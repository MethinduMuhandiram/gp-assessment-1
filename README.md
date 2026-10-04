# GP Inventory Assessment

A small MERN inventory project modeled after the Gunda Power ERP's
product-management conventions:

- React UI
- Redux + Thunk actions
- Axios API calls
- Node.js and Express routes
- MongoDB with Mongoose
- SKU-based product operations
- Stock and audit-related business rules

This is a synthetic assessment project. It contains no production data,
credentials, or proprietary Gunda Power source code.

## Candidate task

See [ASSESSMENT.md](./ASSESSMENT.md). The UI has separate Add stock /
Remove stock actions. The backend stock route and Redux action
intentionally contain `TODO` sections.

## Before the interview

The interviewer should install and start everything before the candidate
arrives so setup time is not included in the 30-minute assessment.

### Requirements

- Node.js 18+
- npm
- Docker, or a local MongoDB instance

### Setup

```bash
cp server/.env server/.env
cp client/.env client/.env
docker compose up -d
npm run install:all
npm run seed
npm run dev
```

Open `http://localhost:5173`.

The API runs at `http://localhost:5050`.

## Resetting between candidates

```bash
npm run seed
```

Reset the Git working tree or provide a fresh copy of this folder before
each candidate.

## Interviewer material

`interviewer/EXPECTED_SOLUTION.md` contains evaluation guidance. Remove
the entire `interviewer` directory before giving the project to a
candidate.
