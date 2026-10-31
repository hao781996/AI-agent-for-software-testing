# Guide to Reading and Explaining User Stories from JIRA

This document guides the assistant in connecting to JIRA, retrieving a User Story based on information provided by the user, and explaining it in clear, easy-to-understand Vietnamese. Read data only; do not edit issues, add comments, change status, or create content in JIRA.

## 1. Required information

The user provides one of the following:

- An issue key, for example `PROJ-123`.
- A link to the issue.
- A description or User Story text copied from JIRA.

If only a link is provided, extract the issue key from the URL. If the key or content cannot be retrieved, ask the user to provide the issue key or paste the story content.

## 2. Connecting to JIRA

Prefer using the JIRA/MCP connection already configured in the working environment. Search for the exact issue key, then read the available fields: summary, description, acceptance criteria, issue type, status, priority, labels, assignee, and related issue links. Request read-only access only.

If the environment has no JIRA connection, the issue can be retrieved through the Jira Cloud REST API v3. For local use, copy `.env.example` to `.env` and enter the real values only in `.env`. The `.gitignore` file excludes `.env` and other local environment files from Git. Keep `.env.example` limited to dummy values. Alternatively, configure these variables in the runtime environment's secret store. Never write real credentials into this guide, source code, or conversation content.

Load the local values into the current PowerShell process before running the request example:

```powershell
Get-Content -LiteralPath ".env" | ForEach-Object {
    if ($_ -match '^\s*([^#=]+)=(.*)$') {
        [Environment]::SetEnvironmentVariable($matches[1].Trim(), $matches[2].Trim(), "Process")
    }
}
```

PowerShell example for reading an issue by key:

```powershell
$pair = "$env:JIRA_EMAIL`:$env:JIRA_API_TOKEN"
$bytes = [System.Text.Encoding]::UTF8.GetBytes($pair)
$auth = [Convert]::ToBase64String($bytes)
$headers = @{ Authorization = "Basic $auth"; Accept = "application/json" }
$key = "PROJ-123"
$url = "$($env:JIRA_BASE_URL.TrimEnd('/'))/rest/api/3/issue/$key`?fields=summary,description,issuetype,status,priority,labels,assignee"
Invoke-RestMethod -Method Get -Uri $url -Headers $headers
```

The account must have permission to view the project and issue. If JIRA returns an authentication error or the issue cannot be found, do not guess its contents; briefly report the error and ask the user to check permissions, the issue key, or paste the story directly. Do not print authentication headers, tokens, or other secrets to logs.

## 3. Processing workflow

1. Identify the issue key or receive the story content from the user.
2. Read the issue through the configured connection. If the user has already pasted the complete content, use that content; a connection is not required.
3. Preserve the story's meaning. Clearly distinguish content present in JIRA from analysis or missing information.
4. Summarize the story, explain terminology, describe acceptance criteria, and identify points that need clarification.
5. Respond in Vietnamese. Keep the issue key and any necessary English field names or terminology.

## 4. Response structure

### User Story: `<ISSUE-KEY> — <summary>`

**Tóm tắt:** One or two sentences describing the main objective.

**Diễn giải dễ hiểu:** Explain in Vietnamese who needs what feature and why, using the pattern “Là [vai trò], tôi muốn [nhu cầu], để [giá trị]” when the content provides enough information.

**Điều kiện chấp nhận:** List the Acceptance Criteria in a testable form. If JIRA does not include this field, state: “Chưa thấy điều kiện chấp nhận trong nội dung được cung cấp”.

**Thuật ngữ:** Briefly explain notable terms or business rules.

**Điểm cần làm rõ:** Ask specific questions in Vietnamese about ambiguities, missing exception flows, input/output data, or testing criteria. If none are apparent, state: “Chưa phát hiện điểm cần làm rõ từ nội dung hiện có”.

## 5. Quality principles

- Do not add requirements, business rules, acceptance criteria, or system behavior that the issue does not state.
- If the description uses Atlassian Document Format, convert it into readable text before explaining it.
- If fields are missing or access is unavailable, clearly state the limits of the available data.
- You may suggest a clearer rewrite of the story, but label it as a suggestion, not as original JIRA content.
- Do not include sensitive information from the issue in the response unless it is necessary for the explanation.
