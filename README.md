# wush

[![Go Reference](https://pkg.go.dev/badge/github.com/coder/wush.svg)](https://pkg.go.dev/github.com/coder/wush)

`wush` is a command line tool that lets you easily transfer files and open
shells over a peer-to-peer wireguard connection. It's similar to
[magic-wormhole](https://github.com/magic-wormhole/magic-wormhole) but:

1. No requirement to set up or trust a relay server for authentication.
1. Powered by Wireguard for secure, fast, and reliable connections.
1. Automatic peer-to-peer connections over UDP.
1. Endless possibilities; rsync, ssh, etc.

## Basic Usage

Install:

```bash
go install github.com/coder/wush/cmd/wush@latest
```

On the host machine:

```bash
$ wush receive
Picked DERP region Toronto as overlay home
Your auth key is:
    >  112v1RyL5KPzsbMbhT7fkEGrcfpygxtnvwjR5kMLGxDHGeLTK1BvoPqsUcjo7xyMkFn46KLTdedKuPCG5trP84mz9kx
Use this key to authenticate other wush commands to this instance.
05:18:59 Wireguard is ready
05:18:59 SSH server listening
```

On the client machine:

```bash
$ wush
┃ Enter the receiver's Auth key:
┃ > 112v1RyL5KPzsbMbhT7fkEGrcfpygxtnvwjR5kMLGxDHGeLTK1BvoPqsUcjo7xyMkFn46KLTdedKuPCG5trP84mz9kx
Auth information:
    > Server overlay STUN address:  Disabled
    > Server overlay DERP home:     Toronto
    > Server overlay public key:    [sEIS1]
    > Server overlay auth key:      [w/sYF]
Bringing Wireguard up..
Wireguard is ready!
Received peer
Peer active with relay  nyc
Peer active over p2p  172.20.0.8:44483
coder@colin:~$
```

## Technical Details

`wush` doesn't require you to trust any 3rd party authentication or relay
servers, instead using x25519 keys to authenticate incoming connections. Auth
keys generated by `wush receive` are separated into a couple parts:

```text
112v1RyL5KPzsbMbhT7fkEGrcfpygxtnvwjR5kMLGxDHGeLTK1BvoPqsUcjo7xyMkFn46KLTdedKuPCG5trP84mz9kx

+---------------------+------------------+---------------------------+----------------------------+
| UDP Address (1-19B) | DERP Region (2B) |  Server Public Key (32B)  |  Sender Private Key (32B)  |
+---------------------+------------------+---------------------------+----------------------------+
| 203.128.89.74:57321 |               21 | QPGoX1GY......488YNqsyWM= | o/FXVnOn.....llrKg5bqxlgY= |
+---------------------+------------------+---------------------------+----------------------------+
```

Senders and receivers communicate over what we call an "overlay". An overlay
runs over one of two currently implemented mediums; UDP or DERP. Each message
over the relay is encrypted with the sender's private key.

**UDP**: The receiver creates a NAT holepunch to allow senders to connect
directly. Wireguard nodes are exchanged peer-to-peer. This mode will only work
if the receiver doesn't have hard NAT.

**DERP**: The receiver connects to the closet DERP relay server. Wireguard nodes
are exchanged through the relay.

In both cases auth is handled the same way. The receiver will only accept
messages encrypted from the sender's private key, to the server's public key.

## Why create another file transfer tool?

Lots of great file tranfer tools exist, but they all have some limitations:

1. Slow speeds due to relay servers.
1. Trusting a 3rd party server for authentication.
1. Limited to only file transfers.

We sought to utilize advancements in userspace networking brought about by
Tailscale to create a tool that could solve all of these problems, and provide
way more functionality.

## Acknowledgements

1. [Tailscale](https://tailscale.com)
1. [Wireguard-go](https://github.com/WireGuard/wireguard-go)
