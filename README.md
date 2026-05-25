# Jelly Stack Docker Compose on MacVLAN

This repository contains a Docker Compose setup for running a Jellyfin media stack as LAN-native services using a macvlan network with static IPs.

Included services:

- Jellyfin
- qBittorrent
- Sonarr
- Radarr
- Lidarr
- Prowlarr
- Jellyseerr
- Flaresolverr

Each container gets its own LAN IP address.

---

## Setup

### 1. Create the macvlan

Example, adjust interface, subnet and gateway to your environment:

```bash
docker network create -d macvlan \
  --subnet=192.168.1.0/24 \
  --gateway=192.168.1.1 \
  -o parent=eth0 \
  lan_macvlan
```

---

### 2. Create the media folders

Example:

```bash
sudo mkdir -p /srv/media/movies
sudo mkdir -p /srv/media/shows
sudo mkdir -p /srv/media/music
sudo mkdir -p /srv/media/downloads
sudo mkdir -p /srv/media2
```

Set ownership to the user used by the containers.

This Compose uses:

```text
PUID=1000
PGID=1000
```

So:

```bash
sudo chown -R 1000:1000 /srv/media
sudo chown -R 1000:1000 /srv/media2
```

---

### 3. Create the config folders

```bash
mkdir -p ./configs/jellyfin
mkdir -p ./configs/qbittorrent
mkdir -p ./configs/sonarr
mkdir -p ./configs/radarr
mkdir -p ./configs/lidarr
mkdir -p ./configs/prowlarr
mkdir -p ./configs/jellyseerr
mkdir -p ./configs/flaresolverr
mkdir -p ./cache/jellyfin
```

---

### 4. Create the `.env` file

Create a file named `.env` next to `docker-compose.yaml`.

This file is required and excluded via `.gitignore`.

```env
###############################################################################
# Jelly Stack – environment configuration
#
# This file is read by Docker Compose via ${VAR} expansion.
# It MUST live in the same directory as docker-compose.yaml.
# Do NOT commit this file to git.
###############################################################################

### ---------------------------------------------------------------------------
### Networking
### ---------------------------------------------------------------------------

# Static IPv4 addresses assigned to each container on the macvlan network.
# These IPs must:
#   - be inside the macvlan subnet
#   - be unused
#   - preferably be outside your router DHCP range

JELLY_FIN_IP=192.168.1.70
QBIT_TORRENT_IP=192.168.1.71
SONARR_IP=192.168.1.72
RADARR_IP=192.168.1.73
LIDARR_IP=192.168.1.74
PROWLARR_IP=192.168.1.75
JELLY_SEER_IP=192.168.1.76
FLARE_SOLVERR_IP=192.168.1.77


### ---------------------------------------------------------------------------
### Data paths
### ---------------------------------------------------------------------------

# Main media directory.
#
# Expected structure:
#   /srv/media/movies
#   /srv/media/shows
#   /srv/media/music
#   /srv/media/downloads

MEDIA_FOLDER=/srv/media

### ---------------------------------------------------------------------------
### Timezone
### ---------------------------------------------------------------------------

# Timezone used by the containers.
# List: https://en.wikipedia.org/wiki/List_of_tz_database_time_zones

JELLY_STACK_TZ=Europe/London
```

---

### 5. Start the stack

```bash
docker compose up -d
```

Check status:

```bash
docker compose ps
```

Check logs:

```bash
docker compose logs jellyfin
docker compose logs qbittorrent
docker compose logs sonarr
docker compose logs radarr
docker compose logs lidarr
docker compose logs prowlarr
docker compose logs jellyseerr
docker compose logs flaresolverr
```

---

## Accessing the services

From another machine on the LAN:

```text
Jellyfin:     http://<JELLY_FIN_IP>:8096
qBittorrent: http://<QBIT_TORRENT_IP>:8080
Sonarr:      http://<SONARR_IP>:8989
Radarr:      http://<RADARR_IP>:7878
Lidarr:      http://<LIDARR_IP>:8686
Prowlarr:    http://<PROWLARR_IP>:9696
Jellyseerr:  http://<JELLY_SEER_IP>:5055
Flaresolverr http://<FLARE_SOLVERR_IP>:8191
```

Example:

```text
http://192.168.1.70:8096
```

---

## Important MacVLAN note

With macvlan, containers behave like separate machines on your LAN.

That is the point.

But it also means the Docker host itself may not be able to reach the containers directly.

So this may work from your laptop:

```bash
curl http://192.168.1.70:8096
```

but fail from the Docker host.

That is normal macvlan behavior.

Fix it only if you actually need host-to-container access.

---

## qBittorrent setup

Open qBittorrent:

```text
http://<QBIT_TORRENT_IP>:8080
```

Set the download path to:

```text
/downloads
```

Sonarr, Radarr and Lidarr also mount the same downloads folder:

```text
/downloads
```

This matters.

Do not point qBittorrent to some random path that the Arr apps cannot see.

---

## Sonarr paths

Inside Sonarr:

```text
TV root folder: /tv
Downloads:      /downloads
```

---

## Radarr paths

Inside Radarr:

```text
Movie root folder: /movies
Downloads:         /downloads
```

---

## Lidarr paths

Inside Lidarr:

```text
Music root folder: /music
Downloads:         /downloads
```

---

## Prowlarr setup

Use Prowlarr to manage indexers.

Then connect Prowlarr to:

```text
Sonarr: http://<SONARR_IP>:8989
Radarr: http://<RADARR_IP>:7878
Lidarr: http://<LIDARR_IP>:8686
```

Use the API keys from each app.

---

## Jellyseerr setup

Open Jellyseerr:

```text
http://<JELLY_SEER_IP>:5055
```

Connect it to Jellyfin:

```text
http://<JELLY_FIN_IP>:8096
```

Then connect it to Sonarr and Radarr:

```text
Sonarr: http://<SONARR_IP>:8989
Radarr: http://<RADARR_IP>:7878
```

---

## Flaresolverr setup

Open:

```text
http://<FLARE_SOLVERR_IP>:8191
```

In Prowlarr, add Flaresolverr as an indexer proxy:

```text
http://<FLARE_SOLVERR_IP>:8191
```

Only use it for indexers that need it.

Do not route everything through it for no reason.

---

## Updating

Pull new images:

```bash
docker compose pull
```

Restart with the new images:

```bash
docker compose up -d
```

Clean unused images:

```bash
docker image prune
```

---

## Stopping

Stop the stack:

```bash
docker compose down
```

This removes the containers, but keeps configs and media.

Do not delete the `configs`, `cache` or media folders unless you actually want to wipe your setup.

---

## Credits

Jellyfin: https://jellyfin.org  
qBittorrent: https://www.qbittorrent.org  
Sonarr: https://sonarr.tv  
Radarr: https://radarr.video  
Lidarr: https://lidarr.audio  
Prowlarr: https://prowlarr.com  
Jellyseerr: https://github.com/Fallenbagel/jellyseerr  
Flaresolverr: https://github.com/FlareSolverr/FlareSolverr
