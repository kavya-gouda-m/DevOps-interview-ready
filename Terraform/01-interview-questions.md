ADVANCED:

𝗧𝗘𝗥𝗥𝗔𝗙𝗢𝗥𝗠 𝗦𝗧𝗔𝗧𝗘 𝗙𝗜𝗟𝗘 𝗖𝗢𝗥𝗥𝗨𝗣𝗧𝗜𝗢𝗡 𝗥𝗘𝗖𝗢𝗩𝗘𝗥𝗬

𝗦𝗰𝗲𝗻𝗮𝗿𝗶𝗼: Your terraform.tfstate file is corrupted or lost, causing infrastructure drift.

✔️ Use terraform state pull to recover the latest state.
✔️ Restore from Terraform Cloud or S3 version history.
✔️ Run terraform refresh cautiously to sync with real infra.
✔️ Store state files in S3 with versioning + DynamoDB locking

𝗣𝗥𝗘𝗩𝗘𝗡𝗧𝗜𝗡𝗚 𝗧𝗘𝗥𝗥𝗔𝗙𝗢𝗥𝗠 𝗗𝗥𝗜𝗙𝗧 & 𝗠𝗔𝗡𝗨𝗔𝗟 𝗖𝗛𝗔𝗡𝗚𝗘𝗦

𝗦𝗰𝗲𝗻𝗮𝗿𝗶𝗼: Unauthorized cloud console changes cause drift between Terraform & infra.

✔️ Run terraform plan frequently to detect drift.
✔️ Use terraform state list & terraform state show for insights.
✔️ Enforce IAM policies to block manual changes.

𝗕𝗟𝗨𝗘-𝗚𝗥𝗘𝗘𝗡 𝗗𝗘𝗣𝗟𝗢𝗬𝗠𝗘𝗡𝗧 𝗨𝗦𝗜𝗡𝗚 𝗧𝗘𝗥𝗥𝗔𝗙𝗢𝗥𝗠

𝗦𝗰𝗲𝗻𝗮𝗿𝗶𝗼: Need zero-downtime infra upgrades (e.g., launching a new AWS app version).

✔️ Use Terraform workspaces for separate environments.
✔️ Deploy a Blue (new) environment while Green (old) stays live.
✔️ Shift traffic using DNS or Load Balancer failover.

𝗦𝗘𝗖𝗨𝗥𝗘𝗟𝗬 𝗠𝗔𝗡𝗔𝗚𝗜𝗡𝗚 𝗦𝗘𝗖𝗥𝗘𝗧𝗦 𝗜𝗡 𝗧𝗘𝗥𝗥𝗔𝗙𝗢𝗥𝗠

𝗦𝗰𝗲𝗻𝗮𝗿𝗶𝗼: Passing sensitive data like API keys & passwords securely.

✔️ Never store secrets in Terraform code or state files.
✔️ Use AWS Secrets Manager, HashiCorp Vault, or SSM Parameter Store.
✔️ Pass secrets via environment variables (TF_VAR_*).

𝗠𝗔𝗡𝗔𝗚𝗜𝗡𝗚 𝗟𝗔𝗥𝗚𝗘-𝗦𝗖𝗔𝗟𝗘 𝗜𝗡𝗙𝗥𝗔 𝗪𝗜𝗧𝗛 𝗧𝗘𝗥𝗥𝗔𝗙𝗢𝗥𝗠 𝗠𝗢𝗗𝗨𝗟𝗘𝗦

𝗦𝗰𝗲𝗻𝗮𝗿𝗶𝗼: Codebase is too large & difficult to manage.

✔️ Break infra into reusable modules.
✔️ Use terraform get & terraform module update to refresh modules.
✔️ Store modules in a private registry or Git repo.

𝗧𝗘𝗥𝗥𝗔𝗙𝗢𝗥𝗠 𝗠𝗨𝗟𝗧𝗜-𝗖𝗟𝗢𝗨𝗗 𝗦𝗧𝗥𝗔𝗧𝗘𝗚𝗬

𝗦𝗰𝗲𝗻𝗮𝗿𝗶𝗼: Deploying infra across AWS, Azure, & GCP.

✔️ Use provider aliasing for multiple clouds.
✔️ Define separate provider modules & call them conditionally.
✔️ Sync infra using remote state storage (S3, GCS, Azure Blob).

𝗗𝗘𝗕𝗨𝗚𝗚𝗜𝗡𝗚 𝗧𝗘𝗥𝗥𝗔𝗙𝗢𝗥𝗠 𝗘𝗫𝗘𝗖𝗨𝗧𝗜𝗢𝗡 & 𝗣𝗘𝗥𝗙𝗢𝗥𝗠𝗔𝗡𝗖𝗘 𝗢𝗣𝗧𝗜𝗠𝗜𝗭𝗔𝗧𝗜𝗢𝗡

𝗦𝗰𝗲𝗻𝗮𝗿𝗶𝗼: terraform apply takes too long or fails unexpectedly.

✔️ Enable debug logs with TF_LOG=DEBUG terraform apply.
✔️ Speed up execution with terraform apply -parallelism=10.
✔️ Disable unnecessary outputs to optimize large Terraform plans.
