# Docker
URL: https://usememos.com/docs/deploy/docker

Docker is the simplest production-style deployment for a single Memos instance.

## Quick start [#quick-start]

```bash
docker run -d \
  --name memos \
  --restart unless-stopped \
  -p 5230:5230 \
  -v ~/.memos:/var/opt/memos \
  neosmemo/memos:stable
```

* The container listens on port `5230`.
* Memos stores persistent data in `/var/opt/memos` inside the container.
* The host path `~/.memos` will contain the SQLite database and local assets.

## Image tags [#image-tags]

Common tag choices:

* `stable`: safest default for production
* versioned tags such as `0.31.0`: fully pinned deployment
* branch-like tags such as `0.30`: latest patch in that line
* `latest`: development-oriented and less suitable for production

## Platform support [#platform-support]

The image ships multi-arch manifests and automatically selects the right variant:

* `linux/amd64` — x86\_64 processors
* `linux/arm64` — ARM 64-bit (Apple Silicon, Raspberry Pi 4+)
* `linux/arm/v7` — ARM 32-bit (Raspberry Pi 2/3)

## Container user [#container-user]

Memos runs as a non-root user inside the container:

* UID `10001`, GID `10001`, username `nonroot`
* The entrypoint script automatically fixes ownership of mounted volumes

If your host volume requires a different UID or GID, override them:

```bash
docker run -d \
  --name memos \
  -p 5230:5230 \
  -v ~/.memos:/var/opt/memos \
  -e MEMOS_UID=1000 \
  -e MEMOS_GID=1000 \
  neosmemo/memos:stable
```

## Environment variables [#environment-variables]

All `MEMOS_*` variables correspond directly to command-line flags. Common ones:

| Variable                                  | Default          | Purpose                                                                         |
| ----------------------------------------- | ---------------- | ------------------------------------------------------------------------------- |
| `MEMOS_PORT`                              | `8081`           | HTTP listen port (image default: `5230`)                                        |
| `MEMOS_ADDR`                              | \`\`             | Bind address (empty = all interfaces)                                           |
| `MEMOS_DATA`                              | `/var/opt/memos` | Data directory                                                                  |
| `MEMOS_DRIVER`                            | `sqlite`         | Database backend                                                                |
| `MEMOS_DSN`                               | auto             | Database connection string                                                      |
| `MEMOS_INSTANCE_URL`                      | \`\`             | Canonical external URL; independent of access policy                            |
| `MEMOS_DEMO`                              | `false`          | Demo mode                                                                       |
| `MEMOS_WEBHOOK_PRIVATE_NETWORK_ALLOWLIST` | empty            | Exact hostnames, IP addresses, or CIDRs allowed as private webhook destinations |
| `MEMOS_ALLOW_PRIVATE_WEBHOOKS`            | `false`          | Deprecated blanket exception for private webhook destinations                   |
| `MEMOS_LOG_LEVEL`                         | `info`           | Log verbosity (`debug`, `info`, `warn`, `error`)                                |

Typical example:

```bash
docker run -d \
  --name memos \
  --restart unless-stopped \
  -p 5230:5230 \
  -v ~/.memos:/var/opt/memos \
  -e MEMOS_PORT=5230 \
  -e MEMOS_DRIVER=sqlite \
  -e MEMOS_INSTANCE_URL=https://memos.example.com \
  neosmemo/memos:stable
```

On a fresh instance, the quick-start command initializes private access because it omits `MEMOS_INSTANCE_URL`. A configured URL initializes public access only when the access setting does not yet exist. In 0.31, select the desired `ACCESS` policy in instance settings or [deployment configuration](/docs/configuration/deployment-configuration#access-policy); changing the URL of an existing instance does not change that policy.

## External databases [#external-databases]

SQLite is the default and works well for a single-node deployment. If you need MySQL or PostgreSQL, provide:

* `MEMOS_DRIVER=mysql` or `MEMOS_DRIVER=postgres`
* `MEMOS_DSN` with the appropriate connection string

MySQL example:

```bash
docker run -d \
  --name memos \
  -p 5230:5230 \
  -v ~/.memos:/var/opt/memos \
  -e MEMOS_DRIVER=mysql \
  -e MEMOS_DSN="user:password@tcp(mysql-host:3306)/memos" \
  neosmemo/memos:stable
