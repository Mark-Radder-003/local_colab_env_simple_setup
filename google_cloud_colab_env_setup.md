# Google Cloud Setup Guide for colab_env

This guide provides detailed instructions for setting up and using Google Cloud tools in your `colab_env` mamba environment.

## Prerequisites

- Mamba installed (you have this ✓)
- `colab_env` environment created with Google Cloud SDK and Python libraries (completed ✓)
- A Google Cloud account (if you don't have one, visit https://cloud.google.com)


# 👇do in your terminal
## Step 1: Activate Your Environment

<!-- First, you need to activate the `colab_env` environment where all Google Cloud tools are installed.

### Initial Setup (One-time only)

If you haven't initialized mamba for your shell yet:

```bash
# Initialize mamba for zsh (your current shell)
eval "$(mamba shell hook --shell zsh)"

# Make it permanent (optional but recommended)
mamba shell init --shell zsh --root-prefix=/Users/marktan/miniforge3
source ~/.zshrc
``` -->

### Activate the Environment

```bash
mamba activate colab_env
```

You should see `(colab_env)` appear in your terminal prompt, indicating the environment is active.

## Step 2: Authenticate with Google Cloud

Authentication allows you to access Google Cloud services using your Google account.

### Basic Authentication

```bash
gcloud auth login
```

This command will:
1. Open your default web browser
2. Ask you to select your Google account
3. Request permission to access Google Cloud resources
4. Display a success message in your terminal


<!-- ### Application Default Credentials (for Python libraries)

For Python scripts to access Google Cloud services:

```bash
gcloud auth application-default login

gcloud auth login --enable-gdrive-access
```

```bash
# If you want to specify scopes (optional but recommended for broader access)
gcloud auth application-default login --scopes=https://www.googleapis.com/auth/cloud-platform
```

This creates credentials that Python libraries like `google-cloud-storage` and `google-cloud-bigquery` will automatically use. -->

## ✅ `OAuth 2.0` Approach

### Using `OAuth 2.0` to Get User Credentials with Google Drive Permissions

This is the most recommended practice, suitable for your current local Notebook environment.

🔧 Steps:

    Create OAuth credentials (one-time operation) - Visit Google Cloud Console - Credentials

    Create OAuth 2.0 Client ID

    Application type: Desktop App

    Download client_secrets.json

Run the following code in your Notebook:

### ✅ Method A: Always Use `client.query()` + Explicit `creds`

- Full functionality, supports Drive
- Flexible integration with Pandas and Notebook code
```python
from google_auth_oauthlib.flow import InstalledAppFlow
from google.cloud import bigquery

SCOPES = [
   "https://www.googleapis.com/auth/bigquery"
   , "https://www.googleapis.com/auth/drive"
   ]
flow = InstalledAppFlow.from_client_secrets_file("client_secrets.json", SCOPES)
creds = flow.run_local_server(port=0)

client = bigquery.Client(credentials=creds, project="your-project-id")

query = "SELECT col2 FROM `your_dataset.atr_modified` GROUP BY 1"
results = client.query(query).result()
```

✅ Success Reason: You actively authorized `OAuth` credentials that include Drive access permissions and injected them into the BigQuery client.


### ✅ Method B (Advanced): Force Inject `creds` into `%%bigquery` Magic Environment

When you try to use the `%%bigquery` magic command to execute the same query:

```sql
%%bigquery col2
SELECT col2 FROM `your_dataset.atr_modified` GROUP BY 1
```

It fails again with Drive permission-related errors.

### 📌 Root Cause Analysis:

- `%%bigquery` defaults to using **Application Default Credentials (ADC)**, not your authorized `creds`
- Therefore it cannot access Google Drive tables

---

### ✅ Recommended Solutions
```python
from google_auth_oauthlib.flow import InstalledAppFlow
from google.cloud.bigquery.magics import context


SCOPES = [
   "https://www.googleapis.com/auth/bigquery"
   , "https://www.googleapis.com/auth/drive"
   ]
flow = InstalledAppFlow.from_client_secrets_file("client_secrets.json", SCOPES)
creds = flow.run_local_server(port=0)


context.credentials = creds

context.project = "your-project-id"
```


## Step 3: Set Your Default Project

Google Cloud organizes resources into projects. You need to set a default project for your commands.

### Find Your Project ID
Use this command to view your project list:

```bash
gcloud projects list
```

This will display:
- Project ID
- Project Name  
- Project Number


### Set Project ID as an Environment Variable

To avoid repeatedly typing your project ID, set it as an environment variable:

```bash
# Set the project ID variable (replace with your actual project ID)
export PROJECT_ID="your actual project ID"

# Set it as the default project
gcloud config set project $PROJECT_ID
```

### Verify Configuration

```bash
# View current configuration
gcloud config list

# View just the current project
gcloud config get-value project

# Check your PROJECT_ID variable
echo $PROJECT_ID
```

<!-- ### Make it Permanent (Optional)

To have the PROJECT_ID available every time you activate your environment:

```bash
# Add to your shell configuration file
echo 'export PROJECT_ID="your-actual-project-id"' >> ~/.zshrc

# Or, better yet, create an environment-specific activation script
mkdir -p ~/miniforge3/envs/colab_env/etc/conda/activate.d
echo 'export PROJECT_ID="your-actual-project-id"' > ~/miniforge3/envs/colab_env/etc/conda/activate.d/env_vars.sh

# Also create deactivation script to clean up
mkdir -p ~/miniforge3/envs/colab_env/etc/conda/deactivate.d
echo 'unset PROJECT_ID' > ~/miniforge3/envs/colab_env/etc/conda/deactivate.d/env_vars.sh
```

Now, whenever you activate `colab_env`, the PROJECT_ID will be automatically set. -->



## Common Google Cloud Commands

### BigQuery Commands

```bash
# List datasets
bq ls

# Query data
bq query --use_legacy_sql=false 'SELECT * FROM dataset.table LIMIT 10'

# Create a dataset
bq mk --dataset --location=US my_new_dataset

# Load data from CSV
bq load --source_format=CSV dataset.table gs://bucket/file.csv
```

### Cloud Storage Commands

```bash
# List buckets
gsutil ls

# List files in a bucket
gsutil ls gs://your-bucket-name/

# Copy file to bucket
gsutil cp local_file.csv gs://your-bucket-name/

# Download file from bucket
gsutil cp gs://your-bucket-name/file.csv ./

# Create a bucket
gsutil mb -l us-central1 gs://your-new-bucket-name/
```

### General gcloud Commands

```bash
# List all projects
gcloud projects list

# Switch between projects
gcloud config set project OTHER_PROJECT_ID

# View current account
gcloud auth list

# Set default region
gcloud config set compute/region us-central1
```

# 👇do in your notebook

## Using Python Libraries

### Basic BigQuery Example

```python
from google.cloud import bigquery
import pandas as pd
import os

# Use the PROJECT_ID environment variable
project_id = os.environ.get('PROJECT_ID')

# Create a client with the project ID
client = bigquery.Client(project=project_id)

# Run a query
query = """
    SELECT name, count
    FROM `bigquery-public-data.usa_names.usa_1910_2013`
    WHERE state = 'CA'
    LIMIT 10
"""

# Execute query and get results as pandas DataFrame
df = client.query(query).to_dataframe()
print(df)
```

### Using pandas-gbq

```python
import pandas_gbq
import os

# Get project ID from environment variable
project_id = os.environ.get('PROJECT_ID')

# Read from BigQuery
df = pandas_gbq.read_gbq(
    'SELECT * FROM `project.dataset.table` LIMIT 1000',
    project_id=project_id
)

# Write to BigQuery
pandas_gbq.to_gbq(
    df, 
    'dataset.new_table',
    project_id=project_id,
    if_exists='replace'
)
```
# bigquery-magics
In a notebook cell:
```python
%load_ext google.cloud.bigquery
```
# Then run queries with:
```python
%%bigquery
SELECT * FROM `bigquery-public-data.samples.shakespeare` LIMIT 5

# Or save results to a DataFrame:
%%bigquery df
SELECT * FROM `bigquery-public-data.samples.shakespeare` LIMIT 5
```



### Cloud Storage Example

```python
from google.cloud import storage
import pandas as pd
import os

# Use the PROJECT_ID environment variable
project_id = os.environ.get('PROJECT_ID')

# Create a client with the project ID
client = storage.Client(project=project_id)

# List buckets
for bucket in client.list_buckets():
    print(bucket.name)

# Upload a file
bucket = client.bucket('your-bucket-name')
blob = bucket.blob('data/myfile.csv')
blob.upload_from_filename('local_file.csv')

# Download a file
blob = bucket.blob('data/myfile.csv')
blob.download_to_filename('downloaded_file.csv')

# Read CSV directly from GCS to pandas
df = pd.read_csv('gs://bucket-name/path/to/file.csv')
```

## Notebook Integration

When using notebooks in `colab_env`:

```python
import os

# Set up project ID in notebook
PROJECT_ID = os.environ.get('PROJECT_ID')

# Load BigQuery magic commands
%load_ext google.cloud.bigquery

# Run queries with %%bigquery magic (uses default project)
%%bigquery df
SELECT * 
FROM `bigquery-public-data.samples.shakespeare`
LIMIT 10

# Or specify project explicitly
%%bigquery df --project $PROJECT_ID
SELECT * 
FROM `{}.dataset.table`
LIMIT 10
```

## Troubleshooting

### Common Issues and Solutions

1. **"You do not currently have an active account selected"**
   ```bash
   gcloud auth login
   ```

2. **"The project id is not set"**
   ```bash
   gcloud config set project YOUR_PROJECT_ID
   ```

3. **Permission denied errors**
   - Ensure your Google account has the necessary permissions in the project
   - Contact your project administrator to grant required roles

4. **Python import errors**
   - Make sure you've activated the `colab_env` environment
   - Verify installation: `mamba list | grep google`

### Useful Environment Commands

```bash
# Check what's installed in your environment
mamba list | grep google

# Update packages
mamba update -n colab_env google-cloud-bigquery google-cloud-storage

# Deactivate environment when done
mamba deactivate
```

## Best Practices

1. **Security**
   - Never share your credentials or commit them to version control
   - Use service accounts for production applications
   - Rotate credentials regularly

2. **Cost Management**
   - Set up budget alerts in Google Cloud Console
   - Use `LIMIT` clauses in BigQuery queries during development
   - Delete unused resources

3. **Performance**
   - Use appropriate data types in BigQuery
   - Partition large tables by date
   - Use Cloud Storage for staging large data transfers

## Additional Resources

- [Google Cloud Documentation](https://cloud.google.com/docs)
- [BigQuery Documentation](https://cloud.google.com/bigquery/docs)
- [Cloud Storage Documentation](https://cloud.google.com/storage/docs)
- [Python Client Libraries](https://cloud.google.com/python/docs/reference)
- [gcloud CLI Reference](https://cloud.google.com/sdk/gcloud/reference)

## Quick Reference Card

```bash
# Environment
mamba activate colab_env          # Activate environment
mamba deactivate                  # Deactivate environment

# Project Setup
export PROJECT_ID="your-project-id"    # Set project ID variable
gcloud config set project $PROJECT_ID  # Set default project
echo $PROJECT_ID                       # Check project ID

# Authentication
gcloud auth login                 # Login to Google Cloud
gcloud auth list                  # List authenticated accounts
gcloud auth application-default login  # Set up ADC for Python

# Configuration
gcloud config list                     # View all settings
gcloud projects list                   # List available projects

# BigQuery (using PROJECT_ID)
bq ls --project_id=$PROJECT_ID         # List datasets
bq query --project_id=$PROJECT_ID "SELECT ..."  # Run query
bq show $PROJECT_ID:dataset.table      # Show table schema

# Cloud Storage
gsutil ls                         # List buckets
gsutil cp file.csv gs://bucket/   # Upload file
gsutil cat gs://bucket/file.txt   # View file contents

# Python Quick Start
python -c "import os; print(f'PROJECT_ID: {os.environ.get(\"PROJECT_ID\")}')"
```
