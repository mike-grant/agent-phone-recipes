# agent-phone recipes

Shared, replayable Android automation recipes for
[agent-phone](https://github.com/mike-grant/agent-phone). A recipe is a YAML file
of steps (tap this element, type into that field, expect this screen) that
`agent-phone run` replays over adb with no LLM in the loop, healing itself with
`--heal` when the app's UI drifts.

```bash
agent-phone recipe search tiktok
agent-phone recipe install tiktok/carousel-draft
agent-phone run tiktok/carousel-draft --param images=~/slides --param caption="hello"
```

This repo is the default recipe source, so it needs no configuration.

## Recipes

| Name | App | What it does |
|---|---|---|
| `tiktok/carousel-draft` | TikTok (`com.zhiliaoapp.musically`) | Builds a photo carousel from a folder of images plus a caption and **saves it to Drafts**. It never publishes: its `forbid` list blocks Post / Your Story. |

## Contributing a recipe

1. Record it against a real device:
   ```bash
   agent-phone record start myapp/do-thing --app com.example.app
   agent-phone tap @e6 --intent "Open the create screen"
   ...
   agent-phone record stop
   ```
   or let the agent loop write it: `agent-phone agent "..." --record myapp/do-thing --yes`.
2. Replace anything personal with params: file paths, captions, account names.
   Declare `type: file` or `type: dir` on path params.
3. Add store metadata:
   ```yaml
   tested:
     date: '2026-10-20'
     app_version: '37.5.4'     # adb shell dumpsys package <pkg> | grep versionName
   min_engine: 0.3.0
   ```
4. Copy it to `recipes/<namespace>/<name>.yaml` (`name:` must match the path),
   then run:
   ```bash
   agent-phone recipe index .          # rewrites index.json
   agent-phone recipe index . --check  # what CI runs
   ```
5. Open a PR. A heal that fixes a recipe after an app update makes a great PR:
   `--heal --save-heals` writes the patched copy to `~/.agent-phone/recipes/`.

### Rules (enforced by `agent-phone recipe validate --strict`, install, and CI)

- `push` / `push_images` values must be a bare `{{param}}`: no hard-coded local paths.
- `push` destinations stay under `/sdcard/`.
- `app` steps only launch the recipe's declared `app` (or packages listed in `apps:`).
- Every step has an `intent` (healing uses it to re-find a moved element).
- Recipes that post, send, buy or delete must default to the safe variant
  (drafts, previews) and use `forbid` to block the irreversible button. Publish
  variants need a clearly different name.

## Your own private store

Any GitHub repo (or local directory) with this layout works as a source:

```bash
export AGENT_PHONE_RECIPE_SOURCES=github:my-org/private-recipes,github:mike-grant/agent-phone-recipes
export AGENT_PHONE_GITHUB_TOKEN=...   # read access to the private repo
agent-phone recipe sync --yes
```

## Disclaimer

These recipes automate third-party apps. You're responsible for complying with
each app's terms of service. Many platforms restrict automated use, and accounts
can be limited for it. The recipes are provided as-is under the MIT licence.
