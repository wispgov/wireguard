# Connecting to Cloudflare WARP with WireGuard

Cloudflare's WARP VPN uses a slightly modified version of the WireGuard protocol, but it remains backwards compatible with the normal WireGuard client software. This means you can connect to it on platforms which don't yet have an official WARP client, e.g. your computer or [EdgeOS-based router](https://github.com/Lochnair/vyatta-wireguard). Short Tutorial rewritten by Xin Snowflakes, and Fuzzing with the REST API of the Cloudflare Registration.

## Step 1

Generate a WireGuard keypair, as usual:

`wg genkey | tee private.key | wg pubkey > public.key`

## Step 2

Register the public half with Cloudflare, changing the fields as appropriate:

`curl -d '{"key":"PASTE_PUBLIC_KEY_HERE", "install_id":"", "warp_enabled":true, "tos":"2019-11-11T00:00:00.000+01:00", "type":"Windows", "locale":"en_GB"}' https://api.cloudflareclient.com/v0a737/reg | tee warp.json`

(`tos` should be the date 'eg: 11-11-2019, november 11, 2019' you read and agreed to their [terms of service](https://www.cloudflare.com/application/terms/))

## Step 3

Find your 'warp.json' tunnel endpoint.

`jq '.config.peers[0]' warp.json`

And the IP address associated for your endpoint in 'warp.json':

`jq '.config.interface.addresses' warp.json`

## Step 4

Construct your WireGuard config file. It should look something like this:

```
[Interface]
Address = 172.16.0.2/32 (this IP is designated with the 'warp.json' IP Address)
ListenPort = 53 (based on DNS vulnerability)
DNS = 1.1.1.1, 1.0.0.1, 8.8.8.8, 8.8.4.4
PrivateKey = xxxxxxxxxxxxxxxxxx (generated via genkey's PrivateKey file)

[Peer]
PublicKey = yyyyyyyyyyyyyyyyyy (the one found on the warp.json file)
AllowedIPs = 0.0.0.0/0 (universal allocation of allowed IP Addresses)
Endpoint = engage.cloudflareclient.com:2408 (can be replaced with the Public IP of the Server)
```

**Notes**
- It seems `ListenPort` must **not** be accessible from the outside, as this appears to cause problems, so just choose any unused port and firewall it off for now
- `PrivateKey` is yours from Step 1
- `PublicKey` is Cloudflare's from Step 3
- You only get one IPv4 so you'll need to use SNAT if you're doing this on a router
- I don't know what netmask they want for IPv6 so have left that out



( C ) - Xin Snowflakes, 09225205353
