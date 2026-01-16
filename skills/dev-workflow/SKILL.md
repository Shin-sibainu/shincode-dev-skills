---
name: dev-workflow
description: Development workflow guide for web projects. Manages requirements definition, design document generation, task ticket creation, and implementation tracking. Use after initial project setup or on existing projects to guide the full development lifecycle.
---

# Development Workflow

Guide users through a structured development workflow from requirements to deployment.

## Execution Mode

This workflow uses a **hybrid approach**:

| Phase | Mode | Description |
|-------|------|-------------|
| Step 1-2 | **Interactive** | Requirements & design - confirm with user |
| Step 3-6 | **Automatic** | Tickets & implementation - run without interruption |

After user approves the design documents, the implementation phase runs automatically until completion.

## When to Use

- After running `/new-webapp` to set up a project
- On existing projects that need structured development
- When user says "help me build this feature" or "let's start development"

## Prerequisites

Ensure the project has a `docs/` directory. If not, create it:

```
docs/
├── requirements.md
├── architecture.md
├── database.md
├── api.md
└── tickets/
```

## Workflow

### Step 1: Requirements Definition

Check if `docs/requirements.md` exists and has content.

**If empty or missing**, use AskUserQuestion to gather:

| Question | Purpose |
|----------|---------|
| What does this app do? | Overview |
| Who are the target users? | User personas |
| What are the core features? | Feature list |
| Any specific tech constraints? | Requirements |

Then generate `docs/requirements.md` with:

```markdown
# Project Requirements

## Overview
[Generated from user answers]

## Target Users
[Generated from user answers]

## Core Features

### Feature 1: [Name]
- Description:
- Priority: High/Medium/Low
- Acceptance Criteria:
  - [ ] Criteria 1
  - [ ] Criteria 2

## Non-Functional Requirements
- Performance:
- Security:
- Scalability:
```

**If already filled**, read and summarize for user confirmation.

### Step 2: Design Document Generation

Read `docs/requirements.md` and generate design documents:

#### 2a. Architecture (docs/architecture.md)

```markdown
# System Architecture

## Overview
[High-level system description]

## Component Diagram
[Text-based diagram or description]

## Tech Stack
- Frontend: [framework]
- Backend: [BaaS/API]
- Database: [DB choice]
- Auth: [auth provider]

## Data Flow
1. User action
2. Frontend handling
3. API call
4. Database operation
5. Response

## Security Considerations
- Authentication flow
- Data validation
- API protection
```

#### 2b. Database Schema (docs/database.md)

```markdown
# Database Schema

## Tables/Collections

### users
| Column | Type | Constraints |
|--------|------|-------------|
| id | uuid | PK |
| email | string | unique, not null |
| created_at | timestamp | default now() |

### [other tables based on requirements]

## Relationships
- users 1:N posts
- posts N:M tags

## Indexes
- users(email)
- posts(user_id, created_at)
```

#### 2c. API Design (docs/api.md)

```markdown
# API Design

## Endpoints

### Authentication
| Method | Endpoint | Description |
|--------|----------|-------------|
| POST | /api/auth/signup | User registration |
| POST | /api/auth/login | User login |

### [Resource Name]
| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | /api/[resource] | List all |
| POST | /api/[resource] | Create new |
| GET | /api/[resource]/[id] | Get one |
| PUT | /api/[resource]/[id] | Update |
| DELETE | /api/[resource]/[id] | Delete |

## Request/Response Examples
[Include sample JSON for key endpoints]
```

#### 2d. User Confirmation (Last Interactive Step)

After generating all design documents, present a summary and ask for approval:

```
Design documents have been generated:
- docs/architecture.md - System architecture
- docs/database.md - Database schema
- docs/api.md - API endpoints

Please review these documents. Once approved, I will:
1. Create task tickets automatically
2. Implement all tickets sequentially
3. Report progress as I go

Ready to proceed with automatic implementation?
```

**If user approves:** Proceed to Step 3 (automatic mode begins)
**If user wants changes:** Make requested edits, then ask again

---

## AUTOMATIC MODE BEGINS

From this point, no user confirmation is needed. Implementation runs until completion.

---

### Step 3: Task Ticket Creation (Automatic)

Split requirements into actionable tickets in `docs/tickets/`:

**Naming Convention:** `XX-short-name.md` (XX = sequence number)

**Ticket Template:**

```markdown
# [XX] Ticket Title

## Description
Brief description of what needs to be done.

## Acceptance Criteria
- [ ] Criterion 1
- [ ] Criterion 2
- [ ] Criterion 3

## Technical Notes
- Implementation hints
- Related files
- Dependencies on other tickets

## Status
- [ ] Not Started
- [ ] In Progress
- [ ] Review
- [ ] Done
```

**Standard Ticket Sequence:**

