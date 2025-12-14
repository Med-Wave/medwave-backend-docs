# Setup Guide

This guide explains how to set up the automated Swagger documentation publishing system.

## Step 1: Create a Personal Access Token (PAT)

1. Go to GitHub Settings: https://github.com/settings/tokens
2. Click "Generate new token (classic)"
3. Give it a descriptive name: `medwave-docs-push-access`
4. Select the following scopes:
   - `repo` (Full control of private repositories)
5. Click "Generate token"
6. **Important**: Copy the token immediately - you won't be able to see it again

## Step 2: Add Token as Repository Secret in Private Repo

1. Go to the **private repository's** settings: https://github.com/Med-Wave/medwave-backend-rust/settings/secrets/actions
2. Click "New repository secret"
3. Name: `DOCS_REPO_TOKEN`
4. Value: Paste the PAT you created in Step 1
5. Click "Add secret"

## Step 3: Enable GitHub Pages in This Repository

1. Go to this repository's settings: https://github.com/Med-Wave/medwave-backend-docs/settings/pages
2. Under "Source", select "GitHub Actions"
3. Save the changes

## Step 4: Add Workflow Step to Private Repository

Add the following step to your private repository's CI/CD workflow (e.g., in `.github/workflows/ci.yml`):

```yaml
- name: Push Swagger to docs repository
  if: success() && github.ref == 'refs/heads/main'
  run: |
    git config --global user.name "github-actions[bot]"
    git config --global user.email "github-actions[bot]@users.noreply.github.com"
    
    # Clone the docs repository
    git clone https://x-access-token:${{ secrets.DOCS_REPO_TOKEN }}@github.com/Med-Wave/medwave-backend-docs.git docs-repo
    cd docs-repo
    
    # Copy the OpenAPI file (adjust the source path as needed)
    cp ../openapi.json ./openapi.json
    # OR if your file is in a different location:
    # cp ../docs/swagger.json ./openapi.json
    
    # Commit and push if there are changes
    git add openapi.json
    if git diff --staged --quiet; then
      echo "No changes to OpenAPI file"
    else
      git commit -m "Update OpenAPI documentation [skip ci]"
      git push origin main
    fi
```

**Important Notes**:
- Adjust the source path based on where your OpenAPI file is located in the private repository
- The `[skip ci]` in the commit message prevents triggering unnecessary workflows in the docs repository
- The `if: success() && github.ref == 'refs/heads/main'` condition ensures this only runs on successful main branch builds

## Step 5: Test the Setup

1. Make a change to your OpenAPI specification in the private repository
2. Commit and push to the `main` branch
3. Check the Actions tab in the private repository to verify the workflow runs successfully
4. Check this repository to verify the `openapi.json` was updated
5. Wait a few minutes for GitHub Pages to deploy
6. Visit https://med-wave.github.io/medwave-backend-docs/ to see the updated documentation

## Troubleshooting

### Workflow fails with authentication error

Make sure:
1. The PAT has the correct scopes (`repo`)
2. The PAT is not expired
3. The secret is named exactly `DOCS_REPO_TOKEN` in the **private repository**
4. The docs repository name is correct in the git clone URL: `Med-Wave/medwave-backend-docs`

### OpenAPI file not found

Check that the source path in the workflow step matches the actual location of your OpenAPI file in the private repository. Common locations:
- `openapi.json` (root of repository)
- `swagger.json` (root of repository)
- `docs/openapi.json`
- `target/openapi.json` (if generated during build)

### GitHub Pages not deploying

1. Verify GitHub Pages is enabled in this repository's settings
2. Check that the source is set to "GitHub Actions"
3. Wait a few minutes for the deployment to complete
4. Check the Actions tab in this repository for deployment status
5. Look for the "Deploy to GitHub Pages" workflow run

### Documentation not showing API endpoints

This is expected initially. The placeholder `openapi.json` contains no endpoints. Once the private repository's workflow runs successfully and pushes the real OpenAPI file, the documentation will display all endpoints.

## Maintenance

### Updating Swagger UI Version

Edit `index.html` in this repository and change the version in the CDN links:

```html
<link rel="stylesheet" type="text/css" href="https://unpkg.com/swagger-ui-dist@5.10.5/swagger-ui.css">
<script src="https://unpkg.com/swagger-ui-dist@5.10.5/swagger-ui-bundle.js"></script>
<script src="https://unpkg.com/swagger-ui-dist@5.10.5/swagger-ui-standalone-preset.js"></script>
```

### Changing When Documentation Updates

The documentation updates whenever the private repository's workflow pushes changes to this repository. To control when this happens, modify the `if` condition in the workflow step:

```yaml
# Only on main branch
if: success() && github.ref == 'refs/heads/main'

# On main and develop branches
if: success() && (github.ref == 'refs/heads/main' || github.ref == 'refs/heads/develop')

# On tagged releases only
if: success() && startsWith(github.ref, 'refs/tags/')
```
