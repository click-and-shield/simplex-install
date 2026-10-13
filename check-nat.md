# Checking NAT configuration

In the following document:
- the variable `DOMAIN` represents the domain name you want to assign to your SimpleX server.
- the _server host_ is the host you want to install SimpleX to.

| Ubuntu               | Windows                  |
|----------------------|--------------------------|
| `DOMAIN=your_domain` | `SET DOMAIN=your_domain` |

**On the _server host_, make sure that your domain is well configured**

```bash
dig "${DOMAIN}"
``` 

**On the _server host_, check if you have a firewall**

```bash
sudo ufw status verbose
```

**On the _server host_, print your public IP address**

```bash
curl -4 https://api.ipify.org && echo
```

**On the _server host_, print your private IP address**

```bash
ip -4 addr
```

On the _server host_, make sure that no process listen on port 80

```bash
sudo ss -lntp 'sport = :80'
```

On the _server host_, make sure that no process listen on port 443

```bash
sudo ss -lntp 'sport = :443'
```

On the _server host_, make sure that no process listen on port 5223

```bash
sudo ss -lntp 'sport = :5223'
```

**Check the network address translation for HTTP**

On the _server host_:

```bash
sudo tcpdump -ni any 'tcp dst port 80'
```
On another host:

```bash
curl curl "http://${DOMAIN}"
```

**Check the network address translation for HTTPS**

On the _server host_:

```bash
sudo tcpdump -ni any 'tcp dst port 443'
```
On another host:

```bash
curl curl "https://${DOMAIN}"
```

**Check the network address translation for HTTPS fallback port**

On the _server host_:

```bash
sudo tcpdump -ni any 'tcp dst port 5223'
```
On another host:

```bash
curl curl "https://${DOMAIN}:5223/"
```

