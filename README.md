# Setting Up brewdata Package with dbt Core and Snowflake

This guide provides a step-by-step walkthrough for setting up and using the brewdata package with dbt Core and Snowflake.

# Documentation
Please visit [docs](https://brewdata.github.io/brewdata-dbt-Snowflake/)
<br>
## Prerequisites

Before you begin, ensure you have the following:

- **Snowflake account** with appropriate permissions
- **Python (>=3.10)** installed on your system
- **Basic familiarity with dbt and Snowflake**

## Step 1: Install dbt Core

To install dbt Core and the Snowflake adapter, run:

```bash
pip install dbt-core dbt-snowflake
```

Verify the installation with:

```bash
dbt --version
```

## Step 2: Initialize a dbt Project

### 1. Create a directory for your dbt project:

```bash
mkdir my_dbt_project
cd my_dbt_project
```

### 2. Initialize a new dbt project:

```bash
dbt init
```

### 3. Configure Snowflake Connection:
During initialization, you'll be prompted to enter your Snowflake connection details:
- **Profile name** (default is the project name)
- **Database name**
- **Schema name**
- **Warehouse name**
- **Username**
- **Password** (or other authentication method)
- **Account identifier**
- **Role** (optional)

## Step 3(Script): Download and Upload the brewdata Package

### 1. download the script file 
- download the the `brewdata_setup.py` and place it in your `<DBT_PROJECT>` directory where `profiles.yml` file is present.

### 2. Run Script
- run the below command to run the script
  ```bash
  python brewdata_setup.py
  ```
- select the profile and it will place the `brewwdata_lib.zip`inside the `BREWDATA_PACKAGE` **stage** of your `Database`.
- _(optional)_ you can provide the stage name while running the script to upload zip file.
  ```bash
  python brewdata_setup.py --stage_name <STAGE_NAME_TO_UPLOAD_PACKAGE>
  ```

## Step 3(Manual): Download and Upload the brewdata Package

### 1. Download the brewdata package

Get the brewdata package ZIP file from the official GitHub repository [here](https://github.com/brewdata/brewdata-dbt-Snowflake/blob/main/brewdata_lib.zip).

### 2. Upload the package to Snowflake

Log in to your Snowflake account via the web UI and follow these steps:

### 3. Create a Snowflake Stage

1. Navigate to **Data** in the left-hand menu.
2. Select your database and schema.
3. Click **Create** > **Stage**.
4. Name the stage, e.g., `<YOUR_SNOWFLAKE_STAGE>`.
5. Choose **Internal Stage** and complete the setup.

### 4. Upload the ZIP File to the Stage

1. Open the newly created stage.
2. Click **Upload** and select the brewdata ZIP file.
3. Wait for the upload to complete.

## Step 4: Run a dbt Model with the brewdata Package

Your dbt-python model (inside the `models` directory) can import the brewdata package and use it within the model.

### Example: Using brewdata in a dbt-Python Model

Create a new Python model (`models/brewdata_dbt_model.py`) and include the following code:

```python
def model(dbt, session):
    # Configure the model with required packages and brewdata import
    dbt.config(
         materialized="table",
         packages=["shapely","transformers","sympy", "faker", "requests", "xmltodict", "xmlschema", "pandas", "numpy", "scikit-learn", "scipy", "tqdm", "pytorch", "datasets"],
         # tested on python 3.10
         # packages=["shapely==2.0.5", "transformers==4.45.2", "sympy==1.13.3", "faker==30.8.1", "requests==2.32.3", "xmltodict==0.13.0", "xmlschema==2.3.1", "pandas==2.0.3", "numpy==1.24.3", "scikit-learn==1.3.0", "scipy==1.10.1", "tqdm==4.66.5", "pytorch==2.3.0", "datasets==2.19.1"],

         imports=['@BREWDATA_PUBLIC.PUBLIC.PUBLIC_STAGE/brewdata_lib.zip'] # change to your @{DB_NAME}.{SCHEMA_NAME}.{STAGE_NAME}/brewdata_lib.zip
    )

    # Import custom brewdata module AFTER the config call
    from brewdata_dbt import FileSyntheticData
    
    # Fetch customer data from an upstream model
    customer_data = dbt.source("public", "customer") # (schema name, table name)
    df = customer_data.to_pandas()

    fsd = FileSyntheticData(df=df, locale='en-US')
    
    # Configuration for synthetic data generation
    config={"columns_config":[{"source_column": "gender",
            "pattern":None, "pattern_id":None, "strategy_id":59, "dependent_fields":[], "tokenization_type": "NA"}
                          ]}
    
    # Generate synthetic data
    fsd.start_synthetic(df, config, "en-US", limit=None, table_data=None, bias=None)
    ddf = fsd.synthetic_file_content
    return ddf
```

### Step 5: Run the Model

Execute the following command to run the model:

```bash
dbt run --select brewdata_dbt_model
```

This will run your dbt model while utilizing the brewdata package.

## Additional Resources

To learn more about configuration options and available strategies, [click here](https://github.com/brewdata/brewdata-dbt-Snowflake/blob/main/brewdata-DBT/readme.md).

---

This guide ensures a smooth installation and setup process for dbt Core and the brewdata package in Snowflake.

