# scoop-azguard

Scoop bucket for [AzGuard](https://github.com/cdobratz/AzGuard) — One command to make sure your Azure and AWS free tiers don't surprise you with a bill.

## Install

```powershell
# Add the bucket
scoop bucket add azguard https://github.com/cdobratz/scoop-azguard

# Install AzGuard
scoop install azguard
```

## Troubleshooting

If azguard doesn't run after install:

1. **Restart your terminal** - This reloads the PATH
2. **Manually add Scoop to PATH:**
   ```powershell
   $env:PATH += ";$env:USERPROFILE\scoop\shims"
   ```
3. **Check installation:**
   ```powershell
   scoop which azguard
   ```

For permanent fix, add the Scoop shims folder to your system PATH.

## Usage

### Azure

```powershell
# Quick overview of your Azure free tier status
azguard status

# Scan for Azure free tier overages
azguard scan

# Add a budget alert ($1-$100)
azguard budget add 5

# Find idle and orphaned resources (read-only)
azguard resources

# Interactive cleanup guide
azguard cleanup

# Show current month costs
azguard cost current
```

### AWS Free Tier Alerts

```powershell
# Check your AWS free tier usage
azguard aws status

# Scan AWS services approaching limits
azguard aws scan

# Set alert thresholds (percentage)
azguard aws alerts --threshold 80

# List free tier eligible services
azguard aws resources

# View AWS cost breakdown
azguard aws cost
```

## AWS Credential Setup

Do not put AWS access keys, secret keys, or session tokens in the AzGuard config file. The [AzGuard configuration guide](https://github.com/cdobratz/AzGuard#configuration) says the same thing. The config file holds non-secret settings such as the region. Cloud secrets are read from the environment or the OS keychain.

Config file: `~/.azguard/config.yaml`

```yaml
azure:
  auth_method: cli  # or service_principal
  subscription_id: YOUR_SUB_ID
  # Do not put client_secret in this file.

aws:
  region: us-east-1  # optional, uses AWS CLI default
  # Do not put access keys in this file.

storage:
  path: ~/.azguard/data.db
```

AzGuard resolves AWS credentials in this order:

1. **Environment variables.** `AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY`, and `AWS_SESSION_TOKEN` (temporary credentials) override the keychain.
2. **OS keychain** (service name `azguard`). If a config file still has `access_key`, `secret_key`, `session_token`, or `client_secret`, the next start copies those values into the keychain and removes them from the file. When the keychain is unavailable, the file is left as it is.
3. **AWS CLI profiles and SSO.** When the access key or secret key is still empty, AzGuard runs `aws configure export-credentials --format env`. That uses the default profile from `aws configure`, the named profile in `AWS_PROFILE`, or an IAM Identity Center (SSO) session after `aws sso login`.

```powershell
# Named profile
aws configure --profile your-profile
$env:AWS_PROFILE = "your-profile"

# IAM Identity Center / SSO
aws configure sso --profile your-profile
aws sso login --profile your-profile
$env:AWS_PROFILE = "your-profile"

# Environment variables. Include AWS_SESSION_TOKEN for temporary credentials.
$env:AWS_ACCESS_KEY_ID = "YOUR_ACCESS_KEY"
$env:AWS_SECRET_ACCESS_KEY = "YOUR_SECRET_KEY"
$env:AWS_SESSION_TOKEN = "YOUR_SESSION_TOKEN"
$env:AWS_DEFAULT_REGION = "us-east-1"
```

Sign in to Azure with the Azure CLI (`az login`). Do not put `client_secret` in the config file. Set `AZURE_CLIENT_SECRET` or leave the secret in the OS keychain.

### Required IAM Permissions

`azguard aws status`, `azguard aws scan`, and `azguard aws cost` call Cost Explorer and Free Tier:

- `ce:GetCostAndUsage` for Cost Explorer queries
- `freetier:GetFreeTierUsage` for free tier usage data

`azguard resources` is read-only. It does not delete anything. Give the Azure subscription the built-in Reader role. On AWS, attach ReadOnlyAccess, or the read-only actions listed in the [AzGuard README](https://github.com/cdobratz/AzGuard#read-only-permissions).

## Requirements

### Azure

- Azure subscription
- Azure CLI (`az`) authenticated (`az login`)

### AWS

- AWS account (free tier eligible)
- Credentials from an AWS CLI profile, IAM Identity Center SSO, or environment variables (see above). Never put secret keys in the config file.

For more commands and options, see the [full documentation](https://github.com/cdobratz/AzGuard).

## Other Install Methods

- **Homebrew:** `brew install cdobratz/azguard/azguard`
- **Direct:** Download from [GitHub Releases](https://github.com/cdobratz/AzGuard/releases)

## License

MIT — See [AzGuard](https://github.com/cdobratz/AzGuard) for details.
