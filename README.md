# IPsec VPN Server on Docker with Libreswan

Docker image to run an IPsec VPN server, with both `IPsec/L2TP` and `Cisco IPsec`.

Based on Debian 10 (Buster) with [Libreswan](https://libreswan.org) (IPsec VPN software) and [xl2tpd](https://github.com/xelerance/xl2tpd) (L2TP daemon).

## How to use this image

### Environment variables

This Docker image uses the following variables, that can be declared in an `env` file:
Filename: `vpn.env`
```
VPN_IPSEC_PSK=your_ipsec_pre_shared_key
VPN_USER=your_vpn_username
VPN_PASSWORD=your_vpn_password
```

Additional VPN users are supported, and can be optionally declared in your `env` file like this. Usernames and passwords must be separated by spaces, and usernames cannot contain duplicates. All VPN users will share the same IPsec PSK.

```
VPN_ADDL_USERS=additional_username_1 additional_username_2
VPN_ADDL_PASSWORDS=additional_password_1 additional_password_2
```

**Note:** In your `env` file, DO NOT put `""` or `''` around values, or add space around `=`. DO NOT use these special characters within values: `\ " '`. A secure IPsec PSK should consist of at least 20 random characters.

All the variables to this image are optional, which means you don't have to type in any environment variable, and you can have an IPsec VPN server out of the box! Read the sections below for details.

### Start the IPsec VPN server

Create a new Docker container from this image (replace `./vpn.env` with your own `env` file):

```
# docker-compose up -d
version: '3.2'
services:
  vpn:
    image: liuxuc63/vpn-libreswan
    restart: always
    env_file:
      - ./vpn.env
    ports:
      - "500:500/udp"
      - "4500:4500/udp"
      - "17:17/udp"
      - "1701:1701/udp"
    privileged: true
    hostname: ipsec-vpn-server
    container_name: ipsec-vpn-server
```

### Retrieve VPN login details

If you did not specify an `env` file in the `docker run` command above, `VPN_USER` will default to `vpnuser` and both `VPN_IPSEC_PSK` and `VPN_PASSWORD` will be randomly generated. To retrieve them, view the container logs:

```
docker logs ipsec-vpn-server
```

Search for these lines in the output:

```
Connect to your new VPN with these details:

Server IP: your_vpn_server_ip
IPsec PSK: your_ipsec_pre_shared_key
Username: your_vpn_username
Password: your_vpn_password
```


## Advanced usage

### Use alternative DNS servers

Clients are set to use [Google Public DNS](https://developers.google.com/speed/public-dns/) when the VPN is active. If another DNS provider is preferred, define `VPN_DNS_SRV1` and optionally `VPN_DNS_SRV2` in your `env` file, then follow instructions above to re-create the Docker container. For example, if you wish to use [Cloudflare's DNS service](https://1.1.1.1):

```
VPN_DNS_SRV1=1.1.1.1
VPN_DNS_SRV2=1.0.0.1
```
