# macos-dotfiles

Dotfile management for macOS using [chezmoi](https://www.chezmoi.io/). Supports **work** and **personal** machine profiles. System setup and Homebrew packages live in the companion repo [praivio/macos-config](https://github.com/praivio/macos-config).

---

## What's managed

| File | Deploys to | Notes |
|---|---|---|
| `dot_zshrc.tmpl` | `~/.zshrc` | Work/personal blocks via template |
| `dot_zsh_aliases.tmpl` | `~/.zsh_aliases` | Common + work-specific aliases |
| `dot_gitconfig.tmpl` | `~/.gitconfig` | Name/email from chezmoi prompts |
| `dot_gitconfig_work` | `~/.gitconfig.work` | Work git identity (included conditionally) |
| `dot_antigenrc` | `~/.antigenrc` | zsh plugin list for antigen |
| `dot_p10k.zsh` | `~/.p10k.zsh` | Powerlevel10k prompt config |
| `dot_gitignore` | `~/.gitignore` | Global gitignore |

---

## First-time setup

If you're setting up a new Mac, use [praivio/macos-config](https://github.com/praivio/macos-config) — its `bootstrap.sh` handles everything including these dotfiles.

To apply dotfiles only (on an existing machine with chezmoi + Homebrew already installed):

```bash
chezmoi init --apply git@github.com:praivio/macos-dotfiles.git
```

You'll be prompted for:
- **Machine type:** `work` or `personal`
- **Full name:** used in `~/.gitconfig`
- **Email address:** used in `~/.gitconfig`

These are stored in `~/.config/chezmoi/chezmoi.toml` and are never committed.

---

## Day-to-day usage

```bash
# Edit a dotfile in your $EDITOR
chezmoi edit ~/.zshrc

# Preview what would change
chezmoi diff

# Apply changes to $HOME
chezmoi apply

# Pull latest from GitHub and apply
chezmoi update

# Shortcuts (defined in ~/.zshrc after first apply)
cze ~/.zshrc    # edit
czd             # diff
cza             # apply
czu             # update
```

---

## Secrets and 1Password integration {#secrets}

**Never commit secrets to this repo.** The `.zshrc` has placeholders showing the pattern.

### Setup

1. Install the 1Password CLI (included in `Brewfile.common`): `brew install 1password-cli`
2. Open **1Password → Settings → Developer → Use the SSH agent** (recommended)
3. Sign in: `eval "$(op signin)"`

### Using secrets in dotfile templates

```bash
# In any .tmpl file:
export MY_TOKEN="{{ onepasswordRead "op://Personal/My Token/credential" }}"
```

The vault path format is `op://vault-name/item-name/field-name`.

### AWS credentials (work profile)

In `~/.zshrc`, there's a commented block in the work section. To activate it:

1. Create a **"AWS CLI"** item in your **Work** vault in 1Password
2. Add fields: `access_key_id` and `secret_access_key`
3. Uncomment the lines in `dot_zshrc.tmpl`:
   ```bash
   export AWS_ACCESS_KEY_ID="$(op read "op://Work/AWS CLI/access_key_id")"
   export AWS_SECRET_ACCESS_KEY="$(op read "op://Work/AWS CLI/secret_access_key")"
   ```
4. Run `chezmoi apply`

### Local overrides (not tracked)

For anything too machine-specific for a template, use `~/.zshrc.local`. It's sourced at the end of `~/.zshrc` but is intentionally not tracked by chezmoi:

```bash
echo 'export MY_SECRET=foo' >> ~/.zshrc.local
```

---

## Work git identity

On work machines, `~/.gitconfig` conditionally includes `~/.gitconfig.work` for repos under `~/work/`. Edit it with:

```bash
chezmoi edit ~/.gitconfig.work
```

Uncomment and fill in the `name` and `email` fields with your work identity.

---

## Adding a new dotfile

```bash
# Track an existing file
chezmoi add ~/.ssh/config

# Track and make it a template (for profile-specific content)
chezmoi add --template ~/.ssh/config

chezmoi diff     # check the result
chezmoi apply    # apply
```

Then commit from the chezmoi source directory:

```bash
chezmoi cd
git add -A
git commit -m "feat: add ssh config"
git push
```

---

## References

- [chezmoi documentation](https://www.chezmoi.io/user-guide/)
- [chezmoi 1Password integration](https://www.chezmoi.io/user-guide/password-managers/1password/)
- [praivio/macos-config](https://github.com/praivio/macos-config) — companion repo for brew + system setup
