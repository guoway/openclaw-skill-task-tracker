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
- **API 變更說明**（for 前端 Web/App Engineer）

#### API 變更說明格式

在 ticket 完成後，必須新增 API 變更說明，讓前端工程師了解 API 異動：

```markdown
## API 變更說明（for 前端 Web/App Engineer）

### 新增 API

| Method | Endpoint | 說明 |
|--------|----------|------|
| POST | `/api/xxx` | API 說明 |

**Request Body**：（如有）
\`\`\`json
{
  "field": "value"
}
\`\`\`

**Response**：（如有）
\`\`\`json
{
  "field": "value"
}
\`\`\`

---

### 修改 API

| API | 變更說明 |
|-----|----------|
| `GET /api/xxx/{id}` | 新增/修改欄位說明 |

**新增/修改欄位**：

| 欄位 | 型別 | 說明 |
|------|------|------|
| `fieldName` | Type | 欄位說明 |

---

### 刪除 API

| Method | Endpoint | 說明 |
|--------|----------|------|
| （如有刪除才填寫） | | |
```

**注意事項**：
- 若無任何 API 變更，仍需標註「無」
- Request/Response 範例使用 code block 包裹
- 欄位說明要清楚標明型別和用途

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

## GuyFish: API 權限設定

當新增需要登入的 API 時，需將 API 註冊至權限表。

### 資料表說明

| 資料表 | 用途 |
|--------|------|
| `ss3a_api` | 定義 API（名稱、URL、HTTP Method 等） |
| `ss3a_func_api_mapping` | 綁定 API 與 Function 的對應關係 |

### API ID 命名規則

| 功能分類 | API ID 範圍 | URL prefix |
|----------|-------------|------------|
| 會員相關 | A51xx | `/api/user/*`, `/api/my/*` |
| Post 相關 | A52xx | `/api/post/*` |

**命名原則**：先查詢該分類目前最大的 API ID，新 API 遞增編號。

### 查詢現有 API ID

```sql
-- 查詢會員相關 API（A51xx）
SELECT api_id, name, url FROM ss3a_api WHERE api_id LIKE 'A51%' ORDER BY api_id DESC;

-- 查詢 Post 相關 API（A52xx）
SELECT api_id, name, url FROM ss3a_api WHERE api_id LIKE 'A52%' ORDER BY api_id DESC;
```

### 新增 API 流程

**Step 1**：新增至 `ss3a_api`

```sql
INSERT INTO ss3a_api (api_id, name, application_id, enabled, url, http_method, create_time, creator)
VALUES ('{API_ID}', '{API名稱}', 'guyfish', 1, '{URL}', '{HTTP方法}', NOW(), 'ADMIN');
```

**Step 2**：綁定至 `ss3a_func_api_mapping`

```sql
INSERT INTO ss3a_func_api_mapping (function_id, api_id) VALUES ('front_function', '{API_ID}');
```

### URL 萬用字元規則

| 萬用字元 | 用途 | 範例 |
|----------|------|------|
| `*` | 單一路徑參數 | `/api/post/like/*` → `/api/post/like/P001` |
| `**` | 多層路徑參數 | `/api/post/comments/**` → `/api/post/comments/P001/0` |

### 注意事項

- 執行前需先確認 API ID 不重複
- `application_id` 固定為 `guyfish`
- 前台 API 綁定 `front_function`
- **只有需登入的 API** 才需要新增至此權限表

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
- [ ] **[GuyFish]** API permission added to ss3a_api (if login required)
