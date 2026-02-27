# Example: Job Description Parsing — A Governed Flow

This example shows one concrete governed flow from a hiring engine: a Job Description PDF is uploaded, parsed into structured items, and later used to ask the candidate about each requirement. It demonstrates steps, invisible auto-execution, the code/LLM division of labor, and data evolution across tables.

---

## Data Evolution Diagram

```
FILE UPLOAD                    STEP 001-003                     STEP 004                        STEP 005-006
(before conversation)          (conversation)                   (invisible, automatic)          (conversation)

    PDF/DOCX                   User messages                    F001 Parser runs                Orchestrator reads
       │                            │                                │                         parsed_data
       ▼                            ▼                                ▼                              │
┌─────────────┐             ┌──────────────┐               ┌──────────────────┐                    ▼
│  Supabase   │             │   sessions   │               │  LLM extracts    │          ┌──────────────────┐
│  Storage    │             │              │               │  meaning from    │          │  LLM asks about  │
│  (bucket)   │             │ current_step │               │  raw JD text     │          │  each item:      │
│             │             │ language     │               │       │          │          │  "Do you have    │
│  jd_1234.pdf│             │ job_id       │               │       ▼          │          │   5 yrs Python?" │
└─────────────┘             └──────────────┘               │  Code adds IDs   │          └────────┬─────────┘
       │                            │                      │  MH01, AD01...   │                   │
       │                    ┌──────────────┐               └────────┬─────────┘                   ▼
       │                    │chat_messages │                        │                    ┌──────────────────┐
       │                    │              │                        ▼                    │   admin_data     │
       │                    │ step 001: Hi │               ┌──────────────────┐          │                  │
       │                    │ step 002: OK │               │   parsed_data    │          │ MH01: "Yes, 7yr" │
       │                    │ step 003: #2 │               │   (per item)     │──reads──▶│ AD01: "OK"       │
       │                    └──────────────┘               │                  │          │ CERT01: "No"     │
       │                                                   │ MH01: "Python"   │          │ LANG01: "Yes"    │
       └───────reads────────────────────────────────────▶  │ MH02: "Postgres" │          └──────────────────┘
                                                           │ AD01: "Remote"   │
                                                           │ CERT01: "AWS"    │
                                                           │ LANG01: "English"│
                                                           └──────────────────┘
```

**Reading it left to right — data evolves in three stages:**

1. **Raw** — An unstructured PDF sits in Storage. A session row tracks where the candidate is. Chat messages accumulate. No structured data exists yet.

2. **Parsed** — F001 turns the raw file into structured rows with IDs. The LLM extracts meaning, the code stamps IDs. This is the first time the system "understands" what the job requires.

3. **Collected** — The orchestrator reads `parsed_data`, asks the candidate about each item, and writes their answers to `admin_data`. Now each JD requirement has a candidate response attached to it.

The key: **each stage only reads from the previous stage's table**. No function calls another function. The database is the conveyor belt.

---

## F001: From Uploaded File to Structured Data in 5 Steps

### The big picture

```
PDF/DOCX file  →  Storage bucket  →  Raw text  →  LLM extracts  →  Code formats  →  DB rows
```

The work is split between **the LLM** (semantic understanding) and **the code** (structure, IDs, storage). Neither does the other's job.

### Step 1 — File goes into Storage

A user uploads a job description (PDF, DOCX, or TXT). The upload route validates it and puts it into a Supabase Storage bucket:

```
Supabase Storage
  └── job-descriptions/
        └── {user_id}/{session_id}/jd_1708934400.pdf
```

It also writes a row to `file_uploads` (metadata: name, size, mime type, path). Then it fires off F001 automatically.

### Step 2 — F001 downloads the raw text

The Edge Function receives `{ session_id, file_url }`. It downloads the file from Storage and extracts the raw text:

```typescript
const { data: fileData } = await supabase.storage
  .from(bucket)
  .download(filePath);

const rawText = await fileData.text();
```

At this point `rawText` is just the unstructured content of the job posting — paragraphs, bullet points, whatever format the employer used.

### Step 3 — One LLM call with JSON mode

The raw text goes to OpenAI with a system prompt that says *"extract and normalize this into categories"*:

```typescript
const completion = await openai.chat.completions.create({
  model: "gpt-4o",
  messages: [
    { role: "system", content: EXTRACTION_PROMPT },
    { role: "user",  content: `Parse this job description:\n\n${rawText}` }
  ],
  temperature: 0.0,
  response_format: { type: "json_object" }
});
```

**The LLM's job:** Understand the meaning. Figure out that "5+ years Python" is a must-have, "AWS certification preferred" is a nice-to-have, and "remote work, dental insurance" are admin/perks. It returns raw JSON:

```json
{
  "job_title": "Senior Backend Engineer",
  "must_haves": ["5+ years Python", "PostgreSQL experience", "REST API design"],
  "nice_to_haves": ["Kubernetes", "GraphQL"],
  "certifications": ["AWS Solutions Architect preferred"],
  "languages": ["English fluent", "German B2"],
  "administrative": ["Remote OK", "Berlin office", "€80-100k"],
  "responsibilities": ["Design microservices", "Mentor junior devs"]
}
```

### Step 4 — Code adds rigid structure

The LLM doesn't generate IDs or timestamps — it's bad at uniqueness and consistency. That's the code's job:

