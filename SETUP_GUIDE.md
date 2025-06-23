# Google Cloud Setup Guide

Complete configuration for Google Cloud Platform integration.

## Prerequisites

- Environment created and activated ([see README.md](./README.md))
- Google Cloud account
- Basic commands available ([see COMMANDS.md](./COMMANDS.md))

## Authentication Methods

### Method 1: Basic Authentication (Recommended)

```bash
mamba activate colab_env
gcloud auth login
gcloud auth application-default login
```

### Method 2: OAuth 2.0 with Drive Access (Advanced)

Required for Google Drive-based BigQuery external tables.

#### Create OAuth Credentials
1. Visit [Google Cloud Console - Credentials](https://console.cloud.google.com/apis/credentials)
2. Create OAuth 2.0 Client ID (Desktop App)
3. Download `Oauth20_client_secrets.json`

#### Use in Python/Jupyter
```python
from google_auth_oauthlib.flow import InstalledAppFlow
from google.cloud import bigquery

SCOPES = [
    "https://www.googleapis.com/auth/bigquery",
    "https://www.googleapis.com/auth/drive"
]

flow = InstalledAppFlow.from_client_secrets_file("Oauth20_client_secrets.json", SCOPES)
creds = flow.run_local_server(port=0)

# Use with BigQuery client
client = bigquery.Client(credentials=creds, project="your-project-id")
```

#### Configure BigQuery Magic Commands
```python
from google.cloud.bigquery.magics import context

context.credentials = creds
context.project = "your-project-id"
```

## Project Configuration

### Find and Set Project
```bash
# List projects
gcloud projects list

# Set project (session-specific)
export PROJECT_ID="your-project-id"
gcloud config set project $PROJECT_ID

# Verify
gcloud config list
echo $PROJECT_ID
```

### Make Project Persistent
```bash
# Create environment activation script
mkdir -p ~/miniforge3/envs/colab_env/etc/conda/activate.d
echo 'export PROJECT_ID="your-project-id"' > ~/miniforge3/envs/colab_env/etc/conda/activate.d/env_vars.sh

# Create deactivation cleanup
mkdir -p ~/miniforge3/envs/colab_env/etc/conda/deactivate.d
echo 'unset PROJECT_ID' > ~/miniforge3/envs/colab_env/etc/conda/deactivate.d/env_vars.sh
```

## Python Library Usage

### BigQuery Examples
```python
from google.cloud import bigquery
import pandas as pd
import os

project_id = os.environ.get('PROJECT_ID')
client = bigquery.Client(project=project_id)

# Run query
query = """
    SELECT name, count
    FROM `bigquery-public-data.usa_names.usa_1910_2013`
    WHERE state = 'CA'
    ORDER BY count DESC
    LIMIT 10
"""
df = client.query(query).to_dataframe()
```

### pandas-gbq Integration
```python
import pandas_gbq

# Read from BigQuery
df = pandas_gbq.read_gbq(
    'SELECT * FROM `project.dataset.table` LIMIT 1000',
    project_id=project_id
)

# Write to BigQuery
pandas_gbq.to_gbq(
    df, 'dataset.new_table',
    project_id=project_id,
    if_exists='replace'
)
```

### Cloud Storage Integration
```python
from google.cloud import storage
import pandas as pd

client = storage.Client(project=project_id)

# List buckets
for bucket in client.list_buckets():
    print(bucket.name)

# Upload/download files
bucket = client.bucket('bucket-name')
blob = bucket.blob('data/file.csv')
blob.upload_from_filename('local_file.csv')

# Read CSV directly from GCS
df = pd.read_csv('gs://bucket-name/path/to/file.csv')
```

## Jupyter Notebook Integration

### Load Extensions
```python
%load_ext google.cloud.bigquery
import os
os.environ['PROJECT_ID'] = 'your-project-id'
```

### BigQuery Magic Usage
```python
# Simple query
%%bigquery
SELECT * FROM `bigquery-public-data.samples.shakespeare` LIMIT 5

# Save to DataFrame
%%bigquery df --project $PROJECT_ID
SELECT name, year, number
FROM `bigquery-public-data.usa_names.usa_1910_current`
WHERE name = 'Alice'
ORDER BY year
LIMIT 10
```

## Troubleshooting

### Common Issues

**"You do not currently have an active account selected"**
```bash
gcloud auth login
```

**"The project id is not set"**
```bash
gcloud config set project YOUR_PROJECT_ID
```

**Permission denied errors**
- Check IAM roles in Google Cloud Console
- Ensure proper permissions (BigQuery User, Storage Object Viewer)

**Python import errors**
```bash
mamba activate colab_env
mamba list | grep google
```

**BigQuery magic not working**
```python
%reload_ext google.cloud.bigquery
!gcloud auth list
```

### Environment Maintenance
```bash
# Update Google Cloud packages
mamba update google-cloud-bigquery google-cloud-storage pandas-gbq

# Clean environment
mamba clean --all
```

## Best Practices

### Security
- Never commit credentials to version control
- Use service accounts for production
- Regularly rotate credentials

### Performance
- Use `LIMIT` in development queries
- Partition large BigQuery tables
- Cache frequently used datasets

### Cost Management
- Set up billing alerts in Google Cloud Console
- Monitor BigQuery slot usage
- Delete unused resources