# SoloDeck community app store

An [umbrelOS](https://umbrel.com) community app store with one app in it:
**Tailscale Subnet Fix** (`solodeck-subnet-fw`).

## Adding the store

In umbrelOS, open the App Store, click the three dots in the top right, choose
**Add community store**, and paste:

```
https://github.com/liviux1/umbrel-solodeck-store
```

The app then appears under Networking. It needs the official Tailscale app
installed first.

## What the app fixes

If you run a Tailscale subnet router on Umbrel, Docker's `DOCKER-USER` chain
drops the forwarded traffic, so a phone on cellular can't reach anything on your
home LAN. That breaks monitoring LAN miners from away, which is what
[SoloDeck](https://solodeck.app) needs.

This app re-applies the rules that make that routing work:

- `DOCKER-USER` ACCEPT both ways between `tailscale0` and your LAN interface
- `MASQUERADE` for the Tailscale range (`100.64.0.0/10`) out of the LAN interface
- IPv4 and IPv6 forwarding

It re-asserts them every 30 seconds, so they survive reboots, power cuts, and
app updates, since umbrelOS owns the container lifecycle.

The LAN interface is read from the default route on every pass rather than
hardcoded. Rules pinned to an interface that's since gone down stop routing
without any visible error, so moving the box between Wi-Fi and wired used to
break it silently.

There's no web UI. Once it's running there's nothing to open.
