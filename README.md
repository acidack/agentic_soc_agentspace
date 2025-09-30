Agentic SOC Agentspace - Quick Start Guide
==========================================

### 1\. Prerequisites

**Enable APIs:**



```
gcloud services enable aiplatform.googleapis.com discoveryengine.googleapis.com storage.googleapis.com cloudbuild.googleapis.com chronicle.googleapis.com

```

**Create Agent's Main Service Account:**



```
# Replace YOUR_PROJECT_ID with your actual project ID
gcloud iam service-accounts create soc-agent-sa --display-name="SOC Agent Service Account"

gcloud projects add-iam-policy-binding YOUR_PROJECT_ID\
  --member="serviceAccount:soc-agent-sa@YOUR_PROJECT_ID.iam.gserviceaccount.com"\
  --role="roles/aiplatform.serviceAgent"

gcloud projects add-iam-policy-binding YOUR_PROJECT_ID\
  --member="serviceAccount:soc-agent-sa@YOUR_PROJECT_ID.iam.gserviceaccount.com"\
  --role="roles/discoveryengine.viewer"

```

**Set User Permissions:** In the Google Cloud IAM console, add the **`Vertex AI Search Admin`** role to your own user account.

### 2\. Setup

**Clone Repository:**



```
git clone --recurse-submodules https://github.com/acidack/agentic_soc_agentspace.git
cd agentic_soc_agentspace

```

**Configure Environment:**



```
cp .env.example .env

```

Now, **edit the `.env` file** and set the following required values:

-   `PROJECT_ID`

-   `PROJECT_NUMBER`

-   `LOCATION`

-   `GOOGLE_CLOUD_LOCATION` (use the same value as `LOCATION`)

-   `AGENTSPACE_PROJECT_ID` (use the same value as `PROJECT_ID`)

-   `AGENTSPACE_PROJECT_NUMBER` (use the same value as `PROJECT_NUMBER`)

-   `STAGING_BUCKET` (must start with `gs://`, e.g., `gs://your-bucket-name`)

**Configure Chronicle API Access (Optional):** If you are using the Chronicle tools, follow these steps to create a dedicated service account and key.

1.  **Create Service Account for Chronicle:**

    

    ```
    gcloud iam service-accounts create chronicle-api-sa --display-name="Chronicle API Service Account"

    ```

2.  **Grant Chronicle Admin Role:**

    

    ```
    # Replace YOUR_PROJECT_ID with your actual project ID
    gcloud projects add-iam-policy-binding YOUR_PROJECT_ID\
      --member="serviceAccount:chronicle-api-sa@YOUR_PROJECT_ID.iam.gserviceaccount.com"\
      --role="roles/chronicle.admin"

    ```

3.  **Create and Download JSON Key:** This command saves the key in your current directory.

    

    ```
    # Replace YOUR_PROJECT_ID with your actual project ID
    gcloud iam service-accounts keys create ./chronicle-key.json\
      --iam-account="chronicle-api-sa@YOUR_PROJECT_ID.iam.gserviceaccount.com"

    ```

4.  **Update `.env` file:** Set the path to the key you just created.

    ```
    SECOPS_SA_PATH=./chronicle-key.json

    ```

**Install Dependencies:**



```
python3 -m venv venv
source venv/bin/activate
pip install -r requirements.txt

```

### 3\. Deploy

**Deploy the Agent:**



```
make agent-engine-deploy

```

After this finishes, copy the `REASONING_ENGINE` ID and `AGENT_ENGINE_RESOURCE_NAME` from the output into your `.env` file.

**Register the Agent:**



```
make agentspace-register

```

After this finishes, copy the `AGENTSPACE_AGENT_ID` from the output into your `.env` file.

### 4\. Run

**Start the User Interface:**



```
make run-ui

```

### 5\. How to Update

To apply any changes to your code or `.env` file, you must re-deploy the agent.



```
# 1. Re-deploy the agent
make agent-engine-deploy

# 2. Update the new engine ID in your .env file
# 3. Restart the UI
make run-ui
```
