### Azure Key Vault sync fails across subscriptions. How do you secure and automate secrets rotation at scale?

#### The Scalable Architecture: An Event-Driven Hub-and-Spoke Model

We'll design a centralized system that reacts to secret lifecycle events and securely distributes them. This is a classic hub-and-spoke model.

- Hub: A central subscription containing our automation logic (Azure Functions) and event routing (Azure Event Grid).
- Spokes: All other subscriptions containing applications and their respective Key Vaults.

This is how it works:

- Event Trigger: When a secret in any Key Vault is about to expire, it emits a SecretNearExpiry event.
- Event Grid: An Azure Event Grid System Topic is configured on each Key Vault. It captures these events and forwards them to a central endpoint in our hub subscription.
- Azure Function: A single, highly-secure Azure Function App in the hub subscription acts as our central processor. It subscribes to the Event Grid topic
- Rotation Logic: The Function executes the rotation logic:
  - Generates a new secret/password.
  - Updates the source of truth (e.g., the database, the service principal).
  - Creates a new version of the secret in the original Key Vault.
- Secure Access: The entire process is authenticated using Managed Identities, eliminating the need for any stored credentials.

#### Securing Cross-Subscription Access with Managed Identities
This is the most critical piece. How does our central Azure Function in Subscription A get permission to update a secret in Subscription B?

- Use a User-Assigned Managed Identity: While a system-assigned identity works for the Function App itself, a user-assigned managed identity is more flexible for cross-subscription scenarios. Create one in your central hub subscription
- Grant IAM Permissions via IaC: The magic is in granting this identity the necessary permissions on the Key Vaults in your spoke subscriptions. This should be codified using Bicep or Terraform.

#### Enforce with Azure Policy:

- Deploy Azure Policies to enforce this pattern. For example:
  - An AuditIfNotExists policy to ensure all Key Vaults have an Event Grid topic configured for secret expiry.
  - A Deny policy that prevents the creation of secrets without an expiration date.
