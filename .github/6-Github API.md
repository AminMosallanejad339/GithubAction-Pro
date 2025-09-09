 

## Github API

```yaml
name: 6- Github API  # Workflow name

on:
    repository_dispatch:
      types: [trigger-fetch-data]  # Specific event type
 
jobs:
  fetch-data:
    runs-on: ubuntu-latest
    steps:
      - name: MySQL fetch
        run: echo "I will retrieve data from MySQL"

      - name: PostgreSQL fetch
        run: echo "I will retrieve data from PostgreSQL."

      - name: Done
        run: echo "Done boy 😎"
```

## Key Concept: Repository Dispatch

**Repository Dispatch** allows you to trigger workflows via GitHub API calls with custom payloads. This is different from other triggers:

- **`workflow_dispatch`**: Manual trigger from GitHub UI
- **`repository_dispatch`**: Programmatic trigger via API with custom data

## How Repository Dispatch Works:

### 1. **Event Type Definition**
```yaml
types: [trigger-fetch-data]
```
- Defines a custom event type
- You can have multiple types: `[type1, type2, type3]`
- Acts as a "webhook endpoint" for your workflow

### 2. **API Trigger Structure**
The API call sends a POST request with a custom payload.

## API Call Examples:

### Basic Trigger:
```bash
curl -X POST \
  -H "Authorization: token YOUR_GITHUB_TOKEN" \
  -H "Accept: application/vnd.github.v3+json" \
  https://api.github.com/repos/OWNER/REPO/dispatches \
  -d '{
    "event_type": "trigger-fetch-data"
  }'
```

### With Custom Payload:
```bash
curl -X POST \
  -H "Authorization: token YOUR_GITHUB_TOKEN" \
  -H "Accept: application/vnd.github.v3+json" \
  https://api.github.com/repos/OWNER/REPO/dispatches \
  -d '{
    "event_type": "trigger-fetch-data",
    "client_payload": {
      "database": "mysql",
      "environment": "production",
      "timestamp": "2024-01-15T10:30:00Z"
    }
  }'
```

## Accessing Payload Data in Workflow:

You can access the custom payload in your workflow steps:

```yaml
steps:
  - name: Log payload
    run: |
      echo "Event type: ${{ github.event.action }}"
      echo "Database: ${{ github.event.client_payload.database }}"
      echo "Environment: ${{ github.event.client_payload.environment }}"
```

## Enhanced Example with Payload Handling:

```yaml
name: 6- Github API

on:
    repository_dispatch:
      types: [trigger-fetch-data, trigger-backup, trigger-report]
 
jobs:
  fetch-data:
    runs-on: ubuntu-latest
    steps:
      - name: Check event type
        run: |
          echo "Event type: ${{ github.event.action }}"
          echo "Payload: ${{ toJson(github.event.client_payload) }}"

      - name: MySQL fetch
        if: ${{ github.event.client_payload.database == 'mysql' || github.event.client_payload.database == 'all' }}
        run: echo "Fetching data from MySQL"

      - name: PostgreSQL fetch
        if: ${{ github.event.client_payload.database == 'postgresql' || github.event.client_payload.database == 'all' }}
        run: echo "Fetching data from PostgreSQL"

      - name: MongoDB fetch
        if: ${{ github.event.client_payload.database == 'mongodb' || github.event.client_payload.database == 'all' }}
        run: echo "Fetching data from MongoDB"

      - name: Process based on environment
        run: |
          echo "Processing for environment: ${{ github.event.client_payload.environment || 'development' }}"

      - name: Done
        run: echo "Done! Event completed successfully 😎"
```

## Real-World Use Cases:

### 1. **External System Integration**
```bash
# From your server/application
curl -X POST https://api.github.com/repos/your-org/your-repo/dispatches \
  -H "Authorization: token $GITHUB_TOKEN" \
  -d '{
    "event_type": "data-sync-completed",
    "client_payload": {
      "source": "external-api",
      "records_processed": 1500,
      "status": "success"
    }
  }'
```

### 2. **Scheduled Tasks with Dynamic Parameters**
```bash
# From cron job or scheduler
curl -X POST https://api.github.com/repos/your-org/your-repo/dispatches \
  -H "Authorization: token $GITHUB_TOKEN" \
  -d '{
    "event_type": "daily-backup",
    "client_payload": {
      "backup_type": "full",
      "retention_days": 7,
      "notify_email": "admin@company.com"
    }
  }'
```

### 3. **CI/CD Pipeline Triggers**
```bash
# From another workflow or system
curl -X POST https://api.github.com/repos/your-org/your-repo/dispatches \
  -H "Authorization: token $GITHUB_TOKEN" \
  -d '{
    "event_type": "deploy-to-production",
    "client_payload": {
      "version": "1.2.3",
      "build_id": "build-12345",
      "requester": "ci-system"
    }
  }'
```

## Required Permissions:

To use repository dispatch, you need:
1. **Personal Access Token** with `repo` scope
2. **Repository access** permissions
3. **GitHub Token** for workflows (if triggering from within GitHub)

## Security Considerations:

```yaml
# Add validation step
- name: Validate payload
  run: |
    if [ -z "${{ github.event.client_payload.secret }}" ] || [ "${{ github.event.client_payload.secret }}" != "${{ secrets.DISPATCH_SECRET }}" ]; then
      echo "Invalid or missing secret token!"
      exit 1
    fi
```

## Comparison with Other Triggers:

| Trigger Type          | Source         | Custom Data       | UI Visible |
| --------------------- | -------------- | ----------------- | ---------- |
| `workflow_dispatch`   | GitHub UI      | Limited inputs    | Yes        |
| `repository_dispatch` | API Call       | Full JSON payload | No         |
| `push`                | Git operations | No                | Yes        |
| `schedule`            | Cron           | No                | Yes        |

**Repository Dispatch** is perfect for:
- External system integrations
- Complex parameter passing
- Automated triggers from other systems
- Building event-driven architectures

 