# p2p_chat

Peer-to-peer chat in Rust over iroh gossip. No server: peers connect directly and messages broadcast to everyone subscribed to the room's topic.

## How it works

- A peer opens a room, which creates a gossip topic and prints an invitation ticket: a base32 string encoding the topic ID and the peer's addresses.
- Other peers join with that ticket. iroh handles the QUIC connections and NAT traversal, and the gossip layer spreads messages peer to peer once the network forms.
- Messages are JSON on the wire, two types: `AboutMe` announces a nickname, `Message` carries chat text. Each message includes a random nonce for deduplication.

## Run

Open a room:

```bash
cargo run -- --name alice open
```

It prints your endpoint ID and a ticket. Join from another terminal or machine:

```bash
cargo run -- --name bob join <ticket>
```

Optionally bind a specific port with `--bind-port <port>`.

Type a line and press Enter to broadcast. Incoming messages show the sender's nickname, or their truncated endpoint ID if they never announced one.

## Implementation notes

- Single binary (`src/main.rs`) on the tokio runtime.
- stdin is blocking, so input runs on its own thread feeding an mpsc channel, while the gossip subscription runs as an async task.
- Transport security is iroh's QUIC/TLS. There is no application-layer encryption, no persistence, and no direct messages: everything is room broadcast.

## Stack

iroh, iroh-gossip, tokio, serde/serde_json, clap, data-encoding, anyhow.
