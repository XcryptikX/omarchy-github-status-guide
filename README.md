# GitHub Status Plugin for Omarchy — Installation Guide

A step-by-step tutorial for installing and configuring the **GitHub Status** bar widget on Omarchy Linux. This widget shows your GitHub notifications, open PRs, review requests, and repositories directly in your status bar.

---

## Prerequisites

- **Omarchy Linux** (tested on v3.8.x+ with Hyprland)
- An active **GitHub account**
- Internet access (no Tailscale/VPN required)

---

## Step 1 — Install the GitHub CLI (`gh`)

The plugin uses your authenticated `gh` CLI session — no tokens are stored by the plugin itself.

```bash
# Install gh via your package manager
sudo pacman -S github-cli

# Or via mise (if you use it)
mise install gh
```

Verify it works:

```bash
gh --version
```

Expected output: `gh version 2.xx.x (...)`

---

## Step 2 — Authenticate `gh` with GitHub

You have two options:

### Option A — Device code flow (browser-based)

```bash
gh auth login --hostname github.com --web
```

1. The CLI prints a one-time device code and a URL (`https://github.com/login/device`)
2. Open that URL in your browser, enter the code
3. Approve the authorization
4. The CLI confirms: `✓ Logged in to github.com account <your-username>`

**If the CLI times out waiting for the browser callback** (common in terminal-only environments), use Option B instead.

### Option B — Personal Access Token (no browser wait)

1. Go to `https://github.com/settings/tokens`
2. Click **Generate new token → Generate new token (classic)**
3. Name it (e.g. `omarchy-shell`), set an expiration
4. Select these scopes:
   - **`repo`** — read your repos, PRs, issues, notifications
   - **`read:user`** — read your profile/login
   - **`read:org`** — optional, only if you collaborate in orgs
5. Copy the token
6. Run:

```bash
echo "ghp_YOUR_TOKEN_HERE" > /tmp/gh-pat.txt
chmod 600 /tmp/gh-pat.txt
gh auth login --hostname github.com --with-token < /tmp/gh-pat.txt
rm -f /tmp/gh-pat.txt
```

Verify authentication:

```bash
gh auth status
gh api user --jq '.login'
```

Expected: `✓ Logged in to github.com account <your-username>`

---

## Step 3 — Install the GitHub Status Plugin

```bash
omarchy plugin add https://github.com/HalmyLyseas/omarchy-github-status.git --yes
```

This clones the plugin into `~/.config/omarchy/plugins/halmylyseas.github-status/`.

---

## Step 4 — Add the Plugin to Your Bar

Edit `~/.config/omarchy/shell.json`. Add the plugin ID to the `plugins` array and place it in whichever bar section you want (here: **upper-left**).

```json
{
  "version": 1,
  "idle": {
    "screensaver": 150,
    "lock": 300
  },
  "bar": {
    "position": "top",
    "transparent": false,
    "centerAnchor": "omarchy.clock",
    "layout": {
      "left": [
        { "id": "omarchy.menu" },
        { "id": "omarchy.workspaces" },
        { "id": "halmylyseas.github-status" }
      ],
      "center": [
        { "id": "omarchy.indicators" },
        { "id": "omarchy.clock", "format": "dddd HH:mm" }
      ],
      "right": [
        { "id": "omarchy.tray" },
        { "id": "omarchy.audio" },
        { "id": "omarchy.power" }
      ]
    }
  },
  "plugins": [
    { "id": "halmylyseas.github-status" }
  ]
}
```

Key points:
- Add `{ "id": "halmylyseas.github-status" }` to the `plugins` array
- Add `{ "id": "halmylyseas.github-status" }` to whichever bar section you want (`left`, `center`, or `right`)
- The order in the section array controls left-to-right placement

---

## Step 5 — Enable and Restart the Shell

```bash
# Enable the plugin in the left section
omarchy plugin enable halmylyseas.github-status --section left

# Restart the Omarchy shell to pick up the new config
omarchy restart shell
```

The GitHub Status icon should now appear in your bar.

---

## Step 6 — Verify It Works

- **Hover** the GitHub icon in your bar — the tooltip should show something like `GitHub Status — N unread · M PRs` (not `not signed in`)
- **Click** it to open the panel with your notifications, PRs, review requests, and repos
- **Check** the plugin is active:

```bash
omarchy plugin list --json | jq '.[] | select(.id == "halmylyseas.github-status")'
```

Expected: `"enabled": true`

---

## Troubleshooting

| Symptom | Fix |
|---------|-----|
| Tooltip says "not signed in" | Run `gh auth status` — if not logged in, re-run Step 2 |
| Tooltip says "gh CLI not found" | Ensure `gh` is on your PATH: `which gh` should return a path |
| Widget not showing at all | Check `omarchy plugin list --json` — plugin should be `"enabled": true` |
| Widget shows but panel is empty | Wait ~60s for the first data fetch, or click the widget to force a refresh |
| Auth was written to `/root/.config/gh/` | You ran `gh` as root. Copy it: `sudo cp /root/.config/gh/hosts.yml ~/.config/gh/hosts.yml && sudo chown $USER:$USER ~/.config/gh/hosts.yml` |

---

## Optional Customization

You can tweak the plugin's behavior by adding settings to its entry in `shell.json`:

```json
{
  "id": "halmylyseas.github-status",
  "dashboardIntervalSec": 180,
  "notificationsIntervalSec": 60,
  "repoLimit": 10,
  "issuesFilter": "focus"
}
```

| Setting | Default | Description |
|---------|---------|-------------|
| `dashboardIntervalSec` | 180 | How often the dashboard (PRs, repos) refreshes, in seconds |
| `notificationsIntervalSec` | 60 | How often notifications are polled, in seconds |
| `repoLimit` | 10 | Max repos shown in the repositories section |
| `issuesFilter` | `focus` | `focus` = subscribed issues only; `all` = every open issue you authored |

Apply with `omarchy restart shell` after editing.

---

## Security Notes

- **Never share your PAT.** Treat it like a password. Revoke it at `https://github.com/settings/tokens` if it's ever exposed.
- The plugin **never sees your token** — all GitHub access goes through your `gh` CLI session. The plugin runs `gh api` / `gh api graphql` as a subprocess.
- The plugin is **read-only** — it never mutates GitHub data (no POST/PATCH/DELETE).

---

## What You Can Do Now

With `gh` authenticated and the plugin connected:

```bash
# Create a new repo and push your project
gh repo create my-project --public --source . --remote origin
git push -u origin main

# Clone an existing repo
gh repo clone owner/repo

# Create a pull request
gh pr create --title "Fix bug" --body "Description here"

# View your open PRs
gh pr list
```

The GitHub Status widget in your bar gives you a live dashboard of all this activity at a glance.

---

## Links

- **Plugin source:** https://github.com/HalmyLyseas/omarchy-github-status
- **Omarchy docs:** https://omarchy.org/
- **GitHub CLI docs:** https://cli.github.com/manual/
- **This guide's repo:** https://github.com/XcryptikX/omarchy-github-status-guide
