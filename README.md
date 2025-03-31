# ChirpStack Simulator

ChirpStack Simulator is an open-source simulator for the [ChirpStack](https://www.chirpstack.io)
open-source LoRaWAN<sup>&reg;</sup> Network-Server (v4). It simulates
a configurable number of devices and gateways, which will be automatically
created on starting the simulation.

This project has been developed together with [TWTG](https://www.twtg.io/).

## Building

The recommended way to compile the simulator code is using [Docker Compose](https://docs.docker.com/compose/).
Example:

```bash
docker-compose run --rm chirpstack-simulator make clean build
```

The binary will be located under `build/chirpstack-simulator`.

## Configuration

For generating a configuration template, use the following command:

```bash
./build/chirpstack-simulator configfile > chirpstack-simulator.toml
```

### Example

```toml
[general]
# Log level
#
# debug=5, info=4, warning=3, error=2, fatal=1, panic=0
log_level=4


# ChirpStack configuration.
[chirpstack]

  # API configuration.
  #
  # This configuration is used to automatically create the:
  #   * Device profile
  #   * Gateways
  #   * Application
  #   * Devices
  [chirpstack.api]

  # JWT token.
  #
  # API key to connect to the ChirpStack API. This API key can created within
  # the ChirpStack web-interface.
  api_key="PUT_YOUR_API_KEY_HERE"

  # Server.
  #
  # This must point to the API interface of ChirpStack.
  # If the server is running on the same machine, keep this to the
  # default value.
  server="127.0.0.1:8080"

  # Insecure.
  #
  # Set this to true when the endpoint is not using TLS.
  insecure=true


  # MQTT integration configuration.
  #
  # This integration is used for counting the number of uplinks that are
  # published by the ChirpStack MQTT integration.
  [chirpstack.integration.mqtt]

  # MQTT server.
  server="tcp://127.0.0.1:1883"

  # Username.
  username=""

  # Password.
  password=""


  # MQTT gateway backend.
  [chirpstack.gateway.backend.mqtt]

  # MQTT server.
  server="tcp://127.0.0.1:1883"

  # Username.
  username=""

  # Password.
  password=""


# Simulator configuration.
[[simulator]]

# Tenant ID.
#
# It is recommended to create a new tenant for simulations.
tenant_id="PUT_YOUR_TENANT_ID_HERE"

# Duration.
#
# This defines the duration of the simulation. If set to '0s', the simulation
# will run until terminated.
duration="5m"

# Activation time.
#
# This is the time that the simulator takes to activate the devices. This
# value must be less than the simulator duration.
activation_time="1m"

  # Device configuration.
  [simulator.device]

  # Number of devices to simulate.
  count=1000

  # Uplink interval.
  uplink_interval="5m"

  # FPort.
  f_port=10

  # Payload (HEX encoded).
  payload="010203"

  # Frequency (Hz).
  frequency=868100000

  # Bandwidth (Hz).
  bandwidth=125000

  # Spreading-factor.
  spreading_factor=7

  # Gateway configuration.
  [simulator.gateway]

  # Min number of receiving gateways.
  min_count=3

  # Max number of receiving gateways.
  max_count=5

  # Event topic template.
  event_topic_template="eu868/gateway/{{ .GatewayID }}/event/{{ .Event }}"

  # Command topic template.
  command_topic_template="eu868/gateway/{{ .GatewayID }}/command/{{ .Command }}"


# Prometheus metrics configuration.
#
# Using Prometheus (and Grafana), it is possible to visualize various
# simulation metrics like:
#   * Join-Requests sent
#   * Join-Accepts received
#   * Uplinks sent (by the devices)
#   * Uplinks sent (by the gateways)
#   * Uplinks sent (by the ChirpStack MQTT integration)
[prometheus]

# IP:port to bind the Prometheus endpoint to.
#
# Metrics can be retrieved from /metrics.
bind="0.0.0.0:9000"
```

## Running the simulator

To start the simulator, execute the following command:

```bash
./build/chirpstack-simulator -c chirpstack-simulator.toml
```

When a duration has been configured, then the simulation will stop after
the given interval. Note that this does not terminate the process! This makes
it possible to still read Prometheus metrics after the simulation has been
completed.

Regardless if a duration has been configured or not, the simulator can be
terminated. When sending an interrupt signal once, the simulation will be
terminated and the simulator will clean up the created gateways, devices,
application and device-profile. When sending an interrupt for the second time,
the simulator will be terminated immediately.

## Prometheus metrics

The ChirpStack Simulator provides various metrics that can be collected using
[Prometheus](https://prometheus.io/) and visualized using [Grafana](https://grafana.com/).

* `device_uplink_count`: The number of uplinks sent by the devices
* `device_join_request_count`: The number of join-requests sent by the devices
* `device_join_accept_count`: The number of join-accepts received by the devices
* `application_uplink_count`: The number of uplinks published by the application integration
* `gateway_uplink_count`: The number of uplinks sent by the gateways
* `gateway_downlink_count`: The number of downlinks received by the gateways

## License

ChirpStack Simulator is distributed under the MIT license. See also
[LICENSE](https://github.com/brocaar/chirpstack-simulator/blob/master/LICENSE).
