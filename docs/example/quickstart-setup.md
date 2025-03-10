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
- **Profile name** 
  (default is the project name)

- **Database name**

- **Schema name**

- **Warehouse name**

- **Username**

- **Password** 
  (or other authentication method)

- **Account identifier**

- **Role** 
  (optional)

## Step 3(Script): Download and Upload the brewdata Package

### 1. Get the setup script
You have two options:

#### Option A: Download the script file
- Download the `brewdata_setup.py` and place it in your `<DBT_PROJECT>` directory where `profiles.yml` file is present

#### Option B: Create the script manually
1. Create a new file named `brewdata_setup.py` in your `<DBT_PROJECT>` directory
2. Copy and paste the following code into the file:
```python
from ast import Import
import os
import yaml
import sys
import subprocess
import argparse
from pathlib import Path

def install_required_packages():
    """Install required packages if not already installed."""
    required_packages = ["pyyaml", "requests", "snowflake-connector-python"]
    for package in required_packages:
        try:
            Import(package.replace("-", ""))
        except ImportError:
            print(f"Installing {package}...")
            subprocess.check_call([sys.executable, "-m", "pip", "install", package])

def get_dbt_profile():
    """Get the dbt profile based on the current directory name."""
    # Get current directory name
    current_dir = str(Path(os.getcwd()).resolve())
    
    # Clean up the directory name to match profile name format
    profile_name = current_dir.replace('-', '_')
    
    # Find profiles.yml location
    if os.name == 'nt':  # Windows
        profiles_path = os.path.join(os.environ['USERPROFILE'], '.dbt', 'profiles.yml')
    else:  # Linux/macOS
        profiles_path = os.path.join(Path(current_dir), 'profiles.yml')
    
    # Check if profiles file exists
    if not os.path.exists(profiles_path):
        raise FileNotFoundError(f"dbt profiles file not found at {profiles_path}")
    
    # Load profiles
    with open(profiles_path, 'r') as f:
        profiles = yaml.safe_load(f)
    
    # Check if the profile exists
    if profile_name not in profiles:
        available_profiles = list(profiles.keys())
        print(f"Profile '{profile_name}' not found in profiles.yml.")
        print(f"Available profiles: {available_profiles}")
        
        # Try to match with available profiles
        for p in available_profiles:
            if p.lower() == profile_name.lower():
                profile_name = p
                print(f"Using profile '{profile_name}' instead.")
                break
        else:
            # Ask user to select a profile
            print("Please select a profile:")
            for i, p in enumerate(available_profiles):
                print(f"{i}: {p}")
            selection = int(input("Enter profile number: "))
            profile_name = available_profiles[selection]
    
    # Get the target profile (default is 'dev')
    target_name = profiles[profile_name].get('target', 'dev')

    # get the target profile
    print(f"Current Selected Target : {target_name} \n Would you like to change it? (y/n)")
    change_target = input()
    while change_target.lower() not in ['y', 'n']:
        print("Invalid input. Please enter 'y' or 'n'.")
        print(f"Current Selected Target : {target_name} \n Would you like to change it? (y/n)")
        change_target = input()
    if change_target.lower() == 'y':
        print("Please select a target:")
        for i, p in enumerate(profiles[profile_name]['outputs'].keys()):
            print(f"{i}: {p}")
        selection = int(input("Enter target number: "))
        target_name = list(profiles[profile_name]['outputs'].keys())[selection]

    target_profile = profiles[profile_name]['outputs'][target_name]
    
    return profile_name, target_profile

def download_brewdata_package(download_path, github_url=None):
    """Download the brewdata package or use a local file."""
    import requests
    
    local_path = os.path.join(download_path, "brewdata_lib.zip")
    
    # Check if a local file already exists
    if os.path.exists(local_path):
        use_local = input(f"Found brewdata_lib.zip in {download_path}. Use this file? (y/n): ")
        if use_local.lower() == 'y':
            print(f"Using existing file at {local_path}")
            return local_path
    
    # Download the file
    print(f"Downloading brewdata package from {github_url}...")
    response = requests.get(github_url, stream=True)
    if response.status_code != 200:
        raise Exception(f"Failed to download package. Status code: {response.status_code}")
    
    # Save the file with progress indicator for large files
    total_size = int(response.headers.get('content-length', 0))
    block_size = 1024  # 1 Kibibyte
    
    with open(local_path, 'wb') as f:
        if total_size > 1024*1024:  # Only show progress for files > 1MB
            print(f"Downloading {total_size//(1024*1024)}MB file...")
            downloaded = 0
            for data in response.iter_content(block_size):
                downloaded += len(data)
                f.write(data)
                done = int(50 * downloaded / total_size)
                print(f"\r[{'=' * done}{' ' * (50-done)}] {downloaded//(1024*1024)}MB/{total_size//(1024*1024)}MB", end='')
            print()
        else:
            f.write(response.content)
    
    print(f"Package downloaded to {local_path}")
    return local_path

def create_snowflake_stage_and_upload(profile, zip_file_path, stage_name=None):
    """Create a Snowflake stage and upload the ZIP file."""
    import snowflake.connector
    
    # Connect to Snowflake
    print(f"Connecting to Snowflake as {profile['user']}@{profile['account']}...")
    conn = snowflake.connector.connect(
        user=profile['user'],
        password=profile['password'],
        account=profile['account'],
        warehouse=profile['warehouse'],
        database=profile['database'],
        schema=profile['schema'],
        role=profile.get('role')
    )
    
    # If stage_name is not provided, create a default one
    if not stage_name:
        stage_name = f"{profile['database']}.{profile['schema']}.BREWDATA_STAGE"
    else :
        stage_name = f"{profile['database']}.{profile['schema']}.{stage_name.upper()}"
    
    try:
        # Create cursor
        cursor = conn.cursor()
        
        # Create stage if not exists
        print(f"Creating stage {stage_name}...")
        cursor.execute(f"CREATE STAGE IF NOT EXISTS {stage_name}")
        
        # Upload ZIP file to stage
        print(f"Uploading ZIP file to stage {stage_name}...")
        # Normalize path for Snowflake PUT command
        zip_file_path_norm = os.path.abspath(zip_file_path).replace('\\', '/')
        # Disable auto-compression to ensure file remains as .zip only
        upload_cmd = f"PUT file://{zip_file_path_norm} @{stage_name} AUTO_COMPRESS=FALSE OVERWRITE=TRUE"
        cursor.execute(upload_cmd)
        print("Upload completed successfully!")
        
        return stage_name
    
    finally:
        conn.close()

def main():
    parser = argparse.ArgumentParser(description='Set up brewdata package with dbt Core and Snowflake')
    parser.add_argument('--url', default="https://github.com/brewdata/brewdata-dbt-Snowflake/releases/download/main/brewdata_lib.zip", help='GitHub URL for the brewdata package')
    parser.add_argument('--download_path', default='.', help='Path to download the brewdata package')
    parser.add_argument('--stage_name', help='Custom stage name (optional)')
    parser.add_argument('--keep_zip', action='store_true', help='Keep the ZIP file after upload')
    args = parser.parse_args()
    
    try:
        # Install required packages
        # install_required_packages()
        
        # Get Snowflake profile from dbt
        profile_name, profile = get_dbt_profile()
        print(f"Using profile: {profile_name}")
        print(f"Snowflake connection: {profile['user']}@{profile['account']}")
        
        # Download brewdata package
        zip_file_path = download_brewdata_package(args.download_path, args.url)
        
        # Create stage and upload
        stage_name = create_snowflake_stage_and_upload(profile, zip_file_path, args.stage_name)
        
        # Delete the ZIP file after upload unless --keep_zip is specified
        if not args.keep_zip and os.path.exists(zip_file_path):
            print(f"Removing temporary ZIP file: {zip_file_path}")
            os.remove(zip_file_path)
        
        # Print instructions for the user
        print("\nSetup Complete!")
        print("===================================")
        print("To use brewdata in your dbt-Python model, include the following in your model file:")
        print(f"""
def model(dbt, session):
    dbt.config(
         materialized="table",
         packages=["shapely", "transformers", "sympy", "faker", "requests", "xmltodict", "xmlschema", 
                   "pandas", "numpy", "scikit-learn", "scipy", "tqdm", "pytorch", "datasets"],
         imports=['@{stage_name}/brewdata_lib.zip'] # change to your @{{DB_NAME}}.{{SCHEMA_NAME}}.{{STAGE_NAME}}/brewdata_lib.zip
    )

    # Import custom brewdata module AFTER the config call
    from brewdata_dbt import FileSyntheticData
    
    # Your code here
    # ...
""")
    
    except Exception as e:
        print(f"Error: {e}")
        import traceback
        traceback.print_exc()
        return 1
    
    return 0

if __name__ == "__main__":
    exit_code = main()
    exit(exit_code)

```

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

Get the brewdata package ZIP file from the official GitHub repository [here](https://github.com/brewdata/brewdata-dbt-Snowflake/releases/download/main/brewdata_lib.zip).

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
    config={"columns_config":[
                              {"source_column": "gender",
                              "pattern":None, 
                              "pattern_id":None, 
                              "strategy_id":59, 
                              "dependent_fields":[],
                              "tokenization_type": "NA"}
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
