# PingKit Agent

PingKit Agent watches your own network from your Home Assistant machine, which stays on,
and sends PingKit on your iPhone an alert when:

- the internet goes down (after about three minutes of failed checks) and when it comes back;
- a new device joins your network (after a one-day learning period);
- a device goes offline, if you turn on Device Offline alerts. A device is reported
  offline after three missed sweeps, which can take up to 45 minutes at the default sweep.

Running the agent is free. iPhone alerts need PingKit Guardian.

This is a Home Assistant app, which Home Assistant called an add-on before 2026.2.

## Pair it with your iPhone

1. In PingKit on your iPhone, open Agents > Add an Agent. It shows an 8-character code.
2. In this app's Configuration tab, paste the code into **Claim code** and save.
3. Start (or restart) the app. The iPhone shows the agent within a minute.

The code works once and expires 10 minutes after your iPhone shows it. You can clear it
after pairing. In Home Assistant, use the Claim code option rather than the status page's
code form.

## The status page

**Open Web UI** (or the sidebar panel, shown to administrators) shows the agent's status
page: pairing, the interface and gateway it watches, the last sweep, recent events and the
outage log. Device names and addresses are on the Devices and Timeline pages. In the
sidebar they open at once: your Home Assistant login already guards it, and its port answers
only Home Assistant. On your network they are behind a six-digit PIN: your iPhone shows it,
and the app's log prints it at every start. Besides pairing, the one thing these pages can
change is Unpair, which asks you to confirm first in both places, and on your network asks for
the PIN again.

The sidebar has a port of its own, which Home Assistant picks and which answers only Home
Assistant, behind your login. The same page is also on your network at
`http://<this machine's IP address>:7474/`, from private addresses only. The IP address
always works. The page also answers the machine's own name as the app sees it, listed on the
`local_names` line of the app's log at every start; it refuses any other name.

## Presence

If you have an MQTT broker, such as the Mosquitto broker app, the agent puts your network
into Home Assistant by itself. Home Assistant hands it the broker's address and login, so
there is nothing to type, and without a broker the agent runs as before.

You get one PingKit Agent device (Internet connectivity, device counts, the last outage and
an event you can use in automations) and one device for each device on your network. Phones,
tablets and anything on a private Wi-Fi address become a device tracker you can add to a
person; the rest become a connectivity sensor.

- A tracker reads home while the agent sees the device on your network, and away once it has
  gone unseen for three minutes. The agent does that waiting itself. A phone that sleeps its
  Wi-Fi may read away while it is in your pocket, so test an automation before you trust it.
- If the agent stops, every entity reads unavailable, not away, so a restart does not send the
  whole family out of the house.
- Each entity follows the agent's own record of the device rather than its MAC address.
  When you remove a device, its entities go too.
- A device on a rotating private address can appear as a new device when the address changes.
  For a device you track, choose Fixed in the iPhone's Wi-Fi settings.

Discovery and state messages are retained on your broker, so the broker keeps your device
names, and anyone who can read that broker can read them. They go nowhere else: the broker is
your own.

## What it can see, and the one privilege

The app uses the host's network (`host_network`). It has to: the neighbour (ARP) table,
Bonjour (mDNS) and UPnP (SSDP) announcements only exist on your real network, and inside
Home Assistant's internal network the agent would see Home Assistant and nothing else. It
asks for no other privilege: no added capabilities, no full access, and an AppArmor profile
of its own that limits it to the files and sockets it uses. That gives it a security rating
of 7 of 8; the point it loses is `host_network`. It keeps its state in the app's own `/data`
folder.

Home Assistant creates that folder owned by root, so the app starts as root for one
step: before it reads any setting from the network, it gives `/data` and its own files to
its unprivileged user, switches to that user for good, and gives up every capability. Its
log says so in its first line, `privileges_dropped`. The Docker and Raspberry Pi versions
never start as root.

Home Assistant builds the app on this machine when you install or update it. The build
adds one line to the published PingKit Agent image (start as root) and compiles nothing,
so it takes seconds.

Every probe is a connection attempt, a ping, or a discovery query to devices on your own
network. Beyond it, the agent's checks reach three fixed public addresses (1.1.1.1, 8.8.8.8
and 9.9.9.9 by default; set your own and it keeps to those), ask the first of them and your
router for PingKit's own server name, and fetch one PingKit address. That is how it tells
your router from your line.
The device list stays on this machine and the iPhone reads it over your network. An
alert may carry a device's name to your iPhone through Apple's push service; it is not
stored on the way.

## Backups

Home Assistant backs the app up while it runs, so the agent keeps watching. A backup includes
`/data`, and with it the agent's pairing key and the token it uses to send alerts, so a
restored backup brings the pairing back. Someone holding such a backup could act as this
agent, so keep backups as private as the machine itself.

A backup also carries the store and the last seven nightly snapshots, which is what the agent
fills from if a restored store turns out damaged. It leaves out crash reports, damaged files
set aside for support, and temporary files.

## Ping

With **Ping** on `auto`, the agent pings where the system allows unprivileged ping
sockets and otherwise uses ARP and TCP checks, which work everywhere. It never opens a
raw socket.

## Alerts to your own services

The **Notification targets** option sends the same alerts to services of your own, free and
with no iPhone needed: one URL per row, masked on screen. For example
`ntfy://192.168.1.5/home-net`, `tgram://bot-token/chat-id`, `discord://webhook-id/webhook-token`,
`pover://user-key@app-token`, `webhook+http://homeassistant.local:8123/api/webhook/your-id`,
or `apprise://192.168.1.5:8000/key` for anything else.

During an outage, a target on your own network hears at once. A target on the internet hears
once it is back, in one message with how long the outage lasted. The agent sends a POST to the
URL you gave and reads only whether it worked.

## Options

Every option has a default; most people set only **Claim code**. Options beat a `config.yaml`
in the data folder, and they take effect when the app restarts. The two settings that say
where to read other settings from, `PINGKIT_CONFIG_FILE` and `PINGKIT_CLAIM_CODE_FILE`, are
for Docker and are not options here.

The ports matter only if 7474 or 7475 is already taken on this machine. If you change
**Status page port**, set the same number for 7474 under Network on this page too, because
the watchdog follows that entry. **Listen address** is the status page's address; the iPhone
connection (7475) always listens on every interface. Leave it at its default unless the
status page documentation says otherwise.

**Status page** off also closes the sidebar.

Alert rules (how far each kind of alert goes, quiet hours, skip repeats, a device's sleep
window) come from the iPhone, or, with no iPhone, from a `rules:` section in the same
`config.yaml` in the data folder (`/data`). What the iPhone sets stays above it, and the
Diagnostics page shows where each rule came from. Every key is in
`docs/headless-agent/install.md` of the agent's repository, "Alert rules with no iPhone".

## Watchdog

Turn on Watchdog on the Info tab and Home Assistant restarts the app if it stops answering.
It asks `/livez`, which reads only the agent's own loops. An internet outage, which the agent
is there to record, does not make it restart. The watchdog reaches the status port, so keep
**Status page** on and **Listen address** at its default while it is on: with either changed,
it finds nothing there and restarts the app over and over.

## Known limits

- The agent watches while Home Assistant runs. It sees nothing while the machine is off.
- Home Assistant OS on a Raspberry Pi needs the 64-bit image: the app is `aarch64` and `amd64`.
- Moving between this repository's branches installs a different app, which pairs again.

## Updating

Updates arrive through the app store like any other app.

## Support

Report a problem at https://github.com/pingkit/agent/issues with the "Copy diagnostics"
text from the status page's Diagnostics page. Please do not paste device names or addresses.
