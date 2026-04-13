---
name: parse_github_url
description: Parse a GitHub URL into owner and repo components
parameters:
  - name: url
    required: true
    type: string
    description: A GitHub repository URL like https://github.com/owner/repo
---

# Parse GitHub URL

Take a GitHub URL string and extract the owner and repo name.

## Logic

Strip optional protocol (`http://` or `https://`) and `www.`.
Split on `/` after `github.com/`.
The first segment is the owner; the second is the repo name.
Strip a trailing `.git` suffix or trailing slash from the repo name if present.

## Output format

If the URL parses successfully:
```
owner: {owner}, repo: {repo}
```

If the URL is not a valid GitHub URL:
```
Invalid GitHub URL: {url}
```

## Examples

Input: `https://github.com/pydantic/monty`
Output: `owner: pydantic, repo: monty`

Input: `https://github.com/openai/whisper.git`
Output: `owner: openai, repo: whisper`

Input: `https://example.com/foo/bar`
Output: `Invalid GitHub URL: https://example.com/foo/bar`

## When to use

- User has a GitHub URL and wants to extract owner/repo
- After search_github returns results, to parse the URL of a specific result