```typescript
function generateIds(items: string[], prefix: string): any[] {
  return items.map((text, index) => ({
    id: `${prefix}${String(index + 1).padStart(2, '0')}`,
    text
  }));
}
```

So `["5+ years Python", "PostgreSQL experience"]` becomes:

```json
[
  { "id": "MH01", "text": "5+ years Python" },
  { "id": "MH02", "text": "PostgreSQL experience" }
]
```

Every category gets its own prefix:

| Category | Prefix | Example IDs |
|---|---|---|
| Administrative | `AD` | AD01, AD02 |
| Must-haves | `MH` | MH01, MH02, MH03 |
| Nice-to-haves | `NH` | NH01, NH02 |
| Certifications | `CERT` | CERT01 |
| Languages | `LANG` | LANG01, LANG02 |
| Responsibilities | `RS` | RS01, RS02 |

### Step 5 — Stored in the database

The formatted result is upserted into `job_descriptions` as one row:

```typescript
await supabase
  .from("job_descriptions")
  .upsert({
    session_id,
    job_title: "Senior Backend Engineer",
    must_haves:    '[{"id":"MH01","text":"5+ years Python"}, ...]',
    nice_to_haves: '[{"id":"NH01","text":"Kubernetes"}, ...]',
    certifications:'[{"id":"CERT01","text":"AWS Solutions Architect"}, ...]',
    languages:     '[{"id":"LANG01","text":"English fluent"}, ...]',
    administrative:'[{"id":"AD01","text":"Remote OK"}, ...]',
    raw_jd_text:   rawText,
    parsing_version: "1.0.2"
  });
```

### The division of labor

| Who | Does what | Does NOT do |
|---|---|---|
| **LLM** | Understands semantics — which text is a skill vs. a perk vs. a cert | Generate IDs, timestamps, or UUIDs |
| **Code** | Adds structure — sequential IDs, formatting, DB writes | Understand what "5+ years Python" means |

This is deliberate. The LLM is good at meaning; the code is good at consistency. Each does only what it's good at.

### Where the data goes next

These structured items are what step 006 (Admin Data Collection) uses later. The orchestrator loads `parsed_data` items and asks the candidate about each one — "Do you have 5+ years Python?" — writing the answers to `admin_data`. The IDs (`MH01`, `AD01`) tie the candidate's response back to the original JD item.

---

## How Steps and Data Evolve in the DB

Imagine watching the database tables change as a candidate chats:

### Step 001 — Language Detection

- User sends their first message (e.g. "Hi" or "Hallo").
- `sessions` row is created: `current_step = '001'`
- Orchestrator reads the `steps` row for 001, assembles a prompt, calls the LLM once.
- LLM responds with JSON: `{ "language": "en", "response": "..." }`
- **DB changes:**
  - `chat_messages` gets 2 rows (user + assistant, tagged `step_id = '001'`)
  - `sessions.confirmed_language` is set to `'en'`
  - `sessions.current_step` advances to `'002'`

### Step 002 — Welcome & GDPR

- User sends next message (e.g. "sure, I accept").
- Orchestrator reads step 002 → sees `message_ref = 'welcome, gdpr_notice'` → loads those templates from `static_messages` table.
- Those templates go into the prompt as data. LLM presents them verbatim and asks for acceptance.
- LLM responds: `{ "response": "...", "action": "accepted" }`
- **DB changes:**
  - `chat_messages` gets 2 more rows (tagged `step_id = '002'`)
  - `sessions.current_step` advances to `'003'`
  - No new data tables touched — this step is purely conversational.

### Step 003 — Job Selection

- Orchestrator reads step 003 → sees `expected_action = 'selection'` → loads job list from `job_listings` table.
- Jobs are injected into the prompt. LLM presents them as options.
- User picks one. LLM responds: `{ "response": "...", "action": "selected_2" }`
- **DB changes:**
  - `chat_messages` gets 2 more rows (tagged `step_id = '003'`)
  - `sessions.selected_job_id` is set to the matching job ID
  - `sessions.current_step` advances to `'004'`

### Step 004 — JD Parsing (invisible)

- This step has `visible = false` — the user never sees it.
- Orchestrator detects an invisible step and **auto-executes** it: calls the `F001 Parser` Edge Function.
- F001 reads the full job description from `job_descriptions`, sends it to OpenAI, gets back categorized items.
- **DB changes:**
  - `parsed_data` gets multiple rows — one per item extracted from the JD (e.g. `AD01: "Start date"`, `MH01: "5 years Java"`, `CERT01: "AWS certified"`)
  - `sessions.current_step` advances to `'005'`
  - No chat messages — the user didn't interact.

### The pattern

| What changes | Where it lives |
|---|---|
| Conversation state (which step, what language, which job) | `sessions` — one row, updated in place |
| Every message ever sent | `chat_messages` — append-only, tagged with step_id |
| Step definitions, method logic, templates | `steps`, `methods`, `static_messages` — **read-only seed data**, never written at runtime |
| Extracted data from functions | `parsed_data`, `admin_data` — written by functions, read by later steps |

The key insight: **functions don't call each other**. F001 writes to `parsed_data`. Later, when step 006 runs, the orchestrator reads `parsed_data` and injects it into the prompt. The database is the communication channel between functions — that's the "data evolution" pattern.
