# Setup

Firstly you need to open the browser at the link <http://localhost:8091> and edit the settings for Zwave, MQTT and the Gateway.

## General

- **Auth**: Enable this to password protect your application. Default credentials are:
  - Username:`admin`
  - Password: `zwave`
- **Log enabled**: Enable logging for zwavejs2mqtt
- **Log level**: Set the log level (Error, Warn, Info, Verbose, Debug, Silly)
- **Log to file**: Enable this to store the logs to a file

### Device values configuration

The Device values configuration table can be found under [General](#general) section and can be used to create valueIds specific configurations for each device. This means that if you create an entry here this configuration will be applied to all device of the same type in your Network.

![Gateway values](../_images/gateway_values_table.png)

> [!NOTE]
> In order to appear in the dropdown list a device must have completed its interview so if you don't find it there please wait for this.
> If it is a Battery powered device try to manually wake up it

Properties of a **valueId configuration**:

- **Device**: The device type. Once scan is complete, the gateway creates an array with all devices types found in the network. A device has a `device_id` that is unique, it is composed by this node properties: `<manufacturerid>-<productid>-<producttype>`.
- **Value**: The valueId you want to configure
- **Device Class**: If the value is a multilevel sensor, a binary sensor or a meter you can set a custom `device_class` to use with home assistant discovery. Check [sensor](https://www.home-assistant.io/components/sensor/#device-class) and [binary sensor](https://www.home-assistant.io/components/binary_sensor/#device-class)
- **Topic**: The topic to use for this value. It is the topic added after topic prefix, node name and location. If gateway type is different than `Manual` this will be ignored
- **QoS**: If specified, overrides MQTT settings QoS level
- **Retain**: If specified, overrides MQTT settings retain flag
- **Post operation**: If you want to convert your value (valid examples: '/10' '/100' '*10' '*100')
- **Parse send**: Enable this to allow users to specify a custom `function(value,valueId,node,logger)` to parse the value sent to MQTT. The function must be sync
- **Parse receive**: Enable this to allow users to specify a custom `function(value,valueId,node,logger)` to parse the value received via MQTT. The function must be sync
- **Enable Poll**: Enable poll of this value by using zwave-js [pollValue](https://zwave-js.github.io/node-zwave-js/#/api/node?id=pollvalue)
- **Poll interval**: Seconds between two poll requests

## Zwave

- **Serial port**: The serial port where your controller is connected
- **Network key** (Optional): Zwave network key if security is enabled. The correct format is like the OZW key but without `0x` `,` and spaces: OZW: `0x5C, 0x14, 0x89, 0x74, 0x67, 0xC4, 0x25, 0x98, 0x51, 0x8A, 0xF1, 0x55, 0xDE, 0x6C, 0xCE, 0xA8` Zwavejs: `5C14897467C42598518AF155DE6CCEA8`
- **Log enabled**: Enable logging for zwave-js websocket server
- **Log level**: Set the log level (Error, Warn, Info, Verbose, Debug, Silly)
- **Log to file**: Enable this to store the logs to a file
- **Commands timeout**: Seconds to wait before automatically stop inclusion/exclusion
- **Hidden settings**: advanced settings not visible to the user interface, you can edit these by setting in the settings.json
  - `zwave.plugin` defines a js script that will be included with the `this` context of the zwave client, for example you could set this to `hack` and include a `hack.js` in the root of the app with `module.exports = zw => {zw.client.on("scan complete", () => console.log("scan complete")}`
  - `zwave.options` overrides options passed to the zwave js Driver constructor [ZWaveOptions](https://zwave-js.github.io/node-zwave-js/#/api/driver?id=zwaveoptions)

## Disable Gateway

Enable this to use Z2M only as a Control Panel

## MQTT

- **Name**: A unique name that identify the Gateway.
- **Host url**: The url of the broker. Insert here the protocol if present, example: `tls://localhost`. Mqtt supports these protocols: `mqtt`, `mqtts`, `tcp`, `tls`, `ws` and `wss`
- **Port**: Broker port
- **Reconnect period**: Milliseconds between two reconnection tries
- **Prefix**: The prefix where all values are published
- **QoS**: Quality Of Service (check MQTT specs) of outgoing packets
- **Retain**: The retain flag of outgoing packets
- **Clean**: Sets the clean flag when connecting to the broker
- **Store**: Enable/Disable persistent storage of packets (QoS > 0). If disabled in memory storage will be used but all packets stored in memory are lost in case of shutdowns or unexpected errors.
- **Allow self signed certs**: When using encrypted protocols, set this to true to allow self signed certificates (**WARNING** this could expose you to man in the middle attacks)
- **Ca Cert and Key**: Certificate Authority, Client Key and Client Certificate files required for secured connections (if broker requires valid certificates, this fields can be leave empty otherwise)
- **Auth**: Enable this if broker requires auth. If so you need to enter also a valid **username** and **password**.

## Gateway

- **Type**: This setting specify the logic used to publish Zwave Nodes Values in MQTT topics. At the moment there are 3 possible configuration, two are automatic (all values are published in a specific topic) and one needs to manually configure which values you want to publish to MQTT and what topic to use. For every gateway type you can set custom topic values, if gateway is not in 'configure manually' mode you can omit the topic of the values (the topic will depends on the gateway type) and use the table to set values you want to `poll` or if you want to scale them using `post operation`

  1. **ValueId Topics**: _Automatically configured_. The topic where zwave values are published will be:

     `<mqtt_prefix>/<?node_location>/<nodeId>/<commandClass>/<endpoint>/<property>/<propertyKey>`

     - `mqtt_prefix`: the prefix set in Mqtt Settings
     - `node_location`: location of the Zwave Node (optional, if not present will not be added to the topic)
     - `nodeId`: the unique numerical id of the node in Zwave network
     - `commandClass`: the command class number of the value
     - `endpoint`: the endpoint number (if the node has more then one endpoint)
     - `property`: the value [property](https://zwave-js.github.io/node-zwave-js/#/api/valueid)
     - `propertyKey`: the value [propertyKey](https://zwave-js.github.io/node-zwave-js/#/api/valueid)

  2. **Named Topics**: _Automatically configured_. The topic where zwave values are published will be:

     `<mqtt_prefix>/<?node_location>/<node_name>/<class_name>/<?endpoint>/<propertyName>/<propertyKey>`

     - `mqtt_prefix`: the prefix set in Mqtt Settings
     - `node_location`: location of the Zwave Node (optional, if not present will not be added to the topic)
     - `node_name`: name of the node, if not set will be `nodeID_<node_id>`
     - `class_name`: the valueId command class name corresponding to given command class number or `unknownClass_<class_id>` if the class name is not known
     - `?endpoint`: Used just with multi-instance devices. The main enpoint (0) will not have this part in the topic but other instances will have: `endpoint_<endpoint>`
     - `propertyName`: the value [propertyName](https://zwave-js.github.io/node-zwave-js/#/api/valueid)
     - `propertyKey`: the value [propertyKey](https://zwave-js.github.io/node-zwave-js/#/api/valueid)

  3. **Configured Manually**: _Needs configuration_. The topic where zwave values are published will be:

     `<mqtt_prefix>/<?node_location>/<node_name>/<value_topic>`

     - `mqtt_prefix`: the prefix set in Mqtt Settings
     - `node_location`: location of the Zwave Node (optional, if not present will not be added to the topic)
     - `node_name`: name of the node, if not set will be `nodeID_<node_id>`
     - `value_topic`: the topic you want to use for that value (taken from gateway values table).

- **Payload type**: The content of the payload when an update is published:

  - **JSON Time-Value**: The payload will be a JSON object like:

    ```json
    {
      "time": 1548683523859,
      "value": 10
    }
    ```

  - **Entire ValueId Object**
    The payload will contain all info of a value from Zwave network:

    ```js
    {
      id: "38-0-targetValue",
      nodeId: 8,
      commandClass: 38,
      commandClassName: "Multilevel Switch",
      endpoint: 0,
      property: "targetValue",
      propertyName: "targetValue",
      propertyKey: undefined,
      type: "number",
      readable: true,
      writeable: true,
      description: undefined,
      label: "Target value",
      default: undefined,
      genre: "user",
      min: 0,
      max: 99,
      step: undefined,
      unit: undefined,
      list: false,
      value: undefined,
      lastUpdate: 1604044669393,
    }
    ```

    Example of a valueId with `states`:

    ```js
    {
      id: "112-0-200",
      nodeId: 8,
      commandClass: 112,
      commandClassName: "Configuration",
      endpoint: 0,
      property: 200,
      propertyName: "Partner ID",
      propertyKey: undefined,
      type: "number",
      readable: true,
      writeable: true,
      description: undefined,
      label: "Partner ID",
      default: 0,
      genre: "config",
      min: 0,
      max: 1,
      step: undefined,
      unit: undefined,
      list: true,
      states: [
        {
          text: "Aeon Labs Standard Product",
          value: 0,
        },
        {
          text: "others",
          value: 1,
        },
      ],
      value: 0,
      lastUpdate: 1604044675644,
    }
    ```

  - **Just value**: The payload will contain only the row Numeric/String/Bool value

- **Use nodes name instead of numeric nodeIDs**: When gateway type is `ValueId` use this flag to force to use node names instead of node ids in topic.
- **Send Zwave Events**: Enable this to send all Zwave client events to MQTT. More info [here](#zwave-events)
- **Include Node info**: Adds in ValueId json payload two extra values with the Name: `nodeName` and Location `nodeLocation` for better graphing capabilities (useful in tools like InfluxDb,Grafana)
- **Ignore status updates**: Enable this to prevent gateway to send an MQTT message when a node changes its status (dead/sleep == false, alive == true)
- **Ignore location**: Enable this to remove nodes location from topics
- **Publish node details**: Creates an `nodeinfo` topic under each node's MQTT tree, with most node details. Helps build up discovery payloads.

## Home Assistant

- **WS Server**: Enable [zwave-js websocket server](https://github.com/zwave-js/zwave-js-server). This can be used by HASS [zwave-js integration](https://www.home-assistant.io/integrations/zwave_js) to automatically create entities
- **MQTT discovery**: Enable this to use MQTT discovery. This is an alternative to Hass Zwave-js integration. (more about this [here](/guide/homeassistant))
- **Discovery Prefix**: The prefix to use to send MQTT discovery messages to HASS
- **Retain Discovery**: Set retain flag to true in discovery messages
- **Manual Discovery**: Don't automatically send the discovery payloads when a device is discovered
- **Entity name template**: Custom Entity name based on placeholders. Default is `%ln_%o`
  - `%ln`: Node location with name `<location-?><name>`
  - `%n`: Node Name
  - `%loc`: Node Location
  - `%p`: valueId property (fallback to device type)
  - `%pk`: valueId property key (fallback to device type)
  - `%pn`: valueId property name (fallback to device type)
  - `%o`: HASS object_id
  - `%l`: valueId label (fallback to object_id)

## Save settings

Once finished press `SAVE` and gateway will start Zwave Network Scan, then go to 'Control Panel' section and wait until the scan is completed to check discovered devices and manage them.

Settings, scenes and Zwave configuration are stored in `JSON` files under project `store` folder that you can easily **import/export** for backup purposes.

## Poll values

Some legacy devices don't report all their values automatically and require polling to get updated values. In contrast to OZW, zwave-js does not automatically poll devices on a regular basis without user interaction. Polling can quickly lead to network congestion and should be used very sparingly and only where necessary.

zwavejs2mqtt allows you to configure scheduled polling on a per-value basis, which you can use to keep certain values updated.
It also allows you to poll individual values on-demand from your automations, which should be preferred over blindly polling all the time if possible.

> [!NOTE]
> Many values can only be polled together. For example `targetValue`, `currentValue` and `duration` for most of the switch-type CCs. Before enabling polling for all values of a node, users should check whether polling a single value already updates the desired other values too.

In order to enable Polling of specific values you need to go to Settings page, expand General section and add a new value to the [Gateway Values table](#gateway-values-table)

Now press on `NEW VALUE` to add a new value or on the `Pen Icon` in actions column of the table to open the dialog to Add/Edit a value. Select the device, the valueId, enable the `Enable Poll` flag and set a `Poll interval` in seconds:

![Edit value](../_images/edit_gateway_value.png)

Press now on `SAVE` to upload your new settings to the server and it will automatically handle the polling based on your settings.

## Config DB Updates

Since version 4.0.0 it's possible to update internal zwave-js devices config database on the fly directly from zwavejs2mqtt UI.

Updates are checked everyday at midnight but you can also check for new updates manually from the UI by clicking on the icon in top right corner, when an update is available a badge will show up next to the icon:

![Config update icon](../_images/config_updates_icon.png)

When you click on the icon, if there is an update available, a dialog like this is shown:

![Config update dialog](../_images/config_updates_dialog.png)

Just press on `INSTALL` and wait until you receive a feedback, if the update fails check logs to see more details about errors. If there is no update available you will see a `CHECK` button instead of `INSTALL` and by pressing it you will trigger a manual check.

### Inside docker containers

By default config updates work by checking the installed version of the module `@zwave-js/config`. Doing such updates inside docker containers requires volumes in order to keep them consistent. Therefore, when running on docker zwave-js will copy the embedded config DB into the `store/.config-db` folder. This folder is not visible/editable from the store ui and should not be touched. The folder path can be customized using the `ZWAVEJS_EXTERNAL_CONFIG` env var, check related [docs](/guide/env-vars) for more info.
