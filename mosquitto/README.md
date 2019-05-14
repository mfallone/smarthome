# README Mosquitto

Docker Setup
```
docker run -it \
	-p 1883:1883 -p 9001:9001 \
	-v mosquitto.conf:/mosquitto/config/mosquitto.conf \
	-v data:/mosquitto/data \
	-v log:/mosquitto/log \
	-v certs:/mosquitto/certs \
	eclipse-mosquitto
```

See [docker-compose](../docker-compose.yaml) file mqtt block for example.

