````md
# dkms-kernel-gc (openSUSE Tumbleweed)

## Overview
`dkms-kernel-gc` removes orphaned DKMS build artifacts and leftover kernel module trees that remain after kernel package removal on **openSUSE Tumbleweed**.

It targets two locations typically affected by DKMS-based drivers (e.g., NVIDIA DKMS):
- `/var/lib/dkms/`
- `/usr/lib/modules/` (Tumbleweed uses usr-merge; `/lib/modules` may be a symlink)

## Supported platform
- **Only:** openSUSE 
- **Not supported / not tested: other distributions

The cleanup logic intentionally **does not** treat the presence of `/usr/lib/modules/<kver>/` as proof that the kernel is installed (directories can remain because they are not empty).

## Requirements
- systemd
- `bash`
- `rpm`
- `dkms` (recommended; if absent, only filesystem cleanup may apply depending on implementation)

## Installation
### 1) Install the script
From the repository root (adjust the script name if different):

```bash
sudo install -m 0755 ./dkms-kernel-gc /usr/local/sbin/dkms-kernel-gc
````

### 2) systemd automation (run after zypper transactions)

Create the service unit:

`/etc/systemd/system/dkms-kernel-gc.service`

```ini
[Unit]
Description=Garbage collect DKMS and orphaned kernel module trees (openSUSE Tumbleweed)

[Service]
Type=oneshot
ExecStart=/usr/local/sbin/dkms-kernel-gc
```

Create the path unit (triggered by zypper/zypp history changes):

`/etc/systemd/system/dkms-kernel-gc.path`

```ini
[Unit]
Description=Run dkms-kernel-gc after zypper transactions (openSUSE Tumbleweed)

[Path]
PathChanged=/var/log/zypp/history

[Install]
WantedBy=multi-user.target
```

Enable:

```bash
sudo systemctl daemon-reload
sudo systemctl enable --now dkms-kernel-gc.path
```
## Browse safery what is going to be removed

```bash
sudo /usr/local/sbin/dkms-kernel-gc --dry-run
```

## Manual run

```bash
sudo /usr/local/sbin/dkms-kernel-gc
```

## Logs

```bash
journalctl -u dkms-kernel-gc.service --no-pager -n 200
```

## Uninstall

Disable automation:

```bash
sudo systemctl disable --now dkms-kernel-gc.path
sudo systemctl daemon-reload
```

Remove files:

```bash
sudo rm -f /etc/systemd/system/dkms-kernel-gc.service
sudo rm -f /etc/systemd/system/dkms-kernel-gc.path
sudo rm -f /usr/local/sbin/dkms-kernel-gc
sudo systemctl daemon-reload
```

## Safety and warranty disclaimer

This software performs removal operations on system paths related to kernels and DKMS. Use at your own risk.

**NO WARRANTY:** This program is distributed in the hope that it will be useful, but **WITHOUT ANY WARRANTY**; without even the implied warranty of **MERCHANTABILITY** or **FITNESS FOR A PARTICULAR PURPOSE**.

## License

GNU **GPL**. See `LICENSE` in this repository.
