# C-PID
PID algorithm implemented in C.

## Overview
This is a simple, yet computationally-efficient implementation of a PID algorithm. It was discretized using *backward differentiation formula*, with the derivative component acting based on the measurement rather than on the error, thus, preventing *derivative kicks*.

It's important to notice that, for best results, the computation function must be called within a fixed time period of `t`. `t` is used in the algorithm's initialization in order to calculate the constants, which are directly multiplied by the current and past error/measurement data. This way, `t` should not be changed frequently, as any change would require a new costly calculation of the constants.

## Installation
Simply copy both `PID.c` and `PID.h` into an appropriate location within your project's source folders. There are no dependencies associated with this implementation of the PID algorithm.

## Usage
- The first step is including the library in your source code:
```C
#include "PID.h"
```

- Then, insantiate a `PID_t` object and initialize it:
```C
PID_t pid;
PID_init(&pid, reset, kp, ti, td, max_output, min_output, sample_period);
```
>Being `sample_period` the forementioned `t`. `reset` should be set to 1 in the first initialization and if the `PID_t` object is to be reused, and 0 otherwise.

- Now it's time to define a setpoint. Should the setpoint change, this function must be called again.
```C
void PID_set_setpoint(&pid, setpoint)
```

- The next step is calling the `compute` function, which takes the measurement as a parameter and yields the output value itself:
```C
void function_to_be_executed_in_a_fixed_period_of_time(PID_t* pid) {
  double measurement = read_data_from_some_sensor();
  double output = PID_compute(pid, measurement);
  write_to_some_actuator(output);
}
```

There are also implementations for setter functions. Be aware that, for maintaining a good computation performance, some of these shouldn't be called frequently, as they trigger constants recalculation, as explained previously. 

- Set PID parameters (Kp, Ti, Td):
```C
void PID_set_parameters(PID_t* pid, double Kp, double Ti, double Td);
```
***This triggers recalculation.*** Use this function to change the PID parameters on-the-go.

- Set output limits:
```C
void PID_set_limits(PID_t* pid, double max_output, double min_output);
```
Use this function to set the output limits of the algorithm. It's necessary to set according values, as this is crucial for preventing *PID integral windup*. Does not trigger recalculation.

- Set sample period:
```C
void PID_set_sample_period(PID_t* pid, double sample_period);
```
***This triggers recalculation.*** Use this function to set the fixed sample period, in seconds, the `compute` function will be called.

## Notes
If it's desired to not take the sampling time into account, simply define `t` as 1; ***however, this is not advised***.

It's possible to have several PID instances running at once. For each desired instance, one should instantiate a different `PID_t` object.

