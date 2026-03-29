# 🎣 Phishing Attack Simulation with Gophish

> **Cybersecurity Project | March 2026**
> **Johnbosco Ibeneme** |
> 📧 chizzyibeneme@gmail.com | 🔗 [LinkedIn](https://linkedin.com/in/chizitem-ibeneme) | 💻 [GitHub](https://github.com/Chizitem-sec)

---

## 📋 Table of Contents

- [Project Overview](#-project-overview)
- [Tools & Technologies](#-tools--technologies)
- [Architecture](#-architecture)
- [Phase 1 — Cloud Deployment on Railway](#phase-1--cloud-deployment-on-railway)
- [Phase 2 — Infrastructure Issue: SMTP Egress Blocked](#phase-2--infrastructure-issue-smtp-egress-blocked)
- [Phase 3 — Local Gophish Deployment (macOS)](#phase-3--local-gophish-deployment-macos)
- [Phase 4 — SMTP Configuration with Mailgun](#phase-4--smtp-configuration-with-mailgun)
- [Phase 5 — Campaign Configuration](#phase-5--campaign-configuration)
- [Phase 6 — Campaign Execution & Results](#phase-6--campaign-execution--results)
- [Challenges & Troubleshooting](#-challenges--troubleshooting)
- [Key Findings](#-key-findings)
- [Skills Demonstrated](#-skills-demonstrated)
- [Ethical Disclaimer](#-ethical-disclaimer)

---

## 📌 Project Overview

This project demonstrates a full phishing attack simulation using **Gophish**, an open-source phishing framework. The goal was to simulate how threat actors craft and deliver phishing emails, and how organizations can leverage such tools defensively for **security awareness training**.

The deployment follows a **hybrid approach**:
- Gophish is first deployed to **Railway** (cloud platform) to demonstrate cloud infrastructure and CI/CD skills
- After identifying SMTP egress restrictions on Railway, the project pivots to a **local macOS deployment** — a real-world troubleshooting decision that mirrors production security engineering

This project covers the full simulation lifecycle: **cloud deployment → SMTP relay configuration → email template crafting → landing page cloning → campaign launch → result tracking**.

---

## 🛠 Tools & Technologies

| Tool | Purpose |
|------|---------|
| **Gophish v0.12.1** | Open-source phishing simulation framework |
| **Railway** | Cloud hosting platform for initial deployment |
| **GitHub** | Version control, repository forking, config management |
| **Mailgun** | SMTP email relay service |
| **ngrok** | Reverse tunnel to expose local phish server publicly |
| **macOS Terminal** | Local binary execution and process management |
| **LinkedIn** | Cloned login page used as credential harvesting landing page |
| **Gmail** | Target inbox for simulation |

---

## 🏗 Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                     HYBRID DEPLOYMENT                        │
│                                                              │
│  ┌──────────────────┐         ┌──────────────────────────┐  │
│  │  Railway (Cloud) │         │  Local macOS (Gophish)   │  │
│  │                  │         │                          │  │
│  │  Admin Panel     │  -OR->  │  Admin Panel  :3333      │  │
│  │  Port :3333      │         │  Phish Server :80        │  │
│  │                  │         │        │                 │  │
│  └──────────────────┘         │        ▼                 │  │
│                               │   ngrok tunnel           │  │
│                               │  (public HTTPS URL)      │  │
│                               └──────────────────────────┘  │
│                                        │                     │
│                    ┌───────────────────▼──────────────────┐ │
│                    │          Mailgun SMTP Relay           │ │
│                    │       smtp.mailgun.org:587            │ │
│                    └───────────────────┬──────────────────┘ │
│                                        │                     │
│                    ┌───────────────────▼──────────────────┐ │
│                    │           Target Inbox               │ │
│                    │     (Simulated Phishing Email)       │ │
│                    └──────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────┘
```

---

## Phase 1 — Cloud Deployment on Railway

### Step 1: Fork the Gophish Repository

Navigated to the official Gophish GitHub repository and forked it to the personal account `Chizitem-sec`.

![Gophish Official Repo](screenshots/01_github_gophish_repo.png)
*Official Gophish repository — Open-Source Phishing Toolkit with 13.7k stars and 2.8k forks*

![Forked Repo](screenshots/02_forked_repo.png)
*Forked repository at Chizitem-sec/gophish — synced with upstream gophish/gophish:master*

---

### Step 2: Set Up Railway and Deploy

Created a Railway account via GitHub OAuth and created a new project by selecting "Deploy from GitHub repo."

![Railway Dashboard](screenshots/03_railway_dashboard.png)
*Railway dashboard — Limited Trial workspace with $5.00 credit, ready to create a new project*

**Issue encountered:** Railway showed "No repositories found" because the GitHub App had not yet been granted repository access.

![Railway No Repos](screenshots/04_railway_no_repos.png)
*Error — Railway cannot find GitHub repositories. GitHub App permissions have not been configured yet*

**Resolution:** Clicked Configure GitHub App → granted access to the forked Gophish repo → refreshed Railway.

![Railway Repo List](screenshots/05_railway_repo_list.png)
*After granting permissions — Chizitem-sec/gophish now appears in the repository selection list*

![Railway ToS](screenshots/06_railway_tos.png)
*Railway Terms of Service and Privacy Policy accepted before proceeding with deployment*

---

### Step 3: Deploy and Monitor Initialization

Selected the forked repository. Railway automatically detected the Go runtime and began building.

![Gophish Initializing](screenshots/07_railway_initializing.png)
*Gophish service initializing on Railway — deployment in progress*

![Gophish Online](screenshots/08_railway_online.png)
*Gophish service successfully deployed and showing Online status on Railway*

---

### Step 4: Generate a Public Domain

Navigated to the service Settings → Networking → Generated a public domain with target port 3333.

![Railway Networking](screenshots/09_railway_networking.png)
*Railway service Networking settings — Generate Domain option visible under Public Networking*

![Generate Domain](screenshots/10_railway_generate_domain.png)
*Generate Service Domain dialog — target port set to 3333 (Gophish admin server port)*

![Domain Generated](screenshots/11_railway_domain_generated.png)
*Public domain generated: gophish-production-10b9.up.railway.app routing to Port 3333*

---

### Step 5: Set Environment Variable

Added the `PORT` environment variable with value `3333` in the Railway Variables tab.

![PORT Variable](screenshots/12_railway_port_variable.png)
*PORT=3333 environment variable added — ensures the admin server binds to the correct port*

---

### Step 6: Edit config.json

Edited `config.json` directly via the GitHub browser editor to configure Gophish for public access.

**Before — default config:**

![Config Before](screenshots/13_config_json_before.png)
*Default config.json — listen_url bound to 127.0.0.1:3333, use_tls set to true, trusted_origins empty array*

**Changes applied:**
```json
"admin_server": {
    "listen_url": "0.0.0.0:3333",
    "use_tls": false,
    "trusted_origins": ["gophish-production-10b9.up.railway.app"]
}
```

**After — updated config:**

![Config After](screenshots/14_config_json_after.png)
*Updated config.json — listen_url changed to 0.0.0.0:3333, use_tls set to false, Railway domain added to trusted_origins with correct JSON string syntax*

Changes were committed directly via GitHub, which automatically triggered a Railway redeployment via CI/CD.

---

### Step 7: Confirm Deployment and Log In


![Deploy Logs](screenshots/16_railway_deploy_logs.png)
*Deploy logs — database migrations completing successfully, phishing server and admin server both starting at correct addresses*

> **Security note:** Temporary admin credentials were retrieved privately from deploy logs and immediately reset upon first login. No credentials are exposed in this repository.

![Gophish Login Railway](screenshots/18_gophish_login_railway.png)
*Gophish admin login page successfully served at the Railway public domain — accessible from any browser*

![Reset Password](screenshots/19_gophish_reset_password.png)
*Gophish prompted an immediate password reset on first login — new secure credentials set*

---

## Phase 2 — Infrastructure Issue: SMTP Egress Blocked

### Sending Profile Configuration Attempt on Railway

After logging into the Railway-hosted Gophish instance, a sending profile was configured with Mailgun SMTP credentials to begin testing email delivery.

![Sending Profile on Railway](screenshots/21_mailgun_smtp_credentials.png)
*Gophish Sending Profile configured on Railway — Name: "Security Test Profile", SMTP From and Username set to Mailgun address, Host: smtp.mailgun.org:587, Ignore Certificate Errors enabled*

### SMTP Timeout Error

Every send attempt resulted in a connection timeout regardless of port or provider:

```
Max connection attempts exceeded - dial tcp 34.160.13.42:465: i/o timeout
```

![SMTP Timeout](screenshots/22_mailgun_smtp_saved.png)
*SMTP timeout error on Railway — "Max connection attempts exceeded" confirms Railway is blocking all outbound SMTP connections at the network level*

**Ports systematically tested:**

| Provider | Port | Result |
|----------|------|--------|
| smtp.gmail.com | 587 | ❌ Timeout |
| smtp.gmail.com | 465 | ❌ Timeout |
| smtp.mailgun.org | 587 | ❌ Timeout |
| smtp.mailgun.org | 465 | ❌ Timeout |

**Root Cause:** Railway enforces egress firewall rules blocking all outbound SMTP traffic. This is a platform-level restriction, not a credentials or configuration error.

**Decision:** A hybrid approach was adopted. The Railway deployment was retained as demonstration of cloud infrastructure skills. Campaign execution was moved to a **local macOS deployment** where SMTP egress is unrestricted.

> This mirrors real-world engineering decisions where infrastructure constraints require adaptive solutions under operational conditions.

---

## Phase 3 — Local Gophish Deployment (macOS)

### Step 1: Create a Dedicated Test Gmail Account

A dedicated throwaway Gmail account was created exclusively for use as the simulated phishing sender identity, isolated from personal accounts.

![Test Gmail Created](screenshots/20_test_gmail_created.png)
*New Google account created: ChizzyTestgophish (chizzytestgophish@gmail.com) — dedicated sender account for the simulation*

---

### Step 2: Download the macOS Binary

Downloaded the pre-compiled macOS binary directly from the Gophish GitHub Releases page.

![Binary Download](screenshots/23_gophish_sending_profile.png)
*GitHub Releases — gophish-v0.12.1-osx-64bit.zip selected and downloading (33.2 MB macOS binary)*

---

### Step 3: Run Gophish in Terminal

```bash
cd ~/Downloads/gophish-v0.12.1-osx-64bit
./gophish
```

**Permission error encountered and resolved:**

```bash
# Error
zsh: permission denied: ./gophish

# Resolution
sudo chmod +x gophish
./gophish
```

![Terminal Startup](screenshots/24_smtp_timeout_error.png)
*Terminal — permission denied error on first attempt, chmod +x applied, Gophish then starts successfully. Database migrations run, phishing server starts on http://0.0.0.0:80, admin server starts on https://127.0.0.1:3333*

The local admin panel was then accessed at `https://localhost:3333`.

---

## Phase 4 — SMTP Configuration with Mailgun

### Step 1: Set Up Mailgun SMTP Credentials

Created a free Mailgun account, navigated to Domain Settings → SMTP Credentials, and created a new SMTP user.

![Mailgun SMTP Settings](screenshots/26_first_email_received.png)
*Mailgun Domain Settings — SMTP credentials tab showing newly created SMTP user. SMTP hostname: smtp.mailgun.org, available ports: 25, 587, 2525, and 465 (SSL/TLS)*

---

### Step 2: Configure the Sending Profile in Gophish

Using the Mailgun SMTP details, the sending profile was configured in the local Gophish admin panel.

![Sending Profile Configured](screenshots/21_mailgun_smtp_credentials.png)
*Gophish Sending Profile — "Security Test Profile" configured with Mailgun SMTP address, host smtp.mailgun.org:587, password set, Ignore Certificate Errors enabled*

---

### Step 3: Sending Profile Saved

![Profile Saved](screenshots/29_landing_page_configured.png)
*Gophish Sending Profiles page — "Profile added successfully!" green banner confirms the profile was saved. Security Test Profile listed with SMTP interface type*

---

## Phase 5 — Campaign Configuration

### Target Group

A target group was created with two simulated employees representing different organizational roles.


<img width="1470" height="956" alt="Screenshot 2026-03-24 at 2 15 40 PM" src="https://github.com/user-attachments/assets/720e34a5-ae79-4d01-85af-6b41410131ae" />
*Target group "Test group" — Jermaine Ibeneme (COO) and Chizitem Ibeneme (CEO) added as phishing simulation targets in the local Gophish instance*

---

### Landing Page

Gophish's Import Site feature was used to clone the LinkedIn login page as the credential harvesting landing page.


<img width="1470" height="956" alt="Screenshot 2026-03-24 at 2 19 41 PM" src="https://github.com/user-attachments/assets/242da682-0f17-44cd-98c1-e54d9ccb3305" />
*Import Site dialog — https://www.linkedin.com/login entered to clone the login page as the fake landing page*

![Landing Page Configured](screenshots/33_gophish_binary_download.png)
*Cloned LinkedIn landing page — Password field and Sign In button rendered in the HTML editor. Capture Submitted Data is checked, redirect to https://www.linkedin.com configured to redirect victims after form submission*

---

### Email Template

An HTML phishing email template was crafted impersonating an IT Security team, using Gophish's `{{.FirstName}}` and `{{.URL}}` template variables for personalization and click tracking.

![Email Template](screenshots/35_email_template_html.png)
*Email Templates editor — Name: "Security Alert", Envelope Sender: LinkedIn Support, Subject: "Urgent: Verify Your Account", HTML body with Gophish template variables {{.FirstName}} and {{.URL}}*

---

### Campaign Setup

All components were linked into a single campaign ready to launch.

![New Campaign](screenshots/36_campaign_setup.png)
*New Campaign form — Email Template: Security Alert, Landing Page: Fake Login Page, URL: http://localhost:3333, Sending Profile: Security Test Profile, Groups: Test group, Launch Date: March 25th 2026*

---

## Phase 6 — Campaign Execution & Results

### ngrok Tunnel

An ngrok tunnel was configured to expose the local Gophish phishing server to the public internet, enabling tracking link functionality.

```bash
ngrok http 80
```

![ngrok Running](screenshots/38_ngrok_running.png)
*ngrok session established — forwarding https://imagistic-carinulate-carolin.ngrok-free.dev → http://localhost:80, Session Status: online, US region*

![ngrok Active](screenshots/39_ngrok_active.png)
*ngrok confirmed active — 31ms latency, forwarding tunnel live*

![ngrok Requests](screenshots/40_ngrok_requests.png)
*ngrok logging live HTTP requests — 404 responses on the base URL are expected. Gophish only serves content when a valid ?rid= tracking parameter is present, confirming the tunnel is correctly forwarding to Gophish*

---

### Phishing Emails Delivered

Campaigns were launched and phishing emails were successfully delivered to target inboxes across multiple iterations.

**Iteration 1 — Initial plain text version:**

![First Email Delivered](screenshots/34_phishing_email_received.png)
*First phishing email delivered to target inbox — Gmail correctly flagged as spam. Body shows plain text with raw tracking URL visible, demonstrating the initial unrefined template before HTML improvements*

**Iteration 2 — Refined HTML version:**

![Styled Phishing Email](screenshots/37_styled_phishing_email.png)
*Refined phishing email — polished HTML layout with Security Alert heading, personalized "Hi Chizitem" greeting, urgency text with bold formatting, and "Verify My Account" CTA button. Gmail danger warning still triggered, demonstrating spam filter and phishing detection in action*

---

### LinkedIn Landing Page

The cloned LinkedIn login page successfully rendered as the credential capture destination.

![LinkedIn Landing Page](screenshots/41_linkedin_landing_page.png)
*Cloned LinkedIn login page — complete with email/phone and password fields, Google and Apple SSO buttons, Sign In button, and LinkedIn branding. Configured to capture submitted credentials before redirecting to the real LinkedIn site*

---

### Campaign Dashboard

![Campaign Dashboard](screenshots/42_campaign_dashboard.png)
*Campaign dashboard — 2 emails sent to Chizitem Ibeneme (CEO, chizzyibeneme@gmail.com) and Jermaine Ibeneme (COO, ibenemejarmaine1@gmail.com), both showing "Email Sent" status. Campaign Timeline visible at top*

---

## ⚠️ Challenges & Troubleshooting

### Challenge 1: Railway GitHub App Permissions

| | |
|---|---|
| **Problem** | Railway showed "No repositories found" when deploying from GitHub |
| **Root Cause** | Railway GitHub App had not been granted repository permissions |
| **Fix** | Configure GitHub App → selected forked repo → saved → refreshed Railway |

---

### Challenge 2: Railway SMTP Egress Blocked

| | |
|---|---|
| **Problem** | All SMTP test emails timed out regardless of provider or port |
| **Error** | `Max connection attempts exceeded - dial tcp [IP]:587: i/o timeout` |
| **Root Cause** | Railway enforces egress firewall rules blocking all outbound SMTP on ports 25, 465, and 587 |
| **Fix** | Pivoted to local macOS deployment; Railway deployment retained for cloud infrastructure demonstration |

---

### Challenge 3: macOS Binary Permission Denied

| | |
|---|---|
| **Problem** | `./gophish` returned `zsh: permission denied` |
| **Root Cause** | Downloaded binary lacked execute permissions on macOS by default |
| **Fix** | `sudo chmod +x gophish` — applied execute permissions, Gophish started successfully |

---

### Challenge 4: Gmail Link Stripping

| | |
|---|---|
| **Problem** | Delivered phishing emails showed the CTA as unclickable plain text |
| **Root Cause** | Gmail's security filters detected the Mailgun sandbox domain as suspicious and actively stripped all hyperlinks. Confirmed by the red "This message might be dangerous" banner |
| **Impact** | Click tracking could not be captured. Email delivery metrics were still recorded |
| **Mitigation** | In real environments: custom verified domain + SPF/DKIM/DMARC authentication + warmed sending reputation |

---

### Challenge 5: ngrok Base URL Returns 404

| | |
|---|---|
| **Observation** | Visiting the base ngrok URL returned a 404 error |
| **Explanation** | This is expected behavior. Gophish's phishing server only serves content when a valid `?rid=` tracking parameter is appended. The 404 confirmed the tunnel was correctly forwarding requests to Gophish |

---

## 🔍 Key Findings

| Finding | Result | Detail |
|---------|--------|--------|
| Cloud Deployment | ✅ Success | Gophish deployed to Railway via GitHub CI/CD |
| SMTP Egress (Railway) | ⚠️ Blocked | Railway blocks all outbound SMTP — documented and mitigated |
| Local Execution | ✅ Success | Gophish ran on macOS after chmod +x fix |
| Email Delivery | ✅ Success | Phishing emails delivered via Mailgun SMTP relay |
| Spam Detection | ✅ Triggered | Gmail correctly classified simulation as dangerous |
| Link Sanitization | ✅ Triggered | Gmail stripped tracking links — email security controls demonstrated |
| Landing Page | ✅ Success | LinkedIn login cloned and configured for credential capture |
| Campaign Tracking | ✅ Partial | Email sent status captured; click tracking blocked by Gmail sanitization |

---

## 💼 Skills Demonstrated

- **Cloud Deployment** — Deployed Go application to Railway via GitHub CI/CD with environment variable management
- **Infrastructure Troubleshooting** — Identified SMTP egress restrictions through systematic testing; implemented hybrid solution
- **Version Control** — Forked repository, edited config files, committed changes via GitHub web editor
- **SMTP & Email Relay** — Configured Mailgun sandbox SMTP with credential management
- **Phishing Simulation** — End-to-end campaign setup: HTML templates, cloned landing pages, target groups, tracking
- **Network Tunneling** — Configured ngrok reverse proxy to expose local phish server publicly
- **CLI & System Administration** — Binary permissions, process execution, log analysis on macOS
- **Security Analysis** — Interpreted spam filter behavior, link sanitization, and email authentication gaps
- **Security Awareness Testing** — Core competency for SOC Analyst, GRC Analyst, and red team roles

---

## 📂 Repository Structure

```
gophish-phishing-simulation/
│
├── README.md
├── config/
│   └── config.json.example
└── screenshots/
    ├── 01_github_gophish_repo.png
    ├── 02_forked_repo.png
    ├── 03_railway_dashboard.png
    ├── 04_railway_no_repos.png
    ├── 05_railway_repo_list.png
    ├── 06_railway_tos.png
    ├── 07_railway_initializing.png
    ├── 08_railway_online.png
    ├── 09_railway_networking.png
    ├── 10_railway_generate_domain.png
    ├── 11_railway_domain_generated.png
    ├── 12_railway_port_variable.png
    ├── 13_config_json_before.png
    ├── 14_config_json_after.png
    ├── 15_railway_deployment_success.png
    ├── 16_railway_deploy_logs.png
    ├── 18_gophish_login_railway.png
    ├── 19_gophish_reset_password.png
    ├── 20_test_gmail_created.png
    ├── 21_mailgun_smtp_credentials.png
    ├── 22_mailgun_smtp_saved.png
    ├── 23_gophish_sending_profile.png
    ├── 24_smtp_timeout_error.png
    ├── 26_first_email_received.png
    ├── 27_users_groups.png
    ├── 28_landing_page_import.png
    ├── 29_landing_page_configured.png
    ├── 33_gophish_binary_download.png
    ├── 35_email_template_html.png
    ├── 36_campaign_setup.png
    ├── 37_styled_phishing_email.png
    ├── 38_ngrok_running.png
    ├── 39_ngrok_active.png
    ├── 40_ngrok_requests.png
    ├── 41_linkedin_landing_page.png
    └── 42_campaign_dashboard.png
```

---

## ⚖️ Ethical Disclaimer

> This project was conducted **strictly in a controlled lab environment** for educational and research purposes only. All simulated phishing emails were sent exclusively to email accounts owned and controlled by the researcher. No unauthorized individuals were targeted at any point. 

---

## 📚 References

- [Gophish Documentation](https://docs.getgophish.com)
- [Alphasec — Phishing Attack Simulation with Gophish](https://alphasec.io/phishing-attack-simulation-with-gophish/)
- [Railway Documentation](https://docs.railway.app)
- [Mailgun SMTP Documentation](https://documentation.mailgun.com)
- [ngrok Documentation](https://ngrok.com/docs)

---

*Project completed: March 25, 2026 | Johnbosco Ibeneme*
