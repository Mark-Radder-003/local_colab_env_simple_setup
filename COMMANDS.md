# Essential Commands

## Environment Management

### Basic Operations
```bash
# Create environment
mamba env create -f colab_env_simple.yml

# Activate environment
mamba activate colab_env

# Deactivate environment
mamba deactivate

# Verify installation
python --version
python -c "import pandas, numpy, matplotlib; print('✓ Data packages ready')"
python -c "import google.cloud.bigquery; print('✓ Google Cloud ready')"
```

### Environment Maintenance
```bash
# Update environment from file
mamba env update -f colab_env_simple.yml --prune

# List all environments
mamba env list

# Remove environment
mamba env remove -n colab_env

# Export current environment
mamba env export > environment_backup.yml
mamba env export --from-history > colab_env_updated.yml
```

## Package Management

```bash
# Install additional package
mamba install package_name

# Update specific package
mamba update package_name

# Search for packages
mamba search package_name
```

## Google Cloud Basic Setup

```bash
# Login to Google Cloud
gcloud auth login

# Set project ID
export PROJECT_ID="your-project-id"
gcloud config set project $PROJECT_ID

# Verify setup
gcloud config list
```

## Quick Jupyter Test

```python
# Load BigQuery extension
%load_ext google.cloud.bigquery

# Test query
%%bigquery
SELECT 'Hello World' as message
```

## Common BigQuery Commands

```bash
# List datasets
bq ls

# Run query
bq query --use_legacy_sql=false 'SELECT * FROM `project.dataset.table` LIMIT 10'

# Create dataset
bq mk --dataset --location=US my_dataset
```

## Cloud Storage Commands

```bash
# List buckets
gsutil ls

# Upload file
gsutil cp local_file.csv gs://bucket-name/

# Download file
gsutil cp gs://bucket-name/file.csv ./
```