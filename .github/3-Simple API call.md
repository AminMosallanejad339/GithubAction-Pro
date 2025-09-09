

## Simple API call

```yaml
name: 3 - Simple API call  # Workflow name

on:
  schedule:
   - cron: '23 06 * * *'   # Scheduled at 06:23 UTC daily
   - cron: '30 06 * * *'   # Scheduled at 06:30 UTC daily
  workflow_dispatch:       # Manual trigger option

jobs:
  fetch_data:
    runs-on: ubuntu-latest
    
    steps:
      - name: Checkout the repository
        uses: actions/checkout@v4  # Get repository code

      - name: Install jq
        run: sudo apt-get install jq  # Install JSON processor

      - name: Fetch data
        run: |
          TIMESTAMP=$(date +%s)  # Unix timestamp
          OUTPUT_FILE="stage/${TIMESTAMP}.csv"  # Dynamic filename
          mkdir -p stage  # Create directory if not exists
          curl -s -H "User-Agent: Chrome/123.0" https://www.sahamyab.com/guest/twiter/list?v=0.1 | jq '.items[] | [.id, .sendTime, .sendTimePersian, .senderName, .senderUsername, .type, .content] | join(",") ' > $OUTPUT_FILE
          echo "Data saved to $OUTPUT_FILE"

      - name: Commit and push changes
        run: |
          git config --local user.email  	"amin@hotmail.com"
          git config --local user.name "amin"
          git add stage/
          git commit -m "Fetch data at $(date)"
          git push
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}  # Authentication
```

## Key Concepts Demonstrated:

### 1. **Scheduled Triggers (Cron)**
```yaml
schedule:
 - cron: '23 06 * * *'   # Minute 23, Hour 06, Daily
 - cron: '30 06 * * *'   # Multiple schedules possible
```
- **Format**: `minute hour day month day-of-week`
- Runs automatically at specified times
- Multiple schedules can be defined

### 2. **API Call with Processing**
```bash
curl -s -H "User-Agent: Chrome/123.0" https://example.com/api | jq '.items[] | [.data] | join(",")'
```
- **`curl`**: Fetches API data
- **`-s`**: Silent mode (no progress)
- **`-H`**: Sets headers (User-Agent to avoid blocking)
- **`jq`**: Processes JSON and converts to CSV format

### 3. **Dynamic File Naming**
```bash
TIMESTAMP=$(date +%s)
OUTPUT_FILE="stage/${TIMESTAMP}.csv"
```
- Uses Unix timestamp for unique filenames
- Prevents file overwrites

### 4. **Automated Git Operations**
```bash
git config --local user.email "email"
git config --local user.name "name"
git add stage/
git commit -m "message"
git push
```
- Configures git user
- Stages new files
- Commits with timestamp
- Pushes changes automatically

### 5. **GitHub Token Authentication**
```yaml
env:
  GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```
- Automatic token provided by GitHub
- Allows pushing to the repository

## Workflow Execution Flow:

1. **Trigger**: Either scheduled (06:23/06:30 UTC) or manual
2. **Checkout**: Gets the latest code
3. **Setup**: Installs required tools (jq)
4. **Data Fetch**: Calls API, processes JSON to CSV
5. **Git Operations**: Commits and pushes new data files
6. **Result**: Repository updated with new data files

## Real-World Applications:

- **Data collection pipelines**: Daily stock prices, weather data, social media metrics
- **Automated backups**: Regular database exports
- **Monitoring**: Periodic health checks and reporting
- **Content aggregation**: News feeds, blog posts, etc.

## Improvement Suggestions:

```yaml
# Add error handling
- name: Fetch data
  run: |
    set -e  # Exit on error
    TIMESTAMP=$(date +%s)
    OUTPUT_FILE="stage/${TIMESTAMP}.csv"
    mkdir -p stage
    if ! curl -s -f -H "User-Agent: Chrome/123.0" https://www.sahamyab.com/guest/twiter/list?v=0.1 | jq -r '.items[] | [.id, .sendTime, .sendTimePersian, .senderName, .senderUsername, .type, .content] | @csv' > $OUTPUT_FILE; then
      echo "API call failed!"
      exit 1
    fi
    echo "Data saved to $OUTPUT_FILE"

# Add file validation
- name: Validate CSV
  run: |
    if [ ! -s "stage/$(ls -t stage/ | head -1)" ]; then
      echo "Error: CSV file is empty!"
      exit 1
    fi
```

 