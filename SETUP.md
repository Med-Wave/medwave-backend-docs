# Setup Guide

This guide explains how to set up the automated Swagger documentation publishing system.

## Step 1: Create a Personal Access Token (PAT)

1. Go to GitHub Settings: https://github.com/settings/tokens
2. Click "Generate new token (classic)"
3. Give it a descriptive name: `medwave-docs-access`
4. Select the following scopes:
   - `repo` (Full control of private repositories)
5. Click "Generate token"
6. **Important**: Copy the token immediately - you won't be able to see it again

## Step 2: Add Token as Repository Secret

1. Go to this repository's settings: https://github.com/Med-Wave/medwave-backend-docs/settings/secrets/actions
2. Click "New repository secret"
3. Name: `PRIVATE_REPO_TOKEN`
4. Value: Paste the PAT you created in Step 1
5. Click "Add secret"

## Step 3: Enable GitHub Pages

1. Go to repository settings: https://github.com/Med-Wave/medwave-backend-docs/settings/pages
2. Under "Source", select "GitHub Actions"
3. Save the changes

## Step 4: Test the Workflow

1. Go to the Actions tab: https://github.com/Med-Wave/medwave-backend-docs/actions
2. Select "Fetch Swagger from Private Repo" workflow
3. Click "Run workflow" button
4. Select branch: `main`
5. Click "Run workflow"

The workflow should:
- Fetch the OpenAPI file from the private repository
- Commit it to this repository
- Trigger the GitHub Pages deployment
- Make the documentation available at: https://med-wave.github.io/medwave-backend-docs/

## Step 5: (Optional) Trigger from Private Repo

To automatically update the documentation when the API changes, add this to your private repository's CI workflow:

```yaml
- name: Trigger documentation update
  if: success()
  run: |
    curl -X POST \
      -H "Accept: application/vnd.github.v3+json" \
      -H "Authorization: token ${{ secrets.DOCS_REPO_TOKEN }}" \
      https://api.github.com/repos/Med-Wave/medwave-backend-docs/dispatches \
      -d '{"event_type":"swagger-updated"}'
```

You'll need to create another PAT with `repo` scope and add it as a secret named `DOCS_REPO_TOKEN` in the private repository.

## Troubleshooting

### Workflow fails with "No OpenAPI/Swagger file found"

The workflow searches for:
- `openapi.json`
- `swagger.json`
- `docs/openapi.json`
- `docs/swagger.json`

If your file is in a different location, edit `.github/workflows/fetch-swagger.yml` and update the search paths.

### Workflow fails with authentication error

Make sure:
1. The PAT has the correct scopes (`repo`)
2. The PAT is not expired
3. The secret is named exactly `PRIVATE_REPO_TOKEN`
4. The private repository name is correct: `Med-Wave/medwave-backend-rust`

### GitHub Pages not deploying

1. Verify GitHub Pages is enabled in repository settings
2. Check that the source is set to "GitHub Actions"
3. Wait a few minutes for the deployment to complete
4. Check the Actions tab for deployment status

### Documentation not showing API endpoints

This is expected initially. The placeholder `openapi.json` contains no endpoints. Once you run the "Fetch Swagger from Private Repo" workflow successfully, the real API documentation will be displayed.

## Maintenance

### Updating Swagger UI Version

Edit `index.html` and change the version in the CDN links:

```html
<link rel="stylesheet" type="text/css" href="https://unpkg.com/swagger-ui-dist@5.10.5/swagger-ui.css">
<script src="https://unpkg.com/swagger-ui-dist@5.10.5/swagger-ui-bundle.js"></script>
<script src="https://unpkg.com/swagger-ui-dist@5.10.5/swagger-ui-standalone-preset.js"></script>
```

### Changing Update Schedule

Edit `.github/workflows/fetch-swagger.yml` and modify the cron schedule:

```yaml
schedule:
  - cron: '0 0 * * *'  # Daily at midnight UTC
```

Common schedules:
- `0 */6 * * *` - Every 6 hours
- `0 0 * * 1` - Every Monday at midnight
- `0 0 1 * *` - First day of each month
