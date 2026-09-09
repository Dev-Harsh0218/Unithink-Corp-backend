# unithink-corp-api

Backend API for the **UniThink** study-abroad consultancy platform. Node.js + Express + MongoDB, with a Nodemailer email pipeline for lead acknowledgements and application status notifications. Deployed on Vercel as serverless functions.

Part of the [UniThink platform](https://github.com/Dev-Harsh0218/unithink-platform). Consumed by [`unithink-corp-web`](https://github.com/Dev-Harsh0218/unithink-corp-web) (staff portal) and receives leads from [`unithink-website`](https://github.com/Dev-Harsh0218/unithink-website) (public marketing site).

## Responsibilities

- **Lead ingestion** — public contact form on the marketing site POSTs here → creates `Lead` in MongoDB → fires acknowledgement email to student + notification email to assigned consultant
- **CRUD for the portal** — students, applications, documents, universities, users
- **Session-based auth** — consultants sign in via the portal → cookie-based session
- **Email pipeline** — Nodemailer transports on status transitions (lead-created, application-submitted, document-approved, offer-received, visa-approved)
- **File upload endpoints** — student documents (transcripts, SOP, LORs, ID scans)

## Stack

- **Node.js** + **Express 4**
- **MongoDB** via **Mongoose** ODM
- **Nodemailer** for email delivery (SMTP)
- **CORS** for cross-origin from the portal + marketing site
- **dotenv** for config
- **moment** for date handling (legacy — could swap for date-fns)
- **Vercel** as the runtime (serverless functions)

## Layout

```
unithink-corp-api/
├── app/                     # request handlers grouped by resource
├── routes/                  # Express router definitions
├── src/                     # models, services, utilities
├── package.json
├── vercel.json              # Vercel serverless config
└── README.md
```

## Endpoints (high level)

- `POST /leads` — public, from marketing site contact form. Body: `{name, email, phone, program, message}`.
- `GET /leads` (auth) — paginated lead inbox for consultants
- `POST /students`, `GET /students`, `PATCH /students/:id` — student profile CRUD
- `POST /applications`, `GET /applications`, `PATCH /applications/:id/status` — university application lifecycle
- `POST /documents/upload` (auth, multipart) — student document upload
- `POST /auth/login`, `POST /auth/logout`, `GET /auth/me` — session

Full route surface: see `routes/`.

## Local dev

```bash
npm install
cp .env.example .env
# Set:
#   MONGODB_URI=mongodb://localhost:27017/unithink
#   SMTP_HOST=...
#   SMTP_USER=...
#   SMTP_PASS=...
#   JWT_SECRET=...
npm run dev              # nodemon on :3000
npm start                # production
```

## Deploy

Vercel (serverless):

```bash
vercel --prod
```

Set env vars in Vercel dashboard:
- `MONGODB_URI` — MongoDB Atlas connection string
- `SMTP_HOST` / `SMTP_PORT` / `SMTP_USER` / `SMTP_PASS` — email transport
- `JWT_SECRET` — session signing
- `CORS_ORIGINS` — comma-separated origins for the marketing + portal domains

## Design choices

- **MongoDB over Postgres** — student application state is variable per country / per university (different doc requirements, different fields). Mongoose's schema flexibility saved months of migration work here.
- **Nodemailer over SendGrid/Postmark** — volume is low (dozens/day), consultant control over templates was important, and self-hosted SMTP avoided per-email pricing.
- **Vercel serverless (not always-on Express)** — traffic is low + bursty (leads come in at unpredictable rates). Cold starts are fine for internal traffic; cost is basically zero when idle.
- **Session cookies (not JWT in localStorage)** — HttpOnly cookies survive page refresh, immune to XSS token theft. Portal is intranet-y, no SPA-token complexity needed.

## Related repos

- [`unithink-platform`](https://github.com/Dev-Harsh0218/unithink-platform) — meta-repo (architecture overview)
- [`unithink-corp-web`](https://github.com/Dev-Harsh0218/unithink-corp-web) — staff portal (main consumer)
- [`unithink-website`](https://github.com/Dev-Harsh0218/unithink-website) — public marketing site (lead source)
