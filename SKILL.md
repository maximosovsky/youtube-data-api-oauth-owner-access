---
name: youtube-data-api-oauth-owner-access
description: Use when connecting owner-authorized YouTube channels to Hermes/subagents via official YouTube Data API OAuth without exposing passwords, cookies, tokens, or client secrets in chat.
version: 1.0.0
author: Hermes Agent
license: MIT
metadata:
  hermes:
    tags: [youtube, oauth, google-cloud, owner-access, data-api, no-secrets]
    related_skills: [youtube-owner-studio-ops]
---

# YouTube Data API OAuth Owner Access

## Overview

This skill sets up owner-authorized YouTube channel access for Hermes or channel-specific subagents using the official **YouTube Data API v3** and **OAuth 2.0 Desktop app** clients.

The owner performs all Google login and consent steps. The agent may inspect local OAuth client JSON structure, start a local OAuth callback server, save local token files, and verify channel ownership with `channels.list(mine=true)`, but must never request or print passwords, 2FA codes, raw cookies, raw OAuth tokens, refresh tokens, or client secret values.

## When to Use

Use this skill when the user wants to:

- connect their own YouTube channels to Hermes or subagents;
- authorize access to private/unlisted/public video inventory;
- verify which channel a Google account/Brand Account controls;
- create a reusable no-secret YouTube OAuth setup workflow;
- enable future channel operations through YouTube Data API OAuth.

Do not use this for public-only YouTube transcript work; use public YouTube tooling or `youtube-content` instead.

## Non-Negotiable Secret Rules

1. Never ask the user to paste passwords, 2FA codes, recovery codes, raw cookies, session tokens, OAuth authorization codes, access tokens, refresh tokens, or full client-secret JSON contents into chat.
2. If the user provides a local JSON path, inspect only safe metadata: file exists, OAuth client type, project id, presence of client id/secret, redirect URIs. Do **not** print `client_secret` or tokens.
3. OAuth tokens may be saved locally only in a project-approved token folder. Do not print token JSON.
4. Cookie-based access is not part of this workflow. It requires a separate explicit approval and should normally be avoided.
5. Deletion is never a default permission. Deleting videos/playlists/comments/assets requires a separate explicit user request for the exact target.

## Google Cloud Setup Checklist

The owner does these steps in Google Cloud Console:

1. Open Google Cloud Console.
2. Select the project for the OAuth application.
3. Enable **YouTube Data API v3**:
   ```text
   https://console.cloud.google.com/apis/api/youtube.googleapis.com/overview
   ```
   If needed, use the exact project URL shown by Google, for example:
   ```text
   https://console.cloud.google.com/apis/api/youtube.googleapis.com/overview?authuser=1
   ```
4. Configure **Google Auth Platform**.
5. Open **Audience**.
6. Keep the app in **Testing** unless the user intentionally publishes/verifies it.
7. Add every account that will authorize as **Test users**.
8. Open **APIs & Services → Credentials**.
9. In **OAuth 2.0 Client IDs**, create or find a client with type **Desktop app**.
10. Click **Download JSON** / the download icon.
11. Save the JSON locally, for example:
    ```text
    C:\100_star\Chanells YT\<channel>-client_secret_<client-id>.json
    ```

Do not create an API key for owner access. Do not use a Service Account for normal YouTube channel owner operations.

## Local OAuth Flow

Use a local callback server and a Desktop app OAuth client.

Recommended scopes:

```text
https://www.googleapis.com/auth/youtube
https://www.googleapis.com/auth/yt-analytics.readonly
```

Read-only inventory can use:

```text
https://www.googleapis.com/auth/youtube.readonly
```

For Brand Accounts, force account/channel selection:

```python
prompt="select_account consent"
```

The user should open the generated `AUTHORIZE_URL`, log in themselves, and choose the correct Brand Account/channel if Google shows a chooser. The agent waits for the local callback and then verifies `channels.list(mine=true)`.

## Safe Client JSON Inspection

Given a path to a downloaded client JSON, verify safely:

