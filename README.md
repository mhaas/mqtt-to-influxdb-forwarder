# IoT MQTT to InfluxDB forwarder #

This tool forwards IoT sensor data from an MQTT broker to an InfluxDB instance.

## MQTT topic structure ##

The topic structure should be path-like, where the first element in the hierarchy contains
the name of the sensor node. Below the node name, the individual measurements are published
as leaf nodes. Each sensor node can have multiple sensors.

The tool takes a list of node names and will auto-publish all measurements found
below these node names. Any measurements which look numeric will be converted to
a float.

### Example MQTT topic structure ###

A simple weather station with some sensors may publish its data like this:

    /weather/uv: 0 (UV indev)
    /weather/temp: 18.80 (Â°C)
    /weather/pressure: 1010.77 (hPa)
    /weather/bat: 4.55 (V)

Here, 'weather' is the node name and 'humidity', 'light' and 'temperature' are
measurement names. 0, 18.80, 1010.88 and 4.55 are measurement values. The units
are not transmitted, so any consumer of the data has to know how to interpret
the raw values.

## Translation to InfluxDB data structure ##

The MQTT topic structure and measurement values are mapped as follows:

- the measurement name becomes the InfluxDB measurement name
- the measurement value is stored as a field named 'value'.
- the node name is stored as a tag named sensor\_node'

### Example translation ###

The following log excerpt should make the translation clearer:

    DEBUG:forwarder.MQTTSource:Received MQTT message for topic /weather/uv with payload 0
    DEBUG:forwarder.InfluxStore:Writing InfluxDB point: {'fields': {'value': 0.0}, 'tags': {'sensor_node': 'weather'}, 'measurement': 'uv'}
    DEBUG:forwarder.MQTTSource:Received MQTT message for topic /weather/temp with payload 18.80
    DEBUG:forwarder.InfluxStore:Writing InfluxDB point: {'fields': {'value': 18.8}, 'tags': {'sensor_node': 'weather'}, 'measurement': 'temp'}
    DEBUG:forwarder.MQTTSource:Received MQTT message for topic /weather/pressure with payload 1010.77
    DEBUG:forwarder.InfluxStore:Writing InfluxDB point: {'fields': {'value': 1010.77}, 'tags': {'sensor_node': 'weather'}, 'measurement': 'pressure'}
    DEBUG:forwarder.MQTTSource:Received MQTT message for topic /weather/bat with payload 4.55
    DEBUG:forwarder.InfluxStore:Writing InfluxDB point: {'fields': {'value': 4.55}, 'tags': {'sensor_node': 'weather'}, 'measurement': 'bat'}

### Example InfluxDB query ###

    select value from bat;
    select value from bat where sensor_node = 'weather' limit 10;
    select value from bat,uv,temp,pressure limit 20; 

The data stored in InfluxDB via this forwarder are easily visualized with [Grafana](http://grafana.org/)

## License ##

See the LICENSE file.

## Versioning ##

[Semantic Versioning](http://www.semver.org)
