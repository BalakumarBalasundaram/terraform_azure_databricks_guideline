federated OIDC authentication via GitHub → Azure → Databricks.

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