```bash
python - <<'PY'
import json, pathlib
p = pathlib.Path(r'C:/path/to/client_secret.json')
print('exists=', p.exists())
if p.exists():
    data = json.loads(p.read_text(encoding='utf-8'))
    kind = 'installed' if 'installed' in data else 'web' if 'web' in data else 'unknown'
    obj = data.get(kind, {})
    print('type=', kind)
    print('project_id=', obj.get('project_id', ''))
    print('has_client_id=', bool(obj.get('client_id')))
    print('has_client_secret=', bool(obj.get('client_secret')))
    print('redirect_uris=', obj.get('redirect_uris', []))
PY
```

Never print the actual `client_secret` value.

## OAuth Script Template

Adjust the three paths/labels, then run in the project folder:

```bash
python - <<'PY'
import pathlib
from google_auth_oauthlib.flow import InstalledAppFlow
from googleapiclient.discovery import build

client_secret = pathlib.Path(r'C:/100_star/Chanells YT/<client-secret-file>.json')
token_path = pathlib.Path(r'C:/100_star/Chanells YT/oauth_tokens/<channel>_<account>_token.json')
token_path.parent.mkdir(parents=True, exist_ok=True)

scopes = [
    'https://www.googleapis.com/auth/youtube',
    'https://www.googleapis.com/auth/yt-analytics.readonly',
]

flow = InstalledAppFlow.from_client_secrets_file(str(client_secret), scopes=scopes)
creds = flow.run_local_server(
    host='localhost',
    port=0,
    open_browser=False,
    prompt='select_account consent',
    authorization_prompt_message='AUTHORIZE_URL: {url}',
    success_message='OAuth complete. You can close this tab and return to Hermes.'
)

token_path.write_text(creds.to_json(), encoding='utf-8')
youtube = build('youtube', 'v3', credentials=creds)
resp = youtube.channels().list(part='id,snippet,statistics,status', mine=True).execute()
items = resp.get('items', [])
print('TOKEN_SAVED:', token_path)
print('CHANNELS_COUNT:', len(items))
for it in items:
    sn = it.get('snippet', {})
    st = it.get('statistics', {})
    status = it.get('status', {})
    print('CHANNEL_ID:', it.get('id', ''))
    print('TITLE:', sn.get('title', ''))
    print('CUSTOM_URL:', sn.get('customUrl', ''))
    print('PUBLISHED_AT:', sn.get('publishedAt', ''))
    print('SUBSCRIBERS:', st.get('subscriberCount', ''))
    print('VIDEOS:', st.get('videoCount', ''))
    print('PRIVACY:', status.get('privacyStatus', ''))
PY
```

If fixed ports are easier for the user, use `host='localhost'` or `host='127.0.0.1'` and an unused port such as `60221`. If the user gets `ERR_CONNECTION_REFUSED`, the local OAuth server has already exited or the wrong/stale URL was opened; start a new OAuth run and give only the fresh URL.

## Verification Commands

After token creation, verify the owner context:

```bash
python - <<'PY'
import pathlib
from google.oauth2.credentials import Credentials
from googleapiclient.discovery import build

TOKEN = pathlib.Path(r'C:/100_star/Chanells YT/oauth_tokens/<token-file>.json')
SCOPES = ['https://www.googleapis.com/auth/youtube', 'https://www.googleapis.com/auth/yt-analytics.readonly']
creds = Credentials.from_authorized_user_file(str(TOKEN), SCOPES)
yt = build('youtube', 'v3', credentials=creds)
resp = yt.channels().list(part='id,snippet,statistics,status', mine=True).execute()
print('CHANNELS_COUNT:', len(resp.get('items', [])))
for it in resp.get('items', []):
    sn = it.get('snippet', {})
    st = it.get('statistics', {})
    print(it.get('id'), sn.get('title'), sn.get('customUrl'), st.get('videoCount'))
PY
```

For complete owner-visible video counts including private/unlisted, use `search.list(forMine=True, type='video')` and then `videos.list(status)` for privacy breakdown:

