# Contributing

- **Design before code.** Update the relevant design artifacts (Value Chain Mapping, Data Evolution, TDD Golden Examples) before touching functions or schema. Read `DESIGN.md` to see what exists.
- **Follow the pattern.** New steps, functions, and skills must follow the governed architecture — DB owns the process, one LLM call per function, defined I/O schemas, everything stored.
- **New examples** should follow the [JD Parsing Example](docs/explanations/jd-parsing-example.md) style: list the steps, show the DB changes, include a diagram.
- **Commits** use [Conventional Commits](https://www.conventionalcommits.org/): `feat:`, `fix:`, `refactor:`, `test:`, `docs:`, `chore:`.
- **Never commit** secrets, `.env` files, or credentials.

---

Questions? Reach out to [Csaba Toldi](https://www.linkedin.com/in/csabatoldi/).
