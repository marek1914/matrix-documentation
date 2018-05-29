<h2 style="padding-top:0">GPIO</h2>

### Device Compatibility
<img class="creator-compatibility-icon" src="../../img/creator-icon.svg">
<img class="voice-compatibility-icon" src="../../img/voice-icon.svg">

GPIO can be used to communicate or receive input from analog and digital components.

* [Creator Pinout](/matrix-creator/resources/pinout.md)
* [Voice Pinout](/matrix-voice/resources/pinout.md)

### Initial Setup

To begin working with the GPIO you need to include these header files.

```language-cpp
#include "matrix_hal/gpio_control.h"
#include "matrix_hal/matrixio_bus.h"
```

You'll also need to setup the `MatrixIOBus` for hardware communication.

```language-cpp
matrix_hal::MatrixIOBus bus; // Makes a bus object for hardware communication
if (!bus.Init()) return false; // Initializes bus
```

Now we will create our `GPIOControl` object.

```language-cpp
matrix_hal::GPIOControl gpio; // Make GPIOControl object
everloop.Setup(&bus); // Specify the MatrixIOBus object that GPIO will use
```

### GPIO Functions

Each GPIO pin can either function as a digital I/0 pin or as a PWM pin. By default the GPIO pins are in I/0 mode.

The following code sets GPIO_0 to function as a digital I/0 pin.

```language-cpp
#define I/0 0
#define PWM 1
gpio.SetFunction(0 , I/0); // (uint16_t pinNumber, uint16_t function)
```

The following code sets GPIO_0 to function as a PWM pin.

```language-cpp
#define I/0 0
#define PWM 1
gpio.SetFunction(0 , PWM); // (uint16_t pinNumber, uint16_t function)
```

### GPIO Input

To get input from a GPIO pin first set the mode.

```language-cpp
#define INPUT 0
#define OUTPUT 1
gpio.SetMode(0 , INPUT); // (uint16_t pinNumber, uint16_t mode)
```

Alternatively you can set the mode for multiple pins at once.

```language-cpp
#define INPUT 0
#define OUTPUT 1
unsigned char inputPinList[8] = {0, 2, 4, 6, 8, 10, 12, 14};
gpio.SetMode(inputPinList, sizeof(inputPinList), INPUT); // (unsigned char *array, int length, uint16_t mode)
```

The code below reads GPIO pin 0.

```language-cpp
uint16_t GPIO_0_output = gpio.GetGPIOValue(0);
```

Alternatively you can read from all pins at once.
Each bit of the 16-bit integer represents a GPIO pin.

```language-cpp
uint16_t GPIO_ALL_output = gpio.GetGPIOValues();
```

### GPIO PWM Input
>Ensure that you have set the GPIO to function as PWM before continuing.
PLACEHOLDER

### GPIO I/0 Output

To output from a GPIO pin first set the mode.

```language-cpp
#define INPUT 0
#define OUTPUT 1
gpio.SetMode(0 , OUTPUT); // (uint16_t pinNumber, uint16_t mode)
```

Alternatively you can set the mode for multiple pins at once.

```language-cpp
#define INPUT 0
#define OUTPUT 1
unsigned char outputPinList[8] = {1, 3, 5, 7, 9, 11, 13, 15};
gpio.SetMode(outputPinList, sizeof(outputPinList), OUTPUT); // (unsigned char *array, int length, uint16_t mode)
```

The code below writes ON(1) to GPIO pin 0.

```language-cpp
#define OFF 0
#define ON 1
gpio.SetGPIOValue(0 , ON); // (uint16_t pinNumber, uint16_t value)
```

Alternatively you can write to multiple pins at once.

```language-cpp
#define OFF 0
#define ON 1
unsigned char outputPinList[8] = {1, 3, 5, 7, 9, 11, 13, 15};
gpio.SetMode(outputPinList, sizeof(outputPinList), ON); // (unsigned char *array, int length, uint16_t value)
```

### GPIO PWM Output
>Ensure that you have set the GPIO to function as PWM before continuing.
PLACEHOLDER

### Reference
