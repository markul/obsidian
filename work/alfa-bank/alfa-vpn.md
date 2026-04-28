# Alfa VPN

## Related Notes

- [[work/alfa-bank/index|Alfa Bank]]
- [[personal/tech/software|Software]]

## `snxctl` IPSec Certificate Validation Failure

Observed on `2026-04-23` with `snx-rs` / `snxctl` `5.2.3` on `optiplex`.

Symptom:

```bash
snxctl connect
```

returns:

```text
Internal IPSec certificate validation failed!
```

Useful checks:

```bash
snxctl --version
timedatectl
snxctl info
systemctl status snx-rs --no-pager
journalctl -u snx-rs -n 160 --no-pager
```

Findings from the working incident:

- System clock was synchronized, so the failure was not caused by local time drift.
- `~/.config/snx-rs/snx-rs.conf` already had `ignore-server-cert=true`; this does not bypass the internal IPSec certificate fingerprint check.
- `snxctl info` reached `mypc.alfabank.ru` and showed the advertised internal CA fingerprint as `GETS CITE SHAM WALT FORM ROAD FLEW BUFF GROW OS DIAL PRY`.
- A root `snx-rs` command daemon was running under `snx-rs.service` for several days; restarting that daemon fixed the connection.

Working fix:

```bash
sudo systemctl restart snx-rs
snxctl connect
```

If the error recurs and restart does not help, try upgrading `snx-rs` before changing VPN config. As checked on `2026-04-23`, upstream latest was `v6.0.0` while the local package was `5.2.3`.

Emergency workaround if IPSec remains broken and VPN is needed immediately:

```bash
cp ~/.config/snx-rs/snx-rs.conf ~/.config/snx-rs/snx-rs.conf.bak
sed -i 's/^tunnel-type=.*/tunnel-type=ssl/' ~/.config/snx-rs/snx-rs.conf
snxctl connect
```

Restore IPSec config afterwards:

```bash
mv ~/.config/snx-rs/snx-rs.conf.bak ~/.config/snx-rs/snx-rs.conf
```
