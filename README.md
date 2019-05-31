# README smarthome

Smarthome is made up of various docker images including:
* grafana
* home-assistant
* influxdb
* mosquitto
* pihole

See [docker-compose.yaml](docker-compose.yaml) for orchestration.

## Migrating to Recorder: Postgres

To set up a `postgres` recorder on a `backend` overlay network.

* In `docker-compose.yaml`,  check that networks are defined:

```yaml
networks:
 frontend:
 backend:

```

* Add a `sql` section and set it to the `backend` network

```yaml
  sql:
  	# ...
    networks:
    	- backend
```

* Make sure hass is registered to both networks

```yaml
  home-assistant:
  	# ...
    networks:
      - frontend
      - backend
```


