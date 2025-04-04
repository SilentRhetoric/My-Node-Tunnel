# My Node Tunnel

This tutorial will teach you how to set up Tailscale and Caddy in conjunction with a home-based Algorand node so that it can be accessed easily from anywhere in the world while remaining private and not exposed to the open internet.

These instructions were developed and tested using a node installed on Linux Debian 12.

## Server Setup

The host machine will be set up with an Algorand node, Tailscale to provide secure, remote VPN access, UFW for for a simple firewall, and Caddy to provide a reverse proxy in front of the node.

### Algorand Node

- Install NodeKit [https://nodekit.run](https://nodekit.run/) to manage a new or existing node
- The node data directory that contains `config.json` and `algod.token` can be seen with `./nodekit debug`
- In `config.json` set and note the EndpointAddress on which algod will listen for requests, such as `"EndpointAddress": "127.0.0.1:0"`
- Copy the algod token, a secret required to access the algod REST API endpoint from `algod.token`

### Tailscale on the Server

- Install Tailscale <https://tailscale.com/kb/1347/installation>
- Log into Tailscale and enable it
- In web dashboard, go to DNS and enable MagicDNS and HTTPS
- Note the Tailnet name and your machine’s domain name will be `machinename.tailnet-name.ts.net`
- Edit the Tailscale config file at `/etc/default/tailscaled` to include a variable that gives Caddy access to get certificates `TS_PERMIT_CERT_UID=caddy`.  See <https://tailscale.com/kb/1190/caddy-certificates> for more info.
- Generate a TLS certificate with `sudo tailscale cert machinename-tailnet.name.ts.net`

### UFW

Use the Uncomplicated Firewall (UFW) to protect the node system from malicious intrusion attempts.

- Installation instructions can be found at <https://help.ubuntu.com/community/UFW>
- Create allow rule for tailscale0 `sudo ufw allow in on tailscale0`
- See <https://tailscale.com/kb/1077/secure-server-ubuntu> for more info

### Fail2Ban

I also like to use fail2ban as additional protection against uninvited visitors. 

- Install fail2ban `apt install fail2ban`
- Full installation instructions can be found at <https://github.com/fail2ban/fail2ban?tab=readme-ov-file#installation>

### Caddy

Caddy is an excellent server written in Go that can be used to provide a reverse proxy from the tailnet address to the node.

- Install Caddy <https://caddyserver.com/docs/install#debian-ubuntu-raspbian>
- Edit the Caddyfile at `/etc/caddy/Caddyfile` to create the reverse proxy from the Tailscale machine address to the node and then `sudo systemctl reload caddy`. The file needs to be editable by Caddy, so don’t edit it with `sudo` or use `sudo -u caddy`.

```
machinename.tailnet-name-ts.net {
    reverse_proxy http://ipaddress:port
}
```

## Client Setup

### Tailscale on the Client

- Install Tailscale <https://tailscale.com/kb/1347/installation>
- Log into Tailscale and enable it

### Lora

- Go to <https://lora.algokit.io>
- In Settings click the `+ Create`  button to add a custom network
- Select your wallets. Set the Algod server to your node address [`https://machinename.tailnet-name.ts.net`](https://machinename.tailnet-name.ts.net) and set the port to 443 for HTTPS.
- If you have an indexer, enter its details here, or utilize Nodely’s indexer [`https://mainnet-idx.algonode.cloud`](https://mainnet-idx.algonode.cloud/) with port 443 and no token.

Now you should be able to use Lora's data querying and transaction submission capabilities through your own personal node from wherever you are in the world!

Happy NODLing!