# Cursor desktop, GitHub sign-in, and Git push from the agent

This guide separates three related ideas that people often mix up:

1. **Using Cursor as a desktop app** — install and open the editor.
2. **Signing into Cursor with GitHub** — links your **Cursor account** to GitHub for Cursor product features (and optional integrations).
3. **Authenticating Git for `git push`** — what the **terminal and the agent** use when they run Git; this is **normal Git + GitHub credentials**, not automatic just because you signed into Cursor.

---

## Part 1 — Install and use Cursor as a desktop app

1. Download the **Cursor** installer for your OS from [https://cursor.com](https://cursor.com) (macOS, Windows, or Linux).
2. Install it like any other desktop application, then open **Cursor** from Applications / Start menu / launcher.
3. **Open a folder or clone a repo:** **File → Open Folder…**, or use the welcome screen to clone from GitHub when Git is configured (see Part 3).

Cursor is a fork of the VS Code experience: same familiar layout (sidebar, editor, integrated terminal, command palette).

**Useful shortcuts**

| Action | macOS | Windows / Linux |
|--------|--------|------------------|
| Command palette | `⌘ Shift P` | `Ctrl Shift P` |
| Integrated terminal | `⌃ \`` (Control + backtick) | `Ctrl \`` |
| Settings | `⌘ ,` | `Ctrl ,` |

---

## Part 2 — Sign into Cursor with GitHub (Cursor account)

This step connects **your Cursor identity** to GitHub (OAuth). Exact labels can change between versions; look for **Sign in**, **Log in**, or **Account** on first launch or in settings.

**Typical flow**

1. Open **Cursor Settings** (gear icon or `⌘ ,` / `Ctrl ,`) and find **Account**, **Sign in**, or **Cursor Settings → General** (wording varies by version).
2. Choose **Continue with GitHub** (or equivalent).
3. Complete the flow in the **browser**: approve Cursor’s access to your GitHub account when prompted.

**Why this matters**

- Unlocks Cursor **account** features tied to your subscription and cloud services.
- May enable **GitHub-related integrations** inside Cursor (e.g. browsing PRs, depending on what Cursor exposes in your build).

**Important:** Signing into Cursor with GitHub **does not by itself** configure **Git credentials** on your machine. You still need Part 3 for `git push` from the terminal or from an agent that runs shell commands.

---

## Part 3 — Authenticate Git so pushes work (including from the agent)

When Cursor’s **agent** runs `git push`, it uses the **same Git and credential setup** as the **integrated terminal** on your machine. There is no separate “agent-only GitHub login” in the typical local setup: you configure Git once; then both you and the agent benefit.

Pick **one** primary method below (HTTPS + GitHub CLI is usually the simplest).

### Option A — HTTPS with GitHub CLI (`gh`) (recommended)

1. Install [GitHub CLI](https://cli.github.com/) (`gh`), e.g. `brew install gh` on macOS if you use Homebrew.
2. In Cursor, open the **integrated terminal** (**Terminal → New Terminal**).
3. Run:

   ```bash
   gh auth login
   ```

4. Choose **GitHub.com**, **HTTPS**, and authenticate via **web browser** (device code) or token, as prompted.
5. Wire Git to use `gh` for credentials:

   ```bash
   gh auth setup-git
   ```

6. In your repo, confirm the remote and push:

   ```bash
   cd /path/to/your/repo
   git remote -v
   git push origin main
   ```

After this, **agent-driven pushes** that run `git push` in the same environment should succeed **as long as** that environment can read the credential helper (same user account, non-sandboxed terminal).

### Option B — SSH keys

1. Generate a key (if you do not have one): `ssh-keygen -t ed25519 -C "your_email@example.com"`.
2. Add the **public** key to GitHub: **GitHub → Settings → SSH and GPG keys → New SSH key**.
3. Set your repo remote to SSH:

   ```bash
   git remote set-url origin git@github.com:OWNER/REPO.git
   ```

4. Test: `ssh -T git@github.com`, then `git push`.

Ensure `ssh-agent` is running and has your key loaded in the session Cursor uses.

### Option C — Personal access token (HTTPS, no `gh`)

1. GitHub: **Settings → Developer settings → Personal access tokens** — create a token with **`repo`** scope (for private repos) or minimal scopes for public-only work.
2. On first `git push`, Git prompts for username/password: use your GitHub **username** and the **token** as the password. macOS can store this in Keychain via Git’s **osxkeychain** helper.

Prefer **fine-grained tokens** with least privilege where possible.

---

## Part 4 — If the agent still cannot push

Check the following:

| Symptom | What to try |
|--------|-------------|
| `could not read Username for 'https://github.com'` | Complete **Option A** in the **integrated terminal**, then retry push from the agent or terminal. |
| `Permission denied (publickey)` | Fix **SSH** key loading or switch remote to **HTTPS** + `gh`. |
| Agent terminal “sandbox” or `Operation not permitted` on `.git/config` | Run `git push` yourself once in the **integrated terminal**, or fix file permissions; some automated shells cannot write `.git/config` even when push works. |
| Organization SSO | On GitHub, **authorize** the token or SSH key for the organization (**SSO → Authorize**). |

**Cloud / remote agents:** If you use Cursor features that run agents **outside** your laptop, those environments may need their own secrets (e.g. a PAT stored in the provider’s secret store). Follow Cursor’s current docs for that product; local desktop agents use your machine’s Git setup.

---

## Quick checklist

- [ ] Cursor **desktop** installed and project folder opened.
- [ ] **Cursor account** signed in with GitHub if you use that sign-in method.
- [ ] **`gh auth login`** + **`gh auth setup-git`** *or* **SSH** *or* **PAT** configured for Git.
- [ ] `git push` succeeds from the **integrated terminal** in the same repo.

---

## References (official / primary)

- Cursor: [https://cursor.com](https://cursor.com) — download and product information.
- GitHub CLI: [https://cli.github.com/manual/gh_auth_login](https://cli.github.com/manual/gh_auth_login)
- GitHub: [Connecting with SSH](https://docs.github.com/en/authentication/connecting-to-github-with-ssh) and [HTTPS cloning with PAT](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/creating-a-personal-access-token)

---

*Sample documentation for the Cursor-doc repository. Update menu names if your Cursor version differs.*
