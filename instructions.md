**Azure Game Highlight Processor - Terraform**

**HighlightProcessor**
This project uses RapidAPI to obtain NCAA game highlights, stores the json file in an Azure Blob and then parses the json file for a video url and downloads the video to the same Azure Storage Blob.

**File Overview**

**Python Files**

.env file stores all over the environment variables, these are variables that we don't want to hardcode into our script.

The config.py script performs the following actions: Loads environment variables from .env using python-dotenv.

Retrieves and centralizes configuration values like API endpoints, RapidAPI Key and Azure Storage account details.

Dynamically builds the Azure Storage Connection string from individual parameters if one isn't provided.

The fetch.py script performs the following actions:

Fetches basketball highlights from the RapidAPI Sports API.

Usess the requests library to call the API with the specified query parameters and headers.

Parses the returned JSON data and uploads it as a file to Azure Blob Storage.

process_one_video.py performs the following actions:

Downloads the previously uploaded JSON file from Azure Blob Storage.

Extracts the first video URL from the JSON data.

Downloads the video URL from the JSON data.

Uploads the downloaded video file to the same Azure Blob Storage container.

run_all.py performs the following actions: Orchestrates the entire workflow, ensuring each step runs in sequence with retry logic.

update_env.py performs the following actions: Automats updating the .env file after Terraform provisioning is complete

Runs the command terraform output -json to capture the newly provisioned resource details, storage account name, key and container name.

Updates the corresponding entries in the .env file so that the Python scripts use the latest resource values.

**Terraform Files**

main.tf performs the following actions:

The primary Terraform configuration file.

Specifies the Azure provider and uses it to provision essential Azure resources.

Uses resource blocks to define theses resources

variables.tf performs the following actions:

Declares variables used by Terraform such as subscription_id, resource_group_name, location, storage_account_name, container_name and environment.

Provides descriptions and default values for better clarity and reusability.

outputs.tf performs the following actions:

Defines the outputs of the Terraform configuration

Outputs include the resource group name, storage account name, primary access key, and blob container name.

Outputs are consumed by other scripts or used for documentation.

terraform.tfvars performs the following actions:

Provides specific values for the variables defined in variables.tf

The file is used to supply the concrete configuration when running Terraform.

**Prerequisites**

Before running the scripts, ensure you have the following:

**1 Create Rapidapi Account**

Rapidapi.com account, will be needed to access highlight images and videos.

For this example we will be using NCAA (USA College Basketball) highlights since it's included for free in the basic plan.

Sports Highlights API is the endpoint we will be using

**2 Verify prerequites are installed**

Python3 should be pre-installed also python3 --version

Git - Cloning the repo

Set up Azure Account

    Azure $200 Credit
    
    Azure For Students
    
    Azure For Startups

**3 Retrieve Azure Account ID**

After logging in, you can run:

```az account show --query id -o tsv```

in the CLI to return the subscription id.

**Project Structure**
```
├── .env                      # Environment variables for app configuration
├── .gitignore                # Files/directories to ignore in Git
├── requirements.txt          # Python dependencies (e.g., requests, azure-storage-blob, etc.)
├── config.py             # Loads environment variables and builds the Azure connection string
├── fetch.py              # Fetches highlights from RapidAPI and uploads JSON to Azure Blob Storage
├── process_one_video.py  # Downloads JSON, extracts video URL, downloads video, and uploads it to Blob Storage
├── run_all.py            # Orchestrates running fetch.py and process_one_video.py sequentially with retry logic
├── update_env.py         # Updates the .env file with Terraform outputs automatically
└── terraform/                # Terraform configuration for provisioning Azure resources
    ├── main.tf               # Defines Azure resource group, storage account, and blob container
    ├── variables.tf          # Declares variables for Terraform configuration
    ├── outputs.tf            # Outputs the storage account name, primary key, and container name
    └── terraform.tfvars      # For variable values
```

![image](https://github.com/user-attachments/assets/042b9204-3f7e-4ef7-9bc0-a5e4c45a1f23)


**AWS to Azure Translation**

For those of you who previously completed the lab in AWS, here is a quick break down of how the services map to Azure: Azure Blob Storage replaces Amazon S3 for storing both highlight metadata(JSON) and processed videos. Authentication now uses DefaultAzureCredential(), which works with Microsoft Entra ID instead of IAM roles. AWS MediaConvert functionality that enhances the video/audio quality was removed since Azure Media Services was deprecated in 2023.

![image](https://github.com/user-attachments/assets/8803739a-77a9-4a9a-83ff-e725025cc668)

**START HERE - Local**

**Step 1: Clone The Repo**
```
git clone https://github.com/charity-web/AzureHighlightProcessorTerraform.git
cd src
```

**Step 2: Update .env file**

1. RAPIDAPI_KEY
2. AZURE_SUBSCRIPTION_ID
3. AZURE_RESOURCE_GROUP

**Step 3: Secure .env file**
```
chmod 600 .env
```

**Step 4: Update terraform.tfvars**

1. subscription_id
2. resource_group_name
3. storage_account_name
4. container_name
   
**Step 5: Setup Python Virtual Environment**

macOS/Linux
```
python3 -m venv venv
source venv/bin/activate
```

Windows
```
python -m venv venv
venv\Scripts\activate
```

**Step 6: Install Project Dependencies**
```
pip install --upgrade pip
pip install -r requirements.txt
```

**Step 7: Provision Azure Infrastructure with Terraform**
```
terraform init
```
```
terraform plan
```
```
terraform apply
```
Type "yes" when prompted

![image](https://github.com/user-attachments/assets/0c50550f-85e1-43e4-8f7c-3bc46682d8d3)


**Step 8: Update .env file automatically**
```
python update_env.py
```
This script will update the following variables in your .env:

1. AZURE_STORAGE_ACCOUNT_NAME
2. AZURE_STORAGE_ACCOUNT_KEY
3. AZURE_BLOB_CONTAINER_NAME
   
**Step 9: Execute The Data Processing Pipeline**

Run:
```
python run_all.py
```
This script will:

Run fetch.py: Fetch highlights from the API and upload the JSON file to Azure Blob Storage. Wait for resources to stabilize. Run process_one_video.py: Download the JSON file, extract the video URL, download the video, and upload it to Azure Blob Storage.

![image](https://github.com/user-attachments/assets/c9a91a19-6707-4032-8eea-ee77c7c11c7a)

Optional - Confirm there is a JSON file and video is uploaded to the container
```
az storage blob list \
    --account-name <YOUR_STORAGE_ACCOUNT_NAME> \
    --container-name <YOUR_CONTAINER_NAME> \
    --query "[].{name:name}" \
    --output table
```
![image](https://github.com/user-attachments/assets/195e1cb5-d451-4f3f-8dbe-1623735f634e)


![image](https://github.com/user-attachments/assets/ef6b85af-5f6a-4ce1-b6ee-2fb8fd0cc8d1)



**What We Learned**
1. Terraform workflow: - Initialization (terraform init): Downloading providers and setting up the working directory - Planning (terraform plan): Previewing changes before applying them - Applying (terraform apply): Creating or updating resources in Azure
2. Integrating Terraform outputs with other automation scripts (like update_env.py) to automatically update configuration files
3. Idempotency - ensuring that running the same configuration repeatedly leads to the same resource state
4. Parameterizing configurations to make deployments flexible and reusable across different environments
   
**Future Enhancements**
1. Automate updating the Azure Subscription ID from the terraform variabel file
2. Deploy the containers to an orchestration platform (Azure Container Instances) for improved scalability and management
3. Automate the Terraform provisioning and .env update as part of the deployment pipeline
