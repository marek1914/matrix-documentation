<h2 style="padding-top: 0">Introduction</h2>

Programs created with HAL allow you to directly access sensors and components on the MATRIX device through C++. This guide will show you how to create and run an LED demo in HAL. The final result being a rainbow LED sequence.

## Creating A Program
>HAL programs are designed to be developed and executed on the MATRIX device.

Gain access to the terminal of your Raspberry Pi:

The following commands will create a demo.cpp file and open it using the nano editor.

```language-bash
touch demo.cpp
nano demo.cpp
```

You'll need to copy all the code below into the open nano editor. Then press `Ctrl-X` and `y` to save and close nano.

```language-cpp
#include <unistd.h>
#include <iostream>
#include <cmath>

#include "matrix_hal/everloop.h"
#include "matrix_hal/everloop_image.h"
#include "matrix_hal/matrixio_bus.h"

namespace hal = matrix_hal;

int main() {
  hal::MatrixIOBus bus; // Makes a bus object for hardware communication.

  if (!bus.Init()) return false; // Initializes Bus.

  hal::EverloopImage image1d(bus.MatrixLeds()); // Make everloopImage object

  hal::Everloop everloop; // Make everloop object

  everloop.Setup(&bus); // Specify the bus object that everloop will use.

  float counter = 0;
 
  const float freq = 0.375;

  while (true) {
    for (hal::LedValue &led : image1d.leds) {
      led.red = 0;
      led.green = 0;
      led.blue = 0;
      led.white = 0;
    }
    for (hal::LedValue &led : image1d.leds) { //Logic for sine wave rainbow.
      led.red = (std::sin(freq*counter + 0)*155+100)/10;
      led.green = (std::sin(freq*counter + M_PI/2)*155+100)/10;
      led.blue = (std::sin(freq*counter + M_PI)*155+100)/10;
      counter = counter + 0.51;
    }
    everloop.Write(&image1d);
    usleep(20000);
  }

  return 0;
}
```
## Compiling your Program
When compiling your program you must link the libmatrix_creator_hal.so file. With g++ the `-l` flag links a library and so `-lmatrix_creator_hal` links the libmatrix_creator_hal.so file.

```language-bash
g++ -o demo.out demo.cpp -std=c++11 -lmatrix_creator_hal
```
## Running your Program
Run the following command to start the demo program.

```language-bash
./demo.out
```

## Next Steps
View our [reference page](../reference) to see what you can do with MATRIX HAL.