# Experiment 10: Shared Resource Management using Counting Semaphore in FreeRTOS

## Objective

To implement a Counting Semaphore in FreeRTOS for controlling access to
shared resources among multiple tasks and observe concurrent execution
behavior.

------------------------------------------------------------------------

## Components Required

-   STM32 Nucleo-F446RE
-   USB Cable
-   STM32CubeIDE
-   FreeRTOS Support Package

------------------------------------------------------------------------

## Theory

A Counting Semaphore allows multiple tasks to access a shared resource
simultaneously up to a predefined limit.

Unlike a binary semaphore:

-   Binary Semaphore → Only one task allowed
-   Counting Semaphore → Multiple tasks allowed

Counting semaphores are commonly used for:

-   Resource pools
-   Peripheral sharing
-   Task synchronization
-   Connection management

------------------------------------------------------------------------

## System Configuration

### Clock Configuration

Clock Frequency:

84 MHz

### RTOS Tasks

Three tasks are created:

1.  TaskA
2.  TaskB
3.  TaskC

All tasks operate with Normal Priority.

### Semaphore Configuration

Semaphore Name:

myCountingSem01

Maximum Count:

2

Initial Count:

2

This means only two tasks can access the resource simultaneously.

------------------------------------------------------------------------

## Program Flow

1.  Task requests semaphore access
2.  If token available → task executes
3.  If unavailable → task waits
4.  After execution → semaphore released
5.  Waiting tasks continue

------------------------------------------------------------------------

## Important Code

Semaphore Acquire:

``` c
osSemaphoreAcquire(
myCountingSem01Handle,
osWaitForever);
```

Semaphore Release:

``` c
osSemaphoreRelease(
myCountingSem01Handle);
```

Task Example:

``` c
printf("Task Running");

HAL_Delay(50);

osSemaphoreRelease(
myCountingSem01Handle);
```

------------------------------------------------------------------------

## Expected Output

When count = 2:

Multiple tasks execute simultaneously producing mixed output.

Example:

AABABBCCBAA

When count = 1:

Output becomes sequential:

AAAAAAAA

BBBBBBBB

CCCCCCCC

------------------------------------------------------------------------

## Observation Table

  Parameter                 Observation
  ------------------------- -------------
  Number of Tasks           
  Semaphore Count           
  Resource Access Pattern   
  Output Behavior           

------------------------------------------------------------------------

## Result

Shared resource management was successfully implemented using a Counting
Semaphore. Multiple tasks accessed resources according to semaphore
count limits.

------------------------------------------------------------------------

## Conclusion

This experiment demonstrates how counting semaphores manage concurrent
access to shared resources in RTOS applications while preventing
uncontrolled resource usage.
