# SoloDeck community app store

A community app store for [umbrelOS](https://umbrel.com) with one app in it:
**Tailscale Subnet Fix** (`solodeck-subnet-fw`).

It keeps LAN devices reachable over Tailscale after a reboot, so you can check
your miners from your phone when you're away from home.

## Before you start

This app only does something useful if you've already set up Tailscale on Umbrel
as a **subnet router**, advertising your home LAN range. That means:

1. The official **Tailscale** app installed from the Umbrel App Store. This app
   lists it as a dependency, so Umbrel will prompt you if it's missing.
2. Your LAN subnet advertised by that Tailscale node, and the route approved in
   the [Tailscale admin console](https://login.tailscale.com/admin/machines).
   See [Tailscale's subnet router guide](https://tailscale.com/kb/1019/subnets).

If you haven't done that, install Tailscale and set up subnet routing first.
This app fixes a problem that only appears once subnet routing is running.

## Adding the store

1. Open the **App Store** from the Umbrel dock.
2. Click the three dots in the top right and choose **Community App Stores**.
3. Paste this URL and click **Add**:

```
https://github.com/liviux1/umbrel-solodeck-store
```

## Installing the app

Community store apps don't show up in the main App Store categories, which is
the step people usually get stuck on. To find this one:

1. Three dots menu, then **Community App Stores** again.
2. Click **Open** next to **SoloDeck**.
3. Install **Tailscale Subnet Fix** from there.

There's no web UI. Once it's installed there's nothing to open, and the tile in
umbrelOS won't take you anywhere.

## Checking it works

From your phone, off Wi-Fi and on cellular with Tailscale connected, open a
device on your home LAN by its local IP, for example `http://192.168.1.50`. If
it loads, routing is working.

To confirm from the Umbrel terminal:

```
sudo iptables -S DOCKER-USER | grep tailscale0
```

You should see two ACCEPT rules naming `tailscale0`.

## What it actually fixes

With a Tailscale subnet router on Umbrel, Docker's `DOCKER-USER` chain drops the
forwarded traffic, so a phone on cellular can't reach anything on your home LAN.
Umbrel runs Tailscale in Docker, so this hits every Umbrel subnet router, and it
comes back every time Docker rebuilds its rules.

The app re-applies what that routing needs:

- `DOCKER-USER` ACCEPT both ways between `tailscale0` and your LAN interface
- `MASQUERADE` for the Tailscale range (`100.64.0.0/10`) out of the LAN interface
- IPv4 and IPv6 forwarding

It re-applies them every 30 seconds, so they survive reboots, power cuts, and app
updates, because umbrelOS owns the container lifecycle.

The LAN interface is read from the default route each time rather than hardcoded.
Rules pinned to an interface that has since gone down stop routing with no visible
error, so moving the box between Wi-Fi and wired used to break it silently.

## What it needs on your system

Worth knowing before you install anything from a community store, this one
included. The container runs privileged with `NET_ADMIN` and `NET_RAW` on the
host network, because editing the host's iptables rules is the whole job.

It stores nothing and listens on nothing. The manifest declares port 8245 only
because Umbrel requires a port; no process binds it. On first run it may fetch
`iptables` from Debian's package repos if the base image doesn't already have
it, which is the only time it touches the network.

The source is short and worth a read: [`docker-compose.yml`](solodeck-subnet-fw/docker-compose.yml).
