# PingKit Agent, Home Assistant app image (lane H8b; docs/headless-agent/packaging.md,
# "Home Assistant /data").
#
# The published image, unchanged, started as root. The Supervisor creates the add-on's /data
# as root (mode 0755) and has no option to hand it to another user, so the image's own user,
# 65532, cannot create its store there. Started as root with SUPERVISOR_TOKEN set, the agent
# itself, before it reads any input, cuts its capabilities to CHOWN, SETUID and SETGID, takes
# /data, becomes 65532 for good with every capability set empty and no_new_privs on, and
# checks that it is so (AddOnPrivileges.swift). Docker, the NAS files and the Pi never use
# this file: they run the published image as 65532 from its first instruction.
#
# The Supervisor builds this on the Home Assistant machine (config.yaml has no `image:`) and
# passes BUILD_VERSION, the add-on's version, which is the image tag. Nothing is compiled,
# copied or run here: one base and one USER line, held so by check-agent-packaging.py.
ARG BUILD_VERSION
FROM ghcr.io/pingkit/agent:${BUILD_VERSION}
USER 0:0
