# RustDesk Self-Host (Docker & Synology Container Manager)

Self-hosted RustDesk server (`hbbs` ID/Rendezvous server + `hbbr` Relay server) ready for local Docker testing and deployment on a Synology NAS via **Container Manager**.

## Files

- [`compose.yml`](./compose.yml) — Docker Compose configuration for `hbbs` and `hbbr` with `ENCRYPTED_ONLY=1` enabled.
- [`.env.example`](./.env.example) — Optional environment variable template.
- [`.gitignore`](./.gitignore) — Excludes `data/` (which holds your private key `id_ed25519`) and `.env` from Git.

---

## 1. Local Docker Test (Windows / macOS / Linux)

1. Start the stack:
   ```bash
   docker compose up -d
   ```
2. Verify both containers are running:
   ```bash
   docker compose ps
   docker compose logs -f
   ```
3. Retrieve your generated public key:
   ```bash
   cat ./data/id_ed25519.pub
   ```
4. Test with a RustDesk client on the same machine or LAN:
   - Open RustDesk -> **Settings** -> **Network** -> **Unlock network settings**
   - **ID Server**: `localhost` (or your PC's LAN IP)
   - **Key**: paste the contents of `./data/id_ed25519.pub`
5. Stop the local test stack when done:
   ```bash
   docker compose down
   ```

---

## 2. Deploy on Synology NAS (Container Manager)

### Step A: Clone this repository onto your Synology NAS

SSH into your Synology NAS (or use a scheduled task / terminal) and clone into `/volume1/docker/rustdesk-server`:

```bash
cd /volume1/docker
git clone https://github.com/hoangprohigher/RustDeskSelfHost.git rustdesk-server
cd rustdesk-server
mkdir -p data
```

*(If your Synology NAS does not have `git` installed, you can install **Git Server** from Synology Package Center or download the ZIP from GitHub and extract it to `/docker/rustdesk-server/`.)*

### Step B: Create the Project in Synology Container Manager

1. Open **Container Manager** in DSM (DSM 7.2+).
2. Go to **Project** -> **Create**.
3. Set:
   - **Project Name**: `rustdesk-server`
   - **Path**: `/docker/rustdesk-server`
   - **Source**: **Use existing `compose.yml`** (Container Manager will automatically detect `compose.yml`).
4. Click **Next** -> **Done** to pull the image and start `rustdesk-hbbs` and `rustdesk-hbbr`.

### Step C: Router Port Forwarding & Synology Firewall

Forward these ports on your router to your Synology NAS's local IP address (and allow them in **DSM Control Panel -> Security -> Firewall** if enabled):

| Port | Protocol | Purpose |
| :--- | :--- | :--- |
| `21115` | TCP | NAT type test (`hbbs`) |
| `21116` | TCP & UDP | ID registration & heartbeat (`hbbs`) |
| `21117` | TCP | Relay service (`hbbr`) |
| `21118` | TCP | Web client support (optional) |
| `21119` | TCP | Web client relay support (optional) |

### Step D: Retrieve Your Public Key & Configure Clients

1. In DSM **File Station**, open `/docker/rustdesk-server/data/id_ed25519.pub` (or run `cat /volume1/docker/rustdesk-server/data/id_ed25519.pub` via SSH).
2. Copy the public key string.
3. On each RustDesk client:
   - Go to **Settings** -> **Network** -> **Unlock network settings**
   - **ID Server**: your Synology DDNS hostname or public IP (e.g., `my-nas.synology.me`)
   - **Relay Server**: leave blank (auto-discovered via ID server) or set the same hostname
   - **Key**: paste the public key from `id_ed25519.pub`
   - Click **Apply** — the status bar at the bottom of RustDesk should turn green (**Ready**).
