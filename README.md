# Azure Terraform Backend Bootstrap

This blueprint automates the "Day 0" infrastructure required to run Terraform at scale in Azure.

## 🏛️ Architecture: Why the Hybrid Mix?
Terraform has a "Chicken and Egg" problem: it needs a Storage Account to store its state, but shouldn't manually create it.

**The Solution:**
- **ARM (Stateless):** We use these native tools to build the "House" (Resource Group, Storage, RBAC).
- **Terraform (Stateful):** Terraform then moves in and manages the application infrastructure, storing its state in the storage created here.

## 🛡️ Governance & Security Best Practices
- **Resilience:** Defaulted to **Standard_ZRS** for zone redundancy.
- **Zero-Trust:** Shared Access Keys are **Disabled**. All access must use Entra ID (Azure AD) tokens.
- **Auditability:** Every read/write/delete operation on the state file is logged to Log Analytics.
- **Immutability:** A `CanNotDelete` Management Lock prevents accidental destruction of the backend.
- **Recovery:** Blob Versioning and 14-day Soft Delete are enabled.

## 🔑 Why GUIDs for Role Definitions?
We use the GUID `ba92f5b4-2d11-453d-a403-e96b0029c9fe` (Storage Blob Data Contributor) because:
1. **Immutability:** Display names (like "Contributor") can be renamed by Microsoft; GUIDs are permanent.
2. **Localization:** GUIDs are the only universal language the Azure API understands across all global regions.
3. **Precision:** Prevents name collisions with custom roles.

## ⚠️ Critical Implementation Notes
1. **Global Uniqueness:** Storage names must be unique across all of Azure. Use a specific prefix.
2. **Deployment Permissions:** The identity running this requires **Owner** or **User Access Administrator** at the Subscription scope to assign RBAC roles.
3. **ZRS Availability:** Ensure the `location` supports Zone-Redundant Storage.
4. **RBAC Delay:** Wait ~5 minutes after deployment before running `terraform init` to allow permissions to sync.

## 🛠️ Usage in Terraform
```hcl
terraform {
  backend "azurerm" {
    resource_group_name  = "rg-terraform-mgmt"
    storage_account_name = "<YOUR_UNIQUE_STORAGE_NAME>"
    container_name       = "tfstate"
    key                  = "prod.terraform.tfstate"
    use_azuread_auth     = true
  }
}
