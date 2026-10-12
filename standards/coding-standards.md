# StackStorm Team Coding Standards

**Version:** 1.0 · **Authors:** Amarawickrama R.H. (SE/2021/004), S.Thanoogithan (SE/2021/005), B.K.Pabasara (SE/2021/009), Lakshani A.T. (SE/2021/041) · **Date:** 2026-10-09

## 1. Language
- Backend: JavaScript, Node.js 20 LTS
- Frontend: JavaScript / JSX, React 18+

## 2. Style guide
- Airbnb JavaScript Style Guide, enforced by ESLint
- Formatting enforced by Prettier. The `.prettierrc` is committed to the repo root
- Principles: SOLID, DRY, YAGNI, clean code (meaningful names, small functions, no magic numbers)

## 3. Naming conventions

| Construct | Convention | Example |
|---|---|---|
| Backend files | camelCase | `customerService.js` |
| React component files | PascalCase | `ProductCard.jsx` |
| Variables and functions | camelCase | `getUserById()` |
| Classes and models | PascalCase | `CustomerService` |
| Constants | SCREAMING_SNAKE_CASE | `ORDER_STATUS.PENDING` |
| Database columns | snake_case | `customer_id` |
| Environment variables | SCREAMING_SNAKE_CASE | `JWT_SECRET` |

## 4. Git
- Commit format: `<type>(<scope>): <short description>`, body with `Closes #NN` or `Refs #NN`
- Types: feat, fix, test, ci, build, perf, refactor, docs, chore, style, revert
- Branches: `feature/<issue>-<slug>`, `fix/<issue>-<slug>`, `hotfix/<slug>`, `release/<version>`
- Feature branches live at most 5 working days. Never commit directly to `main` or `develop`
- Releases are tagged `vMAJOR.MINOR.PATCH`

## 5. Pull requests and reviews
- Every PR targets `develop`, needs 1 approval and a green CI
- Review comment prefixes: `[blocker]`, `[suggestion]`, `[question]`, `[nit]`
- The reviewer resolves a comment, never the author

## 6. Testing
- Arrange-Act-Assert pattern, Given-When-Then test names
- Unit tests: at least 80% coverage on new code, 90% branch coverage on critical business logic
- Integration tests: every API endpoint has at least one
- End-to-end tests: all happy paths and the top 3 critical flows

## 7. Error handling
- Never expose stack traces in production
- Async route handlers use try/catch and call `next(err)`
- Error responses use one format: `{ error: 'CODE', message: 'Readable text' }`

## 8. Secrets
- Never commit secrets, even in private repos
- Local values go in `.env` (git-ignored). `.env.example` holds placeholders only
- CI secrets live in GitHub Secrets