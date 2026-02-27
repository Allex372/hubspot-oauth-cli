# Deploying this project

## CI/CD (GitHub Actions)

The workflow in `.github/workflows/deploy.yml` runs on push to `main`. It uses a **Personal Access Key** (PAK) from GitHub secrets:

- `HUBSPOT_ACCESS_TOKEN` – your HubSpot Personal Access Key  
- `PORTAL_ID` – your HubSpot account (portal) ID  

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
