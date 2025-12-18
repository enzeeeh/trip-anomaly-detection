# GitHub Actions Guide

## What are GitHub Actions?

GitHub Actions is an automation tool that runs workflows automatically when certain events happen in your repository (push, pull request, schedule, etc.). Think of it as a robot assistant that performs tasks for you!

## 📁 Workflow Location

Workflows are stored in: `.github/workflows/`

## 🎯 Our Current Workflow: `notebook-validation.yml`

### What it does:
1. **Validates notebooks** - Checks if .ipynb files have correct syntax
2. **Converts to HTML** - Creates HTML versions of your notebooks
3. **Tests data quality** - Validates your CSV data structure

### When it runs:
- ✅ When you `git push` to `main` or `develop` branches
- ✅ When you create a Pull Request to `main`
- ✅ When you manually trigger it from GitHub UI
- ✅ Only when notebook (`.ipynb`) or Python (`.py`) files change

## 🚀 How to Use

### 1. View Workflow Runs

Go to your GitHub repository:
```
https://github.com/enzeeeh/trip-anomaly-detection/actions
```

You'll see:
- ✅ Green checkmark = Success
- ❌ Red X = Failed
- 🟡 Yellow circle = Running

### 2. Manually Trigger a Workflow

1. Go to **Actions** tab
2. Click on **Notebook Validation** workflow
3. Click **Run workflow** button (right side)
4. Select branch and click green **Run workflow**

### 3. Download Artifacts

After a successful run:
1. Go to the workflow run
2. Scroll down to **Artifacts** section
3. Download `notebook-html` (HTML versions of your notebooks)

## 📝 Common GitHub Actions Use Cases

### Example 1: Run on Schedule (Daily/Weekly)
```yaml
on:
  schedule:
    - cron: '0 0 * * 0'  # Every Sunday at midnight UTC
```

### Example 2: Send Notifications
```yaml
- name: Send Slack notification
  if: failure()
  uses: slackapi/slack-github-action@v1
  with:
    webhook-url: ${{ secrets.SLACK_WEBHOOK }}
```

### Example 3: Run Python Tests
```yaml
- name: Run pytest
  run: |
    pip install pytest
    pytest tests/
```

### Example 4: Deploy to Cloud
```yaml
- name: Deploy to AWS
  run: |
    aws s3 sync ./outputs s3://my-bucket/
```

## 🔐 Using Secrets

For sensitive data (API keys, passwords):

1. Go to **Settings** → **Secrets and variables** → **Actions**
2. Click **New repository secret**
3. Add name and value
4. Use in workflow: `${{ secrets.MY_SECRET }}`

Example:
```yaml
- name: Connect to BigQuery
  env:
    GOOGLE_CREDENTIALS: ${{ secrets.GCP_CREDENTIALS }}
  run: |
    python extract_data.py
```

## 🎨 Customizing the Workflow

### Change Python version:
```yaml
env:
  PYTHON_VERSION: '3.12'  # Change from 3.11 to 3.12
```

### Add more jobs:
```yaml
jobs:
  my-new-job:
    name: My Custom Job
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Do something
        run: echo "Hello!"
```

### Run on different triggers:
```yaml
on:
  push:
    branches: [ main, develop, feature/* ]  # All feature branches
  release:
    types: [ published ]  # When you create a release
  issue_comment:
    types: [ created ]  # When someone comments on an issue
```

## 🛠️ Common Commands

### Test workflow locally (requires act):
```bash
# Install act: https://github.com/nektos/act
act -j validate-notebooks
```

### Check workflow syntax:
```bash
# Install actionlint: https://github.com/rhysd/actionlint
actionlint .github/workflows/*.yml
```

## 📊 Practical Examples for Data Analysis

### 1. Auto-generate reports
```yaml
- name: Generate weekly report
  run: |
    jupyter nbconvert --execute --to html weekly_report.ipynb
    # Upload to S3, send email, etc.
```

### 2. Data validation on new data
```yaml
- name: Validate new data
  run: |
    python scripts/validate_data.py
    if [ $? -eq 0 ]; then
      echo "✓ Data is valid"
    else
      echo "✗ Data validation failed"
      exit 1
    fi
```

### 3. Scheduled data refresh
```yaml
on:
  schedule:
    - cron: '0 2 * * *'  # Daily at 2 AM UTC
jobs:
  refresh-data:
    runs-on: ubuntu-latest
    steps:
      - name: Extract from BigQuery
        run: python extract_bigquery.py
      - name: Commit updated data
        run: |
          git config user.name "GitHub Actions"
          git config user.email "actions@github.com"
          git add trip_data.csv
          git commit -m "chore: update data $(date +%Y-%m-%d)"
          git push
```

## 🔍 Monitoring & Debugging

### View logs:
1. Click on a workflow run
2. Click on a job name
3. Expand each step to see detailed logs

### Debug mode:
Add to your workflow:
```yaml
env:
  ACTIONS_STEP_DEBUG: true
  ACTIONS_RUNNER_DEBUG: true
```

## 💡 Tips

1. **Start small** - Test workflows on a feature branch first
2. **Use caching** - Speed up runs with `cache: 'pip'`
3. **Fail fast** - Use `if: failure()` to handle errors
4. **Matrix builds** - Test multiple Python versions:
   ```yaml
   strategy:
     matrix:
       python-version: ['3.9', '3.10', '3.11', '3.12']
   ```

## 📚 Resources

- [GitHub Actions Documentation](https://docs.github.com/en/actions)
- [Marketplace](https://github.com/marketplace?type=actions) - Pre-built actions
- [Workflow Syntax](https://docs.github.com/en/actions/using-workflows/workflow-syntax-for-github-actions)

## 🎓 Next Steps

1. ✅ Push your workflow to GitHub (already done!)
2. Go to Actions tab and watch it run
3. Try triggering it manually
4. Download the HTML artifacts
5. Customize for your needs!
