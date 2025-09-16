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
