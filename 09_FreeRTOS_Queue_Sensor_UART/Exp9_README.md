# Experiment 9: Inter-Task Communication using FreeRTOS Message Queue

## Objective

To implement Producer-Consumer communication using FreeRTOS queues where
one task generates data and another task receives and processes it.

## Components Required

-   STM32 Nucleo-F446RE
-   USB Cable
-   STM32CubeIDE
-   FreeRTOS Configuration

## Theory

A queue enables safe data transfer between multiple tasks in FreeRTOS.

Producer: Generates data and sends it to queue.

Consumer: Receives data from queue and processes it.

Queue operations:

-   osMessageQueuePut() → Send data
-   osMessageQueueGet() → Receive data

Queues eliminate race conditions caused by shared global variables.

## CubeMX Configuration

### System Settings

-   Clock Frequency: 84 MHz
-   Timebase Source: TIM6
-   Debug Mode: SWV

### Tasks

Task 1:

Name: Sensor_Read

Task 2:

Name: Motion_Control

Priority:

Normal

### Queue Configuration

Queue Name:

myQueue01

Queue Size:

16

Item Size:

sizeof(unsigned int)

## Program Flow

1.  Producer task generates distance values
2.  Data sent to queue
3.  Consumer waits for data
4.  Consumer reads data
5.  Display output

## Important Code

Producer:

``` c
dist = dist + 1;

osMessageQueuePut(
myQueue01Handle,
&dist,
0,
osWaitForever);

osDelay(1000);
```

Consumer:

``` c
osMessageQueueGet(
myQueue01Handle,
&distance,
NULL,
osWaitForever);

printf("Distance=%u",
distance);
```

## Observation Table

  Parameter               Observation
  ----------------------- -------------
  Producer Priority       
  Consumer Priority       
  Queue Size              
  Data Transfer Success   

## Output

Expected:

Distance = 1

Distance = 2

Distance = 3

...

## Result

Inter-task communication was successfully achieved using FreeRTOS
Message Queues. Producer and consumer tasks exchanged data safely
without race conditions.

## Conclusion

This experiment demonstrates synchronization and communication between
multiple tasks using queues in RTOS.
