# NestDroid update channel

This repository is the public update channel for the NestDroid Android shell. It contains **no source code**: only signed manifests and the binary payloads the app downloads.

| Path | What it is |
| --- | --- |
| `channel.json` + `channel.json.sig` | The app catalog: latest version, APK digest, threat database and advisor model. Signed. |
| `apk/` | Optional home for release APKs. The catalog can point at any HTTPS URL (it pins the SHA-256), and the current 1.6.30 build is served from a static mirror. |
| `threats/threats.json` | Known-bad SHA-256 list used by the on-device file advisor. |
| `threats/advisor.json` | Advisor weights (the "model") for the content signals. |
| `runtimes/index.json` + `.sig` | Wine runtime catalog, signed with the same key. `RuntimeProvisioner` can consume it directly. |
| `runtimes/*.zip` | Runtime packs listed by that catalog. |

## How the app trusts this repository

1. `channel.json` must verify against `channel.json.sig` with the ECDSA P-256 public key compiled into the app (`app/res/raw/runtime_feed_keys.txt`, key id `nestdroid-channel-2026-11`). A modified catalog fails the check and nothing is downloaded.
2. Every payload is pinned by SHA-256 and size in the signed catalog, so replacing a file on this repository cannot change what the device installs.
3. An APK is only installed when it is signed by the same key as the installed app (the system package manager enforces this) **and** its SHA-256 matches the signed catalog.
4. The catalog is fetched over HTTPS only; a redirect away from HTTPS is rejected.

The repository is public, so anyone can download these files. What nobody else can do is produce a payload the app accepts: that requires the private signing key, which is held by the publisher and never committed here.

## What is in here today

- **App**: 1.6.30, served from `https://nestdroid-update-mirror-25devl.freebuff.site/nestdroid-1.6.30.apk` with SHA-256 `bf52388b9142d7199aaf8bd7bf17bd27ed5188d230a25e8064725e9f2aa91100`.
- **Threat database**: `2026.11.21`, currently the harmless EICAR test signature. It exists so the update path and the content matcher can be exercised without shipping malware.
- **Runtime catalog**: one pack, `synthetic-display-fixture-5` — a synthetic display fixture used by the project's device tests. **There is still no real Wine image**: Wine, Box64/FEX and the Windows runtimes are third-party binaries with their own licences and are not built by this project. The catalog slot is ready for a real pack when one exists.

## Publishing an update

```sh
./bump-version.sh patch
./build.sh --offline                       # -> build/dist/nestdroid-X.Y.Z.apk
$EDITOR tools/channel/release.json         # versionName, versionCode, apk path, notes
python3 tools/channel/publish.py --spec tools/channel/release.json \
    --key /path/to/channel-key.pem --out build/channel-out
```

Then put the APK where `apkUrl` points — `apk/` in this repo, or any public HTTPS host — and commit `build/channel-out/{channel.json,channel.json.sig,threats/,runtimes/}`. The script prints the APK SHA-256 that the signed catalog pins; the uploaded file must match it byte for byte.

Data-only updates (threat database, advisor model) do not need an APK: publish a new `threats/threats.json`, re-sign the catalog with a higher version, and devices pick it up on the next check.

The private key is the whole trust anchor. Losing it means no device can be updated again; leaking it means someone else can sign updates. Keep it offline and back it up.
