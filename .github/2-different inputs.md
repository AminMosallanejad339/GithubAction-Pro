## Different inputs

```yaml
name: 2 - different inputs  # Workflow name

on:  # Trigger definition
    workflow_dispatch:  # Manual trigger with inputs
        inputs:
            MySQL:
                required: false
                type: boolean  # Boolean input (true/false)
            PostgreSQL:
                required: false
                type: boolean  # Another boolean input
                
jobs: 
    fetch-data:
        runs-on: ubuntu-latest
        steps:
          - name: MySQL fetch
            if: ${{ github.event.inputs.MySQL == 'true' }}  # Conditional execution
            run: echo "I will retrieve data from MySQL"
    
          - name: PostgreSQL fetch
            if: ${{ github.event.inputs.PostgreSQL == 'true' }}  # Conditional execution
            run: echo "I will retrieve data from PostgreSQL."
    
          - name: Done
            run: echo "You did it 😎"  # Always runs (no condition)

          - name: Some Log prop.
            run: |  # Multi-line command
             echo "The name of event is = ${{ github.event_name }}" 
             echo "Commit SHA is = ${{ github.sha }}"
             echo "Runners is = ${{ runner.os }}"
```

## Key Concepts Demonstrated:

### 1. **Workflow Inputs**
```yaml
inputs:
    MySQL:
        required: false
        type: boolean
```
- Creates interactive checkboxes in GitHub UI
- Users can select which databases to process
- `required: false` means both can be unchecked

### 2. **Conditional Execution with `if`**
```yaml
if: ${{ github.event.inputs.MySQL == 'true' }}
```
- Step only runs if MySQL checkbox is checked
- Boolean inputs become strings: `'true'` or `'false'`

### 3. **GitHub Context Variables**
```yaml
echo "The name of event is = ${{ github.event_name }}"
echo "Commit SHA is = ${{ github.sha }}"
echo "Runners is = ${{ runner.os }}"
```
- **`github.event_name`**: Name of triggering event (`workflow_dispatch`)
- **`github.sha`**: Commit SHA that triggered the workflow
- **`runner.os`**: Operating system of the runner (`Linux`)

## How It Works in Practice:

1. **User triggers workflow** from GitHub Actions tab
2. **Sees checkboxes** for MySQL and PostgreSQL:
   ```
   [ ] MySQL
   [ ] PostgreSQL
   ```
3. **Only selected steps run** based on user choices
4. **Non-conditional steps** (like "Done") always execute

## API Trigger Example:

To trigger this via API with specific inputs:

```bash
curl -X POST \
  -H "Authorization: token YOUR_GITHUB_TOKEN" \
  -H "Accept: application/vnd.github.v3+json" \
  https://api.github.com/repos/OWNER/REPO/actions/workflows/FILE_NAME/dispatches \
  -d '{
    "ref": "main",
    "inputs": {
      "MySQL": true,
      "PostgreSQL": false
    }
  }'
```

## Real-World Use Case:

This pattern is perfect for:
- **Selective data processing**: Choose which databases to backup
- **Environment selection**: Dev/Staging/Production deployments  
- **Feature toggles**: Enable/disable specific operations
- **Multi-region processing**: Select regions to deploy to

## Improvement Suggestions:

```yaml
# Add default values and descriptions
inputs:
    MySQL:
        description: 'Process MySQL database'
        required: false
        type: boolean
        default: false
    PostgreSQL:
        description: 'Process PostgreSQL database' 
        required: false
        type: boolean
        default: false

# Add validation step
- name: Validate inputs
  if: ${{ github.event.inputs.MySQL != 'true' && github.event.inputs.PostgreSQL != 'true' }}
  run: |
    echo "Error: At least one database must be selected!"
    exit 1
```

 