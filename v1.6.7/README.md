# Chiselled InfluxDB v1.6.7

This directory contains the image recipes of Chiselled InfluxDB v1.6.7. These images are smaller in size,
hence less prone to vulnerabilities. Know more about chisel [here](https://github.com/canonical/chisel).

### Building the image(s)

> [!IMPORTANT]
> You need to have rockcraft installed in order to build any rock.  Check the [installation guide](https://documentation.ubuntu.com/rockcraft/en/latest/how-to/get-started/#getting-started)

Build the rock by running the following instruction:

```sh
$ rockcraft pack
```

Convert the rock to a Docker image using the `skopeo` tool:

```sh
$ rockcraft.skopeo --insecure-policy copy \
    oci-archive:influxdb_1.6.7_amd64.rock \
    docker-daemon:influxdb:1.6.7
```

### Run the image(s)

The image has the "influxd" binary as the entrypoint.

```sh
$ docker run -p 8086:8086 influxdb:1.6.7
2025-03-24T21:50:36.358Z [pebble] Started daemon.
2025-03-24T21:50:36.365Z [pebble] POST /v1/services 3.627196ms 202
2025-03-24T21:50:36.369Z [pebble] Service "influxdb" starting: /usr/bin/influxd
2025-03-24T21:50:37.374Z [pebble] GET /v1/changes/1/wait 1.008448644s 200
2025-03-24T21:50:37.374Z [pebble] Started default services with change 1.
```

### Pushing and Pulling measurements from the database

InfluxDB expose a HTTP server on port 8086, why is why we bind that port to our host machine
port 8086 with `-p` option in the docker command above.

We can now use that API to create a database, insert a data point and retrieve it,
on a separate cmd:

1. Create a database:

```sh
$ curl -i -XPOST http://127.0.0.1:8086/query --data-urlencode "q=CREATE DATABASE testdb"
```

2. Insert a data point:

```sh
$ data_point="test_measurement,tag_key=tag_value value=0.64 1434055562000000000"
$ curl -i -XPOST 'http://127.0.0.1:8086/write?db=testdb' --data-binary "${data_point}"
```

3. Retrieve the data point in JSON format:

```sh
$ curl -G 'http://127.0.0.1:8086/query?db=testdb' --data-urlencode 'q=SELECT * FROM "test_measurement"'
```
