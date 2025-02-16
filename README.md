# rosenpass-setup

## Installation:
1. Dependencies
```sh
sudo apt-get install libsodium-dev libclang-dev cmake pkg-config git build-essential
sudo apt-get --yes install wireguard
```
Install Rust
```sh
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
```
2. Rosenpass installation:
```sh
git clone https://github.com/rosenpass/rosenpass.git
cd rosenpass
git checkout v0.2.2
cargo build --release
sudo install target/release/rosenpass /usr/local/bin
sudo install rp /usr/local/bin
```

Note: If cargo build unsupported feature error, check which rust toolchain you are on & switch if needed:
```sh
rustup show
rustup override set stable 
```
## Setup with WireGuard:

1. Generate Keypairs:
Server:
```sh
rp genkey server.rosenpass-secret
rp pubkey server.rosenpass-secret server.rosenpass-public
```
Client:
```sh
rp genkey client.rosenpass-secret
rp pubkey client.rosenpass-secret client.rosenpass-public
```

2. Copy the `public` key directories to the other peer:
```sh
scp -r server@<server_ip>:~/server.rosenpass-public ~/
```
```sh
scp -r client@<client_ip>:~/client.rosenpass-public ~/
```

3. Start the client & server:
Server:
```sh
sudo rp exchange server.rosenpass-secret dev rosenpass0 listen 127.0.0.1:9999 \
peer client.rosenpass-public allowed-ips fe80::/64
```
Client:
```sh
sudo rp exchange client.rosenpass-secret dev rosenpass0 \
peer server.rosenpass-public endpoint 127.0.0.1:9999 allowed-ips fe80::/64
```
