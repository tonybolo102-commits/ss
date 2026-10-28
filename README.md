# noVNC Google Desktop Starter

A minimal browser UI for a **self-hosted noVNC session**. The browser connects to a VNC server through `websockify`; a desktop browser running inside that remote desktop can then open Google services.

> This is a starter, not a hosted remote desktop service. You provide and secure the remote Linux desktop, VNC server, and websockify endpoint.

## Architecture

`Browser → noVNC web client → WebSocket (websockify) → VNC server → Linux desktop + browser`

Google services are opened **inside the remote desktop**, not embedded in an iframe. Google commonly blocks framing/authentication flows, so use a normal browser in the remote desktop.

## Quick start (Docker Compose)

Requirements: Docker Engine + Docker Compose plugin.

1. Copy `.env.example` to `.env` and change `VNC_PASSWORD` to a long unique password.
2. Start the remote desktop:

   ```bash
   docker compose up -d
   ```

3. Open `http://localhost:6080/vnc.html` on the same machine.
4. Enter the VNC password from `.env`.
5. The desktop should show a lightweight Linux session. Open Chromium/Firefox and navigate to `https://www.google.com/`, Gmail, Drive, or another Google service.

The included `docker-compose.yml` uses a maintained community image (`dorowu/ubuntu-desktop-lxde-vnc`) for a quick local prototype. For production, pin a reviewed image digest/version and review its security posture.

## Use this minimal custom website

`site/index.html` is a minimal landing page that loads the upstream noVNC client from a CDN and connects to `ws://localhost:6080/websockify` by default. It is useful when the page is served on the same host as the local demo.

To run it, serve the `site` directory:

```bash
python3 -m http.server 8080 --directory site
```

Then open `http://localhost:8080`. If your backend is on a different host, edit `NOVNC_URL` in `site/app.js` to the full URL of the noVNC client page, or set the `data-novnc-url` attribute on the page. For a real deployment, serve everything over HTTPS and use `wss://`.

## Backend notes

The Compose demo exposes port `6080` for the desktop/noVNC web endpoint. The `websockify` WebSocket bridge is part of the image. If building your own backend, you need:
- a VNC server (for example TigerVNC) running on the remote desktop,
- `websockify` bridging a WebSocket port to the VNC TCP port,
- a browser-accessible noVNC static client.

Do not expose raw VNC (typically port 5900) to the public internet. Put access behind HTTPS, authentication, network restrictions/VPN, and preferably a reverse proxy. Set strong credentials, keep the host patched, and avoid shared demo credentials. A browser remote desktop can access signed-in Google accounts, so treat it like a sensitive workstation.

## Production checklist

- HTTPS + WSS
- Authentication in front of noVNC (SSO/OIDC or reverse-proxy auth)
- Restrict access by VPN/IP allowlist where appropriate
- Per-user isolated desktop containers/VMs
- Strong VNC password and no public raw VNC port
- Resource limits, logging, patching, and lifecycle cleanup
- Review Google account policies and avoid storing credentials in images
