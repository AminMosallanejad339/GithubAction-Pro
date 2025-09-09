## GitHub Actions Hello - world

```yaml
name: 1 - hello - world  # Name of the workflow (appears in GitHub UI)

on: workflow_dispatch  # Trigger: manual execution via GitHub UI/API

jobs:
    first-job:  # First job definition
        runs-on: ubuntu-latest  # Runner environment
        steps:
            - name: Print hello  # Step name
              run: echo "Hello World"  # Command to execute

    second-job:  # Second job (runs in parallel with first-job)
        runs-on: ubuntu-latest
        steps:
             - name: Print Hi to you
               run: echo "Hi to you" 

             - name: I will suc.
               run: exit 0  # Exit with success code (0)

             - name: Say Goodbye
               run: echo "Goodbye"
```

## Key Concepts:
- **Workflow**: Entire YAML file (.github/workflows/*.yml)
- **Job**: Group of steps (run in parallel by default)
- **Step**: Individual task within a job
- **Runner**: Environment where jobs execute

## File Transfer Workflow Example

Here's a workflow that transfers files to cloud storage (AWS S3 example):

```yaml
name: File Transfer to Storage

on:
  workflow_dispatch:  # Manual trigger
    inputs:
      file_path:
        description: 'Path to file to transfer'
        required: true
        type: string
      destination_bucket:
        description: 'Destination bucket name'
        required: true
        type: string

env:
  AWS_REGION: us-east-1

jobs:
  file-transfer:
    runs-on: ubuntu-latest
    
    steps:
    - name: Checkout code
      uses: actions/checkout@v4  # Gets your repository code

    - name: Configure AWS credentials
      uses: aws-actions/configure-aws-credentials@v2
      with:
        aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
        aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
        aws-region: ${{ env.AWS_REGION }}

    - name: Transfer file to S3
      run: |
        # Check if file exists
        if [ ! -f "${{ github.event.inputs.file_path }}" ]; then
          echo "File not found: ${{ github.event.inputs.file_path }}"
          exit 1
        fi
        
        # Upload to S3
        aws s3 cp "${{ github.event.inputs.file_path }}" \
          "s3://${{ github.event.inputs.destination_bucket }}/$(basename ${{ github.event.inputs.file_path }})"
        
        echo "File transferred successfully!"

    - name: Notify completion
      run: echo "Transfer completed at $(date)"
```

## Alternative Storage Options

### For Azure Blob Storage:
```yaml
- name: Upload to Azure Blob
  uses: azure/CLI@v1
  with:
    azcliversion: 2.0.72
    inlineScript: |
      az storage blob upload \
        --account-name ${{ secrets.AZURE_STORAGE_ACCOUNT }} \
        --container-name ${{ secrets.AZURE_CONTAINER }} \
        --name $(basename ${{ github.event.inputs.file_path }}) \
        --file "${{ github.event.inputs.file_path }}" \
        --auth-mode key
```

### For Google Cloud Storage:
```yaml
- name: Upload to GCS
  uses: google-github-actions/upload-cloud-storage@v1
  with:
    credentials: ${{ secrets.GCP_CREDENTIALS }}
    destination: gs://${{ secrets.GCS_BUCKET }}/$(basename ${{ github.event.inputs.file_path }})
    source: ${{ github.event.inputs.file_path }}
```

## API Call to Trigger Workflow

To trigger this via API:

```bash
curl -X POST \
  -H "Authorization: token YOUR_GITHUB_TOKEN" \
  -H "Accept: application/vnd.github.v3+json" \
  https://api.github.com/repos/OWNER/REPO/actions/workflows/FILE_NAME/dispatches \
  -d '{
    "ref": "main",
    "inputs": {
      "file_path": "path/to/your/file.txt",
      "destination_bucket": "your-bucket-name"
    }
  }'
```

## Setup Steps:

1. **Create workflow file**: `.github/workflows/file-transfer.yml`
2. **Add secrets**: Store credentials in GitHub Secrets (Settings → Secrets → Actions)
3. **Test manually**: Use "Run workflow" in GitHub UI
4. **Call via API**: Use the API endpoint to trigger programmatically

