---
layout: post
title: A pattern for creating SSH keys for your projects
date: 2026-01-09 13:44 -0600
---

# Create an SSH key (naming matters)

## Naming scheme

**Pattern**

```
id_ed25519_<your-username>_<your-machine>_<project>
```

**Why**

* **username** → identifies the human
* **machine** → limits blast radius if a device is lost
* **project** → scopes access intentionally

**Rule**
👉 Replace **every `<…>` placeholder** with *your own values*.
Do **not** copy examples literally.

**Example (DO NOT COPY AS-IS)**

```
id_ed25519_bob_bobmachine_payments
```

---

## 1. Generate the key (replace placeholders)

```bash
ssh-keygen -t ed25519 -a 100 \
  -f ~/.ssh/id_ed25519_<your-username>_<your-machine>_<project> \
  -C "<your-username>@<your-machine> | project=<project>"
```

Example (illustration only):

```bash
ssh-keygen -t ed25519 -a 100 \
  -f ~/.ssh/id_ed25519_alice_macbook_platform \
  -C "alice@macbook | project=platform"
```

Set a passphrase when prompted.

---

## 2. Load the key

```bash
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_ed25519_<your-username>_<your-machine>_<project>
```

---

## 3. Add SSH config entry

Edit `~/.ssh/config`:

```sshconfig
Host <alias>
  HostName <host-or-ip>
  User <remote-user>
  IdentityFile ~/.ssh/id_ed25519_<your-username>_<your-machine>_<project>
  IdentitiesOnly yes
```

Example (illustration only):

```sshconfig
Host platform-box
  HostName 203.0.113.10
  User ubuntu
  IdentityFile ~/.ssh/id_ed25519_alice_macbook_platform
  IdentitiesOnly yes
```

---

## 4. Share the public key

```bash
cat ~/.ssh/id_ed25519_<your-username>_<your-machine>_<project>.pub
```

Send the output to the server admin.

---

## 5. Connect

```bash
ssh <alias>
```

---

### One-line reminder for your teammates

**If you see `<something>` in a command, you must replace it with your own value.**
