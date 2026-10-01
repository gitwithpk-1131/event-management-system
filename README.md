# Eventora – Event Management System

A full-stack web application for browsing, booking, and managing
events — with role-based access for regular users and staff,
integrated payments, automated email notifications, and a staff
dashboard for event and booking oversight.

## Live Demo

**https://eventora-oclm.onrender.com**

## GitHub Link

**https://github.com/gitwithpk-1131/event-management-system**

## Features

- User authentication with role-based access (user / staff)
- Event creation, browsing, and booking
- Stripe-integrated payment handling for paid bookings
- Automated email notifications via Nodemailer
- Staff dashboard for managing events, bookings, and feedback
- Session management with MongoDB-backed persistent sessions
- Certificate generation and attendance/event reports
- CSRF protection, rate limiting, and Helmet security middleware
- Contact form with email delivery

## Tech Stack

- **Node.js / Express** — backend server and routing
- **MongoDB / Mongoose** — database and data modeling
- **EJS** — server-rendered views
- **express-session + connect-mongo** — persistent, database-backed sessions
- **Stripe** — payment processing
- **Nodemailer** — transactional email (bookings, contact form, notifications)
- **Helmet, csurf, express-rate-limit** — security middleware
- **Bootstrap** — frontend styling
- **PDFKit** — certificate/report generation

## Project Structure

```
eventora/
├── app.js                    # Express app entry point, middleware, routes
├── package.json
├── vercel.json                # (legacy) Vercel serverless config
├── .env                        # Environment variables (not committed)
├── models/
│   ├── Event.js
│   ├── Booking.js
│   ├── Feedback.js
│   ├── User.js
│   ├── certificate.js
│   └── session.js
├── routes/
│   ├── auth.js
│   ├── events.js
│   ├── bookings.js
│   ├── feedback.js
│   ├── sessions.js
│   ├── certificate.js
│   ├── reports.js
│   ├── contact.js
│   ├── staff.js
│   └── dashboard.js
├── middleware/
│   ├── auth.js                # requireLogin guard
│   ├── rateLimit.js
│   └── validators.js
├── helpers/
│   └── notifyOrganizer.js
├── utils/
│   └── mailer.js
├── public/
│   ├── css/styles.css
│   ├── js/scripts.js
│   └── images/
└── views/
    ├── layouts/main.ejs
    ├── partial/ (header, footer, messages)
    ├── auth/ (login, register, forgot/reset password)
    ├── events/ (list, details, create, edit, manage)
    ├── bookings/ (list, confirm, success)
    ├── dashboard/ (user, staff)
    ├── sessions/, feedback/, reports/
    ├── home.ejs, contact.ejs, certificate.ejs, error.ejs
```

## Setup & Run Locally

```bash
npm install
```

Create a `.env` file in the project root with:

```
MONGODB_URI=mongodb+srv://<username>:<password>@<cluster-url>/event_management?retryWrites=true&w=majority
SESSION_SECRET=your_session_secret
STRIPE_SECRET_KEY=your_stripe_secret_key
STRIPE_PUBLISHABLE_KEY=your_stripe_publishable_key
EMAIL_USER=your_gmail_address
EMAIL_PASS=your_gmail_app_password
PORT=5000
```

Then start the server:

```bash
npm start
```

Open `http://localhost:5000` in your browser.

## Deployment Notes

This project is deployed on **Render** as a persistent Node.js server.
It was originally attempted on Vercel, but Vercel's serverless function
model isn't well-suited to a traditional session-based Express +
MongoDB app (cold starts, repeated database connections, and
session/cookie handling all behave differently in a serverless
environment) — Render's persistent server model matches this app's
architecture much better.

A few real debugging lessons from getting this deployed and stable:

- **`app.set('trust proxy', 1)`** is required when running behind
  Render/Vercel's reverse proxy, or `express-rate-limit` throws on the
  `X-Forwarded-For` header.
- **MongoDB Atlas SSL/TLS errors** (`ssl3_read_bytes:tlsv1 alert
  internal error`) that persisted across multiple hosting platforms
  were ultimately resolved by provisioning a fresh Atlas cluster —
  pointing to an intermittent issue with the original cluster's
  connection layer rather than application code.
- **`querySrv ENOTFOUND`** errors connecting to a `mongodb+srv://`
  URI are typically a DNS issue, not a database issue — some
  ISP/router DNS servers don't properly support the SRV record lookups
  this connection format relies on. Switching to a public DNS resolver
  (e.g. 8.8.8.8) resolves it.
- Error handlers that reference `req.session.user` should guard
  against `req.session` being `undefined` (e.g.
  `req.session ? req.session.user : null`), since a session can fail
  to attach on certain error paths — otherwise the error handler
  itself can throw.

## Disclaimer

This project was built as a learning/portfolio project. Stripe is
configured in test mode; no real payments are processed.