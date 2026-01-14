---
name: new-webapp
description: Interactive web development project setup. Guides users through selecting framework, BaaS, authentication, and payment providers, then generates the project. Use when user wants to create a new web application, start a new web project, set up a web service, or needs a project scaffold.
---

# Web Project Setup

Guide users through an interactive setup process to create a new web application with their preferred technology stack.

## Workflow

### Step 1: Framework Selection

Use AskUserQuestion to ask which framework:

| Option | Description |
|--------|-------------|
| **Next.js** (Recommended) | Full-stack React with App Router, SSR, API routes |
| **React + Vite** | Client-side React, fast dev experience |

### Step 2: BaaS Selection

Ask about backend needs:

| Option | Description |
|--------|-------------|
| **Supabase** (Recommended) | Open-source, Postgres, Auth, Storage |
| **Turso** | Edge SQLite, libSQL, global replication |
| **None** | Custom backend later |

### Step 3: Authentication

Ask about authentication:

| Option | Description |
|--------|-------------|
| **Supabase Auth** | If Supabase selected, use built-in auth |
| **better-auth** (Recommended) | Modern, framework-agnostic, TypeScript-first |
| **Clerk** | Drop-in UI, fast setup |

### Step 4: Payment (Optional)

Ask if subscription/payment needed:

| Option | Description |
|--------|-------------|
| **Stripe Subscription** | 3-tier plans (Basic/Pro/Enterprise), Customer Portal, Webhooks |
| **None** | No payment |

If Stripe selected, generate:
- 3-tier pricing page component
- Checkout API route
- Webhook handler for subscription events
- Customer portal redirect
- Subscription status check utilities

### Step 5: File Storage (Optional)

Ask if file/image upload is needed:

| Option | Description |
|--------|-------------|
| **Supabase Storage** | If Supabase selected, use built-in storage |
| **Cloudflare R2** | S3-compatible, edge-optimized, cost-effective |
| **None** | No file storage |

If storage selected, generate:
- Upload API route
- File upload component with drag & drop
- Image optimization utilities (if applicable)

### Step 6: Email (Optional)

Ask if email functionality is needed:

| Option | Description |
|--------|-------------|
| **Resend** | Modern email API, React Email templates |
| **None** | No email |

If Resend selected, generate:
- Email sending utility
- Email templates (welcome, password reset, notification)
- API route for sending emails

### Step 7: Generate Project

Based on selections:

1. Create directory structure
2. Generate package.json with dependencies
3. Create configuration files
4. Set up environment template (.env.example)
5. Generate starter files
6. **Create docs/ directory with requirement templates**

#### docs/ Directory Structure

Always create the following documentation structure:

```
docs/
├── requirements.md      # Main requirements document (user fills this)
├── architecture.md      # System architecture (AI generates from requirements)
├── database.md          # DB schema design (AI generates)
├── api.md               # API endpoints design (AI generates)
└── tickets/             # Task tickets (AI splits from requirements)
    ├── 01-setup.md
    ├── 02-auth.md
    └── ...
```

#### requirements.md Template

```markdown
# Project Requirements

## Overview
<!-- Describe what this application does -->

## Target Users
<!-- Who will use this application? -->

## Core Features
<!-- List main features with priority -->

### Feature 1: [Name]
- Description:
- Priority: High/Medium/Low
- Acceptance Criteria:

### Feature 2: [Name]
...

## Non-Functional Requirements
- Performance:
- Security:
- Scalability:

## Tech Stack
- Framework: [selected]
- Database: [selected]
- Auth: [selected]
- Payment: [selected]
- Storage: [selected]
- Email: [selected]
```

#### Workflow After Generation

1. User fills `docs/requirements.md` with project details
2. AI reads requirements and generates:
   - `docs/architecture.md` - System design
   - `docs/database.md` - DB schema
   - `docs/api.md` - API design
3. AI splits requirements into tickets in `docs/tickets/`
4. Each ticket has TODO checkboxes for progress tracking

After generation, provide:
- Setup steps (install dependencies, configure env)
- Development commands
- Guide to fill requirements.md
- Links to relevant documentation

## Reference Files

Load the appropriate reference based on framework selection:

- **Next.js**: See [references/nextjs.md](references/nextjs.md)
- **React + Vite**: See [references/react-vite.md](references/react-vite.md)

These files contain dependency versions, configuration patterns, and code templates.
