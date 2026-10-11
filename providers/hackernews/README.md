# hackernews — the example connector

The smallest complete connector, and the one to try first. It wraps Hacker
News's **official public API** ([HackerNews/API](https://github.com/HackerNews/API)),
so it works with no login, no extension, and no reverse engineering. It also
shows the shape of a connector:

| File | Shows |
| --- | --- |
| `openapi.yaml` | path params (`item`, `user`), `security: []` public operations, a hidden helper (`topIds`), and a `foreach` workflow (`top`: ids → stories) |
| `auth.manifest.json` | the capture manifest every connector needs (here: HN's `user` cookie; none of the operations use it) |
| `skills/read/` | an agent skill, installable with `mastro skills add hackernews` |

```bash
mastro hackernews top --limit 5               # installs itself on first use
mastro hackernews item --id 8863 --json
mastro validate providers/hackernews
```

Real connectors capture a browser session and replay it against an app's own
(unofficial) API — see
[docs/AUTHORING.md](https://github.com/mastro-hq/mastro-connect/blob/main/docs/AUTHORING.md),
or let an agent build one with the `mastro-author` skill.
