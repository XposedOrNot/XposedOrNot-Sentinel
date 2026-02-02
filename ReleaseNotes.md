# Release Notes

## v1.0.0 (2026-02-01)

### 🎉 Initial Release

**XposedOrNot for Microsoft Sentinel** — Breach intelligence data connector with fully automated deployment.

#### Components
- **Data Connector** — Logic App that syncs breach data every 12 hours
- **Workbook** — Breach intelligence dashboard with analytics
- **Analytics Rule** — Incident detection for new breaches

#### Features
- ✅ One-click deployment via Azure Portal
- ✅ Fully automated post-deployment (KeyVault, DCR, initial sync)
- ✅ Zero manual configuration required
- ✅ Dashboard populates within 3-5 minutes

#### Data Collected
- Email breach exposures
- Password risk levels (plaintext, hashed, etc.)
- Exposed data types
- Breach details and timelines

#### Requirements
- Microsoft Sentinel enabled workspace
- XposedOrNot API key from [plus.xposedornot.com](https://plus.xposedornot.com)

---

*For support, visit [plus.xposedornot.com/contact](https://plus.xposedornot.com/contact)*
