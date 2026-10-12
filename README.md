# mastro connectors

Community connectors for [mastro-connect](https://github.com/mastro-hq/mastro-connect),
the framework for driving web apps from the terminal through your own browser
session. Each connector is data, not code: `providers/<id>/` holds a capture
manifest, an OpenAPI 3.1 spec, a README, and agent skills.

| Connector | Login | What it covers | Last verified |
| --- | --- | --- | --- |
| [`hackernews`](providers/hackernews) | no | front page, items, users — the example connector, over HN's official API | 2026-10-11 |
| [`reservecalifornia`](providers/reservecalifornia) | optional | California State Parks campsite search and availability (no login); your own reservations (login) | 2026-10-11 |
| [`depop`](providers/depop) | yes | search listings, your profile, list your own items for sale | 2026-06-08 |

## Install

Nothing to install up front: the first time you call a connector from this
repo, mastro fetches it (pinned to a commit) and runs the command.

```bash
npx mastro-connect hackernews top --limit 5       # just Node; mastro brings its own runtime
npx mastro-connect reservecalifornia parks "big sur" --json
npx mastro-connect setup                          # pick connectors, extension, agent skills
```

`mastro providers update` re-fetches everything you've added. Connectors that
need a login also need the capture extension once: `mastro extension install`.

## Build one

Have your agent do it: `mastro skills add mastro-author`, then ask it to make a
connector for an app you use. By hand:
[AUTHORING.md](https://github.com/mastro-hq/mastro-connect/blob/main/docs/AUTHORING.md).
Check your work with `mastro validate providers/<id>` (CI runs it on every PR).

A good connector PR:

- acts on the **user's own account**, at a human pace (`rate_limit` set);
- is **read-first**; anything that changes state is a workflow with
  `--dry-run`, and its skill requires user confirmation;
- never solves CAPTCHAs or bot challenges — if the app needs a real browser,
  use `via_browser` so requests run in the user's own tab;
- contains **no HARs, cookies, tokens, or real people's data** (fixtures are
  synthetic);
- says in its README that it's unofficial and not affiliated with the service.

## Disclaimer & takedowns

These connectors are **unofficial**. None is affiliated with or endorsed by the
services it talks to; names are used only to identify them. They replay the
requests each app's own web frontend makes, from the user's machine, with the
user's own session. Using them may conflict with a service's Terms of Service
— you are responsible for how you use them.

**If you operate one of these services and want its connector removed, open an
issue on this repo** and it will be taken down promptly, no argument needed.

## License

[MIT](LICENSE)
