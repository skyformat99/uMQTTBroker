# uMQTTBroker
MQTT Broker library for ESP8266 Arduino

You can start an MQTT broker in any ESP Arduino project. Just clone (or download the zip-file and extract it) into the libraries directory of your Arduino ESP8266 installation.

Thanks to Tuan PM for sharing his MQTT client library https://github.com/tuanpmt/esp_mqtt as a basis with us. The modified code still contains the complete client functionality from the original esp_mqtt lib, but it has been extended by the basic broker service.

The broker does support:
- MQTT protocoll versions v3.1 and v3.1.1 simultaniously
- a smaller number of clients (at least 8 have been tested, memory is the issue)
- retained messages
- LWT
- QoS level 0
- username/password authentication
 
The broker does not yet support:
- QoS levels other than 0
- many TCP(=MQTT) clients
- non-clear sessions
- TLS

## API

The broker is started by simply including:

```c
#include "uMQTTBroker.h"
```
and then calling
```c
bool MQTT_server_start(uint16_t portno, uint16_t max_subscriptions, uint16_t max_retained_topics);
```
in the "setup()" function. Now it is ready for MQTT connections on all activated interfaces (STA and/or AP). The MQTT server will run in the background and you can connect with any MQTT client. Your Arduino project might do other application logic in its loop.

Your code can locally interact with the broker using these functions:

```c
bool MQTT_local_publish(uint8_t* topic, uint8_t* data, uint16_t data_length, uint8_t qos, uint8_t retain);
bool MQTT_local_subscribe(uint8_t* topic, uint8_t qos);
bool MQTT_local_unsubscribe(uint8_t* topic);

void MQTT_server_onData(MqttDataCallback dataCb);
```

With these functions you can publish and subscribe topics as a local client like you would with any remote MQTT broker. The provided dataCb is called on each reception of a matching topic, no matter whether it was published from a remote client or the "MQTT_local_publish()" function.

Username/password authentication is provided with the following interface:

```c
typedef bool (*MqttAuthCallback)(const char* username, const char *password, struct espconn *pesp_conn);

void MQTT_server_onAuth(MqttAuthCallback authCb);

typedef bool (*MqttConnectCallback)(struct espconn *pesp_conn, uint16_t client_count);

void MQTT_server_onConnect(MqttConnectCallback connectCb);
```

If an *MqttAuthCallback* function is registered with MQTT_server_onAuth(), it is called on each connect request. Based on username, password, and optionally the connection info (e.g. the IP address) the function has to return *true* for authenticated or *false* for rejected. If a request provides no username and/or password these parameter strings are empty. If no *MqttAuthCallback* function is set, each request will be admitted.

The *MqttConnectCallback* function does a similar check for the connection, but it is called right after the connect request before the MQTT connect request is processed. This is done in order to reject requests from unautorized clients in an early stage. The number of currently connected clients (incl. the current one) is given in the *client_count* paramater. With this info you can reject too many concurrent connections.

If you want to force a cleanup when the broker as a WiFi client (WIFI_STA mode) has lost connectivity to the AP, call:
```c
void MQTT_server_cleanupClientCons();
```
This will remove all broken connections, publishing LWT if defined.

Sample: in the Arduino setup() initialize the WiFi connection (client or SoftAP, whatever you need) and somewhere at the end add these line:
```c
MQTT_server_start(1883, 30, 30);
```

You can find a sample sketch in the examples.
