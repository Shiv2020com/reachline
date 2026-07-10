# Reachline — Backend Architecture & Build Plan

This document describes how to turn the frontend prototype (`index.html`) into a fully working website with a real backend: form submissions, an admin panel, and the Creator Recommendation Book.

---

## 1. Recommended Stack

| Layer | Recommendation | Why |
|---|---|---|
| Frontend | Next.js (React) | Fast page loads, built-in SEO (meta tags, sitemaps), easy to convert the existing HTML/CSS into components |
| Backend | Next.js API routes, or a separate Node.js + Express service | Handles form submissions, admin auth, creator approval logic |
| Database | PostgreSQL (via Supabase, Neon, or Railway) | Structured data — advertisers, creators, inquiries, approvals |
| Auth (admin only) | NextAuth.js or Clerk, email/password or magic link | Public users don't need accounts; only your internal team logs in |
| File storage | Cloudflare R2 or AWS S3 | For creator media kits, screenshots of analytics, etc. |
| Email notifications | Resend or Postmark | Sends confirmation emails and internal alerts when a form is submitted |
| Hosting | Vercel (frontend + API routes) | One-click deploy, free tier is enough to start, scales automatically |

This stack keeps things in one codebase (Next.js does both frontend and backend), which is simpler to maintain for a first version.

---

## 2. Database Schema (core tables)

**advertisers_inquiries**
- id, full_name, business_name, email, phone, channels (array), budget_range, website_url, notes, status (new / contacted / active / closed), created_at

**creator_applications**
- id, full_name, channel_name, platform (YouTube / Facebook Page / Instagram / Other), audience_size, email, channel_link, niche_description, status (pending / approved / rejected), created_at

**recommendation_book** (only approved creators appear here)
- id, creator_application_id (foreign key), display_name, platform, niche, audience_size, is_featured, listed_at

**contact_messages**
- id, name, email, message, created_at, replied (boolean)

**admin_users**
- id, email, password_hash, role (admin / editor)

---

## 3. Core API Endpoints

```
POST   /api/advertisers          -> create a new advertiser inquiry
POST   /api/creators              -> create a new creator application
POST   /api/contact               -> create a contact message

GET    /api/admin/inquiries       -> (auth required) list advertiser inquiries
GET    /api/admin/applications    -> (auth required) list creator applications
PATCH  /api/admin/applications/:id -> approve or reject a creator application
GET    /api/recommendation-book   -> public list of approved creators (for a public directory page, if you want one)
```

Each POST endpoint should:
1. Validate the input (required fields, valid email format).
2. Store the record in the database.
3. Send a confirmation email to the submitter.
4. Send an internal notification (email or Slack webhook) to your team.

---

## 4. Admin Panel (internal use)

A simple protected `/admin` section where your team can:
- View new advertiser inquiries and mark them as contacted/active/closed
- Review creator applications and approve or reject them
- See approved creators automatically appear in the Recommendation Book
- Reply to contact messages

This can be a set of simple authenticated pages in the same Next.js app — no need for a separate admin tool for a first version.

---

## 5. Performance Checklist (for fast loading)

- Serve images in modern formats (WebP/AVIF) and lazy-load anything below the fold.
- Use Next.js's built-in image optimization instead of raw `<img>` tags.
- Keep font loading minimal — the prototype uses 3 font families, which is already a reasonable ceiling.
- Enable caching headers for static assets (Vercel does this automatically).
- Avoid loading analytics/marketing scripts synchronously — load them with `async`/`defer` or after page load.
- Run a Lighthouse audit before launch and aim for 90+ on Performance and SEO.

---

## 6. Suggested Build Order

1. Convert the provided HTML prototype into Next.js pages/components (mostly a direct copy of the markup and CSS).
2. Set up the PostgreSQL database and the three core tables (inquiries, applications, contact).
3. Wire the three forms to real API routes that save to the database and send confirmation emails.
4. Build the protected `/admin` pages to view and manage submissions.
5. Add the public Recommendation Book listing (optional, but useful — it lets advertisers browse approved creators directly).
6. Deploy to Vercel, connect your domain, run a Lighthouse/performance pass, then launch.

---

## Notes on the Frontend Prototype

The `index.html` file delivered alongside this plan is a working single-file demo of every page described in your brief: Home, Services, For Advertisers, For Creators, About Us, Why Us, Privacy Policy, and Contact. Navigation works, forms are fully laid out, and submitting a form shows a confirmation message — but nothing is saved yet, since there's no backend behind it. Use it as the visual and structural reference when building the real Next.js version.