```

PostgreSQL example:

```bash
docker run -d \
  --name memos \
  -p 5230:5230 \
  -v ~/.memos:/var/opt/memos \
  -e MEMOS_DRIVER=postgres \
  -e MEMOS_DSN="postgres://user:password@postgres-host:5432/memos?sslmode=disable" \
  neosmemo/memos:stable
```

When using an external database, the mounted volume is still relevant for local assets and instance data.

## Secrets with `_FILE` suffix [#secrets-with-_file-suffix]

The entrypoint script supports `*_FILE` variable suffixes so you can pass credentials via Docker secrets instead of plain environment variables:

```bash
echo "postgres://user:password@host:5432/memos" | docker secret create memos_dsn -

docker run -d \
  --name memos \
  -p 5230:5230 \
  -v ~/.memos:/var/opt/memos \
  -e MEMOS_DRIVER=postgres \
  -e MEMOS_DSN_FILE=/run/secrets/memos_dsn \
  --secret memos_dsn \
  neosmemo/memos:stable
```

This works for any `MEMOS_*` variable.

## Container management [#container-management]

```bash
docker logs -f memos
docker stop memos
docker start memos
docker restart memos
```

## Upgrades [#upgrades]

```bash
docker pull neosmemo/memos:stable
docker stop memos
docker rm memos
docker run -d \
  --name memos \
  --restart unless-stopped \
  -p 5230:5230 \
  -v ~/.memos:/var/opt/memos \
  neosmemo/memos:stable
```

Because data is stored in the mounted volume, replacing the container is usually safe as long as you keep the same data directory and have backups.
![](docs/images/logo.png)

[![Downloads](https://img.shields.io/badge/dynamic/json?color=green&label=Downloads&query=totalString&url=https%3A%2F%2Fraw.githubusercontent.com%2Ftopjohnwu%2Fmagisk-files%2Fcount%2Fcount.json&cacheSeconds=1800)](https://raw.githubusercontent.com/topjohnwu/magisk-files/count/count.json)

#### This is not an officially supported Google product

## Introduction

Magisk is a suite of open source software for customizing Android, supporting devices higher than Android 6.0.<br>
Some highlight features:

- **MagiskSU**: Provide root access for applications
- **Magisk Modules**: Modify read-only partitions by installing modules
- **MagiskBoot**: The most complete tool for unpacking and repacking Android boot images
- **Zygisk**: Run code in every Android applications' processes

## Downloads

[Github](https://github.com/topjohnwu/Magisk/releases) is the only source where you can get official Magisk information and downloads.

## Useful Links

- [Installation Instruction](https://topjohnwu.github.io/Magisk/install.html)
- [Building and Development](https://topjohnwu.github.io/Magisk/build.html)
- [Magisk Documentation](https://topjohnwu.github.io/Magisk/)
- [Zygisk module sample](https://github.com/topjohnwu/zygisk-module-sample)

## Bug Reports

**Only bug reports from Debug builds will be accepted.**

For installation issues, upload both boot image and install logs.<br>
For Magisk issues, upload boot logcat or dmesg.<br>
For Magisk app crashes, record and upload the logcat when the crash occurs.

## Translation Contributions

Default string resources for the Magisk app and its stub APK are located here:

- `app/core/src/main/res/values/strings.xml`
- `app/stub-res/src/main/res/values/strings.xml`

Translate each and place them in the respective locations (`[module]/src/main/res/values-[lang]/strings.xml`).

## License

    Magisk, including all git submodules are free software:
    you can redistribute it and/or modify it under the terms of the
    GNU General Public License as published by the Free Software Foundation,
    either version 3 of the License, or (at your option) any later version.

    This program is distributed in the hope that it will be useful,
    but WITHOUT ANY WARRANTY; without even the implied warranty of
    MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
    GNU General Public License for more details.

    You should have received a copy of the GNU General Public License
    along with this program.  If not, see <http://www.gnu.org/licenses/>.
