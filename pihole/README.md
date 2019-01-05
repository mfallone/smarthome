# README - pihole

Download docker image: `docker pull pihole/pihole`

Run the dockerized instance (connect on http://${IP}:80/admin)

```bash
IP_LOOKUP="$(ip route get 8.8.8.8 | awk '{ print $NF; exit }')"  # May not work for VPN / tun0
IPv6_LOOKUP="$(ip -6 route get 2001:4860:4860::8888 | awk '{for(i=1;i<=NF;i++) if ($i=="src") print $(i+1)}')"  # May not work for VPN / tun0
IP="${IP:-$IP_LOOKUP}"  # use $IP, if set, otherwise IP_LOOKUP
IPv6="${IPv6:-$IPv6_LOOKUP}"  # use $IPv6, if set, otherwise IP_LOOKUP
DOCKER_CONFIGS="$(pwd)"  # Default of directory you run this from, update to where ever.

echo "### Make sure your IPs are correct, hard code ServerIP ENV VARs if necessary\nIP: ${IP}\nIPv6: ${IPv6}"
docker run -d \
    --name pihole \
    -p 53:53/tcp -p 53:53/udp \
    -p 67:67/udp \
    -p 80:80 \
    -p 443:443 \
    -v "${DOCKER_CONFIGS}/pihole/:/etc/pihole/" \
    -v "${DOCKER_CONFIGS}/dnsmasq.d/:/etc/dnsmasq.d/" \
    -e ServerIP="${IP}" \
    -e ServerIPv6="${IPv6}" \
    --restart=unless-stopped \
    --cap-add=NET_ADMIN \
    --dns=127.0.0.1 --dns=1.1.1.1 \
    pihole/pihole:latest

echo -n "Your password for https://${IP}/admin/ is "
docker logs pihole 2> /dev/null | grep 'password'

```

The directories `pihole` and `dnsmasq.d` are required for persistence.

### docker-compose

See [docker-compose](../docker-compose.yaml) for integration example.

* `ServerIP=` should be set to the host containers IP address.
* `WEBPASSWORD=` is generated automatically but can be set by using a docker-compose overrride file.
* HTTPS certificate paths are overridden 


## Enabling HTTPS

Mostly follow [this](https://discourse.pi-hole.net/t/enabling-https-for-your-pi-hole-web-interface/5771) guide.

`smarthome` is set up to use a dockerized certbot.  Create a `combined.pem` file
for `lighttpd` to use.  It will also need access to `fullchain.pem`.

```bash
BASE_DIR=/opt/smarthome

sudo cat ${BASE_DIR}/certbot/letsencrypt/live/pihole.example.com/privkey.pem \
         ${BASE_DIR}/certbot/letsencrypt/live/pihole.example.com/cert.pem | \
sudo tee ${BASE_DIR}/certbot/letsencrypt/live/pihole.example.com/combined.pem
```

Create a new file, `external.conf` and put the following:

```
$HTTP["host"] == "pihole.example.com" {
  # Ensure the Pi-hole Block Page knows that this is not a blocked domain
  setenv.add-environment = ("fqdn" => "true")

  # Enable the SSL engine with a LE cert, only for this specific host
  $SERVER["socket"] == ":443" {
    ssl.engine = "enable"
    ssl.pemfile = "/certs/combined.pem"
    ssl.ca-file =  "/certs/fullchain.pem"
    ssl.honor-cipher-order = "enable"
    ssl.cipher-list = "EECDH+AESGCM:EDH+AESGCM:AES256+EECDH:AES256+EDH"
    ssl.use-sslv2 = "disable"
    ssl.use-sslv3 = "disable"       
  }

  # Redirect HTTP to HTTPS
  $HTTP["scheme"] == "http" {
    $HTTP["host"] =~ ".*" {
      url.redirect = (".*" => "https://%0$0")
    }
  }
}

```

Add the new volumes to the pihole portion of `docker-compose.yaml`.  Make sure 
to create any required directories and replace _example.com_ with your domain.

```yaml
    - volumes:
      - /opt/smarthome/pihole/lighttpd/external.conf:/etc/lighttpd/external.conf
      - /opt/smarthome/certbot/letsencrypt/live/pihole.example.com/fullchain.pem:/certs/fullchain.pem:ro
      - /opt/smarthome/certbot/letsencrypt/live/pihole.example.com/combined.pem:/certs/combined.pem:ro
```
