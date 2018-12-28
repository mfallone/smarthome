# README - grafana

### docker-compose

See [docker-compose](../docker-compose.yaml) for example implementation.

__NOTE__: The `user: <uid>`  can be adjusted to reflect any user, or can be omitted.

### Configuration files

`grafana.ini` is set as the default configuration file (`/etc/grafana/grafana.ini`)
in docker-compose.

### Generate a certificate 

Certificates can be created using certbot and linked to the `/certs` path inside the container.

[certbot](../certbot) can be used to generate the certificate.  An example:
```bash
BASE_PATH=/opt/smarthome/certbot
docker run -it --rm --name certbot \
            -v "${BASE_PATH}/letsencrypt:/etc/letsencrypt" \
            -v "${BASE_PATH}/lib/letsencrypt:/var/lib/letsencrypt" \
            -v "${BASE_PATH}/conf:/conf" \
            certbot/dns-google certonly \
                -d grafana.example.com -d example.com \
                --preferred-challenges dns-01 \
                --server https://acme-v02.api.letsencrypt.org/directory \
                --dns-google-credentials /conf/dns-example.json

```
