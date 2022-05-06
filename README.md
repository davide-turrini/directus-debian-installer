# Directus installer (through Docker🐳 on Debian) 🚀
## 1. Pick up a VPS on Hetzner (see [Hetzner](https://console.hetzner.cloud/)) 💸

- Go to https://console.hetzner.cloud/ and select for a VPS with the following params
    - Debian10-1vCPU-2GB
    - 20GB SSD
    - 20T Traffic
    - Firewall
        - Incoming traffic: SSH TCP 22 OPEN 🚨
        - Outgoing traffic: ALL OPEN 🚨

and take note of the machine public IP address!

## 2. Configure DNS (see [doc](https://indomus.it/guide/come-installare-e-configurare-caddy-con-docker-su-raspbian-di-raspberry-pi/)) 💻

You can safely skip this step if you already have a stable DNS setup.


On [DuckDNS](https://www.duckdns.org/captcha), sign in with your account, copy the 'token' and use it to configure DuckDNS with the following HTTP GET REQUEST

```
https://www.duckdns.org/update?domains={YOURVALUE}&token={YOURVALUE}[&ip={YOURVALUE}][&ipv6={YOURVALUE}][&verbose=true][&clear=true]
```

#### HTTP API Specification
The update URL can be requested on HTTPS or HTTP. It is recommended that you always use HTTPS
We provide HTTP services for unfortunate users that have HTTPS blocked
You can update your domain(s) with a single HTTPS get to DuckDNS

The domain can be a single domain - or a comma separated list of domains.
The domain does not need to include the .duckdns.org part of your domain, just the subname.
If you do not specify the IP address, then it will be detected - this only works for IPv4 addresses
You can put either an IPv4 or an IPv6 address in the ip parameter
If you want to update BOTH of your IPv4 and IPv6 records at once, then you can use the optional parameter ipv6
to clear both your records use the optional parameter clear=true
A normal good response is

```
OK
```
A normal bad response is
```
KO
```
if you add the &verbose=true parameter to your request, then OK responses have more information

```
OK 
127.0.0.2 [The current IP address for your update - can be blank]
2002:DB7::21f:5bff:febf:ce22:8a2e [The current IPV6 address for your update - can be blank]
UPDATED [UPDATED or NOCHANGE]
```

#### HTTP Parameters
- **domains** - REQUIRED - comma separated list of the subnames you want to update
- **token** - REQUIRED - your account token
- **ip** - OPTIONAL - if left blank we detect IPv4 addresses, if you want you can supply a valid IPv4 or IPv6 address
- **ipv6** - OPTIONAL - a valid IPv6 address, if you specify this then the autodetection for ip is not used
- **verbose** - OPTIONAL - if set to true, you get information back about how the request went
- **clear** - OPTIONAL - if set to true, the update will ignore all ip's and clear both your records




### Special no-parameter request format
Some very basic routers can only make requests without parameters
For these requirements the following request is possible

```
https://duckdns.org/update/{YOURDOMAIN}/{YOURTOKEN}[/{YOURIPADDRESS}]
```

Restrictions
- **YOURDOMAIN** - REQUIRED - only a single subdomain
- **YOURTOKEN** - REQUIRED - your account token
- **YOURIPADDRESS** - OPTIONAL - if left blank we detect IPv4 addresses, if you want to over-ride this, with a valid IPv4 or IPv6 address

## 3. Configure Caddy server (see [doc](https://caddyserver.com/)) ⚙️
to create the Directus/Caddy files / folder type the following..

```
mkdir -m 777 ~/uploads
mkdir -m 777 ~/database
touch ~/database/db.sqlite
chmod 777 -R ~/database
mkdir -p ~/caddy/data
nano ~/caddy/Caddyfile
```

and in the nano editor now add this

```

(nocors) {
  @options {
    method OPTIONS
  }

  header {
      Access-Control-Allow-Origin *
      Access-Control-Allow-Credentials true
      Access-Control-Allow-Methods *
      Access-Control-Allow-Headers *
  }
  respond @options 204
}

bieffe.duckdns.org { 
  reverse_proxy directus:8055

  import nocors
}
```


## 4. Install docker (see [doc](https://docs.docker.com/engine/install/debian/)) ⬇️
```bash
curl https://raw.githubusercontent.com/davide-turrini/directus-debian-installer/master/install.sh > install.sh && chmod +x install.sh && sudo ./install.sh && rm -rf install.sh
```

## 5. Download docker compose file (see [doc](https://strapi.io/documentation/developer-docs/latest/setup-deployment-guides/installation/docker.html#creating-a-strapi-project)) ⬇️
```bash
curl https://raw.githubusercontent.com/davide-turrini/directus-debian-installer/master/docker-compose.yaml > docker-compose.yaml
```

and change the `PUBLIC_URL` conf to your DOMAIN NAME.

## 6. Build image (see [doc](https://strapi.io/documentation/developer-docs/latest/setup-deployment-guides/installation/docker.html#creating-a-strapi-project)) 🛠️
```bash
sudo docker-compose pull
```

## 7. Create container (see [doc](https://strapi.io/documentation/developer-docs/latest/setup-deployment-guides/installation/docker.html#creating-a-strapi-project)) ⏯️
Execute Docker image detaching the terminal
```bash
sudo docker-compose up -d
```
Execute Docker image without detaching the terminal
```bash
sudo docker-compose up
```
