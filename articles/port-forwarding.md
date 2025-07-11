# Port Forwarding

---

Port forwarding is a technique that allows a local server (player-hosted server) to be accessible over the internet (WAN). However, many Internet Service Providers (ISPs) block this functionality by default to reduce potential security risks. As a result, self-hosted game servers may be inaccessible to other players.

Some games do not include built-in relay or NAT traversal solutions to work around port forwarding restrictions, such as Minecraft, Terraria, and Valheim.

Below are common solutions to bypass NAT (Network Address Translation) restrictions and enable connectivity.

---

## Cloud Hosting Providers

Hosting your game server on a cloud platform that provides a public IP address is one of the most reliable methods.

Popular cloud service providers include:
- Amazon Web Services (AWS)
- Google Cloud Platform (GCP)
- Microsoft Azure

---

## Relay SDKs

Relay services act as intermediaries (bridges) between clients and servers, facilitating communication without direct port forwarding. While effective, they often introduce additional latency.

Popular relay solutions:
- Steamworks
- Epic Online Services (EOS)
- Unity Relay

---

## Reverse Proxy Services

A reverse proxy runs alongside the game server on the host machine and exposes it to the internet. This approach is conceptually similar to using a relay.

Common reverse proxy tools:
- Ngrok (TCP only)
- Playit.gg
- Hamachi

---

## Contacting Your ISP

If your ISP restricts NAT or blocks incoming connections, you may request a public IP address. Be aware that this might incur additional charges or require specific configuration.

---

## Solution Comparison

| Solution        | Pros                            | Cons                                                              |
|------------------|----------------------------------|-------------------------------------------------------------------|
| Cloud Hosting   | Lowest latency, reliable          | Higher cost, less control over the environment                    |
| Relay SDK       | Seamless integration, user-friendly | Additional latency, requires compatible transport layer           |
| Reverse Proxy   | Cost-effective, flexible           | Added latency, manual setup often required for players            |
| Public IP (ISP) | Works universally with games       | May involve extra cost and technical setup                        |
