# MT Logistics Automation — Delhivery & Safe Express Integration

📄 **Detailed Design & PRD:**   : https://docs.google.com/document/d/150RSD6H4voUCT1J2aG1oHMoekKn2Xz82wr4Hz8nzDwQ/edit?usp=sharing

## Problem Statement

After every MT shipment booking, the team manually logs into Delhivery or Safe Express, copies LR number, freight cost, dispatch date, appointment date, and other fields, then re-enters them into the internal portal. This process is error-prone, delayed, and unscalable as MT shipment volumes grow — causing data lag across Logistics, Operations, and Finance.

## Solution Overview

A single Next.js 14 application that integrates directly with Delhivery and Safe Express courier APIs to automate the full MT logistics lifecycle. On booking, all shipment data is captured and punched into the internal portal automatically — via REST API or direct DB write. A polling cron runs every 30 minutes to sync live tracking updates, dispatch dates, appointment dates, and delivery confirmations. POD images are fetched post-delivery and routed to Finance instantly. A freight reconciliation engine flags invoice variances automatically. All activity is visible in a role-based operations dashboard with live KPIs for Logistics, Ops, and Finance. The system is packaged as a Docker image and deployed on GCP Cloud Run — no external platform lock-in.


**High-level flow:**
```
MT Order Created
      │
      ▼
Booking API (Delhivery / Safe Express)
      │
      ├── AWB/LR + Freight Cost → Internal Portal Sync
      ├── Label PDF → Supabase Storage → Warehouse Print Queue
      │
      ▼
Tracking Cron (every 30 min)
      │
      ├── Status Updates → Portal Sync (dispatch date, OFD, delivery)
      ├── POD Capture → Supabase → Finance Notified
      └── Alerts → Google Chat + Email
      │
      ▼
Operations Dashboard (Logistics / Ops / Finance)
```

## Tech Stack

- **Frontend:** Next.js 14 (App Router) · React Server Components · Recharts · SWR
- **Backend:** Next.js API Routes (Node.js) · TypeScript · Prisma ORM
- **Database:** PostgreSQL (GCP Cloud SQL)
- **Cloud/Infra:** Docker · GCP Cloud Run · GCP Cloud Scheduler · Supabase Storage
- **Auth:** NextAuth.js (role-based access)
- **Notifications:** Resend (Email) · Google Chat Incoming Webhook


## Features

- **Dual courier support** — Delhivery and Safe Express on a shared integration pipeline; courier selected per shipment at booking time
- **Auto portal data punch** — LR number, freight cost, dispatch date, appointment date, delivery date, and POD URL synced to internal portal with zero manual input
- **Two sync modes** — REST API push (Scenario A) or direct DB write (Scenario B), configured per portal type
- **Live tracking** — all active AWBs polled every 30 minutes; status, location, and timestamps kept current in the portal
- **Automated label routing** — label PDF fetched on booking and pushed to warehouse print queue via Supabase Storage
- **POD capture** — proof-of-delivery image fetched post-delivery and linked to the shipment record; Finance notified instantly
- **Freight reconciliation** — Delhivery and Safe Express invoices matched against booked cost; variances above ₹50 flagged automatically
- **Sync health monitoring** — every portal data punch logged with status; failed syncs retried 3× with exponential backoff and alerted via Google Chat + Email
- **Operations dashboard** — live KPIs including shipment status breakdown, courier performance, SLA compliance %, avg TAT, RTO% monitoring, freight cost analysis, and warehouse heatmap
- **Role-based access** — separate views for Logistics, Operations, and Finance teams
- **GCP-native deployment** — Docker image deployable on Cloud Run or GKE; cron jobs managed via GCP Cloud Scheduler