```bash
python - <<'PY'
import pathlib
from collections import Counter
from google.oauth2.credentials import Credentials
from googleapiclient.discovery import build

TOKEN = pathlib.Path(r'C:/100_star/Chanells YT/oauth_tokens/<token-file>.json')
SCOPES = ['https://www.googleapis.com/auth/youtube', 'https://www.googleapis.com/auth/yt-analytics.readonly']
creds = Credentials.from_authorized_user_file(str(TOKEN), SCOPES)
yt = build('youtube', 'v3', credentials=creds)
ids, page, total = [], None, None
while True:
    resp = yt.search().list(part='id', forMine=True, type='video', maxResults=50, pageToken=page).execute()
    total = resp.get('pageInfo', {}).get('totalResults', total)
    ids += [it['id']['videoId'] for it in resp.get('items', []) if it.get('id', {}).get('videoId')]
    page = resp.get('nextPageToken')
    if not page:
        break
privacy = Counter()
for i in range(0, len(ids), 50):
    resp = yt.videos().list(part='status', id=','.join(ids[i:i+50]), maxResults=50).execute()
    for it in resp.get('items', []):
        privacy[it.get('status', {}).get('privacyStatus', 'unknown')] += 1
print('search_totalResults:', total)
print('listed_ids:', len(ids))
print('privacy:', dict(privacy))
PY
```

## Handling Common Errors

### 403: app not verified / access_denied

The OAuth app is in Testing and the account is not a test user.

Fix:

```text
Google Auth Platform → Audience → Test users → Add users → <account email> → Save
```

Then restart OAuth with a fresh URL.

### 403: YouTube Data API v3 disabled

The project has not enabled `youtube.googleapis.com`.

Fix:

```text
Google Cloud Console → APIs & Services → Library/Enabled APIs → YouTube Data API v3 → Enable
```

If CLI access has permission, this can be attempted:

```bash
gcloud services enable youtube.googleapis.com --project=<project-id>
```

If this returns `PERMISSION_DENIED`, the owner must enable it in Console.

### `MismatchingStateError`

The user opened a stale callback URL or mixed old OAuth tabs.

Fix:

1. Kill the old background OAuth process if still running.
2. Close old `localhost` / `127.0.0.1` tabs.
3. Start a fresh OAuth run.
4. Provide only the newest `AUTHORIZE_URL`.

### `ERR_CONNECTION_REFUSED` on localhost

The local server is not listening anymore, the wrong host/port was used, or the user opened a stale URL.

Fix: restart OAuth and provide the fresh URL. Consider switching between `localhost` and `127.0.0.1`.

### `CHANNELS_COUNT: 0`

OAuth succeeded, but the selected Google identity has no channel in API context.

Likely causes:

- wrong Google account selected;
- selected the personal Google identity instead of the Brand Account/channel;
- account can open Studio but OAuth consent did not choose the Brand Account.

Fix:

1. Confirm `https://studio.youtube.com/` shows the target channel for that Google account.
2. Restart OAuth with `prompt='select_account consent'`.
3. In the Google/YouTube chooser, select the target Brand Account/channel, not just the email identity.
4. Verify again with `channels.list(mine=true)`.

## Channel/Subagent Operating Policy

For each connected channel, keep a local token file named clearly, for example:

```text
oauth_tokens/maximosovsky_educamp_tv_token.json
oauth_tokens/7-years_world_problems_rimmile_token.json
oauth_tokens/smdbar_leskdpl_token.json
oauth_tokens/zhanzhuang_7713mobile_token.json
```

A channel subagent should receive only:

- target channel handle/title;
- expected account email;
- token file path;
- allowed operations;
- deletion prohibition;
- verification requirement.

Do not pass token JSON contents into prompts.

Default allowed operations, if the user grants full owner work:

- inventory public/private/unlisted videos;
- read and edit metadata;
- edit titles/descriptions/tags/playlists/thumbnails/visibility/scheduling;
- prepare migration/export/import manifests;
- read analytics when scope allows.

Forbidden unless separately approved:

- deleting videos;
- deleting playlists;
- deleting comments;
- destructive channel/account actions.

## Verification Checklist

- [ ] YouTube Data API v3 enabled for the OAuth project.
- [ ] Google Auth Platform configured.
- [ ] Audience / Test users includes the account being authorized while app is in Testing.
- [ ] OAuth client type is Desktop app.
- [ ] Client JSON is local; no secrets printed.
- [ ] OAuth token saved locally; token contents not printed.
- [ ] `channels.list(mine=true)` returns the expected channel id/title/custom URL.
- [ ] For Brand Accounts, the target channel was selected during consent.
- [ ] For owner-visible inventory, `search.list(forMine=True)` count/privacy breakdown was run when needed.
- [ ] Deletion remains blocked unless separately and explicitly approved for a specific target.
