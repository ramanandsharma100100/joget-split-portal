# Spilt Portal (Joget app `split_portal`)

Source for the **Spilt Portal** app on Joget DX (`https://joget2.excitel.in`), stored as Joget's own export files.

| File | What it holds |
|---|---|
| `appDefinition.xml` | Forms, datalists, userview, environment variables, plugin defaults, messages |
| `package.xpdl` | Workflow process definition |

A Joget `.jwa` export is a zip of these two files, so this repo holds everything an app export contains.

**Not stored here:** records in the `app_fd_*` database tables, uploaded files, custom plugin `.jar` files, users and groups, and the app's Git settings.

---

## Pushing changes from Joget to this repo

Do this each time you change the app in Joget and want the change in Git.

### 1. Export the app from Joget

1. Log in to `https://joget2.excitel.in/jw/web/console/app/split_portal/2/builders`.
2. Open **Settings** and click **Export**. The browser downloads `APP_split_portal-2-<date>.jwa` (the exact name can differ).

If you changed a version other than `2`, use that version's URL.

### 2. Replace the files in the repo

```bash
cd ~/joget-split-portal
git pull                                   # get the latest from GitHub first
unzip -o ~/Downloads/APP_split_portal-*.jwa appDefinition.xml package.xpdl -d .
```

If more than one export is in `~/Downloads`, use the full name of the newest file instead of the `*`.

### 3. Review what changed

```bash
git status
git diff --stat
```

`appDefinition.xml` is a large file, so `git diff` can be long. Check that you see the forms and lists you meant to change.

### 4. Commit and push

```bash
git add appDefinition.xml package.xpdl
git commit -m "Update <form/list/process name>: <what changed>"
git push
```

Write the commit message about the change itself, for example:
`Update kyc_verification_split_portal: add PAN number field`.

---

## Restoring or moving the app

Zip the two files back into a `.jwa` and import it:

```bash
cd ~/joget-split-portal
zip split_portal.jwa appDefinition.xml package.xpdl
```

Then in Joget go to **All Apps → Import App** and choose `split_portal.jwa`.

To import an older version, check out that commit first:

```bash
git log --oneline                          # find the commit
git checkout <commit-id> -- appDefinition.xml package.xpdl
zip split_portal.jwa appDefinition.xml package.xpdl
git checkout main -- appDefinition.xml package.xpdl   # put the files back afterwards
```

---

## Joget's automatic Git sync

Joget can push to this repo by itself whenever the app is saved. It's configured in the app's Git settings (**Settings → Git** in the App Composer). As of 2026-11-04 it was turned on but had never pushed anything.

For it to work:

1. **Token:** create a fine-grained GitHub token with access to this repo only and the **Contents: Read and write** permission. Put it in the Git password field, and set an expiry date you'll remember to renew.
2. **Network:** the Joget server must be able to reach `github.com` over HTTPS. Company firewalls or proxies can block this.
3. **Exclude setting:** the exclude field takes a list of files to leave out of commits. Leave it empty unless you need it.

If it still doesn't push, save a form and check the server log at `wflow/logs/joget.log` in the Joget install folder for lines containing `git`:

| Error contains | Likely cause |
|---|---|
| `not authorized`, `401`, `403` | Token is wrong, expired, or can't write to the repo |
| `connect timed out`, `UnknownHost` | Server can't reach GitHub |
| `rejected`, `non-fast-forward` | GitHub has commits Joget doesn't; pull or re-sync first |

When auto sync works, keep pushing manual exports the same way. Run `git pull` before each manual push so the two don't conflict.

---

## Security

This repository is **public**, so anyone can read these files.

- Never commit tokens, passwords, database connection strings or `.jwa` files that contain them. Search before every commit:
  ```bash
  grep -niE "github_pat_|ghp_|password\"?\s*[:=]\s*\"[^\"]+|jdbc:" appDefinition.xml package.xpdl
  ```
  Field names such as `c_procare_password` are fine. A hit with an actual value is not.
- Keep the GitHub token only in Joget's Git settings, never in this repo.
- To limit who can see the app's design, switch the repo to private under **Settings → General → Danger Zone**.
