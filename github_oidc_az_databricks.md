# 🔑 Simplified OIDC Relationship (Terraform via GitHub → Azure → Databricks)

| Step | Who | What happens | Why it matters |
|------|-----|--------------|----------------|
| 1 | **GitHub Actions** | Workflow requests an **OIDC token** from GitHub | Proves the workflow’s identity |
| 2 | **Azure AD App** | Validates GitHub’s OIDC token and issues an **Azure AD token** | Lets Terraform act as a service principal |
| 3 | **Terraform Databricks Provider** | Uses `auth_type = "github-oidc-azure"` to exchange the Azure AD token | Gets a short-lived **Databricks token** |
| 4 | **Databricks Workspace** | Accepts the Databricks token and allows API calls | Terraform can manage clusters, jobs, repos, etc. |

---

👉 **In short: federated OIDC authentication via GitHub → Azure → Databricks.**

**GitHub workflow → Azure AD (trust) → Databricks (API access).**

- GitHub proves *who it is*  
- Azure AD trusts GitHub and issues an access token  
- Terraform Databricks provider exchanges it for a Databricks token  
- Terraform uses that token to manage Databricks resources

Sure! Here's the explanation in **Markdown format**, covering **App Registration → Federated Credential → Terraform configuration**:

````markdown
# 🔑 GitHub OIDC → Azure → Databricks (Terraform) Flow

This guide shows how **GitHub Actions**, **Azure AD**, and **Databricks** connect using OIDC for Terraform authentication.

---

## 1. App Registration (Azure AD)

Create an Azure AD App Registration and Service Principal.  
This represents Terraform as an identity.

```bash
# Create App Registration
az ad app create --display-name "databricks-terraform-oidc"

# Get the App ID (Client ID)
APP_ID=$(az ad app list --display-name "databricks-terraform-oidc" --query "[0].appId" -o tsv)

# Create Service Principal
az ad sp create --id $APP_ID
````

* **Purpose:** Identity Terraform in Azure AD.

---

## 2. Federated Credential

Add a federated identity credential to trust GitHub Actions OIDC tokens.

```bash
az ad app federated-credential create \
  --id $APP_ID \
  --parameters '{
    "name": "github-main",
    "issuer": "https://token.actions.githubusercontent.com",
    "subject": "repo:myorg/myrepo:ref:refs/heads/main",
    "audiences": ["api://AzureADTokenExchange"]
  }'
```

* **Issuer:** GitHub OIDC endpoint
* **Subject:** Only this repo/branch is trusted
* **Audience:** Azure token exchange endpoint

✅ No secrets or PATs required.

---

## 3. Terraform Configuration

Configure the Databricks provider to use **OIDC via Azure**.

```hcl
provider "databricks" {
  auth_type                   = "github-oidc-azure"
  azure_tenant_id             = var.tenant_id
  azure_workspace_resource_id = module.dbworkspace.databricks_workspace_azure_id
  host                        = module.dbworkspace.databricks_host
}
```

---

## 4. GitHub Actions Workflow

```yaml
permissions:
  id-token: write   # allow OIDC
  contents: read

jobs:
  terraform:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Azure Login
        uses: azure/login@v2
        with:
          client-id: ${{ secrets.AZURE_CLIENT_ID }}     # $APP_ID
          tenant-id: ${{ secrets.AZURE_TENANT_ID }}
          subscription-id: ${{ secrets.AZURE_SUBSCRIPTION_ID }}

      - name: Terraform Apply
        run: terraform apply -auto-approve
```

---

## 5. How it Connects

1. GitHub workflow requests an OIDC token from `https://token.actions.githubusercontent.com`.
2. Azure AD validates the token against the **federated credential**.
3. Azure AD issues an **AAD token** to Terraform.
4. Terraform Databricks provider exchanges it for a **Databricks token**.
5. Terraform uses that token to manage Databricks resources.

---

## 🔄 Summary Table

| Component                | Role       | Purpose                                        |
| ------------------------ | ---------- | ---------------------------------------------- |
| **App Registration**     | Identity   | Represents Terraform in Azure AD               |
| **Federated Credential** | Trust rule | Allows only specific repo/branch to get tokens |
| **Terraform Provider**   | Consumer   | Uses OIDC token to authenticate to Databricks  |
| **GitHub Actions**       | Source     | Requests OIDC token to prove workflow identity |
| **Databricks Workspace** | Target     | Accepts Databricks token for API operations    |

```

---

If you want, I can also make a **visual diagram in Markdown (ASCII/flow style)** showing the GitHub → Azure → Databricks flow—it makes it really easy to understand at a glance. Do you want me to do that?
```



That’s why your databricks provider has fields like:
auth_type = "github-oidc-azure"
azure_tenant_id
azure_workspace_resource_id
host

This flow is needed because Terraform must:
Authenticate to Azure AD (using OIDC trust from GitHub Actions).
Exchange that for a temporary AAD token.
Use that token to get a Databricks workspace access token behind the scenes.

Call the Databricks REST API.
