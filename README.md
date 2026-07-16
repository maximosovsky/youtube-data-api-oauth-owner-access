<div align="center">

# 🔐 YouTube Data API OAuth Owner Access

![YouTube](https://img.shields.io/badge/YouTube-Data_API_v3-FF0000?style=for-the-badge&logo=youtube&logoColor=white)
![Google Cloud](https://img.shields.io/badge/Google_Cloud-OAuth_2.0-4285F4?style=for-the-badge&logo=googlecloud&logoColor=white)
![Python](https://img.shields.io/badge/Python-3.11+-3776AB?style=for-the-badge&logo=python&logoColor=white)
![Hermes](https://img.shields.io/badge/Hermes-Skill-111111?style=for-the-badge)
![License](https://img.shields.io/badge/License-MIT-lightgrey?style=for-the-badge)

**A no-secrets Hermes skill for connecting owner-authorized YouTube channels through official YouTube Data API OAuth.**

</div>

> This repository turns a fragile Google OAuth setup flow into a repeatable Hermes skill: enable the right API, configure Google Auth Platform, download a Desktop app OAuth JSON locally, authorize in the browser, and verify the channel with `channels.list(mine=true)` — without exposing secrets in chat.

<div align="center">

<!-- Preview image placeholder: add a real screenshot/GIF here when one exists. -->

<a href="#-quick-start">Quick Start</a> · <a href="#-features">Features</a> · <a href="#-tech-stack">Tech Stack</a> · <a href="#-roadmap">Roadmap</a> · <a href="#-license">License</a>

</div>

---

## 💡 Concept

YouTube owner access should not depend on pasted passwords, cookies, or improvised browser sessions. This skill documents a safe OAuth owner-access path for Hermes and subagents: the owner authenticates directly with Google, Hermes stores only local OAuth artifacts, and every channel connection is verified through the official API.

The workflow is designed for multi-channel operators who need repeatable access to public, unlisted, and private video inventory, metadata work, analytics scopes, and future channel-specific agents — while keeping deletion and secrets outside the default operating path.

---

## ✨ Features

| Feature | Description |
|---------|-------------|
| No-secret OAuth workflow | Uses Google OAuth Desktop app clients without asking for passwords, 2FA codes, cookies, authorization codes, or token JSON in chat. |
| Google Cloud setup checklist | Covers YouTube Data API v3, Google Auth Platform, Audience, Test users, Credentials, Desktop app clients, and Download JSON. |
| Local callback flow | Starts a local OAuth server, gives the owner an authorization URL, saves tokens locally, and verifies the selected channel. |
| Brand Account handling | Uses `prompt="select_account consent"` and explains how to avoid authorizing an empty personal YouTube identity. |
| Owner-context verification | Requires `channels.list(mine=true)` before claiming channel access. |
| Private/unlisted inventory check | Includes a `search.list(forMine=True)` pattern for owner-visible video counts and privacy breakdowns. |
| Error playbook | Documents fixes for app-not-verified 403, disabled API 403, stale OAuth state, localhost refusal, and zero-channel OAuth. |
| Safe subagent policy | Defines allowed channel operations and blocks deletion unless the user separately approves a specific target. |

---

## 🚀 Quick Start

```bash
git clone https://github.com/maximosovsky/youtube-data-api-oauth-owner-access.git
cd youtube-data-api-oauth-owner-access
mkdir -p "$LOCALAPPDATA/hermes/skills/media/youtube-data-api-oauth-owner-access"
cp SKILL.md "$LOCALAPPDATA/hermes/skills/media/youtube-data-api-oauth-owner-access/SKILL.md"
```

Then start a fresh Hermes session and load:

```text
/skill youtube-data-api-oauth-owner-access
```

<details>
<summary>⚙️ Google Cloud prerequisites</summary>

1. Open Google Cloud Console.
2. Select the OAuth project.
3. Enable **YouTube Data API v3**.
4. Configure **Google Auth Platform**.
5. Open **Audience** and add every authorizing account as a **Test user** while the app is in Testing.
6. Open **APIs & Services → Credentials**.
7. Create or find an **OAuth 2.0 Client ID** with type **Desktop app**.
8. Click **Download JSON**.
9. Save the JSON locally; do not paste it into chat.

</details>

<details>
<summary>🔒 Secret handling rules</summary>

| Artifact | Rule |
|----------|------|
| Passwords / 2FA / recovery codes | Never paste into chat. |
| Raw cookies / session tokens | Not used by this workflow. |
| OAuth client JSON | Keep local; inspect only safe metadata. |
| OAuth token JSON | Keep local; never print into prompts or chat. |
| Deletion operations | Blocked until a separate explicit approval names the exact target. |

</details>

---

## 🏗️ Tech Stack

| Layer | Technology |
|-------|------------|
| Agent runtime | Hermes Agent skill system |
| Authorization | Google OAuth 2.0 Desktop app flow |
| API | YouTube Data API v3 and optional YouTube Analytics scope |
| Verification | `channels.list(mine=true)` and `search.list(forMine=True)` |
| Implementation examples | Python, `google-auth-oauthlib`, `google-api-python-client` |
| Repository safety | `.gitignore` blocks local tokens, OAuth JSON, cookies, and `.env` files |

<details>
<summary>📁 Project Structure</summary>

```text
youtube-data-api-oauth-owner-access/
├── .gitignore
├── README.md
├── SKILL.md
├── llms.txt
└── llms-full.txt
```

</details>

---

## 🗺️ Roadmap

- [x] Document official YouTube Data API OAuth owner access.
- [x] Add no-secret handling rules for passwords, 2FA, cookies, client secrets, and tokens.
- [x] Add Google Cloud setup checklist and common error fixes.
- [x] Add owner-context verification with `channels.list(mine=true)`.
- [x] Publish as a standalone private GitHub repository.
- [ ] Add a small helper script that starts OAuth from a redacted channel config file.
- [ ] Add a token health-check script that reports channel identity without printing token contents.
- [ ] Add example subagent prompts for per-channel YouTube operations.

---

## 🤝 Contributing

Fork → `feature/name` → PR.

Keep contributions focused on safer OAuth setup, clearer troubleshooting, and stronger secret hygiene. Do not add real OAuth credentials, token JSON, cookies, channel-private data, or screenshots that expose account details.

---

## 📄 License

Maxim Osovsky. Licensed under [MIT](LICENSE).
