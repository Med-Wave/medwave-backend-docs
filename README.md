# MedWave Backend Documentation

This public repository automatically publishes API documentation (OpenAPI/Swagger) from the private [medwave-backend-rust](https://github.com/Med-Wave/medwave-backend-rust) repository to GitHub Pages.

## 🌐 Live Documentation

The API documentation is available at: [https://med-wave.github.io/medwave-backend-docs/](https://med-wave.github.io/medwave-backend-docs/)

## 🔄 How It Works

This repository uses GitHub Actions workflows to:

1. **Fetch Swagger Documentation**: The `fetch-swagger.yml` workflow pulls the OpenAPI specification from the private repository without exposing any source code
2. **Deploy to GitHub Pages**: The `deploy-pages.yml` workflow publishes the documentation as a static site with Swagger UI

## 🔧 Setup Instructions

### Prerequisites

1. **GitHub Token**: Create a Personal Access Token (PAT) with `repo` scope to access the private repository
   - Go to GitHub Settings → Developer settings → Personal access tokens → Tokens (classic)
   - Generate new token with `repo` scope
   - Add it as a repository secret named `PRIVATE_REPO_TOKEN`

2. **GitHub Pages**: Enable GitHub Pages for this repository
   - Go to Repository Settings → Pages
   - Source: GitHub Actions

### Configuration

The workflow automatically searches for the OpenAPI file in these locations within the private repo:
- `openapi.json`
- `swagger.json`
- `docs/openapi.json`
- `docs/swagger.json`

If your OpenAPI file is in a different location, update the `fetch-swagger.yml` workflow.

## 🚀 Triggering Updates

The documentation can be updated in three ways:

1. **Manual Trigger**: Run the "Fetch Swagger from Private Repo" workflow manually from the Actions tab
2. **Scheduled**: Automatically runs daily at midnight UTC
3. **Repository Dispatch**: Trigger from the private repository when the API changes

### Triggering from Private Repo

Add this to your private repository's CI/CD workflow:

```yaml
- name: Trigger docs update
  run: |
    curl -X POST \
      -H "Accept: application/vnd.github.v3+json" \
      -H "Authorization: token ${{ secrets.DOCS_REPO_TOKEN }}" \
      https://api.github.com/repos/Med-Wave/medwave-backend-docs/dispatches \
      -d '{"event_type":"swagger-updated"}'
```

## 📁 Repository Structure

```
.
├── .github/
│   └── workflows/
│       ├── fetch-swagger.yml    # Fetches OpenAPI spec from private repo
│       └── deploy-pages.yml     # Deploys to GitHub Pages
├── index.html                   # Swagger UI frontend
├── openapi.json                 # API specification (auto-updated)
├── LICENSE
└── README.md
```

## 🔒 Security

- **No Source Code Exposure**: Only the OpenAPI specification file is copied, not the source code
- **Token Security**: The private repository access token is stored as a GitHub secret
- **Public Access**: The published documentation is publicly accessible via GitHub Pages

## 📝 License

See [LICENSE](LICENSE) file for details.
