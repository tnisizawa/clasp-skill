# Clasp (GAS Project Management) — English Translation

> **Note**: This is an English translation for human readers. Agents load `SKILL.md` (Japanese). When `SKILL.md` changes, update this file in the same commit.

Google Apps Script management skill via CLI.

## Reference Index

Read the relevant file for your use case.

| What you want to do | Reference |
|---|---|
| **Push code** | This file — "push flow" section (below) |
| First-time setup / install / `clasp login` | `references/setup.md` (Japanese) |
| Connect to a GAS project (initial connection / Case A/B decision) | `references/init.md` (Japanese) |
| Pull changes from the web editor (`clasp pull` / daily sync) | `references/pull.md` (Japanese) |
| Push distribution build for multi-environment (dev/release) | `references/deploy-dist.md` (Japanese) |
| Deploy / rollback / version revert (library / web app) | `references/deploy.md` (Japanese) |
| GAS web app iframe issues / image display | `references/webapp.md` (Japanese) |
| Fetch execution logs with `clasp logs` | `references/logs.md` (Japanese) |

---

## Push Flow (Most Common)

### Prerequisites
- `.clasp.json` must exist
- `clasp` must be installed

### 1. Check / Create `appsscript.json`

Create if missing. If it exists, check for missing or extra scopes.

```json
{
  "timeZone": "Asia/Tokyo",
  "dependencies": {},
  "exceptionLogging": "STACKDRIVER",
  "runtimeVersion": "V8",
  "oauthScopes": [
    "https://www.googleapis.com/auth/spreadsheets"
  ]
}
```

**Add only the scopes your project needs.** Prefix each scope with `https://www.googleapis.com/auth/`.

**Common permissions:**
| Permission | Purpose |
|------|------|
| `gmail.readonly` | Read Gmail |
| `gmail.send` | Send email |
| `drive` | Google Drive operations |
| `documents` | Google Docs operations |
| `spreadsheets` | Spreadsheet operations |
| `forms` | Create / edit Google Forms |
| `calendar` | Google Calendar operations |
| `script.scriptapp` | Set triggers |
| `script.external_request` | UrlFetchApp (external API calls) |
| `script.container.ui` | UI dialogs / sidebars |

### 2. Check `.clasp.json`

```bash
ls .clasp*.json
```

**If multiple files exist (e.g., `.clasp.dev.json`, `.clasp.prod.json`):**
- Ask the user which environment to push to
- Temporarily copy the selected config file to `.clasp.json`, push, then restore it
- If permanently switching `.clasp.json`, confirm the intent with the user first

```bash
CLASP_BAK=".clasp.json.bak.$(date +%Y%m%d%H%M%S)"
node -e "const fs=require('fs');fs.existsSync('.clasp.json')&&fs.copyFileSync('.clasp.json',process.argv[1])" "$CLASP_BAK"
printf '%s\n' "$CLASP_BAK" > .clasp.json.bak-path
node -e "const fs=require('fs');fs.copyFileSync(process.argv[1],'.clasp.json')" ".clasp.<env>.json"
```

### 2.5. Check `.claspignore` (Exclude Files from Push)

`.claspignore` specifies files to exclude from push (similar syntax to `.gitignore`).

**Default behavior when `.claspignore` is absent (clasp 3.x)**: All files are ignored initially; only `appsscript.json` / `.gs` / `.js` / `.ts` / `.html` files under `rootDir` are included. Additionally, `.git/**` and `node_modules/**` are always excluded.
→ In other words, **plain JS/HTML projects usually work fine without it**, but create one if you want explicit control over what gets pushed.

Typical example (exclude docs and tests from push):

```
*.md
node_modules/**
tests/**
```

To preview push targets before pushing, run `clasp show-file-status` (old alias: `clasp status`).

### 3. Run `clasp push`

**Important: In Git Bash, clasp output cannot be captured directly, so always run it via Node.js.**

```bash
run_clasp_push() {
  local push_status=0
  local restore_status=0

  node -e "const{execSync}=require('child_process');try{execSync('clasp push --force',{stdio:'inherit'})}catch(e){process.exit(e.status||1)}" || push_status=$?

  # Restore .clasp.json only if it was temporarily switched for multi-environment
  node -e "const fs=require('fs');let bak=process.argv[1];const marker='.clasp.json.bak-path';if(!bak&&fs.existsSync(marker)){bak=fs.readFileSync(marker,'utf8').trim()}if(bak&&fs.existsSync(bak)){fs.copyFileSync(bak,'.clasp.json');fs.unlinkSync(bak)}if(fs.existsSync(marker)){fs.unlinkSync(marker)}" "${CLASP_BAK:-}" || restore_status=$?

  [ "$restore_status" -eq 0 ] || { echo "failed to restore .clasp.json" >&2; return "$restore_status"; }
  [ "$push_status" -eq 0 ] || { echo "clasp push failed" >&2; return "$push_status"; }
}

run_clasp_push
```

### Verification Points
- Confirm the message "Pushed X files." is displayed
- If this message does not appear, the push did not complete — re-run
- On error, review the message and attempt to fix

### OAuth Scope Insufficient Error

If you add code that uses new GAS APIs (FormApp, DriveApp, etc.) without adding the corresponding scope to `oauthScopes`, a runtime error will occur.

```
Exception: You do not have permission to call FormApp.create. Required permissions: https://www.googleapis.com/auth/forms
```

**Fix:**
1. Add the missing scope to `oauthScopes` in `appsscript.json` (see the permissions table above)
2. Push with `clasp push` to apply
3. When the user runs a function from the GAS editor or spreadsheet, a re-authorization dialog will appear

### Reminder (Bound Scripts Only)

After a successful `clasp push`, if the destination is a bound script (a project tied to a spreadsheet), display a reminder to rename the spreadsheet with a date suffix.

- Development build: "📝 Please rename the development spreadsheet to '{project name}_dev_YYYYMMDD'!"
- Distribution build: For workflows requiring version identification or protection against accidental overwrite of the bound spreadsheet, add a dated-rename reminder echo at the comment position inside the `deploy-dist.sh` template

---

## Deployment Type Decision (Required)

**When asked to deploy, always determine the type before choosing a procedure.**

| Type | Identifying Hint | Update Procedure |
|------|-----------|---------|
| **Library** | No `doGet`/`doPost` | Update via CLI with `clasp deploy -i <id>` → `references/deploy.md` (Japanese) |
| **Web App** | Has `doGet`/`doPost` | Existing deployments can be updated via CLI. Must verify the existing URL works after update → `references/deploy.md` (Japanese) |

**Initial web app creation, execution user settings, and access permission changes must be done in the GAS UI.** To reflect new code in an existing URL, use `clasp update-deployment <id>` or `clasp deploy -i <id>`, but always open the public URL to verify after updating.
