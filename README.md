# MedWave Backend Documentation

This public repository publishes API documentation (OpenAPI/Swagger) from the private [medwave-backend-rust](https://github.com/Med-Wave/medwave-backend-rust) repository to GitHub Pages.

## 🌐 Live Documentation

The API documentation is available at: [https://med-wave.github.io/medwave-backend-docs/](https://med-wave.github.io/medwave-backend-docs/)

## 🔄 How It Works

1. The private repository's CI/CD pipeline pushes the OpenAPI specification directly to this repository's `main` branch
2. When changes are pushed, the `deploy-pages.yml` workflow automatically deploys the documentation to GitHub Pages using Swagger UI

## 🔧 Setup Instructions

### Prerequisites

1. **GitHub Token for Private Repo**: Create a Personal Access Token (PAT) with `repo` scope to allow the private repository to push to this repository
   - Go to GitHub Settings → Developer settings → Personal access tokens → Tokens (classic)
   - Generate new token with `repo` scope
   - Add it as a repository secret named `DOCS_REPO_TOKEN` in the **private repository**

2. **GitHub Pages**: Enable GitHub Pages for this repository
   - Go to Repository Settings → Pages
   - Source: GitHub Actions

### Configuration in Private Repository

Add the following step to your private repository's CI/CD workflow to push the OpenAPI specification to this repository:

```yaml
- name: Push Swagger to docs repository
  run: |
    git config --global user.name "github-actions[bot]"
    git config --global user.email "github-actions[bot]@users.noreply.github.com"
    
    # Clone the docs repository
    git clone https://x-access-token:${{ secrets.DOCS_REPO_TOKEN }}@github.com/Med-Wave/medwave-backend-docs.git docs-repo
    cd docs-repo
    
    # Copy the OpenAPI file (adjust the source path as needed)
    cp ../openapi.json ./openapi.json
    
    # Commit and push if there are changes
    git add openapi.json
    if git diff --staged --quiet; then
      echo "No changes to OpenAPI file"
    else
      git commit -m "Update OpenAPI documentation [skip ci]"
      git push origin main
    fi
```

**Note**: Adjust the source path (`../openapi.json`) based on where your OpenAPI file is located in the private repository.

## 📁 Repository Structure

```
.
├── .github/
│   └── workflows/
│       └── deploy-pages.yml     # Deploys to GitHub Pages
├── index.html                   # Swagger UI frontend
├── openapi.json                 # API specification (pushed from private repo)
├── LICENSE
└── README.md
```

## 🔒 Security

- **No Source Code Exposure**: Only the OpenAPI specification file is pushed from the private repository
- **Token Security**: The access token is stored as a GitHub secret in the private repository
- **Public Access**: The published documentation is publicly accessible via GitHub Pages

## 📝 License

See [LICENSE](LICENSE) file for details.
