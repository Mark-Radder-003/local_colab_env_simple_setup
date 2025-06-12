# Mamba Environment for Data Analysis who wants to work on Google Colab(Cloud based) Env and Local Env simultaneously

This project provides a simple Mamba environment setup (`colab_env_simple.yml`) tailored for data analysis tasks, with a focus on seamless integration with Google Cloud Platform services.

## Purpose

The main goal is to offer a straightforward, cross-platform, and easy-to-maintain Python environment that includes essential packages for:
- General data manipulation and analysis
- Data visualization
- Machine learning
- Geospatial analysis
- Google Cloud connectivity (BigQuery, Cloud Storage)

## Environment Specification File

- **`colab_env_simple.yml`**: This file contains the list of packages for the `colab_env` environment.

## Creating the Environment

Create this environment using Mamba:

### Using Mamba

```bash
# Create the environment from the YAML file
mamba env create -f colab_env_simple.yml

# Activate the environment
mamba activate colab_env
```


After activation, your terminal prompt should indicate the active environment, for example: `(colab_env) your_username@hostname:~$`.

## What's Included?

The `colab_env` environment comes with:

- **Python:** 3.12
- **Jupyter Ecosystem:** `jupyter`, `notebook`, `ipykernel` (for running Jupyter notebooks)
- **Data Analysis & Manipulation:** `pandas`, `numpy`, `pyarrow`
- **Data Visualization:** `matplotlib`, `seaborn`
- **Statistical Analysis:** `scipy`, `statsmodels`
- **Machine Learning:** `scikit-learn`
- **Geospatial Analysis:** `geopandas`
- **Google Cloud Platform:**
    - `google-cloud-sdk`: Command-line tools for Google Cloud (like `gcloud`, `bq`, `gsutil`)
    - `google-auth`, `google-auth-oauthlib`: For authentication with Google Cloud services
    - `google-cloud-storage`: Python client for Google Cloud Storage
    - `google-cloud-bigquery`: Python client for Google BigQuery
    - `pandas-gbq`: To easily read from and write to BigQuery with Pandas DataFrames
    - `bigquery-magics`: For using BigQuery SQL directly in Jupyter notebooks (`%bigquery` and `%%bigquery`)

## Setting Up and Using Google Cloud

Follow these steps to configure and use Google Cloud tools within the `colab_env`.

### 1. Activate Your Environment

Ensure the `colab_env` is active:

```bash
mamba activate colab_env
```

### 2. Authenticate with Google Cloud

**a. Basic CLI Authentication (for `gcloud`, `bq`, `gsutil`):**

This command will open a browser window for you to log in with your Google account and authorize access.

```bash
gcloud auth login
```

**b. Application Default Credentials (ADC) (for Python libraries):**

This allows Python scripts and libraries (like `google-cloud-storage`, `google-cloud-bigquery`) to automatically find and use your credentials.

```bash
gcloud auth application-default login
```
For broader access scopes (optional but often useful):
```bash
gcloud auth application-default login --scopes=https://www.googleapis.com/auth/cloud-platform
```

### 3. Set Your Default Google Cloud Project

Google Cloud organizes resources into projects. You need to tell `gcloud` which project to use by default.

**a. List your projects:**
If you don't know your Project ID, list your available projects:
```bash
gcloud projects list
```
Note the `PROJECT_ID` you want to use.

**b. Set the default project:**
Replace `YOUR_PROJECT_ID` with your actual Project ID.
```bash
# Set the project ID (replace with your actual project ID)
export PROJECT_ID="YOUR_PROJECT_ID"

# Set it as the default project for gcloud
gcloud config set project $PROJECT_ID
```
To make the `PROJECT_ID` environment variable persistent across terminal sessions or when reactivating the environment, you can add `export PROJECT_ID="YOUR_PROJECT_ID"` to your shell's configuration file (e.g., `~/.bashrc`, `~/.zshrc`) or use Conda's environment activation scripts.

**c. Verify configuration:**
```bash
gcloud config list project
# or
echo $PROJECT_ID
```

## Common Google Cloud Commands

Here are some frequently used commands for BigQuery and Cloud Storage.

### BigQuery

```bash
# List datasets in your default project
bq ls

# Run a query (uses standard SQL by default)
bq query --use_legacy_sql=false 'SELECT name, SUM(number) as total_count FROM `bigquery-public-data.usa_names.usa_1910_current` GROUP BY name ORDER BY total_count DESC LIMIT 10'

# Create a new dataset (e.g., my_new_dataset)
bq mk --dataset --location=US my_new_dataset

# Load data from a local CSV file into a BigQuery table
# bq load --source_format=CSV your_dataset.your_table ./local_file.csv schema.json
# (Requires a schema file or inline schema definition)

# Load data from a CSV file in Google Cloud Storage
# bq load --source_format=CSV --autodetect your_dataset.your_table gs://your_bucket_name/path/to/file.csv
```

### Cloud Storage (`gsutil`)

```bash
# List all your Cloud Storage buckets
gsutil ls

# List contents of a specific bucket
gsutil ls gs://your-bucket-name/

# Create a new bucket (bucket names must be globally unique)
# gsutil mb -l US-CENTRAL1 gs://your-new-unique-bucket-name/

# Copy a local file to a bucket
gsutil cp local_file.txt gs://your-bucket-name/

# Download a file from a bucket
gsutil cp gs://your-bucket-name/remote_file.txt ./
```

