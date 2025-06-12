# Colab Environment Setup Instructions

## Environment Specification File

**`colab_env_simple.yml`** - Environment specification with all essential packages for data analysis and Google Cloud

## How to Recreate the Environment

```bash
# On any machine with mamba installed:
mamba env create -f colab_env_simple.yml

# Or with conda:
conda env create -f colab_env_simple.yml

# Activate the environment:
mamba activate colab_env
```

## Sharing Your Environment

To share with others or use on another machine:

1. **Copy the YAML file** to the new machine
2. **Run the create command** shown above
3. **Activate the environment**: `mamba activate colab_env`

## Updating the Specification

If you install new packages and want to update your spec file:

```bash
# Activate your environment first
mamba activate colab_env

# Install new package
mamba install package_name

# Manually add the package name to colab_env_simple.yml
# Or create a new export:
mamba env export --from-history > colab_env_simple_updated.yml
```

## What's Included

The environment includes:
- **Python 3.13**
- **Jupyter ecosystem**: jupyter, notebook, ipykernel
- **Data analysis**: pandas, numpy, matplotlib, seaborn, scipy, statsmodels
- **Machine learning**: scikit-learn
- **Geospatial**: geopandas
- **Google Cloud**: google-cloud-sdk, google-auth, google-cloud-storage, google-cloud-bigquery
- **BigQuery integration**: pandas-gbq, bigquery-magics
- **Data formats**: pyarrow

## Why This Simple YAML Format?

For your use case as a data analyst, the simple YAML format is best because:

1. **Cross-platform**: Works seamlessly on Mac, Linux, and Windows
2. **Always up-to-date**: Gets latest compatible versions of packages
3. **Easy to maintain**: Just add/remove package names as needed
4. **No conflicts**: Lets mamba/conda resolve dependencies automatically
5. **Clean and readable**: No version numbers or build strings to manage

## Quick Commands Reference

```bash
# Create environment from file
mamba env create -f colab_env.yml

# Update existing environment from file
mamba env update -f colab_env.yml

# Export current environment
mamba env export > my_env.yml

# List all environments
mamba env list

# Remove an environment
mamba env remove -n colab_env
```