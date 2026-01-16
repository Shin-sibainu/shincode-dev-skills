---
name: dev-workflow
description: Development workflow guide for web projects. Manages requirements definition, design document generation, task ticket creation, and implementation tracking. Use after initial project setup or on existing projects to guide the full development lifecycle.
---

# Development Workflow

Guide users through a structured development workflow from requirements to deployment.

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

### Step 3: Task Ticket Creation

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

### Step 4: Implementation Mode

Ask user which ticket to work on:

```
Which ticket would you like to implement?
- 01-project-setup (Not Started)
- 02-database-setup (Not Started)
- 03-auth-implementation (Not Started)
...
```

For each ticket:

1. **Read** the ticket requirements
2. **Implement** the code changes
3. **Update** ticket checkboxes as criteria are met
4. **Test** the implementation
5. **Mark** ticket as Done when complete
6. **Ask** user to proceed to next ticket or take a break

### Step 5: Progress Tracking

Provide progress summary when asked:

```markdown
## Development Progress

| Ticket | Status | Progress |
|--------|--------|----------|
| 01-project-setup | Done | 100% |
| 02-database-setup | In Progress | 60% |
| 03-auth-implementation | Not Started | 0% |

Overall: 35% complete (2/6 tickets done)
```

### Step 6: Deployment Checklist

When all tickets are done, provide deployment checklist:

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

Users can say:

| Command | Action |
|---------|--------|
| "show progress" | Display progress summary |
| "next ticket" | Move to next pending ticket |
| "update requirements" | Edit requirements.md |
| "regenerate docs" | Regenerate design documents |
| "deploy checklist" | Show deployment checklist |

## Important Notes

- Always read existing docs before generating new ones
- Preserve user edits when regenerating documents
- Ask for confirmation before overwriting files
- Keep tickets small and focused (1-4 hours of work each)
- Update ticket status in real-time during implementation