1. `01-project-setup.md` - Environment setup, dependencies
2. `02-database-setup.md` - Schema creation, migrations
3. `03-auth-implementation.md` - Authentication flow
4. `04-core-feature-1.md` - First main feature
5. `05-core-feature-2.md` - Second main feature
6. ... (based on requirements)
7. `XX-testing.md` - Test coverage
8. `XX-deployment.md` - Deployment preparation

### Step 4: Implementation Mode (Automatic)

Process all tickets sequentially without user interruption.

#### 4a. Automatic Execution Flow

```
FOR each ticket in docs/tickets/ (sorted by number):
  1. Log: "Starting ticket: [XX-name]"
  2. Read ticket requirements
  3. Detect domain and load appropriate skill patterns
  4. Implement all code changes
  5. Update ticket checkboxes
  6. Mark ticket as Done
  7. Log: "Completed ticket: [XX-name]"
  8. Immediately proceed to next ticket
END FOR

Log: "All tickets completed!"
```

#### 4b. Skill Auto-Detection

Automatically detect and apply specialized skill patterns based on ticket content:

| Ticket Keywords | Applied Skill | Action |
|-----------------|---------------|--------|
| UI, component, page, layout, design | `/frontend-design` patterns | Use shadcn/ui, Tailwind best practices |
| Stripe, payment, subscription, checkout | `/stripe-setup` patterns | Use Stripe SDK patterns from skill |
| auth, login, signup, session | `new-webapp` auth reference | Apply auth patterns |
| database, schema, migration | `new-webapp` DB reference | Apply DB patterns |
| API, endpoint, route | `new-webapp` API reference | Apply API patterns |

**No user confirmation needed** - skills are applied automatically based on content.

#### 4c. Progress Logging

During automatic execution, output progress updates:

```
[1/6] Starting: 01-project-setup
      - Installing dependencies...
      - Creating directory structure...
      - Done ✓

[2/6] Starting: 02-database-setup
      - Applying skill: database patterns
      - Creating schema...
      - Running migrations...
      - Done ✓

[3/6] Starting: 03-auth-implementation
      - Applying skill: new-webapp auth patterns
      - Setting up better-auth...
      - Creating auth routes...
      - Done ✓

...

[6/6] Starting: 06-deployment-prep
      - Creating deployment config...
      - Done ✓

✅ All 6 tickets completed!
```

#### 4d. Available Skills Reference

**From this marketplace (shincode-dev-skills):**

| Skill | Command | Use For |
|-------|---------|---------|
| Stripe Setup | `/stripe-setup` | Payment, subscription, webhooks |

**From official/other marketplaces:**

| Skill | Source | Use For |
|-------|--------|---------|
| Frontend Design | `example-skills:frontend-design` | UI components, pages, styling |
| XLSX | `example-skills:xlsx` | Spreadsheet operations |
| PDF | `example-skills:pdf` | PDF generation/manipulation |

**Note:** Check available skills with `/plugin` command. Skills may vary based on installed plugins.

### Step 5: Progress Tracking (Automatic)

Progress is logged automatically during implementation. Final summary is displayed after completion:

```markdown
## Development Complete!

| Ticket | Status |
|--------|--------|
| 01-project-setup | ✅ Done |
| 02-database-setup | ✅ Done |
| 03-auth-implementation | ✅ Done |
| 04-core-features | ✅ Done |
| 05-ui-components | ✅ Done |
| 06-deployment-prep | ✅ Done |

All 6 tickets completed successfully!
```

### Step 6: Deployment Checklist (Automatic)

Automatically displayed after all tickets are complete:

```markdown
## Pre-Deployment Checklist

### Environment
- [ ] Production environment variables set
- [ ] Database connection configured
- [ ] API keys secured (not in code)

### Security
- [ ] HTTPS enabled
- [ ] CORS configured
- [ ] Rate limiting enabled
- [ ] Input validation complete

### Performance
- [ ] Images optimized
- [ ] Code minified
- [ ] Caching configured

### Testing
- [ ] All tests passing
- [ ] Manual QA complete
- [ ] Cross-browser tested

### Monitoring
- [ ] Error tracking setup (Sentry)
- [ ] Analytics configured
- [ ] Logging enabled
```

## Commands Within Workflow

**During Interactive Phase (Step 1-2):**

| Command | Action |
|---------|--------|
| "update requirements" | Edit requirements.md |
| "regenerate docs" | Regenerate design documents |
| "show design" | Display current design documents |

**During Automatic Phase (Step 3-6):**

The automatic phase runs without interruption. However, if the user needs to stop:

| Command | Action |
|---------|--------|
| "pause" or "stop" | Pause after current ticket completes |
| "show progress" | Display current progress |

**After Completion:**

| Command | Action |
|---------|--------|
| "deploy checklist" | Show deployment checklist again |
| "run tests" | Execute test suite |

## Important Notes

- **Step 1-2**: Always confirm with user before proceeding
- **Step 3-6**: Run automatically without interruption
- Keep tickets small and focused (30min-2hours of work each)
- Update ticket status in real-time during implementation
- Log progress clearly so user can follow along
- Apply specialized skills automatically based on ticket content
