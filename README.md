<div align="center">

# 🌉 thousandbridge

### **The world's first custom connector wiring Cisco ThousandEyes into Microsoft Power Platform, Defender XDR, and Cisco XDR**

*30 operations. Works with agents in AWS, Azure, on-prem. Zero inbound firewall rules. Built because nobody else did.*

---

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg?style=for-the-badge)](https://opensource.org/licenses/MIT)
[![Power Platform](https://img.shields.io/badge/Power_Platform-Ready-742774?style=for-the-badge&logo=microsoft&logoColor=white)](https://make.powerautomate.com)
[![Cisco ThousandEyes](https://img.shields.io/badge/Cisco_ThousandEyes-v7_API-1BA0D7?style=for-the-badge&logo=cisco&logoColor=white)](https://developer.cisco.com/docs/thousandeyes/)
[![Defender XDR](https://img.shields.io/badge/Defender_XDR-Integrated-0078d4?style=for-the-badge&logo=microsoft&logoColor=white)](https://learn.microsoft.com/en-us/defender-xdr/)
[![Cisco XDR](https://img.shields.io/badge/Cisco_XDR-Integrated-049fd9?style=for-the-badge&logo=cisco&logoColor=white)](https://www.cisco.com/site/us/en/products/security/xdr/)

[![AWS Ready](https://img.shields.io/badge/AWS-Ready-FF9900?style=flat-square&logo=amazon-aws&logoColor=white)](https://aws.amazon.com)
[![Azure Ready](https://img.shields.io/badge/Azure-Ready-0078D4?style=flat-square&logo=microsoft-azure&logoColor=white)](https://azure.microsoft.com)
[![On-Prem Ready](https://img.shields.io/badge/On--Prem-Ready-6A1B9A?style=flat-square)](https://www.thousandeyes.com)
[![First of its Kind](https://img.shields.io/badge/first-of_its_kind-ff0066?style=flat-square)](#-what-this-is)
[![Production Ready](https://img.shields.io/badge/production-ready-success?style=flat-square)](#-production-proof--real-200-ok-responses)
[![PRs Welcome](https://img.shields.io/badge/PRs-welcome-brightgreen.svg?style=flat-square)](http://makeapullrequest.com)

</div>

---

## 🚀 What this is

> *Cisco ThousandEyes monitors networks for 30,000+ enterprises including 145 of the Fortune 500. Microsoft Power Platform has 33+ million monthly active users. Cisco XDR and Microsoft Defender XDR together secure the majority of the Fortune 500. **And the connector linking all four — the bridge that would let any of those customers turn network telemetry into AI-driven security operations — had never been built. Not by Cisco. Not by Microsoft. Not by anyone.** So I built it.*

📝 **Read the full engineering field report:** [The Impossible Bridge — A Senior Engineer's Field Notes](https://powerofautomation2025.blogspot.com)

---

## ⚡ The architecture

```
┌─────────────────────────────────────────────────────────────────────────┐
│  MICROSOFT POWER PLATFORM (Azure-hosted, 33M MAU)                       │
└─────────────────────────────────────────────────────────────────────────┘
   💼 Power Automate    🎨 Power Apps    🤖 Copilot Studio
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────────────┐
│  🔌 thousandbridge — 30 Operations                                      │
└─────────────────────────────────────────────────────────────────────────┘
                              │
                       HTTPS 443 outbound
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────────────┐
│  ☁️  ThousandEyes Cloud — api.thousandeyes.com                          │
└─────────────────────────────────────────────────────────────────────────┘
                              ▲
                Agents push telemetry IN (outbound only)
                              │
        ┌─────────────────────┼─────────────────────┐
        │                     │                     │
   🟧 AWS EA              🟦 Azure EA          🟪 On-Prem EA
   EC2 / EKS              VM / AKS             VMware / Docker
        │                     │                     │
        └── all push HTTPS 443 OUTBOUND ────────────┘
              to data.thousandeyes.com

  + Power Automate flows fan out to:
  → 🛡️ Microsoft Defender XDR (via Sentinel Log Analytics)
  → 🛡️ Cisco XDR (via Incidents API)
  → 💬 Microsoft Teams (Adaptive Cards)
  → 🎫 ServiceNow (Incident creation)
```

**TL;DR:** Same connector serves AWS, Azure, on-prem agents. Same flows feed both XDR platforms. **Lab → Production migration is one host change + one credential rotation.**

---

## 📦 What's in the repo

| File | Purpose |
|---|---|
| 📦 `thousandeyes-simulator.zip` | Node.js v3.0 simulator — local ThousandEyes-on-a-laptop |
| 📄 `thousandeyes-connector.yaml` | OpenAPI 2.0 spec — 30 operations, ready to import |
| 📖 `COMPLETE-BEGINNER-GUIDE.html` | 14-part guide with SVG screenshots and color-coded callouts |
| 📰 `BLOG-POST-cisco-recruiters.html` | The full engineering story with embedded proof screenshots |
| 🟧 `deployment/aws-enterprise-agent.md` | Deploy ThousandEyes Enterprise Agent in AWS |
| 🟦 `deployment/azure-enterprise-agent.md` | Deploy ThousandEyes Enterprise Agent in Azure |
| 🟪 `deployment/onprem-enterprise-agent.md` | Deploy on VMware/Hyper-V/Docker |
| 🛡️ `xdr/defender-xdr-flow.json` | Power Automate template — alerts to Defender XDR |
| 🛡️ `xdr/cisco-xdr-flow.json` | Power Automate template — alerts to Cisco XDR |
| 🤖 `copilot-studio/netops-agent.md` | System prompt + starters for the NetOps Copilot |

---

## 🌍 Works with ThousandEyes deployed ANYWHERE

The custom connector doesn't talk to your agents directly. It talks to **ThousandEyes Cloud** (`api.thousandeyes.com`), which already has visibility into every agent you've deployed — regardless of where they live.

### 🟧 AWS deployment

```bash
# Quick start — Marketplace AMI
aws ec2 run-instances \
  --image-id ami-0abc1234def56789 \   # ThousandEyes Enterprise Agent AMI
  --instance-type t3.small \           # 2 vCPU, 2GB RAM
  --key-name your-key \
  --security-group-ids sg-eaagent \    # outbound 443 only
  --subnet-id subnet-xxx
```

**Security Group required rules** — **outbound only, no inbound**:

| Direction | Port | Destination | Purpose |
|---|---|---|---|
| Outbound | TCP 443 | `*.thousandeyes.com` | Telemetry upload + agent registration |
| Outbound | UDP 123 | Any NTP | Time sync (critical for test timestamps) |
| Outbound | UDP/TCP 53 | Your VPC DNS | Name resolution |

📖 **Full guide:** [deployment/aws-enterprise-agent.md](./deployment/aws-enterprise-agent.md)

### 🟦 Azure deployment

```bash
# Quick start — Marketplace VM
az vm create \
  --resource-group rg-te-agents \
  --name te-ea-azure-frankfurt \
  --image cisco:thousandeyes-enterprise-agent:standard:latest \
  --size Standard_B2s \                # 2 vCPU, 4GB RAM
  --vnet-name vnet-prod \
  --subnet subnet-agents \
  --nsg nsg-eaagent                    # outbound 443 only
```

**NSG required rules** — outbound only:

| Direction | Port | Destination | Purpose |
|---|---|---|---|
| Outbound | TCP 443 | `*.thousandeyes.com` | Telemetry upload |
| Outbound | UDP 123 | Any NTP | Time sync |
| Outbound | UDP/TCP 53 | Azure DNS or your resolver | Name resolution |

**Optional:** Azure Private Link to ThousandEyes for traffic that stays off the public internet.

📖 **Full guide:** [deployment/azure-enterprise-agent.md](./deployment/azure-enterprise-agent.md)

### 🟪 On-prem deployment

Multiple form factors — pick what your DC team prefers:

| Form factor | When to use |
|---|---|
| **VMware OVA** | Existing VMware estate |
| **Hyper-V VHDX** | Windows Server / Azure Stack HCI |
| **Docker container** | Container-first shops, RKE2/K3s/Talos clusters |
| **Physical appliance** | Edge sites, branch offices, FedRAMP environments |

📖 **Full guide:** [deployment/onprem-enterprise-agent.md](./deployment/onprem-enterprise-agent.md)

---

## 🔥 The firewall table your network team needs

This is the table I had to compile from three different Cisco docs. **Save this — your firewall team will ask for it.**

### From Enterprise Agents (AWS / Azure / on-prem) → ThousandEyes Cloud

| Destination | Port | Protocol | Purpose |
|---|---|---|---|
| `data.thousandeyes.com` | 443 | TCP | Test results upload |
| `agents.thousandeyes.com` | 443 | TCP | Agent registration + config pull |
| `app.thousandeyes.com` | 443 | TCP | OAuth + management |
| `scribe.thousandeyes.com` | 443 | TCP | Agent diagnostics + logs |
| Any public NTP | 123 | UDP | Time sync — critical for valid timestamps |
| Your DNS resolver | 53 | UDP/TCP | Name resolution |

### From Power Platform → ThousandEyes Cloud (production)

| Destination | Port | Protocol | Purpose |
|---|---|---|---|
| `api.thousandeyes.com` | 443 | TCP | Custom connector API calls |

### From On-Prem Gateway (lab only) → Microsoft Azure

| Destination | Port | Protocol | Purpose |
|---|---|---|---|
| `*.servicebus.windows.net` | 443 | TCP | Azure Service Bus relay |
| `*.frontend.clouddatahub.net` | 443 | TCP | Gateway control plane |
| `*.azure-api.net` | 443 | TCP | API Management |
| `login.microsoftonline.com` | 443 | TCP | M365 authentication |

> ### ✅ The genius of this design
> **Every connection is outbound HTTPS 443 from the customer side.** No inbound firewall rules. No port forwarding. No DMZ. No VPN. No exposed services. Same pattern works whether your agents are in AWS Mumbai or on bare metal in Buffalo.

---

## 🛡️ XDR integration patterns

### Microsoft Defender XDR

```
ThousandEyes Alert (network anomaly)
    ↓ Power Automate trigger (every 5 min)
ListAlerts(state=active) → for each new alert
    ↓ Enrich with GetAlert + FilterOutages
Build custom event payload
    ↓ Microsoft Sentinel connector
Send to Log Analytics Workspace
    Custom table: ThousandEyes_Alerts_CL
    ↓
Defender XDR auto-correlates with:
  • Identity sign-in anomalies
  • Endpoint EDR signals
  • Email threat indicators
  • Cloud app session risks
    ↓
Unified incident in Defender XDR portal
```

📖 **Template:** [xdr/defender-xdr-flow.json](./xdr/defender-xdr-flow.json) — import into Power Automate.

### Cisco XDR

```
ThousandEyes Alert (CRITICAL severity)
    ↓ Power Automate trigger
ListAlerts → for each new CRITICAL alert
    ↓ Translate to CTIM (Cyber Threat Intelligence Model)
POST https://visibility.amp.cisco.com/iroh/iroh-int/incident
    With OAuth Bearer token
    ↓
Cisco XDR creates incident linked to:
  • Affected URLs (target_observables)
  • Affected ASNs (network-observable)
  • Source agents (host-observable)
    ↓
Cisco XDR fuses with:
  • Umbrella DNS resolutions
  • Secure Endpoint EDR
  • Duo MFA events
  • Meraki dashboard signals
  • Talos threat intelligence
    ↓
SOC analyst sees full Cisco portfolio view
```

📖 **Template:** [xdr/cisco-xdr-flow.json](./xdr/cisco-xdr-flow.json) — import into Power Automate.

---

## 🎯 The 30 operations

<details>
<summary><b>Click to expand the full API surface (14 resource families)</b></summary>

### 👥 Account & Users (2)
`ListAccountGroups` · `GetCurrentUser`

### 🌍 Agents (2)
`ListAgents` · `GetAgent`

### 🧪 Tests (7)
`ListAllTests` · `ListTestsByType` · `GetTest` · `CreateTest` · `UpdateTest` · `DeleteTest` · `RunInstantTest`

### 📊 Test Results (3)
`GetNetworkTestResults` · `GetHttpTestResults` · `GetPathVisualization`

### 🚨 Alerts (2)
`ListAlerts` · `GetAlert`

### 📜 Alert Rules (4)
`ListAlertRules` · `GetAlertRule` · `CreateAlertRule` · `DeleteAlertRule`

### 🤫 Suppression Windows (2)
`ListSuppressionWindows` · `CreateSuppressionWindow`

### 📈 Dashboards (2)
`ListDashboards` · `GetDashboard`

### 🌐 Internet Insights (2)
`ListCatalogProviders` · `FilterOutages`

### 🛰️ BGP, Tags, Usage, Integrations (4)
`ListBgpMonitors` · `ListTags` · `GetUsage` · `ListWebhookOperations`

</details>

---

## 🚀 Quick start (30 minutes)

```bash
# 1. Clone
git clone https://github.com/PowerofAutomation2026/thousandbridge.git
cd thousandbridge

# 2. Extract the simulator
unzip thousandeyes-simulator.zip
cd thousandeyes-simulator

# 3. Configure for LAN binding (so on-prem gateway can reach it)
cat > .env << EOF
PORT=4000
HOST=0.0.0.0
PERSIST_FILE=./simulator-data.json
EOF

# 4. Install + start
npm install
npm start
# Watch for banner: "ThousandEyes API Simulator – v3.0 (multi-auth + OAuth 2.0)"
# Copy the bearer token from section "2. Basic auth"
```

Then in Power Platform:

| Step | Action |
|---|---|
| 1️⃣ | Install [On-Premises Data Gateway](https://aka.ms/OnPremisesDataGatewayInstaller) |
| 2️⃣ | Edit `thousandeyes-connector.yaml` — change `host:` to your LAN IP (e.g. `10.0.5.42:4000`) |
| 3️⃣ | Power Automate → Custom connectors → **+ New → Import OpenAPI** → upload the YAML |
| 4️⃣ | ✅ Check **"Connect via on-premises data gateway"** on General tab |
| 5️⃣ | Security tab auto-lands on **Basic authentication** (because of the YAML) |
| 6️⃣ | Create connection: **Username = `user`** · **Password = your raw token** · **Gateway = your gateway** |
| 7️⃣ | Test `ListAgents` → 🎉 **200 OK with 14 agents** |

📖 **Full walkthrough with SVG screenshots:** open `COMPLETE-BEGINNER-GUIDE.html` in a browser.

---

## 🏗️ The auth puzzle (and how it was solved)

Power Platform's on-premises data gateway has a **fundamental limitation**: when "Connect via on-premises data gateway" is enabled, the auth dropdown only shows three options:

```
✅ No authentication
✅ Basic authentication
✅ Windows authentication
❌ API Key (not available with gateway)
❌ OAuth 2.0 (not available with gateway)
```

But ThousandEyes uses OAuth2 Bearer tokens. So how do you bridge them?

**The Basic Auth Wrapper trick** — 8 lines of code:

```javascript
if (authHeader.startsWith('Basic ')) {
  const b64 = authHeader.substring(6);
  const decoded = Buffer.from(b64, 'base64').toString('utf8');
  const colonIdx = decoded.indexOf(':');
  // Username is ignored. Password IS the bearer token.
  token = decoded.substring(colonIdx + 1);
}
```

Power Platform sends Basic auth ✅ Gateway accepts it ✅ Simulator extracts the token ✅ **Zero security loss, full gateway compatibility.**

For production, flip `securityDefinitions` from `basic` to `apiKey` and point at `api.thousandeyes.com`. **Same connector. Same 30 operations. Same flows. Different security envelope.**

---

## 🪨 The 11 cliffs I fell off (documented so you don't)

| # | Cliff | Time lost | Fix |
|---|---|---|---|
| 1 | Phantom API Key dropdown (gateway limitation) | 90 min | Basic auth wrapper |
| 2 | v1.0 simulator can't decode Basic auth | 120 min | Multi-auth `auth.js` (v3.0) |
| 3 | `127.0.0.1` binding blocks gateway | 60 min | Set `HOST=0.0.0.0` |
| 4 | Firewall Public profile silently fails | 45 min | Domain profile only |
| 5 | Tokens regenerate on every restart | 30 min | Set `PERSIST_FILE` |
| 6 | Route ordering bug (`/alerts/rules` vs `/alerts/:id`) | 45 min | Specific routes first |
| 7 | SQL JOIN not supported in micro-parser | 30 min | JS-side lookups |
| 8 | Closure bug in WHERE clause | 20 min | Scope `argIdx` per row |
| 9 | `activeAlerts: undefined` from COUNT | 20 min | `.get().c` |
| 10 | Token format ambiguity (`Bearer ` vs raw) | 30 min | Big red callouts in guide |
| 11 | Power Platform connector cache | 15 min | Wait 5 min between delete & recreate |

**Every fix is baked into the deliverables.** Following the README, you hit zero of these.

---

## ✅ Production proof — real 200 OK responses

The full blog post includes embedded screenshots of real Power Platform connector tests:

- ✅ `ListSuppressionWindows` → **HTTP 200 OK** with maintenance window data, schema validation succeeded
- ✅ `ListBgpMonitors` → **HTTP 200 OK** with Equinix Ashburn and PCCW Singapore monitors, schema validation succeeded

📰 **See the proof:** [BLOG-POST-cisco-recruiters.html](./BLOG-POST-cisco-recruiters.html)

---

## 🛡️ Production-grade by design — defense in depth

| Layer | Guarantee |
|---|---|
| 🔐 **ThousandEyes RBAC** | Token tied to least-privilege role. `CreateTest`/`DeleteTest` return 403 unless explicitly granted. |
| 🚦 **Power Platform throttling** | Per-connector call limits prevent runaway flows. |
| 📋 **DLP policies** | Connector classified as Business data, can't mix with Personal connectors. |
| 📝 **Audit logging** | Microsoft Purview + ThousandEyes audit trail = bidirectional traceability. |
| 🏢 **Gateway boundary** | Lab simulator never touches public internet. |
| 🔑 **Encrypted credentials** | Tokens in Power Platform connection store, encrypted at rest, never logged. |
| 🛡️ **XDR correlation** | Every alert pushed to Defender XDR + Cisco XDR for unified hunting. |

---

## 📜 License

[MIT](LICENSE) — use it, fork it, ship it. Just don't blame me if you accidentally fix everything at your company.

---

## 🧑‍💻 About the author

**Kerolos** — Senior Infrastructure & Security Engineer

Ex-Cisco · Currently at a major US bank · Specializing in AI-powered automation at the Microsoft × Cisco intersection.

I build the connectors and integrations your customers actually want. If your team is building at the intersection of network observability, AI agents, XDR, and developer experience — **let's talk**.

- 💼 [LinkedIn](https://www.linkedin.com/in/kirolos-erian-39014349/)
- 📝 [Power of Automation Blog](https://powerofautomation2025.blogspot.com)
- 🐙 [GitHub Profile](https://github.com/PowerofAutomation2026)

---

## 🚀 What's next

The connector pattern extends across the entire Cisco portfolio:

- 🌐 **Cisco Meraki connector** — Dashboard API for cloud-managed networks
- 🛡️ **Cisco Umbrella connector** — DNS security telemetry → Sentinel + Defender XDR
- 🔐 **Cisco Duo connector** — MFA event correlation with Sentinel UEBA
- 📞 **Cisco Webex Control Hub connector** — Device health and call quality from Teams
- 🤖 **Cisco XDR + Defender XDR fusion Copilot** — One agent knows your entire Cisco + Microsoft security estate

**Write the connector once. Wire it into Teams, Power Apps, and Copilot Studio.**
That's the real promise — and we're still in the first inning.

---

## 🙏 Acknowledgments

- **Cisco ThousandEyes Engineering** — for an API surface clean enough that this was possible
- **Cisco XDR & Microsoft Defender XDR teams** — for ingest APIs that make this fusion real
- **Microsoft Power Platform team** — for the custom connector framework (please make API Key work with the on-prem gateway 🙏)
- **The 2020 NETWORG blog post** — for documenting the gateway auth limitation Microsoft still hasn't put in their official docs
- **Every NOC engineer who's been a human router between five SaaS tools at 9 AM** — this is for you

---

<div align="center">

### ⭐ If this helped you, star the repo

**Made with ☕, 🐍, and the conviction that *nobody should manually pivot between five tools.***

</div>
