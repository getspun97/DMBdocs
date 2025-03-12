﻿<div align="center" style="max-width: 100%; height: auto;">
  <h1>🎬 Debrid Media Bridge 🎬</h1>
  <a href="https://github.com/I-am-PUID-0/DMB">
    <picture>
      <source media="(prefers-color-scheme: dark)" srcset="https://github.com/I-am-PUID-0/DMB/assets/36779668/d0cbc785-2e09-41da-b226-924fdfcc1f21">
      <img alt="DMB" src="https://github.com/I-am-PUID-0/DMB/assets/36779668/d0cbc785-2e09-41da-b226-924fdfcc1f21" style="max-width: 100%; height: auto;">
    </picture>
  </a>
</div>

## 📜 Description

**Debrid Media Bridge (DMB)** is an All-In-One (AIO) docker image for the unified deployment of **[Riven Media's](https://github.com/rivenmedia)**, **[yowmamasita's](https://github.com/yowmamasita)**, **[iPromKnight's](https://github.com/iPromKnight/zilean)**, **[Nick Craig-Wood's](https://github.com/ncw)**, **[Michael Stonebraker's](https://en.wikipedia.org/wiki/Michael_Stonebraker)**, and **[Dave Page's](https://github.com/dpage)** projects -- **[Riven](https://github.com/rivenmedia/riven)**, **[Zurg](https://github.com/debridmediamanager/zurg-testing)**, **[Zilean](https://github.com/iPromKnight/zilean)**, **[rclone](https://github.com/rclone/rclone)**, **[PostgreSQL](https://www.postgresql.org/)**, and **[pgAdmin 4](https://www.pgadmin.org/)**.

> ⚠️ **IMPORTANT**: Docker Desktop **CANNOT** be used to run DMB. Docker Desktop does not support the [mount propagation](https://docs.docker.com/storage/bind-mounts/#configure-bind-propagation) required for rclone mounts.
>
> ![image](https://github.com/I-am-PUID-0/DMB/assets/36779668/aff06342-1099-4554-a5a4-72a7c82cb16e)
>
> See the wiki for [alternative solutions](https://github.com/I-am-PUID-0/DMB/wiki/Setup-Guides) to run DMB on Windows through WSL2.

## 🌟 Features

See the DMB [Wiki](https://github.com/I-am-PUID-0/DMB/wiki) for a full list of features and settings.

## 🐳 Docker Hub

A prebuilt image is hosted on [Docker Hub](https://hub.docker.com/r/iampuid0/dmb).

## 🏷️ GitHub Container Registry

A prebuilt image is hosted on [GitHub Container Registry](https://github.com/I-am-PUID-0/DMB/pkgs/container/DMB).

## 🐳 Docker-compose

> [!NOTE]
> The below examples are not exhaustive and are intended to provide a starting point for deployment.
> Additionally, the host directories used in the examples are based on [Define the directory structure](https://github.com/I-am-PUID-0/DMB/wiki/Setup-Guides#define-the-directory-structure) and provided for illustrative purposes and can be changed to suit your needs.

```YAML
services:
  DMB:
    container_name: DMB
    image: iampuid0/dmb:latest                                      ## Optionally, specify a specific version of DMB w/ image: iampuid0/dmb:2.0.0
    stop_grace_period: 30s                                           ## Adjust as need to allow for graceful shutdown of the container
    shm_size: 128mb                                                  ## Increased for PostgreSQL
    stdin_open: true                                                 ## docker run -i
    tty: true                                                        ## docker run -t
    volumes:
      - /home/username/docker/DMB/config:/config                     ## Location of configuration files. If a Zurg config.yml and/or Zurg app is placed here, it will be used to override the default configuration and/or app used at startup.
      - /home/username/docker/DMB/log:/log                           ## Location for logs
      - /home/username/docker/DMB/Zurg/RD:/zurg/RD                   ## Location for Zurg RealDebrid active configuration
      - /home/username/docker/DMB/Zurg/mnt:/data:shared              ## Location for rclone mount to host
      - /home/username/docker/DMB/Riven/data:/riven/backend/data     ## Location for Riven backend data
      - /home/username/docker/DMB/Riven/mnt:/mnt                     ## Location for Riven symlinks
      - /home/username/docker/DMB/PostgreSQL/data:/postgres_data     ## Location for PostgreSQL database
      - /home/username/docker/DMB/pgAdmin4/data:/pgadmin/data        ## Location for pgAdmin 4 data
      - /home/username/docker/DMB/Zilean/data:/zilean/app/data       ## Location for Zilean data
    environment:
      - TZ=
      - PUID=
      - PGID=
      - DMB_LOG_LEVEL=INFO
      - ZURG_INSTANCES_REALDEBRID_API_KEY=
      - RIVEN_FRONTEND_ENV_ORIGIN=http://0.0.0.0:3000               ## See Riven documentation for more details
    # network_mode: container:gluetun                               ## Example to attach to gluetun vpn container if realdebrid blocks IP address
    ports:
      - "3005:3005"                                                 ## DMB Frontend
      - "3000:3000"                                                 ## Riven Frontend
      - "5050:5050"                                                 ## pgAdmin 4
    devices:
      - /dev/fuse:/dev/fuse:rwm
    cap_add:
      - SYS_ADMIN
    security_opt:
      - apparmor:unconfined
      - no-new-privileges
```

## 🎥 Example Plex Docker-compose

> [!NOTE]
> The Plex server must be started after the rclone mount is available. The below example uses the `depends_on` parameter to delay the start of the Plex server until the rclone mount is available. The rclone mount must be shared to the Plex container. The rclone mount location should not be added to the Plex library. The Riven symlink location must be shared to the Plex container and added to the Plex library.

```YAML
services:
  plex:
    image: plexinc/pms-docker:latest
    container_name: plex
    devices:
      - /dev/dri:/dev/dri
    volumes:
      - /home/username/docker/plex/library:/config
      - /home/username/docker/plex/transcode:/transcode
      - /home/username/docker/DMB/Zurg/mnt:/data            ## rclone mount location from DMB must be shared to Plex container. Don't add to plex library
      - /home/username/docker/DMB/Riven/mnt:/mnt            ## Riven symlink location from DMB must be shared to Plex container. Add to plex library
    environment:
      - TZ=${TZ}
      - PLEX_UID=                                           ## Same as PUID
      - PLEX_GID=                                           ## Same as PGID
      - PLEX_CLAIM=claimToken                               ## Need for the first run of Plex - get claimToken here https://www.plex.tv/claim/
    ports:
      - "32400:32400"
    healthcheck:
      test: curl --connect-timeout 15 --silent --show-error --fail http://localhost:32400/identity
      interval: 1m00s
      timeout: 15s
      retries: 3
      start_period: 1m00s
    depends_on:                                            ## Used to delay the startup of plex to ensure the rclone mount is available.
      DMB:                                                 ## set to the name of the container running rclone
        condition: service_healthy
        restart: true                                       ## Will automatically restart the plex container if the DMB container restarts
```

## 🌐 Environment Variables

The following table lists all available environment variables used by the container. The environment variables are set via the `-e` parameter or via the docker-compose file within the `environment:` section or with a .env file saved to the config directory. Value of this parameter is listed as `<VARIABLE_NAME>=<Value>`

Global Variables:
| Variable       | Default  | Description                                                       | Required for DMB |
| -------------- | -------- | ------------------------------------------------------------------|----------------- |
| `PUID`         | `1000`   | Your User ID | :heavy_check_mark: |
| `PGID`         | `1000`   | Your Group ID |:heavy_check_mark: |
| `TZ`           | `(null)` | Your time zone listed as `Area/Location` | :heavy_check_mark: |

DMB Variables:
| Variable       | Default  | Description                                                       | Required for DMB |
| -------------- | -------- | ------------------------------------------------------------------|----------------- |
| `DMB_LOG_LEVEL`  | `INFO` |Set your DMB log level.  Choices are `INFO` or `DEBUG`  |
| `DMB_LOG_NAME`   | `DMB` | Name of the DMB log file |
| `DMB_LOG_DIR`    | `/log` | Path to the DMB log file |
| `DMB_LOG_COUNT`  | `2` | Number of DMB log files |
| `DMB_LOG_SIZE`   | `10M` | Max size of DMB log file |
| `DMB_COLOR_LOG`  | `true` | Color code log for better readability |
| `DMB_PLEX_TOKEN` | `(null)` | Enter your Plex token |
| `DMB_PLEX_ADDRESS` | `(null)` | ip address of your plex machine written as `http://<IP_ADDRESS>:<PORT>` |
| `DMB_GITHUB_TOKEN` | `(null)` | Enter your GitHub token |

DMB API Variables:
| Variable       | Default  | Description                                                       | Required for DMB |
| -------------- | -------- | ------------------------------------------------------------------|----------------- |
| `DMB_API_SERVICE_ENABLED` | `true` | Enables or disables DMB API service |
| `DMB_API_SERVICE_PROCESS_NAME` | `DMB API` | Name of the DMB API service process |
| `DMB_API_SERVICE_LOG_LEVEL` | `INFO` | Set the DMB API log level. Choices are `INFO` or `DEBUG` |
| `DMB_API_SERVICE_HOST` | `127.0.0.1` | The ip address of your DMB host machine |
| `DMB_API_SERVICE_PORT` | `8000` | The port used for the DMB API service |

DMB Frontend Variables:
| Variable       | Default  | Description                                                       | Required for DMB |
| -------------- | -------- | ------------------------------------------------------------------|----------------- |
| `DMB_FRONTEND_ENABLED` | `true` |
| `DMB_FRONTEND_PROCESS_NAME` | `"DMB Frontend"` |
| `DMB_FRONTEND_REPO_OWNER` | `I-am-PUID-0` |
| `DMB_FRONTEND_REPO_NAME` | `dmbdb` |
| `DMB_FRONTEND_RELEASE_VERSION_ENABLED` | `false` |
| `DMB_FRONTEND_RELEASE_VERSION` | `1.1.0` |
| `DMB_FRONTEND_BRANCH_ENABLED` | `false` |
| `DMB_FRONTEND_BRANCH` | `main` |
| `DMB_FRONTEND_SUPPRESS_LOGGING` | `false` |
| `DMB_FRONTEND_LOG_LEVEL` | `INFO` |
| `DMB_FRONTEND_ORIGINS` | `http://0.0.0.0:3005` |
| `DMB_FRONTEND_HOST` | `0.0.0.0` |
| `DMB_FRONTEND_PORT` | `3005` |
| `DMB_FRONTEND_AUTO_UPDATE` | `false` |
| `DMB_FRONTEND_AUTO_UPDATE_INTERVAL` | `24` |
| `DMB_FRONTEND_CLEAR_ON_UPDATE` | `true` |
| `DMB_FRONTEND_EXCLUDE_DIRS` | `(null)`
| `DMB_FRONTEND_PLATFORMS` | `pnpm` |
| `DMB_FRONTEND_COMMAND` | `node .output/server/index.mjs` |
| `DMB_FRONTEND_CONFIG_DIR` | `/dmb/frontend` |

PostgreSQL Variables:
| Variable       | Default  | Description                                                       | Required for DMB |
| -------------- | -------- | ------------------------------------------------------------------|----------------- |
| `POSTGRES_ENABLED` | `true` |
| `POSTGRES_PROCESS_NAME` | `PostgreSQL` |
| `POSTGRES_SUPPRESS_LOGGING` | `false` |
| `POSTGRES_LOG_LEVEL` | `INFO` |
| `POSTGRES_HOST` | `127.0.0.1` |
| `POSTGRES_PORT` | `5432` |
| `POSTGRES_CONFIG_DIR` | `/postgres_data` |
| `POSTGRES_CONFIG_FILE` | `/postgres_data/postgresql.conf` | 
| `POSTGRES_INITDB_ARGS` | `--data-checksums` |
| `POSTGRES_USER` | `DMB` |
| `POSTGRES_PASSWORD` | `postgres` |
| `POSTGRES_SHARED_BUFFERS` | `128MB` |
| `POSTGRES_MAX_CONNECTIONS` | `100` |
| `POSTGRES_RUN_DIRECTORY` | `/run/postgresql` |
| `POSTGRES_COMMAND` | `postgres -D {postgres_config_dir} -c config_file={postgres_config_file}` |

pgAdmin Variables:
| Variable       | Default  | Description                                                       | Required for DMB |
| -------------- | -------- | ------------------------------------------------------------------|----------------- |
| `PGADMIN_ENABLED` | `true` |
| `PGADMIN_PROCESS_NAME` | `pgAdmin4` |
| `PGADMIN_CONFIG_DIR` | `/pgadmin/data` |
| `PGADMIN_CONFIG_FILE` | `/pgadmin/data/config_local.py` |
| `PGADMIN_LOG_FILE` | `/pgadmin/data/pgadmin4.log` |
| `PGADMIN_PORT` | `5050` |
| `PGADMIN_DEFAULT_SERVER` | `0.0.0.0` |
| `PGADMIN_SETUP_EMAIL` | `DMB@DMB.DMB` |
| `PGADMIN_SETUP_PASSWORD` | `postgres` |

Rclone Variables:
| Variable       | Default  | Description                                                       | Required for DMB |
| -------------- | -------- | ------------------------------------------------------------------|----------------- |
| `RCLONE_INSTANCES_REALDEBRID_ENABLED` | `true` |
| `RCLONE_INSTANCES_REALDEBRID_PROCESS_NAME` | `"rclone w/ RealDebrid"` |
| `RCLONE_INSTANCES_REALDEBRID_SUPPRESS_LOGGING` | `false` |
| `RCLONE_INSTANCES_REALDEBRID_LOG_LEVEL` | `INFO` |
| `RCLONE_INSTANCES_REALDEBRID_KEY_TYPE` | `RealDebrid` |
| `RCLONE_INSTANCES_REALDEBRID_ZURG_ENABLED` | `true` |
| `RCLONE_INSTANCES_REALDEBRID_MOUNT_DIR` | `/data` |
| `RCLONE_INSTANCES_REALDEBRID_MOUNT_NAME` | `rclone_RD` |
| `RCLONE_INSTANCES_REALDEBRID_CACHE_DIR` | `/cache` |
| `RCLONE_INSTANCES_REALDEBRID_CONFIG_DIR` | `/config` |
| `RCLONE_INSTANCES_REALDEBRID_CONFIG_FILE` | `/config/rclone.config` |
| `RCLONE_INSTANCES_REALDEBRID_ZURG_CONFIG_FILE` | `/zurg/RD/config.yml` |
| `RCLONE_INSTANCES_REALDEBRID_COMMAND` | `(null)` |
| `RCLONE_INSTANCES_REALDEBRID_API_KEY` | `(null)` |

Riven Backend Variables:
| Variable       | Default  | Description                                                       | Required for DMB |
| -------------- | -------- | ------------------------------------------------------------------|----------------- |
| `RIVEN_BACKEND_ENABLED` | `true` |
| `RIVEN_BACKEND_PROCESS_NAME` | `"Riven Backend"` |
| `RIVEN_BACKEND_REPO_OWNER` | `rivenmedia` |
| `RIVEN_BACKEND_REPO_NAME` | `riven` |
| `RIVEN_BACKEND_RELEASE_VERSION_ENABLED` | `false` |
| `RIVEN_BACKEND_RELEASE_VERSION` | `v0.20.1` |
| `RIVEN_BACKEND_BRANCH_ENABLED` | `false` |
| `RIVEN_BACKEND_BRANCH` | `release-please--branches--main` |
| `RIVEN_BACKEND_SUPPRESS_LOGGING` | `false` |
| `RIVEN_BACKEND_LOG_LEVEL` | `INFO` |
| `RIVEN_BACKEND_HOST` | `127.0.0.1` |
| `RIVEN_BACKEND_PORT` | `8080` |
| `RIVEN_BACKEND_AUTO_UPDATE` | `false` |
| `RIVEN_BACKEND_AUTO_UPDATE_INTERVAL` | `24` |
| `RIVEN_BACKEND_SYMLINK_LIBRARY_PATH` | `/mnt` |
| `RIVEN_BACKEND_CLEAR_ON_UPDATE` | `true` |
| `RIVEN_BACKEND_EXCLUDE_DIRS` | `/riven/backend/data` |
| `RIVEN_BACKEND_ENV_COPY_SOURCE` | `/riven/backend/data/.env` |
| `RIVEN_BACKEND_ENV_COPY_DESTINATION` | `/riven/backend/src/.env` |
| `RIVEN_BACKEND_PLATFORMS` | `python` |
| `RIVEN_BACKEND_COMMAND` | `/riven/backend/venv/bin/python src/main.py` |
| `RIVEN_BACKEND_CONFIG_DIR` | `/riven/backend` |
| `RIVEN_BACKEND_CONFIG_FILE` | `/riven/backend/data/settings.json` |
| `RIVEN_BACKEND_WAIT_FOR_DIR` | `/data/rclone_RD/__all__` |

Riven Frontend Variables:
| Variable       | Default  | Description                                                       | Required for DMB |
| -------------- | -------- | ------------------------------------------------------------------|----------------- |
| `RIVEN_FRONTEND_ENABLED` | `true` |
| `RIVEN_FRONTEND_PROCESS_NAME` | `"Riven Frontend"` |
| `RIVEN_FRONTEND_REPO_OWNER` | `rivenmedia` |
| `RIVEN_FRONTEND_REPO_NAME` | `riven-frontend` |
| `RIVEN_FRONTEND_RELEASE_VERSION_ENABLED` | `false` |
| `RIVEN_FRONTEND_RELEASE_VERSION` | `v0.17.0` |
| `RIVEN_FRONTEND_BRANCH_ENABLED` | `false` |
| `RIVEN_FRONTEND_BRANCH` | `release-please--branches--main` |
| `RIVEN_FRONTEND_SUPPRESS_LOGGING` | `false` |
| `RIVEN_FRONTEND_LOG_LEVEL` | `INFO` |
| `RIVEN_FRONTEND_HOST` | `127.0.0.1` |
| `RIVEN_FRONTEND_PORT` | `3000` |
| `RIVEN_FRONTEND_AUTO_UPDATE` | `false` |
| `RIVEN_FRONTEND_AUTO_UPDATE_INTERVAL` | `24` |
| `RIVEN_FRONTEND_CLEAR_ON_UPDATE` | `true` |
| `RIVEN_FRONTEND_EXCLUDE_DIRS` | `(null)` |
| `RIVEN_FRONTEND_PLATFORMS` | `pnpm` |
| `RIVEN_FRONTEND_COMMAND` | `node build` |
| `RIVEN_FRONTEND_CONFIG_DIR` | `/riven/frontend` |
| `RIVEN_FRONTEND_ENV_DIALECT` | `postgres` |

Zilean Variables:
| Variable       | Default  | Description                                                       | Required for DMB |
| -------------- | -------- | ------------------------------------------------------------------|----------------- |
| `ZILEAN_ENABLED` | `true` |
| `ZILEAN_PROCESS_NAME` | `Zilean` |
| `ZILEAN_REPO_OWNER` | `iPromKnight` |
| `ZILEAN_REPO_NAME` | `zilean` |
| `ZILEAN_RELEASE_VERSION_ENABLED` | `false` |
| `ZILEAN_RELEASE_VERSION` | `v3.3.0` |
| `ZILEAN_BRANCH_ENABLED` | `false` |
| `ZILEAN_BRANCH` | `main` |
| `ZILEAN_SUPPRESS_LOGGING` | `false` | 
| `ZILEAN_LOG_LEVEL` | `INFO` |
| `ZILEAN_HOST` | `127.0.0.1` |
| `ZILEAN_PORT` | `8182` |
| `ZILEAN_AUTO_UPDATE` | `false` |
| `ZILEAN_AUTO_UPDATE_INTERVAL` | `24` |
| `ZILEAN_CLEAR_ON_UPDATE` | `true` |
| `ZILEAN_EXCLUDE_DIRS` | `/zilean/app/data` |
| `ZILEAN_ENV_COPY_SOURCE` | `/zilean/app/data/.env` |
| `ZILEAN_ENV_COPY_DESTINATION` | `/zilean/app/src/.env` |
| `ZILEAN_PLATFORMS` | `python dotnet` |
| `ZILEAN_COMMAND` | `/zilean/app/zilean-api` | 
| `ZILEAN_CONFIG_DIR` | `/zilean` | 
| `ZILEAN_CONFIG_FILE` | `/zilean/app/data/settings.json` |

Zurg Variables:
| Variable       | Default  | Description                                                       | Required for DMB |
| -------------- | -------- | ------------------------------------------------------------------|----------------- |
| `ZURG_INSTANCES_REALDEBRID_ENABLED` | `true` |
| `ZURG_INSTANCES_REALDEBRID_PROCESS_NAME` | `"Zurg w/ RealDebrid"` |
| `ZURG_INSTANCES_REALDEBRID_REPO_OWNER` | `debridmediamanager` |
| `ZURG_INSTANCES_REALDEBRID_REPO_NAME` | `zurg-testing` |
| `ZURG_INSTANCES_REALDEBRID_RELEASE_VERSION_ENABLED` | `false` |
| `ZURG_INSTANCES_REALDEBRID_RELEASE_VERSION` | `v0.9.3-final` | 
| `ZURG_INSTANCES_REALDEBRID_SUPPRESS_LOGGING` | `false` |
| `ZURG_INSTANCES_REALDEBRID_LOG_LEVEL` | `INFO` |
| `ZURG_INSTANCES_REALDEBRID_AUTO_UPDATE` | `false` |
| `ZURG_INSTANCES_REALDEBRID_AUTO_UPDATE_INTERVAL` | `1` |
| `ZURG_INSTANCES_REALDEBRID_CLEAR_ON_UPDATE` | `false` |
| `ZURG_INSTANCES_REALDEBRID_EXCLUDE_DIRS` | `(null)` |
| `ZURG_INSTANCES_REALDEBRID_KEY_TYPE` | `RealDebrid` |
| `ZURG_INSTANCES_REALDEBRID_CONFIG_DIR` | `/zurg/RD` |
| `ZURG_INSTANCES_REALDEBRID_CONFIG_FILE` | `/zurg/RD/config.yml` |
| `ZURG_INSTANCES_REALDEBRID_COMMAND` | `/zurg/RD/zurg` |
| `ZURG_INSTANCES_REALDEBRID_PORT` | `(null)` |
| `ZURG_INSTANCES_REALDEBRID_USER` | `(null)` |
| `ZURG_INSTANCES_REALDEBRID_PASSWORD` | `(null)` | | :heavy_check_mark: |
| `ZURG_INSTANCES_REALDEBRID_API_KEY` | `(null)` |

## 🌐 Ports Used

> [!NOTE]
> The below examples are default and may be configurable with the use of additional environment variables.

The following table describes the ports used by the container. The mappings are set via the `-p` parameter or via the docker-compose file within the `ports:` section. Each mapping is specified with the following format: `<HOST_PORT>:<CONTAINER_PORT>[:PROTOCOL]`.

| Container port | Protocol | Description                                                                          |
| -------------- | -------- | ------------------------------------------------------------------------------------ |
| `3005`         | TCP      | DMB frontend - a web UI is accessible at the assigned port                           |
| `3000`         | TCP      | Riven frontend - A web UI is accessible at the assigned port                         |
| `8080`         | TCP      | Riven backend - The API is accessible at the assigned port                           |
| `5432`         | TCP      | PostgreSQL - The SQL server is accessible at the assigned port                       |
| `5050`         | TCP      | pgAdmin 4 - A web UI is accessible at the assigned port                              |
| `8182`         | TCP      | Zilean - The API and Web Ui (/swagger/index.html) is accessible at the assigned port |
| `9090`         | TCP      | Zurg - A web UI is accessible at the assigned port                                   |

## 📂 Data Volumes

The following table describes the data volumes used by the container. The mappings
are set via the `-v` parameter or via the docker-compose file within the `volumes:` section. Each mapping is specified with the following
format: `<HOST_DIR>:<CONTAINER_DIR>[:PERMISSIONS]`.

| Container path   | Permissions | Description                                                                                                                                                                                                                                 |
| ---------------- | ----------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `/config`        | rw          | This is where the application stores the rclone.conf, and any files needing persistence. CAUTION: rclone.conf is overwritten upon start/restart of the container. Do NOT use an existing rclone.conf file if you have other rclone services |
| `/log`           | rw          | This is where the application stores its log files                                                                                                                                                                                          |
| `/data`          | shared      | This is where rclone will be mounted.                                                                                                                                                                                                       |
| `/zurg/RD`       | rw          | This is where Zurg will store the active configuration and data for RealDebrid. Not required when only utilizing Riven                                                                                                                      |
| `/riven/data`    | rw          | This is where Riven will store its data. Not required when only utilizing Zurg                                                                                                                                                              |
| `/riven/mnt`     | rw          | This is where Riven will set its symlinks. Not required when only utilizing Zurg                                                                                                                                                            |
| `/postgres_data` | rw          | This is where PostgreSQL will store its data. Not required when only utilizing Zurg                                                                                                                                                         |
| `/pgadmin/data`  | rw          | This is where pgAdmin 4 will store its data. Not required when only utilizing Zurg                                                                                                                                                          |

## 📝 TODO

See the [DMB roadmap](https://github.com/users/I-am-PUID-0/projects/6) for a list of planned features and enhancements.

## 🛠️ DEV

### Tracking current development for an upcoming release:

- [Pre-Release Changes](https://gist.github.com/I-am-PUID-0/7e02c2cb4a5211d810a913f947861bc2#file-pre-release_changes-md)
- [Pre-Release TODO](https://gist.github.com/I-am-PUID-0/7e02c2cb4a5211d810a913f947861bc2#file-pre-release_todo-md)

### Development support:

- The repo contains a devcontainer for use with vscode.
- Bind mounts will need to be populated with content from this repo

## 🚀 Deployment

DMB allows for the simultaneous or individual deployment of any of the services

For additional details on deployment, see the [DMB Wiki](https://github.com/I-am-PUID-0/DMB/wiki/Setup-Guides#deployment-options)

## 🌍 Community

### DMB

- For questions related to DMB, see the GitHub [discussions](https://github.com/I-am-PUID-0/DMB/discussions)
- or create a new [issue](https://github.com/I-am-PUID-0/DMB/issues) if you find a bug or have an idea for an improvement.
- or join the DMB [discord server](https://discord.gg/8dqKUBtbp5)

### Riven Media

- For questions related to Riven, see the GitHub [discussions](https://github.com/orgs/rivenmedia/discussions)
- or create a new [issue](https://github.com/rivenmedia/riven/issues) if you find a bug or have an idea for an improvement.
- or join the Riven [discord server](https://discord.gg/VtYd42mxgb)

## 🍻 Buy **[Riven Media](https://github.com/rivenmedia)** a beer/coffee? :)

If you enjoy the underlying projects and want to buy Riven Media a beer/coffee, feel free to use the [GitHub sponsor link](https://github.com/sponsors/dreulavelle/)

## 🍻 Buy **[yowmamasita](https://github.com/yowmamasita)** a beer/coffee? :)

If you enjoy the underlying projects and want to buy yowmamasita a beer/coffee, feel free to use the [GitHub sponsor link](https://github.com/sponsors/debridmediamanager)

## 🍻 Buy **[Nick Craig-Wood](https://github.com/ncw)** a beer/coffee? :)

If you enjoy the underlying projects and want to buy Nick Craig-Wood a beer/coffee, feel free to use the website's [sponsor links](https://rclone.org/sponsor/)

## 🍻 Buy **[PostgreSQL](https://www.postgresql.org)** a beer/coffee? :)

If you enjoy the underlying projects and want to buy PostgreSQL a beer/coffee, feel free to use the [sponsor link](https://www.postgresql.org/about/donate/)

## ✅ GitHub Workflow Status

![GitHub Workflow Status](https://img.shields.io/github/actions/workflow/status/I-am-PUID-0/DMB/docker-image.yml)
