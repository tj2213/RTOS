# Experiment 3: Ultrasonic Distance Measurement using HC-SR04 with STM32F446RE

## Objective

To interface the HC-SR04 ultrasonic sensor with STM32F446RE and measure
object distance using echo pulse timing. The measured distance is
displayed through UART communication, while an onboard LED indicates
proximity status.

## Components Required

-   STM32 Nucleo-F446RE Board
-   HC-SR04 Ultrasonic Sensor
-   USB Cable
-   Jumper Wires

## Hardware Connections

  STM32 Pin   HC-SR04 Pin   Function
  ----------- ------------- ----------------
  PA8         TRIG          Trigger Signal
  PA9         ECHO          Echo Signal
  3.3V        VCC           Power Supply
  GND         GND           Ground

Additional: - PA5 → LED - PA2/PA3 → USART2

## Theory

Distance formula:

d = (v × t) / 2

where v = speed of sound and t = echo time.

## CubeMX Configuration

-   Clock: 84 MHz
-   TIM4 Prescaler: 83
-   USART2: 115200 baud

## Workflow

1.  Send trigger pulse
2.  Read echo duration
3.  Calculate distance
4.  Send UART output
5.  Control LED

## Observation Table

  Actual (cm)   Measured (cm)   LED
  ------------- --------------- -----
  5                             
  10                            
  15                            
  20                            
  25                            

## Result

HC-SR04 was successfully interfaced with STM32F446RE and distance was
measured using ultrasonic echo timing.
