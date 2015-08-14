# Introduction #

The most important part of a RC flying object is the failsafe routine.
Never, I repeat NEVER take off without a working failsafe routine.
If you do not know how to setup the failsafe or what your copter does if it looses tx signal - DO NOT FLY.
Nothing is more dangerous that an out of control multirotor. You have spinning blades that can hurt people.

The failsafe routine of multiwii determines what is happening if the connection between RX and TX is lost.

**Beware:** Some RX have a built in Failsafe routine, that prevent the FC to recognize a signal loss. Be sure to check the behaviour of you RX system BEFORE first take off.

# Details #

If a loss of TX Signal is detected the copter enters stable mode after a guard time if an ACC is present and applies a configured throttle for a given time before it shuts down the motors.

There are different options that control the behavior of the multirotor after loss of TX signal:

## #define FAILSAFE ##
Failsafe function activated if TX signal loss found.

## #define FAILSAFE\_PIN ##
On which pin should the FC check for a signal loss

## #define FAILSAVE\_DELAY ##
How long does the copter need to notice a loss of TX  signal.

## #define FAILSAVE\_OFF\_DELAY ##
How long does the copter try to land before switching off the motors.
If you set this value to 0 the copter will shut down immediately - sometimes the savest solution. Be aware, that the copter may drift and crash into anything while trying to land.

## #define FAILSAVE\_THROTTLE ##
How much throttle is applied while trying to land. This value is dependent on the weight of your copter, the strenght of your motors and on the size of your propellers. You need to test, which value is right to enter a descending movement.

# Features to come #

  * Failsafe position hold and land
  * Failsafe return home
  * Failsafe baro landing
  * Failsafe ultrasonic landing