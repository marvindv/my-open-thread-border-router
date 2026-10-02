# my-open-thread-border-router

Standalone OpenThread Border Router to be integrated into Home Assistant via ethernet (or probably wifi).

## Hardware

- [Sonoff Dongle Plus MG24](https://sonoff.tech/en-de/products/sonoff-zigbee-thread-usb-dongle-dongle-plus-mg24)
- Raspberry Pi 3B+

## Software

- Raspberry Pi OS Lite (64 bit) based on Debian Trixie (2026-06-18)
- [Docker](https://docs.docker.com/engine/install/debian/)
- [nelletto/hass-otbr-docker](https://github.com/nelletto/hass-otbr-docker)

## Setup

0. If you want to add this Border Router to an existing Thread network: Make sure that the credentials are send to Home Assistant via the companion app before starting.
1. Install Docker
2. Enable IPv6 support on local network.
3. Enable mDNS and IGMP snooping on the network.
4. Configured the docker host with the following sysctl (replace `eth0` with `wlan0` or a different interface, depending on what is used to connect to the local network):

   ```bash
   sudo tee /etc/sysctl.d/99-otbr.conf <<'EOF'
   net.ipv6.conf.all.disable_ipv6 = 0
   net.ipv4.ip_forward = 1
   net.ipv6.conf.all.forwarding = 1
   net.ipv6.conf.eth0.accept_ra_rt_info_max_plen = 64
   net.ipv6.conf.eth0.accept_ra = 2
   EOF
   sudo sysctl --system
   ```

   IPv6 forwarding is required so that other hosts (like the Matter server) are able to reach Thread devices. If forwarding is activated, `accept_ra` has to be set to `2` to still accept Router Advertisements, which might contain _Route Information Options_ for the OMR prefix of other Border Routers.

5. Look up the baudrate for the device, for the Sonoff Dongle Plus MG24 it is 460800.
6. On the docker host:
   ```bash
   # Find out the OpenThread RCP USB device path.
   ls -la /dev/serial/by-id/*
   # Find out the ethernet interface
   ip a
   # Create compose file, fill in the appropriate values for the env variables and start.
   mkdir hass-otbr-docker
   cd hass-otbr-docker
   vim docker-compose.yaml
   docker compose up -d
   ```
7. Goto `https://<my_ha_url>/config/thread` > ... > Add Open Thread Border Router and enter `http://<otbr_ip>:8081`
