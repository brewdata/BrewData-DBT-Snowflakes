# Setting Up BrewData Package with dbt Core and Snowflake

This guide walks you through the process of setting up and using the BrewData package with dbt Core and Snowflake.

## Prerequisites

- Snowflake account with appropriate permissions
- Python installed on your system
- Basic familiarity with dbt and Snowflake

## Step 1: Install dbt Core

Install dbt Core using pip:

```bash
pip install dbt-core dbt-snowflake
```

## Step 2: Initialize a dbt Project

1. Create an empty folder for your dbt project:

```bash
mkdir my_dbt_project
cd my_dbt_project
```

2. Initialize a new dbt project:

```bash
dbt init
```

3. During initialization, you'll be prompted to configure your Snowflake connection. Provide the following information:
   - Profile name (default is the project name)
   - Database name
   - Schema name
   - Warehouse name
   - Username
   - Password (or other authentication method)
   - Account identifier
   - Role (optional)

## Step 3: Configure BrewData Package

1. Create a Python file in the models directory named `brewdata_dbt.py`:

```bash
mkdir -p models
touch models/brewdata_dbt.py
```

2. Add the following content to `brewdata_dbt.py`:

```python
import os
# Your Python code will go here
```

3. Create a configuration YAML file named `config.yaml` in the models directory:

```bash
touch models/config.yaml
```

4. Add the following content to `config.yaml`:

```yaml
version: 2

models:
  - name: sftest1
    config:
      packages:
        - "shapely"
        - "transformers"
        - "sympy"
        - "faker"
        - "requests"
        - "xmltodict"
        - "xmlschema"
        - "pandas"
        - "numpy"
        - "scikit-learn"
        - "scipy"
        - "tqdm"
        - "pytorch"
        - "datasets"
      imports:
        - "@BREWDATA_PUBLIC.PUBLIC.PUBLIC_STAGE/brewdata_lib.zip"
```

## Step 4: Download and Upload the BrewData Package

1. Download the BrewData package ZIP file from the provided URL:
   ```bash
   curl -O https://abc.l.zip
   ```

2. Log in to your Snowflake account through the web UI.

3. Navigate to the database you configured during dbt initialization.

4. Create a new stage named `PYTHON_PACKAGES`:
   - Click on "Data" in the left navigation
   - Select your database and schema
   - Click "Create" and select "Stage"
   - Name it `PYTHON_PACKAGES`
   - Configure it as an internal stage
   - Click "Create"

5. Upload the downloaded ZIP file to the stage:
   - Select the newly created stage
   - Click "Upload" and select the downloaded ZIP file
   - Wait for the upload to complete

## Step 5: Run Your dbt Model

Execute your dbt model with the following command:

```bash
dbt run --select brewdata_dbt
```

This will run the model with the BrewData package imported and available for use.

## Troubleshooting

- **Connection Issues**: Verify your Snowflake credentials in the `profiles.yml` file.
- **Package Import Errors**: Ensure the path to the ZIP file in the `imports` section of your config file is correct.
- **Permission Errors**: Make sure your Snowflake role has the necessary permissions to access the stage and execute Python code.

## Advanced Configuration

For more advanced usage, you can:

1. Create additional Python models that import functions from the BrewData package
2. Configure different package dependencies for different models
3. Set up CI/CD pipelines to automate the deployment process

## Additional Resources

- [dbt Documentation](https://docs.getdbt.com/)
- [Snowflake Python UDFs Documentation](https://docs.snowflake.com/en/sql-reference/udf-python.html)
- [BrewData Package Documentation](https://docs.brewdata.com/) (replace with actual documentation URL)

---

Feel free to contribute to this guide by submitting pull requests or opening issues on GitHub.