# Azure AD: Tenant ID vs Object ID

## 1. Tenant ID
- **What it is:** The globally unique identifier (GUID) for your **Azure AD directory**.
- **Scope:** Applies to the **entire Azure AD tenant** (organization).
- **Use cases:**
  - Federated authentication (OIDC, SAML)
  - Azure login for Terraform or scripts
  - Linking subscriptions to a directory

> Example: `xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx`

---

## 2. Object ID
- **What it is:** The globally unique identifier (GUID) for an **individual object in Azure AD**, such as:
  - User
  - Service Principal
  - Application
- **Scope:** Specific to a single object within the tenant.
- **Use cases:**
  - Assigning roles to a service principal
  - RBAC and access policies
  - Identifying Terraform’s Azure AD App Registration or SP

> Example: `yyyyyyyy-yyyy-yyyy-yyyy-yyyyyyyyyyyy`

---

## 3. Relationship Between Tenant ID and Object ID

| Concept | Tenant ID | Object ID |
|---------|-----------|-----------|
| Scope | Entire Azure AD directory | Individual object (user, SP, app) |
| Uniqueness | Globally unique for tenant | Globally unique within Azure AD |
| Example | `xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx` | `yyyyyyyy-yyyy-yyyy-yyyy-yyyyyyyyyyyy` |
| Use with Terraform | Used in `azure_tenant_id` for authentication | Used in `client_id` (App Registration) or `principal_id` (SP) |
| How they connect | An Object ID **lives inside a Tenant**; every object belongs to exactly one Tenant | Each object references the Tenant it belongs to |

---

### 🔹 Example

1. **Tenant ID:** `11111111-1111-1111-1111-111111111111`  
   - Represents your Azure AD directory “MyCompanyTenant”.

2. **App Registration Object ID:** `22222222-2222-2222-2222-222222222222`  
   - Represents a Service Principal for Terraform in that tenant.

**So:**  
Tenant ID identifies the “directory”, Object ID identifies **who/what** inside that directory.

---

### ✅ TL;DR
- **Tenant ID** → “Which Azure AD directory are we in?”  
- **Object ID** → “Which user/app/service principal inside that directory?”  
- Every object in Azure AD references exactly one Tenant ID.