## Using Python Libraries with Google Cloud

### Example: BigQuery with `google-cloud-bigquery` and `pandas-gbq`

```python
from google.cloud import bigquery
import pandas as pd
import pandas_gbq
import os

# Ensure your PROJECT_ID is set as an environment variable
project_id = os.environ.get('PROJECT_ID')

if not project_id:
    project_id = "your-default-project-id" # Fallback if not set

# Using google-cloud-bigquery client
client = bigquery.Client(project=project_id)
query_job = client.query("""
    SELECT state, SUM(number) as total_births
    FROM `bigquery-public-data.usa_names.usa_1910_current`
    WHERE year > 2000
    GROUP BY state
    ORDER BY total_births DESC
    LIMIT 5
""")
results_df = query_job.to_dataframe()
print("Top 5 states by births after 2000 (using google-cloud-bigquery):")
print(results_df)

# Using pandas-gbq for simplicity
sql_query = """
    SELECT gender, SUM(number) as total_births
    FROM `bigquery-public-data.usa_names.usa_1910_current`
    WHERE year > 2000
    GROUP BY gender
    ORDER BY total_births DESC
"""
gbq_df = pandas_gbq.read_gbq(sql_query, project_id=project_id)
print("\nTotal births by gender after 2000 (using pandas-gbq):")
print(gbq_df)

# Writing a DataFrame to BigQuery (example)
# data_to_upload = {'col1': [1, 2], 'col2': ['a', 'b']}
# df_to_upload = pd.DataFrame(data_to_upload)
# pandas_gbq.to_gbq(df_to_upload, 'your_dataset.your_new_table', project_id=project_id, if_exists='replace')
# print("\nDataFrame uploaded to your_dataset.your_new_table")
```

### Example: Cloud Storage with `google-cloud-storage`

```python
from google.cloud import storage
import os

project_id = os.environ.get('PROJECT_ID')
if not project_id:
    project_id = "your-default-project-id" # Fallback if not set

client = storage.Client(project=project_id)

# Example: List buckets (requires appropriate permissions)
print("\nBuckets (requires storage.buckets.list permission):")
try:
    for bucket in client.list_buckets():
        print(bucket.name)
except Exception as e:
    print(f"Could not list buckets: {e}")

# Example: Uploading a file (replace with your bucket and file names)
# try:
#     bucket_name = "your-actual-bucket-name"
#     bucket = client.bucket(bucket_name)
#     blob = bucket.blob("my_test_upload.txt")
#     blob.upload_from_string("This is a test file from google-cloud-storage Python client.")
#     print(f"File uploaded to {blob.public_url}")
# except Exception as e:
#     print(f"Could not upload file: {e}")
```

### Jupyter Notebooks and BigQuery Magics

In a Jupyter Notebook cell, after installing `google-cloud-bigquery` and authenticating:

1. Load the BigQuery extension:
   ```python
   %load_ext google.cloud.bigquery
   ```

2. Run queries directly (uses the `gcloud` configured default project unless specified):
   ```python
   %%bigquery
   SELECT
     word,
     SUM(word_count) AS count
   FROM `bigquery-public-data.samples.shakespeare`
   WHERE word LIKE '%love%'
   GROUP BY word
   ORDER BY count DESC
   LIMIT 5
   ```

3. Save query results to a Pandas DataFrame:
   ```python
   %%bigquery my_dataframe --project YOUR_PROJECT_ID
   SELECT name, year, number
   FROM `bigquery-public-data.usa_names.usa_1910_current`
   WHERE name = 'Alice'
   ORDER BY year
   LIMIT 10
   ```
   Then you can use `my_dataframe` in subsequent cells.

## Why This Simple YAML Format?

The `colab_env_simple.yml` uses a straightforward list of package names without specific versions (except for Python itself). This approach is beneficial because:

1.  **Cross-platform Compatibility**: Works well across macOS, Linux, and Windows.
2.  **Latest Versions**: Mamba will resolve and install the latest compatible versions of the listed packages.
3.  **Ease of Maintenance**: Simple to add or remove packages.
4.  **Dependency Resolution**: Relies on Mamba's robust dependency resolver to handle compatibility.
5.  **Readability**: Clean and easy to understand the core packages included.

## Quick Commands Reference

```bash
# Create environment from file
mamba env create -f colab_env_simple.yml

# Activate environment
mamba activate colab_env

# Deactivate environment
mamba deactivate

# Update environment from (modified) file
mamba env update -f colab_env_simple.yml --prune

# Export current environment (e.g., after installing new packages)
mamba env export > colab_env_updated.yml

# List all environments
mamba env list

# Remove an environment
mamba env remove -n colab_env
```

## Additional Resources

- [Colab Environment Setup Instructions (`colab_env_setup_instructions.md`)](./colab_env_setup_instructions.md)
- [Google Cloud Setup Guide (`google_cloud_colab_env_setup.md`)](./google_cloud_colab_env_setup.md)
- [Google Cloud Documentation](https://cloud.google.com/docs)
- [Mamba Documentation](https://mamba.readthedocs.io/en/latest/)
```
