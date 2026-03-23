---
title: "GitLab"
date: 0001-01-01
summary: "GitLab Connector #  Register GitLab Connector #  curl -XPOST &#34;http://localhost:9000/connector/&#34; -d &#39;{ &#34;name&#34;: &#34;GitLab Connector&#34;, &#34;description&#34;: &#34;Fetch repositories, issues, merge requests, wikis, and snippets from GitLab.&#34;, &#34;icon&#34;: &#34;/assets/icons/connector/gitlab/icon.png&#34;, &#34;category&#34;: &#34;website&#34;, &#34;tags&#34;: [ &#34;git&#34;, &#34;code&#34;, &#34;vcs&#34;, &#34;website&#34; ], &#34;url&#34;: &#34;http://coco.rs/connectors/gitlab&#34;, &#34;assets&#34;: { &#34;icons&#34;: { &#34;default&#34;: &#34;/assets/icons/connector/gitlab/icon.png&#34;, &#34;repository&#34;: &#34;/assets/icons/connector/gitlab/repository.png&#34;, &#34;issue&#34;: &#34;/assets/icons/connector/gitlab/issue.png&#34;, &#34;merge_request&#34;: &#34;/assets/icons/connector/gitlab/merge_request.png&#34;, &#34;wiki&#34;: &#34;/assets/icons/connector/gitlab/wiki.png&#34;, &#34;snippet&#34;: &#34;/assets/icons/connector/gitlab/snippet.png&#34; } }, &#34;processor&#34;: { &#34;enabled&#34;: true, &#34;name&#34;: &#34;gitlab&#34; } }&#39;  Use gitlab as a unique identifier, as it is a builtin connector."
---

# GitLab Connector

## Register GitLab Connector

```shell
curl -XPOST "http://localhost:9000/connector/" -d '{
  "name": "GitLab Connector",
  "description": "Fetch repositories, issues, merge requests, wikis, and snippets from GitLab.",
  "icon": "/assets/icons/connector/gitlab/icon.png",
  "category": "website",
  "tags": [
    "git",
    "code",
    "vcs",
    "website"
  ],
  "url": "http://coco.rs/connectors/gitlab",
  "assets": {
    "icons": {
      "default": "/assets/icons/connector/gitlab/icon.png",
      "repository": "/assets/icons/connector/gitlab/repository.png",
      "issue": "/assets/icons/connector/gitlab/issue.png",
      "merge_request": "/assets/icons/connector/gitlab/merge_request.png",
      "wiki": "/assets/icons/connector/gitlab/wiki.png",
      "snippet": "/assets/icons/connector/gitlab/snippet.png"
    }
  },
  "processor": {
    "enabled": true,
    "name": "gitlab"
  }
}'
```

> Use `gitlab` as a unique identifier, as it is a builtin connector.

## Use the GitLab Connector

The GitLab Connector allows you to index repositories, issues, merge requests, wikis, and snippets from your GitLab account or group.

### Configure GitLab Datasource

To configure your GitLab connection, you'll need to provide a Personal Access Token (PAT) and specify the scope of the data you wish to index.

`base_url`: (Optional) The base URL of your self-hosted GitLab instance (e.g., `https://gitlab.example.com`). If left empty, it will default to `https://gitlab.com`.

`token`: A GitLab Personal Access Token (PAT) with at least the `api` scope. This is required for authentication.

`owner`: The username or group name that owns the repositories you want to index (e.g., `infinilabs`).

`repos`: (Optional) A list of specific repository names to index. If left empty, the connector will index all repositories belonging to the `owner` that the `token` has access to.

`index_issues`: (Optional) A boolean (`true` or `false`) to enable indexing of issues. Defaults to `true`.

`index_merge_requests`: (Optional) A boolean (`true` or `false`) to enable indexing of merge requests. Defaults to `true`.

`index_wikis`: (Optional) A boolean (`true` or `false`) to enable indexing of wikis. Defaults to `true`.

`index_snippets`: (Optional) A boolean (`true` or `false`) to enable indexing of snippets. Defaults to `true`.

### Datasource Configuration

Each datasource has its own sync configuration and GitLab settings:

```shell
curl -H 'Content-Type: application/json' -XPOST "http://localhost:9000/datasource/" -d '{
    "name": "My Organization Repos",
    "type": "connector",
    "enabled": true,
    "connector": {
        "id": "gitlab",
        "config": {
            "base_url": "https://gitlab.com",
            "token": "YourPersonalAccessToken",
            "owner": "infinilabs",
            "repos": [
                "coco-server",
                "console"
            ],
            "index_issues": true,
            "index_merge_requests": true,
            "index_wikis": true,
            "index_snippets": true
        }
    },
    "sync": {
        "enabled": true,
        "interval": "30s"
    }
}'
```

## Supported Config Parameters for GitLab Connector

Below are the configuration parameters supported by the GitLab Connector:

### Datasource Config Parameters

| **Field**               | **Type**   | **Description**                                                                                          |
|-------------------------|------------|----------------------------------------------------------------------------------------------------------|
| `base_url`              | `string`   | Optional. The base URL of your self-hosted GitLab instance.                                              |
| `token`                 | `string`   | Your GitLab Personal Access Token (PAT) with `api` scope (required).                                     |
| `owner`                 | `string`   | The username or group name to scan (required).                                                            |
| `repos`                 | `[]string` | Optional. A list of repository names to index. If empty, all repositories for the owner will be indexed. |
| `index_issues`          | `boolean`  | Optional. Whether to index issues. Defaults to `true`.                                                   |
| `index_merge_requests`  | `boolean`  | Optional. Whether to index merge requests. Defaults to `true`.                                           |
| `index_wikis`           | `boolean`  | Optional. Whether to index wikis. Defaults to `true`.                                                    |
| `index_snippets`        | `boolean`  | Optional. Whether to index snippets. Defaults to `true`.                                                 |
| `sync.enabled`          | `boolean`  | Enable/disable syncing for this datasource.                                                              |
| `sync.interval`         | `string`   | Sync interval for this datasource (e.g., "30s", "5m", "1h").                                             |

