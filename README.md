# rosenpass-setup

## Installation:
1. Dependencies
```sh
sudo apt-get install libsodium-dev libclang-dev cmake pkg-config git build-essential
sudo apt-get --yes install wireguard wireguard-tools resolvconf
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

Note: If cargo build results in unsupported feature error, check which rust toolchain you are on & switch if needed:
```sh
rustup show
rustup override set stable 
```
## Setup with WireGuard:

for a quick wg setup, check this: [wg.md](https://github.com/lakshya-chopra/rosenpass-setup/blob/main/wg.md)

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

3. Create `rosenpass0.conf` on both the peers:

Client:
```conf
[Interface]
PrivateKey = <secret-key>
Address = 10.10.10.2/24
ListenPort = 51820

[Peer]
PublicKey = zQB5oIxO53rxuBy6y83kfu/A33YG8r/HXpsh32Npv2c=
AllowedIPs = 10.10.10.3/32
Endpoint = 192.168.6.233:51820
PersistentKeepalive = 25
```
Server:
```conf
[Interface]
PrivateKey = <secret-key>
Address = 10.10.10.3/24
ListenPort = 51820

[Peer]
PublicKey = vp3n9ajZ7Bb6W7gTVYJkYd15DGi/eQQmVYr2s52MbWs=
AllowedIPs = 10.10.10.2/32
Endpoint = 192.168.6.232:51820
PersistentKeepalive = 25
```
The secret & public keys can be found at: 
1. Client:
   
sk: `client.rosenpass-secret/wgsk`
pk: `client.rosenpass-public/wgpk`

2. Server:
   
sk: `server.rosenpass-secret/wgsk`
pk: `server.rosenpass-public/wgpk`

[Turn off any other wg interface if running.]

Add the IPs to the `rosenpass0` device:
```sh
sudo ip a add 10.10.10.2 dev rosenpass0
```
```sh
sudo ip a add 10.10.10.3 dev rosenpass0
```

Run the below cmd on both the peers:
```sh
wg-quick up rosenpass0
```


![image](https://github.com/user-attachments/assets/41984a82-37ec-4d91-84f1-e7773c65b1c9)


4.  Test & Observe the connection:
Ping:
```sh
ping 10.10.10.3
```
```sh
watch -n 0.2 'sudo wg show all; sudo wg show all preshared-keys
```
to view the latest handshakes:
```sh
sudo wg show rosenpass0 latest-handshakes
```

```sh
sudo wg show rosenpass0 transfer
```

**PCAP:**

Captured at the endpoint IP:

![image](https://github.com/user-attachments/assets/bf0948bf-22da-4681-b1b9-7e82207a47aa)

This encrypted packet is however in plain-text form at `rosenpass0` (this is the case for `wg0` too).

![image](https://github.com/user-attachments/assets/ab4b5ab0-5920-422b-90e8-8e18b267f394)

This suggests that wg intercepts the packet at `eth0`, identifies the peer and verifies integrity via `Poly1305`. If the authentication succeeds, the packet is decrypted using `ChaCha20` and sent to the network stack (`wg0` or `rosenpass0`).
