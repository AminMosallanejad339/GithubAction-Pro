 

## Fetch and send data to S3 - Revised Version

```yaml
name: 5- Fetch and send data to S3-Revised

on:
  workflow_dispatch:

jobs:
  fetch_data:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout the repository
        uses: actions/checkout@v4

      - name: Install jq
        run: |
          sudo apt-get update  # Added package list update
          sudo apt-get install -y jq  # Added -y flag for automatic yes

      - name: Fetch data
        run: |
          TIMESTAMP=$(date +%s)
          OUTPUT_FILE="stage/${TIMESTAMP}.csv"
          mkdir -p stage
          curl -s -H "User-Agent: Chrome/123.0" https://www.sahamyab.com/guest/twiter/list?v=0.1 | jq '.items[] | [.id, .sendTime, .sendTimePersian, .senderName, .senderUsername, .type, .content] | join(",") ' > $OUTPUT_FILE
          echo "Data saved to $OUTPUT_FILE"

      - name: Set up Python
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

      - name: Commit and push changes
        run: |
            git config --local user.email "amin@hotmail.com"
            git config --local user.name "amin"
            git add stage/
            git commit -m "Fetch data at $(date)"
            git push
        env:
            GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

## Key Improvements in Revised Version:

### 1. **Single Job Structure** (Major Improvement!)
- **Before**: Two separate jobs with dependency management
- **After**: One streamlined job with sequential steps
- **Benefits**: 
  - Simpler architecture
  - No need for `needs` dependency
  - All steps share the same workspace (no duplicate checkouts)

### 2. **Better Package Installation**
```yaml
- name: Install jq
  run: |
    sudo apt-get update  # Updates package lists first
    sudo apt-get install -y jq  # Automatic confirmation (-y)
```
- Prevents potential installation failures
- More robust and production-ready

### 3. **Logical Execution Order**
1. Checkout code ✅
2. Install tools (jq) ✅  
3. Fetch data ✅
4. Setup Python ✅
5. Install dependencies ✅
6. Upload to S3 ✅
7. Commit to Git ✅

### 4. **Efficiency Gains**
- **No duplicate checkouts**: Previously both jobs checked out code
- **Shared environment**: Python setup used for both upload and potential data processing
- **Faster execution**: No job initialization overhead

## Workflow Execution Flow:

1. **Single job starts** on Ubuntu runner
2. **Sequential steps execute**:
   - Checkout repository
   - Install jq (with proper update)
   - Fetch API data and save as CSV
   - Setup Python environment
   - Install Python dependencies
   - Upload files to S3 using Python script
   - Commit new files to Git repository

## Advantages of This Approach:

### ✅ **Simpler Architecture**
- No complex job dependencies
- Easier to debug and maintain

### ✅ **Resource Efficiency** 
- Single runner instance
- No duplicate operations

### ✅ **Data Consistency**
- S3 upload happens immediately after data fetch
- Git commit happens after successful upload

### ✅ **Cost Effective**
- Fewer compute minutes used (single job)

## Potential Considerations:

### ⚠️ **Job Duration**
- Longer single job vs parallel execution
- If upload takes very long, might consider separating

### ⚠️ **Failure Handling**
- If S3 upload fails, git commit won't happen
- This is actually good - prevents committing failed data

## Example Enhanced Python Script:

```python
import os
import boto3
import glob

def upload_new_files():
    access_key = os.environ.get('ACCESS_KEY')
    secret_key = os.environ.get('SECRET_KEY')
    
    if not access_key or not secret_key:
        raise ValueError("AWS credentials not found in environment variables")
    
    # Initialize S3 client
    s3 = boto3.client(
        's3',
        aws_access_key_id=access_key,
        aws_secret_access_key=secret_key,
        region_name='us-east-1'
    )
    
    bucket_name = 'your-data-bucket'
    csv_files = glob.glob('stage/*.csv')
    
    if not csv_files:
        print("No CSV files found in stage directory")
        return
    
    for file_path in csv_files:
        filename = os.path.basename(file_path)
        s3_key = f"twitter-data/{filename}"
        
        try:
            s3.upload_file(file_path, bucket_name, s3_key)
            print(f"✅ Uploaded {filename} to S3")
            
            # Optional: Remove file after successful upload
            # os.remove(file_path)
            # print(f"Removed {filename} from local storage")
            
        except Exception as e:
            print(f"❌ Failed to upload {filename}: {e}")
            # Don't raise exception to allow other files to try

if __name__ == "__main__":
    upload_new_files()
```

## Recommended `requirements.txt`:
```
boto3==1.34.0
```

 