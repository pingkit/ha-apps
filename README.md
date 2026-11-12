<p align="center">
  <picture>
    <source media="(prefers-color-scheme: dark)" srcset="pingkit-agent/dark_logo.png">
    <img src="pingkit-agent/logo.png" alt="PingKit Agent" width="250" height="100">
  </picture>
</p>

# PingKit apps for Home Assistant

PingKit Agent watches your own home network from the Home Assistant machine you already
keep running, and tells [PingKit](https://pingkit.app/) on your iPhone when something
changes.

[![Add this repository to your Home Assistant](https://my.home-assistant.io/badges/supervisor_add_addon_repository.svg)](https://my.home-assistant.io/redirect/supervisor_add_addon_repository/?repository_url=https%3A%2F%2Fgithub.com%2Fpingkit%2Fha-apps)

Or add `https://github.com/pingkit/ha-apps` by hand: Settings > Apps > Install app, then
Repositories in the menu at the top right. Then install **PingKit Agent**. Home Assistant
called apps add-ons before 2026.2, and the two words mean the same here.

| App | What it does |
|---|---|
| [PingKit Agent](pingkit-agent/) | Watches your own network from Home Assistant and alerts PingKit on your iPhone |

## What it does

- **Internet down and back.** Your iPhone hears when the internet at home goes down and
  when it comes back, with an outage timeline and a report you can send your provider.
- **New devices.** An alert when a device joins your network for the first time.
- **Devices going offline,** if you turn it on for a device.
- **Speed tests** on a schedule or when you ask from your iPhone.
- **Home Assistant entities** over MQTT once the Mosquitto broker is installed, and a page
  of its own in the sidebar.

Running the agent is free, and so are its sidebar page and alerts to services you run
yourself. Alerts on your iPhone need PingKit Guardian, a subscription in the iPhone app.

## What it needs

- Home Assistant OS or Supervised, on a 64-bit ARM board (a Raspberry Pi 4 or 5, for
  example) or a 64-bit Intel or AMD machine.
- [PingKit](https://apps.apple.com/app/id6759291812) on your iPhone to pair it: Monitor >
  Agents > Add an Agent shows the code to enter.

It runs on your network only while the Home Assistant machine is running. With Guardian,
PingKit's server notices when that machine or its connection goes quiet and tells your
iPhone.

## Privacy and security

- The app asks Home Assistant for one privilege, the host's network, so it can read the
  neighbour table and hear mDNS and SSDP. It runs with a custom AppArmor profile and no
  Supervisor or Home Assistant API role.
- It only watches your own network. Every check is a connect attempt, a ping or a
  discovery query.
- Security reports: support@pingkit.app, as published in
  [security.txt](https://pingkit.app/.well-known/security.txt).

More at [pingkit.app/agent](https://pingkit.app/agent/). Problems or ideas: open an issue
in this repository.
