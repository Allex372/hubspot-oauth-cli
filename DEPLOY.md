# Deploying this project

## Before you start (HubSpot)

- **Disable old GitHub integration** (if you used it before): In HubSpot go to **Settings → Tools → Developer Projects** and turn off the built-in GitHub integration. Using our workflow or the official HubSpot project action avoids conflicts.

## CI/CD (GitHub Actions)

**Don’t commit `hubspot.config.yml`** — it would contain your token. HubSpot recommends using **environment variables** and **`hs project upload --use-env`** so the CLI reads auth from `HUBSPOT_ACCOUNT_ID` and `HUBSPOT_PERSONAL_ACCESS_KEY`. In practice, with current CLI versions, `--use-env` often fails (“No config file found” or “refresh token was malformed” when using a PAK). So this repo **builds `hubspot.config.yml` in the workflow** from GitHub secrets and then runs `hs project upload` (no `--use-env`). The file exists only in the runner and is never committed.

**Secrets to set** (GitHub → Settings → Secrets and variables → Actions):

- `HUBSPOT_ACCESS_TOKEN` – your HubSpot Personal Access Key  
- `PORTAL_ID` – your HubSpot account ID (use `HUBSPOT_ACCOUNT_ID` as the value if you prefer; the workflow uses it as account ID)  

### If you see "Failed to fetch schemas"

The HubSpot CLI fetches CRM schemas during upload. That can fail if your PAK doesn’t have the right scope. The scope **`crm.schemas.custom.read`** is required for that step but **is not available for all accounts** in the PAK scope list (e.g. it may be limited by plan or app type).

**Options:**

1. **Deploy from your machine**  
   - Run `hs account auth` and complete auth with your PAK.  
   - From the repo root: `hs project upload`.  
   - Use this for releases if CI keeps failing with "Failed to fetch schemas".

2. **Ask HubSpot**  
   - In [HubSpot Community](https://community.hubspot.com) or via support, ask whether:  
     - `crm.schemas.custom.read` can be enabled for your account/PAK, or  
     - there is a way to skip schema fetch for app-only projects (no custom objects).

3. **If the scope appears for your account**  
   - In HubSpot: **Development → Keys → Personal Access Key** → edit your key.  
   - Add **`crm.schemas.custom.read`** (or the scope HubSpot indicates for “fetch schemas”).  
   - Create a new PAK if you can’t add scopes to the existing one, then update the `HUBSPOT_ACCESS_TOKEN` secret in GitHub.

## Local deploy (no CI)

```bash
hs account auth
cd /path/to/hubspot-oauth-cli
hs project upload
```

Use the same PAK (with the scopes you use in the app) so upload and schema fetch succeed.

## Official HubSpot Project Action (alternative)

You can also use the [official HubSpot project action](https://github.com/HubSpot/hubspot-project-actions) with `account_id` and `personal_access_key` inputs (secrets: e.g. `PORTAL_ID` → account_id, `HUBSPOT_ACCESS_TOKEN` → personal_access_key). If you see **"refresh token was malformed"**, the action’s `--use-env` path doesn’t work with PAK in current CLI versions; keep using the manual workflow in this repo instead.
