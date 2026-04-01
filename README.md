# Windows Media Server Stack (Docker)

> This repository contains the configuration and setup instructions for a fully automated media server stack running on **Windows via Docker Desktop**. 
>
> It handles automated searching, downloading, renaming, and serving of TV shows and movies using an interconnected suite of *arr applications, secured locally behind a reverse proxy.

---

## 🛠️ The Tech Stack

* **Plex:** Media server front-end.
* **Sonarr:** TV show automation and management.
* **Radarr:** Movie automation and management.
* **Prowlarr:** Indexer manager (syncs search providers to Sonarr/Radarr).
* **FlareSolverr:** Proxy to bypass Cloudflare protection on indexers.
* **qBittorrent:** Torrent download client.
* **Nginx Proxy Manager (NPM):** Reverse proxy for easy local URLs (e.g., `sonarr.local`).

---

## 📋 Prerequisites

1. **Docker Desktop** installed on Windows.
2. A dedicated folder on your drive (e.g., `F:\PLEXER`).
3. Your Plex Claim Token (get a fresh one at plex.tv/claim right before starting).
4. A valid `docker-compose.yml` file placed in your root folder.

---

## 📂 Step 1: Folder Structure & Startup

Create your root directory and place your `docker-compose.yml` file inside it. Docker will automatically create the subfolders when you start the stack.

### Directory Layout:

```text
F:\PLEXER\
├── docker-compose.yml
├── downloads/
├── media/
│   ├── movies/
│   └── tv/
├── plex/
├── prowlarr/
├── qbittorrent/
├── radarr/
└── sonarr/
```

### Spin Up the Stack:

Open a terminal or PowerShell in your `F:\PLEXER` folder and run:

```bash
docker-compose up -d
```

---

## ⚙️ Step 2: Application Configuration

### qBittorrent (`localhost:8082`)

* **Login:** Find the temporary password by running `docker logs qbittorrent` in your terminal. Username is `admin`.
* **Fix Unauthorized Error:** Go to **Tools -> Options -> Web UI**. Uncheck "Enable Cross-Site Request Forgery (CSRF) protection" and check "Bypass authentication for clients on localhost".
* **Set Categories:** In the left sidebar, right-click and create two categories: 
  * `tv-sonarr` (Save path: `/downloads`)
  * `movies-radarr` (Save path: `/downloads`)

### Prowlarr (`localhost:9696`)

* **Add FlareSolverr:** Go to **Settings -> Indexers -> Add (+)**. Select FlareSolverr. Set Host to `http://flaresolverr:8191`.
* **Add Indexers:** Add your preferred torrent trackers. Ensure the FlareSolverr tag is applied if needed to bypass Cloudflare.
* **Sync Apps:** Go to **Settings -> Apps -> Add Sonarr and Radarr**. Use `http://sonarr:8989` and `http://radarr:7878` as the Prowlarr Sync URLs. Grab their respective API keys from their Settings -> General pages.

### Sonarr & Radarr Root Folders

* **Sonarr (`localhost:8989`):** Go to **Settings -> Media Management -> Add Root Folder** -> Select `/tv`.
* **Radarr (`localhost:7878`):** Go to **Settings -> Media Management -> Add Root Folder** -> Select `/movies`.

---

## 🔗 Step 3: The Crucial Windows Connections

### Connect the Download Client

In both Sonarr and Radarr, go to **Settings -> Download Clients -> Add (+) -> qBittorrent**. 
* Set the Host to `qbittorrent`
* Set the Port to `8082`
* Set the Category to `tv-sonarr` (for Sonarr) and `movies-radarr` (for Radarr).

### ⚠️ Windows Remote Path Mapping (Mandatory)

Because Docker translates Windows paths differently, you must map the paths so Sonarr/Radarr can import finished downloads. In Sonarr/Radarr, go to **Settings -> Download Clients**. Scroll to Remote Path Mappings and click the (+).

* **Host:** `qbittorrent`
* **Remote Path:** `/downloads/`
* **Local Path:** `/downloads/`

### Link to Plex

In Sonarr/Radarr, go to **Settings -> Connect -> Add (+) -> Plex Media Server**. 
* Set Host to `plex`
* Set Port to `32400`
* Authenticate your account and check "On Download" and "On Upgrade".

---

## 🌐 Step 4: Beautiful Local URLs (Reverse Proxy)

Instead of using ports, you can use Nginx Proxy Manager to create clean local URLs.

### 1. Edit Windows Hosts File

Open Notepad as **Administrator**. Open `C:\Windows\System32\drivers\etc\hosts`. Add the following to the bottom of the file and save:

```text
# Media Lab Services
127.0.0.1    sonarr.local
127.0.0.1    radarr.local
127.0.0.1    plex.local
127.0.0.1    qbit.local
127.0.0.1    prowlarr.local
```

### 2. Configure Nginx Proxy Manager (`localhost:81`)

Log in with `admin@example.com` and password `changeme`. Go to **Hosts -> Proxy Hosts -> Add Proxy Host**. Add an entry for each service using the Docker container names. **Ensure Websockets Support is checked for all of them.**

| Domain Name | Scheme | Forward Hostname / IP | Forward Port |
| :--- | :--- | :--- | :--- |
| `sonarr.local` | `http` | `sonarr` | 8989 |
| `radarr.local` | `http` | `radarr` | 7878 |
| `qbit.local` | `http` | `qbittorrent` | 8082 |
| `prowlarr.local` | `http` | `prowlarr` | 9696 |
| `plex.local` | `http` | `plex` | 32400 |

> ✨ You can now access your entire lab simply by typing `http://radarr.local` (or any of the others) into your browser!