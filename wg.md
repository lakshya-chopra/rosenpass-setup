# wireguard quick setup

Add wg0 iface:
```sh
ip link add dev wg0 type wireguard
```

Generate keys:
```sh
wg genkey | tee privatekey | wg pubkey > publickey
```
Create a `wg0.conf` file:
```sh
sudo vim /etc/wireguard/wg0.conf
```
and paste the following at the client:
```conf
[Interface]
PrivateKey = <secret-key>
Address = 10.10.10.2/24
ListenPort = 51820

[Peer]
PublicKey = UuHHNNwZHNtF410w7mSPRdacuRO/eaA1NvJWqC1gdRc=
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
PublicKey = fx92rKzVcwAAOCg3Z5mZ3+SCYNtEFozcs/3Yr2R/u1E=
AllowedIPs = 10.10.10.2/32
Endpoint = 192.168.6.232:51820
PersistentKeepalive = 25
```
Now, we will set up the iface using `wg-quick`, we could have used `wg` as well (though it has some limitations; for example: we cannot specify `Address` in the conf).

Enable & start the `wg-quick` service
```sh
sudo systemctl enable wg-quick@wg0
sudo systemctl start wg-quick@wg0
```

```sh
sudo wg show
```
![image](https://github.com/user-attachments/assets/69c90db5-fafb-40f0-b510-423eed2d65cf)


Test
```sh
ping 10.10.10.3
```
