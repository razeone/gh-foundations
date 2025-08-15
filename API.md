# GitHub API Documentation

The GitHub API enables developers to interact programmatically with GitHub. It supports REST and GraphQL endpoints.

---

## REST API
- **Base URL:** `https://api.github.com/`
- **Authentication:** Personal Access Token (PAT) via headers.
- **Common Endpoints:**
  - `/users/{username}`: Get user info
  - `/repos/{owner}/{repo}`: Get repo info
  - `/issues`: Manage issues
  - `/pulls`: Manage pull requests
- **Docs:** [GitHub REST API Docs](https://docs.github.com/en/rest)

## GraphQL API
- **Base URL:** `https://api.github.com/graphql`
- **Authentication:** PAT required.
- **Advantages:** Flexible queries, fetch multiple resources in one request.
- **Docs:** [GitHub GraphQL API Docs](https://docs.github.com/en/graphql)

## Authentication
- Use a Personal Access Token (PAT) with appropriate scopes.
- Example header: `Authorization: Bearer <TOKEN>`

## Rate Limits
- **REST:** 5,000 requests/hour/user.
- **GraphQL:** Based on query complexity.

## Example: List Repositories (REST)
```bash
curl -H "Authorization: Bearer <TOKEN>" https://api.github.com/user/repos
```

---

> For more, see [GitHub API Docs](https://docs.github.com/en/rest)
