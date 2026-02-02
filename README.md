# XposedOrNot Breach Intelligence Integration for Microsoft Sentinel

[![Deploy to Azure](https://aka.ms/deploytoazurebutton)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fxposedornot%2FXposedOrNot-Sentinel%2Fmain%2FmainTemplate.json)
[![Version](https://img.shields.io/badge/version-1.0.0-blue.svg)](CHANGELOG.md)
[![License](https://img.shields.io/badge/license-MIT-green.svg)](LICENSE)

<p align="center">
  <img src="https://raw.githubusercontent.com/XposedOrNot/XposedOrNot-Sentinel/main/images/logo.svg" alt="XposedOrNot Logo" width="120">
</p>

## Why Breach Intelligence in Your SIEM?

Data breaches happen. What matters is knowing about them before attackers use stolen credentials against your organization. **XposedOrNot** monitors over **10.5 billion exposed records** across **659 verified breaches**, and this integration brings that intelligence right into Microsoft Sentinel.

**Key Benefits:**
- 🔍 **Proactive Threat Detection**: Spot compromised credentials before they're used in attacks
- 📊 **Unified Security View**: Correlate breach exposures with your existing security data
- ⚡ **Automated Monitoring**: Continuous sync of breach data for your monitored domains
- 🎯 **Prioritized Response**: Risk-scored exposures help you focus remediation efforts
- 📈 **Executive Dashboards**: Pre-built workbook for security posture reporting

This integration works with both **XposedOrNot Community Edition** (free) and **XposedOrNot Plus** (commercial):

| Edition | Use Case | API Key Source |
|---------|----------|----------------|
| **Community** | Individual/small team monitoring | [xposedornot.com](https://xposedornot.com) |
| **Plus - Enterprise** | Monitor your organization's domains | [plus.xposedornot.com](https://plus.xposedornot.com) |
| **Plus - ThreatIntel** | Monitor partners and customers | [plus.xposedornot.com](https://plus.xposedornot.com) |

Once deployed, the integration automatically pulls breach exposure data for your monitored domains. Your security team can then identify at-risk accounts, trigger password resets, and detect credential-based attacks.

---

## Installation

You can deploy this solution in two ways:

### Option 1: One-Click Deploy (Recommended)

Click the **Deploy to Azure** button above. The Azure Portal wizard walks you through configuration. Just provide your workspace details and API key.

### Option 2: Manual Deployment (CLI)

```bash
az deployment group create \
  -g <resource-group> \
  -f mainTemplate.json \
  -p workspaceName=<workspace> \
     workspaceResourceGroup=<workspace-rg> \
     xonApiKey=<your-api-key>
```

---

## Prerequisites

Before you begin, make sure you have:

### Required Resources

> ⚠️ **Important:** This template deploys **into an existing Sentinel workspace**. It does not create a new workspace.

- ✅ **Microsoft Sentinel-enabled Log Analytics workspace** (must exist before deployment)
- ✅ **XposedOrNot API key**: Get yours from:
  - [plus.xposedornot.com](https://plus.xposedornot.com) (Enterprise plans)
  - [xposedornot.com](https://xposedornot.com) (Free community API key works too!)

**Don't have a Sentinel workspace yet?** Create one first:
```bash
# Create Log Analytics workspace
az monitor log-analytics workspace create \
  --resource-group <your-rg> \
  --workspace-name <your-workspace> \
  --location <region>

# Enable Sentinel
az sentinel onboarding-state create \
  --resource-group <your-rg> \
  --workspace-name <your-workspace> \
  --name default
```

### Required Permissions

| Permission | Scope | Why It's Needed |
|------------|-------|-----------------|
| **Contributor** or **Owner** | Resource Group | Deploy Azure resources (Logic App, Key Vault, DCR) |
| **Microsoft Sentinel Contributor** | Workspace | Deploy workbook and analytics rule |

> 💡 **Tip:** Deploying to a different resource group than your Sentinel workspace? You'll need Contributor on both.

---

## ⚡ After Installation

**That's it, you're done!** 🎉

The connector starts automatically. Just wait for data to show up.

| Timeline | What Happens |
|----------|--------------|
| **0 min** | Deployment completes |
| **~2 min** | First data sync runs automatically |
| **~5-7 min** | Data appears in workbook |

> **Note:** The 2-minute delay lets Azure permissions propagate before the first sync.

**Optional steps:**
- [Grant yourself Key Vault access](#optional-grant-yourself-key-vault-access): Only needed if you want to view/update the API key later
- [Enable Analytics Rule](#step-3-enable-analytics-rule-after-24-hours): Recommended after 24h of data

**Need to update your API key later?** Check out [Managing the API Key](#-managing-the-api-key)

---

## Workbook Preview

<p align="center">
  <img src="https://raw.githubusercontent.com/XposedOrNot/XposedOrNot-Sentinel/main/images/workbook-screenshot.png" alt="XposedOrNot Sentinel Workbook" width="800">
</p>

*Breach intelligence workbook showing exposure analytics, risk breakdown, and timeline.*

**📍 How to access:** After deployment, go to **Microsoft Sentinel** → **Workbooks** → **My workbooks** → **XposedOrNot Breach Intelligence**

---

## What's Deployed

| Resource | Purpose |
|----------|---------|
| Key Vault | Securely stores API key (RBAC-enabled) |
| Data Collection Endpoint | Ingestion endpoint |
| Data Collection Rule | Schema and routing |
| Logic App | Scheduled data polling (auto-starts) |
| Workbook | Breach intelligence dashboard |
| Analytics Rule | New breach detection (disabled by default) |

**Total: 9 resources.** The Logic App uses Managed Identity for all access, so no manual authorization needed.

---

## Quick Start

### Step 1: Deploy the Template

Click the **Deploy to Azure** button above, or use the CLI command in the Installation section.

**Done!** The connector starts automatically. First data appears in ~5 minutes.

### Step 2: Verify Data (after 5 minutes)

Run in Log Analytics or check the Workbook:
```kql
XonBreachDetails_CL
| take 10
```

### Step 3: Enable Analytics Rule (after 24 hours)

1. Go to **Microsoft Sentinel** → **Analytics**
2. Find **"XposedOrNot - New Breach Exposure Detected"**
3. Click **Enable**

---

## Optional: Grant Yourself Key Vault Access

Only needed if you want to view or update the API key later.

The Key Vault uses RBAC. You need the **Key Vault Secrets Officer** role.

**Option A: Azure Portal**
1. Go to **Resource Group** → **kv-xon-[suffix]** (Key Vault)
2. Click **Access control (IAM)** in left menu
3. Click **+ Add** → **Add role assignment**
4. Select **Key Vault Secrets Officer**
5. Click **Next** → **+ Select members** → Search for your name → **Select**
6. Click **Review + assign**

**Option B: Azure CLI**
```bash
# Get your user ID
USER_ID=$(az ad signed-in-user show --query id -o tsv)

# Get Key Vault ID
KV_ID=$(az keyvault list -g <resource-group> --query "[?contains(name,'kv-xon')].id" -o tsv)

# Assign role
az role assignment create \
  --role "Key Vault Secrets Officer" \
  --assignee "$USER_ID" \
  --scope "$KV_ID"
```

⏳ **Wait 2-3 minutes** for the role to propagate before accessing secrets.

---

## 🔑 Managing the API Key

### View Current API Key

1. **Azure Portal** → **Resource Group** → **kv-xon-[suffix]**
2. Click **Secrets** in left menu
3. Click **xon-api-key**
4. Click current version → **Show Secret Value**

> ⚠️ **Seeing "unauthorized to view"?** You need Key Vault Secrets Officer role. See above.

### Update API Key

1. Go to Key Vault → **Secrets** → **xon-api-key**
2. Click **+ New Version**
3. Enter your new API key in **Secret value**
4. Click **Create**

The Logic App automatically picks up the new key on its next run.

### Update via CLI

```bash
az keyvault secret set \
  --vault-name kv-xon-<suffix> \
  --name xon-api-key \
  --value "your-new-api-key"
```

---

## Parameters

| Parameter | Required | Default | Description |
|-----------|----------|---------|-------------|
| `workspaceName` | Yes | - | Log Analytics workspace name |
| `workspaceResourceGroup` | Yes | - | Workspace resource group |
| `xonApiKey` | Yes | - | XposedOrNot API key |
| `pollingFrequencyHours` | No | 12 | Sync interval (1, 6, 12, 24) |
| `deployWorkbook` | No | true | Deploy dashboard |
| `deployAnalyticsRule` | No | true | Deploy detection rule |

---

## Data Schema

**Table:** `XonBreachDetails_CL`

| Column | Type | Description |
|--------|------|-------------|
| TimeGenerated | datetime | Ingestion time |
| Email | string | Exposed email |
| EmailDomain | string | Email domain part |
| Domain | string | Monitored domain |
| BreachName | string | Breach name |
| BreachedDate | datetime | When breach occurred |
| PasswordRisk | string | plaintext, easytocrack, unknown, stronghash |
| ExposedDataTypes | string | Types of exposed data |
| ExposedRecords | real | Records in breach |
| BreachDescription | string | Breach details |
| IsSearchable | boolean | Searchable in XON |
| SnapshotId | string | Sync run ID |

---

## Sample Queries

**High-risk exposures:**
```kql
XonBreachDetails_CL
| where PasswordRisk in ('plaintext', 'easytocrack')
| summarize Count=count() by Email, PasswordRisk
| order by Count desc
```

**Breaches by domain:**
```kql
XonBreachDetails_CL
| summarize 
    Exposures = count(),
    Breaches = dcount(BreachName)
  by Domain
| order by Exposures desc
```

**Recent activity:**
```kql
XonBreachDetails_CL
| where TimeGenerated > ago(24h)
| summarize Count=count() by bin(TimeGenerated, 1h)
| render timechart
```

---

## Troubleshooting

### "Unauthorized to view secrets" in Key Vault

**Cause:** Key Vault uses RBAC and you don't have permission.

**Fix:**
1. Go to Key Vault → **Access control (IAM)**
2. Add **Key Vault Secrets Officer** role to yourself
3. Wait 2-3 minutes for propagation

### Logic App fails with "Forbidden"

**Cause:** RBAC permissions haven't propagated yet.

**Fix:** Wait 5 minutes and retry. If still failing:
```bash
# Check Logic App's assigned roles
PRINCIPAL=$(az logic workflow show -g <rg> -n <name> --query identity.principalId -o tsv)
az role assignment list --assignee $PRINCIPAL -o table
```

Should show:
- **Key Vault Secrets User** on Key Vault
- **Monitoring Metrics Publisher** on DCR

### No data in workbook / table not found

**Cause:** Data hasn't synced yet, or there's a Logic App issue.

**Timeline check:**
- **Less than 7 minutes since deployment?** Wait. First sync runs at ~2 min, data appears at ~5-7 min.
- **More than 10 minutes?** Check steps below.

**Fix:**
1. Check Logic App state: Should be **Enabled**
2. Check run history in Logic App: Look for errors
3. If first run failed: Wait 1 minute for permissions, then click **Run Trigger** → **Scheduled_Poll**
4. Verify API key is valid at [xposedornot.com](https://xposedornot.com) or [plus.xposedornot.com](https://plus.xposedornot.com)

### Workbook shows "Awaiting data sync" but Logic App succeeded

**Cause:** Log Analytics ingestion delay (this is normal).

**Fix:** Wait 3-5 minutes after Logic App run completes. Data ingestion has some latency.

**Verify data arrived:**
```kql
XonBreachDetails_CL | count
```

---

## Architecture

```
XposedOrNot API ──► Logic App ──► Data Collection Endpoint/Rule
                        │                      │
                        ▼                      ▼
                   Key Vault            Log Analytics Workspace
                  (API Key)                    │
                                               ▼
                                       Microsoft Sentinel
                                      (Workbook + Analytics)
```

**Data Flow:**
1. **Logic App** runs on schedule (default: every 12 hours)
2. Retrieves API key securely from **Key Vault**
3. Calls **XposedOrNot API** to fetch breach data for monitored domains
4. Sends data to **Log Analytics** via Data Collection Rule
5. **Sentinel Workbook** visualizes the data; **Analytics Rule** creates incidents

---

## Security Notes

- **API Key** is stored in Key Vault, never in Logic App code
- **Key Vault** uses RBAC (not access policies) for modern security
- **Logic App** uses Managed Identity for all Azure access
- All connections use **TLS 1.2+**
- Key Vault has **soft-delete** enabled (7 days recovery)

---

## RBAC Roles Used

| Identity | Role | Scope | Purpose |
|----------|------|-------|---------|
| Logic App (MSI) | Key Vault Secrets User | Key Vault | Read API key |
| Logic App (MSI) | Monitoring Metrics Publisher | DCR | Write to Sentinel |
| You (deployer) | Key Vault Secrets Officer | Key Vault | Manage API key (optional) |

---

## Version History

### v1.0.0 (Current)
- Initial release
- Data Connector with Logic App (Managed Identity)
- Key Vault integration with RBAC-only security
- Pre-built Workbook for breach intelligence visualization
- Analytics Rule for new breach detection
- One-click Deploy to Azure with guided wizard
- Automatic first sync after deployment

---

## Open Source 💚

This project is part of the **XposedOrNot** open-source ecosystem. We believe security tools should be transparent, community-driven, and accessible to everyone.

**Why open source?**
- 🔍 **Transparency**: See exactly how your breach data is handled
- 🛡️ **Trust**: No black boxes, no hidden code
- 🤝 **Community**: Built by security professionals, for security professionals
- 🚀 **Innovation**: Your contributions make it better for everyone

Explore our other projects:
- [XposedOrNot-API](https://github.com/XposedOrNot/XposedOrNot-API): The API powering breach checks
- [XposedOrNot-Website](https://github.com/XposedOrNot/XposedOrNot-Website): The public web interface

---

## Show Your Support! ⭐

If this integration helps protect your organization:

- 🌟 **Star this repo** to help others discover it
- 🍴 **Fork it** and make it your own
- 🐛 **Report issues** so we can squash bugs
- 💡 **Contribute** with PRs (check [CONTRIBUTING.md](CONTRIBUTING.md))
- 📢 **Share** with your security team

Every star, fork, and contribution helps the security community. **Thank you!** 🙏

---

## Support

- **API Documentation**:
  - Community Edition: [api.xposedornot.com/docs](https://api.xposedornot.com/docs)
  - Plus (Commercial): [console.xposedornot.com/docs](https://console.xposedornot.com/docs)
- **Issues**: [GitHub Issues](https://github.com/xposedornot/XposedOrNot-Sentinel/issues)
- **Email**: help@xposedornot.com

## License

MIT License - see [LICENSE](LICENSE)
