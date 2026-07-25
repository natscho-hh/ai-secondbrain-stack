# Backup Strategy

Git already protects your vault. It does not protect what `.gitignore` excludes: API keys, media, agent memory, attachments. This guide is the reasoning an agent should follow to design a backup for **this** user, with **their** existing means. It is a recipe, not a product. No script is shipped, because the right building blocks depend on what the user already has.

## 1. Two scenarios, two different answers

People say "backup" and mean one of two things. They need different designs, and mixing them produces an over-engineered system that protects neither well.

| Scenario | What it needs | Size | Time budget |
|---|---|---|---|
| **"I have to work from another device today"** | secrets, configs, agent memory | megabytes | minutes |
| **"My machine is dead, give me everything back"** | media, archives, working data | hundreds of megabytes or more | hours or days |

The first one is about **portability**. The second is about **recovery**. Design them separately, then let the first one carry the credentials for the second. That way the whole chain hangs on two things the user always has with them.

## 2. What actually needs backing up

Apply one rule to every candidate:

> **Can the user recreate this in under four hours, alone, without asking anyone?**

**Yes** means it does not get backed up. It gets a note describing how to recreate it. **No** means it goes into the backup.

Typical outcome:

| Include | Exclude |
|---|---|
| `.env` files and API keys | `node_modules`, `.venv`, build output, caches |
| Agent memory directories | Agent plugin and skill directories (reinstallable) |
| Attachments excluded by `.gitignore` | Anything already tracked in git |
| Media, renders, masters | Cloud-service OAuth credential files |
| Local databases, exports | Agent session transcripts |

Two of these deserve an explicit warning.

**Never back up OAuth credential files** (the token caches agents write on login). Re-authenticating takes thirty seconds. In a backup they are pure risk, and they will be invalid by the time anyone restores them.

**Treat agent session transcripts as a hazard, not an asset.** They routinely contain credentials the user pasted into a conversation. Scan them before deciding, see section 6.

## 3. The sync-folder trap

This is the single most common way a homemade backup dies silently.

Deduplicating backup tools (restic, kopia, borg) **rewrite existing files** during maintenance. A sync client (iCloud, OneDrive, Dropbox, Google Drive) reacts by creating conflict copies, and with files-on-demand it replaces local data with placeholders. The repository ends up corrupted, and nothing announces it. The user finds out during the restore, which is the worst possible moment.

Two safe patterns:

- **Talk to the storage API, not the synced folder.** `rclone` has backends for every major consumer cloud, and restic can use rclone as its transport. No local folder, no sync client, no conflicts. If the provider also syncs that folder to the machine, exclude it in the client settings.
- **Write immutable archives instead.** One encrypted archive file per week, written once and never touched again. Sync handles that perfectly. You lose deduplication and fine-grained versioning, which is an acceptable trade for a secondary copy.

## 4. Choosing targets: cheapest thing that already exists

Do not recommend a new paid service before checking what the user already pays for. Ask, then map their answer onto these:

| What they already have | Good for | Cost | Notes |
|---|---|---|---|
| GitHub account (private repo) | portability axis, encrypted secrets | free | Small text-shaped data only. Not a media store: no retention policy, no deduplication, uploads cannot resume |
| Consumer cloud (OneDrive, iCloud, Drive, Dropbox) | full backup via rclone, or immutable archives | already paid | Never as a synced folder for a deduplicating repo, see section 3 |
| Password manager with file storage | the encryption key itself, nothing more | already paid | A key is a few kilobytes. Do not put the backup there |
| NAS or a second local disk | full backup | already owned | Fast, but same building, same flood, same burglar. Needs an off-site partner |
| Object storage (B2, S3, R2, Scaleway) | full backup | a few cents per month for typical vault sizes | The clean answer when nothing above fits |
| Web hosting included in a plan | usually **not suitable** | already paid | Most shared-hosting terms forbid use as pure file or backup storage. Check before recommending |

Two rules that override cost:

**Match the restore channel to the constraint.** Ask what the user can connect on a replacement device. If the honest answer is "only my GitHub account", then a backup that requires a different login is useless in exactly the situation it was built for. Put the bootstrap on the channel that always works, and let it carry the credentials for everything else.

**Do not put the backup on the same account as the original.** If the vault lives in a GitHub repo, a GitHub-only backup dies with that account. The second copy belongs somewhere else.

## 5. Encryption and keys

Encrypt client-side, before anything leaves the machine. Then the storage provider is a dumb container and their terms, their staff, and their breaches stop being your problem.

- `age` for archives, restic and kopia bring their own encryption.
- Encrypt to **more than one recipient** where the tool allows it, so a single lost key is not fatal.
- Store the private key in a **password manager plus one offline copy** (printed, or on paper in a drawer). A password manager alone is a single point of failure.
- The bootstrap script that installs the tools must stay **unencrypted**. If the thing that decrypts the archive lives inside the archive, there is no way in.

## 6. Secret hygiene comes first

Before the first backup run, scan everything that will be backed up for plaintext credentials: `.env` files are fine, that is their job, but tokens sitting in transcripts, notes, or logs are not.

If the scan finds something: **rotate the credential first, then redact the files, then back up.** Backing up first only copies the problem to more places.

Make this a permanent gate, not a one-off. A backup job that runs unattended should refuse to run when the scanner finds plaintext credentials outside their allowed locations.

## 7. A backup nobody restored is a rumour

Two mechanisms, both required.

**A restore drill, quarterly.** Restore into an empty directory and compare the result byte for byte against the originals, not by eye. Write down the outcome even when it passes. A drill that cannot fail is not a drill.

**A watchdog that runs somewhere else.** Scheduled jobs die quietly: a shell exits zero despite an error, a laptop sleeps through the trigger, a stale lock blocks every following run. The machine cannot report its own silence. Have each job write a timestamp somewhere the user's other device can see it (a file in the vault repo works), and have something outside the machine complain when a timestamp gets too old. A scheduled CI job that opens an issue costs nothing and needs no key, because it only reads dates.

## 8. Write down the recovery path

Produce a `Backup & Recovery` note in the vault containing the numbered steps for a stranger, in order, with the real commands. During a real recovery nobody reconstructs a design from memory. State explicitly what is **not** in the backup and how to get it back another way.

## 9. Reference design

One arrangement that satisfies everything above, as a starting point rather than a prescription:

1. **Bootstrap axis:** a private git repo holding the `.env` files, agent memory, and configs, encrypted with `age`, next to an unencrypted install script and the recovery runbook. Key in the password manager and on paper.
2. **Recovery axis:** restic through rclone to whichever consumer cloud the user already pays for, credentials taken from axis 1.
3. **Third copy:** one immutable encrypted archive per week at a *different* provider, with rotation.
4. **Watchdog:** a scheduled CI job checking the freshness of the timestamps each axis writes.

Skipping step 3 is reasonable for a small vault. Skipping step 4 is how people find out in October that the backup stopped in June.
