## Fetch and send data to s3

```yaml
name: 4 - Fetch and send data to s3

on:
  workflow_dispatch:  # Manual trigger

jobs:
  fetch_data:  # First job: Data collection
    runs-on: ubuntu-latest

    steps:
      - name: Checkout the repository
        uses: actions/checkout@v4

      - name: Install jq
        run: sudo apt-get install jq

      - name: Fetch data
        run: |
          TIMESTAMP=$(date +%s)
          OUTPUT_FILE="stage/${TIMESTAMP}.csv"
          mkdir -p stage
          curl -s -H "User-Agent: Chrome/123.0" https://www.sahamyab.com/guest/twiter/list?v=0.1 | jq '.items[] | [.id, .sendTime, .sendTimePersian, .senderName, .senderUsername, .type, .content] | join(",") ' > $OUTPUT_FILE
          echo "Data saved to $OUTPUT_FILE"

      - name: Commit and push changes
        run: |
          git config --local user.email "amin@hotmail.com"
          git config --local user.name "amin"
          git add stage/
          git commit -m "Fetch data at $(date)"
          git push
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}

  send-to-bucket:  # Second job: Storage upload
    runs-on: ubuntu-latest
    needs: fetch_data  # Dependency: runs only after fetch_data completes

    steps:
      - name: Checkout the repository
        uses: actions/checkout@v4

      - name: set up python
        uses: actions/setup-python@v5
        with:
          python-version: '3.10'

      - name: Install dependencies
        run: |
          pip install -r requirements.txt

      - name: Run upload script
        run: |
          python send-to-object-storage.py
        env:
          ACCESS_KEY: ${{ secrets.S3_BUCKET_ACCESS_KEY }}
          SECRET_KEY: ${{ secrets.S3_BUCKET_SECRET_ACCESS_KEY }}
```

## Key Concepts Demonstrated:

### 1. **Multi-Job Workflow with Dependencies**
```yaml
needs: fetch_data  # Critical: ensures order of execution
```
- **`send-to-bucket`** job waits for **`fetch_data`** to complete
- Jobs run in parallel by default, but `needs` creates sequence
- If `fetch_data` fails, `send-to-bucket` won't run

### 2. **Job Separation**
- **Job 1**: Data fetching and git operations
- **Job 2**: Cloud storage upload (different environment setup)

### 3. **Python Environment Setup**
```yaml
- name: set up python
  uses: actions/setup-python@v5
  with:
    python-version: '3.10'
```
- Uses GitHub's official Python action
- Specific Python version (3.10) ensures consistency

### 4. **Dependency Management**
```yaml
- name: Install dependencies
  run: |
    pip install -r requirements.txt
```
- Installs Python packages from requirements.txt
- Ensures script has all required libraries

### 5. **Secure Credential Handling**
```yaml
env:
  ACCESS_KEY: ${{ secrets.S3_BUCKET_ACCESS_KEY }}
  SECRET_KEY: ${{ secrets.S3_BUCKET_SECRET_ACCESS_KEY }}
```
- Credentials stored in GitHub Secrets (secure)
- Passed as environment variables to Python script

## Workflow Execution Flow:

1. **Trigger**: Manual via GitHub UI/API
2. **Job 1 (fetch_data)**:
   - Checks out code
   - Installs jq
   - Fetches API data → saves as CSV
   - Commits and pushes data to git
3. **Job 2 (send-to-bucket)**:
   - Waits for Job 1 to complete
   - Checks out latest code (with new data)
   - Sets up Python environment
   - Installs dependencies
   - Runs Python script with AWS credentials

## Example Python Script (`send-to-object-storage.py`):

```python
import os
import boto3
from datetime import datetime

# Get credentials from environment variables
access_key = os.environ.get('ACCESS_KEY')
secret_key = os.environ.get('SECRET_KEY')

# Initialize S3 client
s3 = boto3.client(
    's3',
    aws_access_key_id=access_key,
    aws_secret_access_key=secret_key,
    region_name='us-east-1'  # Adjust as needed
)

def upload_files_to_s3():
    bucket_name = 'your-bucket-name'  # Set your bucket name
    stage_dir = 'stage/'
    
    # Upload all CSV files in stage directory
    for filename in os.listdir(stage_dir):
        if filename.endswith('.csv'):
            file_path = os.path.join(stage_dir, filename)
            s3_key = f"twitter-data/{filename}"
            
            try:
                s3.upload_file(file_path, bucket_name, s3_key)
                print(f"Successfully uploaded {filename} to S3")
            except Exception as e:
                print(f"Error uploading {filename}: {e}")

if __name__ == "__main__":
    upload_files_to_s3()
```

## Required `requirements.txt`:
```
boto3==1.28.0
```

## Setup Requirements:

1. **Create GitHub Secrets**:
   - `S3_BUCKET_ACCESS_KEY`: AWS access key
   - `S3_BUCKET_SECRET_ACCESS_KEY`: AWS secret key

2. **Create Python script** with S3 upload logic

3. **Create requirements.txt** with boto3 dependency

## Improvement Suggestions:

```yaml
# Add error handling and notifications
- name: Notify on failure
  if: failure()
  uses: actions/github-script@v6
  with:
    script: |
      github.rest.issues.create({
        owner: context.repo.owner,
        repo: context.repo.repo,
        title: 'Workflow failed',
        body: 'The fetch and upload workflow failed!'
      })
```

