# GPIO

The GPIO driver on current version supports:<a href="https://github.com/matrix-io/matrix-creator-malos/blob/master/docs/gpio_diagram.jpg"><img src="https://github.com/matrix-io/matrix-creator-malos/blob/master/docs/gpio_diagram.jpg?raw=true" align="right" width="420" /></a>

* GPIO pin input
* GPIO pin output
* GPIO updates (state from all pins)

The driver follows the [CORE protocol](../index.md#protocol).

### GPIO electrical characteristics

* GPIO voltage: 0.60-4.10 VDC (<a href="https://github.com/matrix-io/matrix-creator-quickstart/wiki/Data-Sheets" target="_blank">details</a>)
* current 10mA max
* all GPIO pins need pullups

### 0MQ Port
```
20049
```
### Protocol buffers

```language-protobuf
// GPIO handler params
message GpioParams {
  // GPIO to config
  uint32 pin = 1;

  // GPIO mode input/output
  enum EnumMode {
    INPUT = 0;
    OUTPUT = 1;
  }
  EnumMode mode = 2;

  // GPIO value
  uint32 value = 3;

  // GPIO all values
  uint32 values = 4;
}
```
The message is defined in <a href="https://github.com/matrix-io/protocol-buffers/blob/master/matrix_io/malos/v1/driver.proto" target="_blank">driver.proto</a>.

### Keep-alives

If you start sending keep-alive messages it will start returning data every second so you can omit the configuration for this device.


### Errors

This driver reports errors when an invalid configuration is sent.


### Write

All pins on matrix creator start as inputs. For change to outputs the driver need message for each pin on OUTPUT mode. (see Javascript example below)


### Read

The driver will send a serialized message of integer *values* which represents of state from all GPIO pins [see figure 1](gpio_diagram.jpg). For example: *values=5* represents *101* (pin 0 on 1, pin 1 on 0 and pin 2 on 1).


This is a sample output given by the example described below.

```language-bash
$ node test_gpio.js 
Sending pings every 5 seconds
==> pin 0 set to: true
<== GPIO pins register:  1
```
(pin0 set on true and GPIO register return values field on 1, only pin 0 set 1)

```language-bash
$ node test_gpio.js 
Sending pings every 5 seconds
==> pin 0 set to: true
<== GPIO pins register:  101
```
(pin0 set on true and GPIO register return values field on 3 (binary 101) pin 0 on 1, pin 2 on 1 all then on 0)


### JavaScript example

Enhanced description of the <a href="https://github.com/matrix-io/matrix-creator-malos/blob/master/src/js_test/test_gpio.js" target="_blank">sample source code</a>.

First, define the address of the MATRIX Creator. In this case we make it be `127.0.0.1`
because we are connecting from the local host but it needs to be different if we
connect from another computer. There is also the base port reserved by CORE for
the Pressure driver.

```language-javascript
var creator_ip = '127.0.0.1'
var creator_gpio_base_port = 20013 + 36
```

#### Load the protocol buffers used in the example.

```language-javascript
var protoBuf = require("protobufjs");
// parse proto file
var protoBuilder = protoBuf.loadProtoFile('../../protocol-buffers/malos/driver.proto')
// Parse matrix_malos package (namespace).
var matrixMalosBuilder = protoBuilder.build("matrix_malos")
```

#### Subscribe to the errors reported by the driver (optional)

```language-javascript
var zmq = require('zmq')

var errorSocket = zmq.socket('sub')
errorSocket.connect('tcp://' + creator_ip + ':' + (creator_gpio_base_port + 2))
errorSocket.subscribe('')
errorSocket.on('message', function(error_message) {
  process.stdout.write('Message received: Pressure error: ' + error_message.toString('utf8') + "\n")
});
```

#### Configure GPIO pins modes and set values for GPIO outputs:
All the drivers are configured using the message `DriverConfig` (see <a href="https://github.com/matrix-io/protocol-buffers/blob/master/matrix_io/malos/v1/driver.proto" target="_blank">driver.proto</a>).
##### Output mode:

instance driver config object 
```language-javascript
  var config = new matrixMalosBuilder.DriverConfig
```

with GpioParams proto message set GPIO 0 to output mode 
```language-javascript
  var gpio_cfg_cmd = new matrixMalosBuilder.GpioParams
  gpio_cfg_cmd.set_pin(0)
  gpio_cfg_cmd.set_mode(matrixMalosBuilder.GpioParams.EnumMode.OUTPUT)
```

set GPIO pin value (0 or 1)
```language-javascript
  gpio_cfg_cmd.set_value(1);
```

set on config driver params, encode and send
```language-javascript
  config.set_gpio(gpio_cfg_cmd)
  configSocket.send(config.encode().toBuffer())
```

##### Input mode:

instance driver config object 
```language-javascript
  var config = new matrixMalosBuilder.DriverConfig
```

set 250 ms between updates for example: (optional, default 1 second)
```language-javascript
  config.delay_between_updates = .250
```

configure sending updates 2 seconds after pings (optional)
```language-javascript
  config.timeout_after_last_ping = 2.0
```

with `GpioParams` for example set GPIO 1 to input mode:
```language-javascript
  var gpio_cfg_cmd = new matrixMalosBuilder.GpioParams
  gpio_cfg_cmd.set_pin(1)
  gpio_cfg_cmd.set_mode(matrixMalosBuilder.GpioParams.EnumMode.INPUT)
```

set on config driver params, encode and send
```language-javascript
  config.set_gpio(gpio_cfg_cmd)
  configSocket.send(config.encode().toBuffer())
```

#### Subscribe to GPIO updates

Where is where the updates are received by subscribing to the `data update port` of the driver.
The subscription is initiated by the line `updateSocket.subscribe('')`.

```language-javascript
var updateSocket = zmq.socket('sub')
updateSocket.connect('tcp://' + creator_ip + ':' + (creator_gpio_base_port + 3))
updateSocket.subscribe('')
```

Register update callback:
```language-javascript
updateSocket.on('message', function(gpio_buffer) {
  var gpioData = new matrixMalosBuilder.GpioParams.decode(gpio_buffer)
  // output in binary format all 15 pins of GPIO
  console.log('<== GPIO pins register: ',dec2bin(gpioData.values))
});
```
The driver will send on callback a serialized message of integer *values* which represents of state from all GPIO pins

#### Keep-alive messages

An empty keep-alive message is sent to the driver every 2 seconds to make sure it keeps
sending data updates.

```language-javascript
var pingSocket = zmq.socket('push')
pingSocket.connect('tcp://' + creator_ip + ':' + (creator_gpio_base_port + 1))
// Ping the first time.
pingSocket.send(''); 
// Ping every 2 seconds (same to delay between updates)
setInterval(function() {
  pingSocket.send(''); // ping for new updates
}, 2000);
```

