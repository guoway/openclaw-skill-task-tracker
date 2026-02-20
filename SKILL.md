---
name: task-tracker
description: Task tracker workflow for implementing backend APIs from tickets (Redmine, Jira, Linear, etc.). Use when starting work on a ticket/task/issue that requires backend implementation. Covers requirement clarification, feature branch creation, implementation, testing, commit, and ticket update.
---

# Task Tracker Workflow

Standard workflow for implementing backend APIs from task tracker tickets.

## When to Use

- Starting work on a Redmine/Jira/Linear ticket
- Implementing backend API from requirements
- Following a structured development process

## Workflow

### 1. Requirement Clarification

**Goal**: Understand what to build before coding.

Steps:
1. Read ticket (subject, description, attachments)
2. Check design specs (Figma, wireframes) if available
3. Compare with existing code (API, Entity, DTO, Repository)
4. Ask clarifying questions using **interview-style**:
   - One question at a time
   - Prefer multiple choice (A/B/C)
   - Stop asking when requirements are clear

**Key principle**: Search codebase first, ask questions second.

### 2. Create Feature Branch

**Rule**: One ticket = one feature branch.

```bash
git checkout main
git pull
git checkout -b feature/redmine-{ticket-id}-{feature-name}
```

Example: `feature/redmine-1250-home06-carousel`

### 3. Implementation

**Model preference**:
- Planning: Use codex
- Implementation: Use glm-5 (or as specified in project preferences)

**Follow project conventions**:
- Pagination: Use existing patterns (e.g., `NativeSearchWithPageRequest`)
- DTO naming: Can differ from Entity, use product semantics
- API response: Include all fields needed by UI

### 4. Testing

**Required**:
- Write unit tests (Service, Controller, DTO as needed)
- Generate coverage report

```bash
mvn org.jacoco:jacoco-maven-plugin:0.8.12:prepare-agent test \
    org.jacoco:jacoco-maven-plugin:0.8.12:report \
    -Dtest=YourTest
```

**Coverage target**: Critical logic should be tested.

### 5. Commit & Push

**Commit message**: Follow project language preference (default: Chinese for this project).

```bash
git add -A
git commit -m "#{ticket-id} 功能描述

- 實作內容 1
- 實作內容 2
- 測試: X tests, coverage Y%"
git push -u origin feature/redmine-{ticket-id}-{feature-name}
```

### 6. Update Ticket

Update the task tracker ticket with:
- Implementation summary
- Test results
- Git branch and commit info
- Merge request link
- Change status (e.g., New → Resolved)

## Key Rules

1. **One ticket = one feature branch** (never share branches)
2. **Clarify before coding** (interview-style questions)
3. **Write tests with coverage report**
4. **Update ticket when done**
5. **Follow project conventions** (naming, patterns, language)

## Project Preferences

Store project-specific preferences in MEMORY.md:
- Default language (e.g., Chinese)
- Model preference (codex for planning, glm-5 for implementation)
- Commit message language
- API patterns

## Example: Redmine

```bash
# Get ticket info
curl -H "X-Redmine-API-Key: {api-key}" \
    "https://redmine.example.com/issues/{id}.json"

# Update ticket
curl -X PUT \
    -H "X-Redmine-API-Key: {api-key}" \
    -H "Content-Type: application/json" \
    -d '{"issue": {"notes": "...", "status_id": 3}}' \
    "https://redmine.example.com/issues/{id}.json"
```

## Checklist

Before marking ticket complete:
- [ ] Feature branch created
- [ ] Implementation done
- [ ] Tests written and passing
- [ ] Coverage report generated
- [ ] Committed and pushed
- [ ] Ticket updated with notes
- [ ] Ticket status updated
- [ ] MR link provided
