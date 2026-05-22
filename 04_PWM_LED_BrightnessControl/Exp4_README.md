# Experiment 4: PWM-Based LED Brightness Control using STM32F446RE

## Objective

To generate PWM signals using TIM2 and control onboard LED brightness by
varying duty cycle continuously from 0% to 100% and back.

## Components Required

-   STM32 Nucleo-F446RE
-   USB Cable
-   Onboard LED (PA5)

## Theory

PWM (Pulse Width Modulation) controls average output voltage by changing
ON time relative to total period.

Duty Cycle:

Duty Cycle (%) = (CCR / ARR) × 100

PWM Frequency:

Frequency = Clock / ((PSC+1) × (ARR+1))

Higher duty cycle → Brighter LED

## Hardware Configuration

  Parameter    Value
  ------------ --------
  Timer        TIM2
  Channel      CH1
  Output Pin   PA5
  Clock        84 MHz
  Prescaler    839
  ARR          1000

## CubeMX Setup

1.  Configure PA5 as TIM2_CH1
2.  Set timer parameters
3.  Enable PWM Generation
4.  Generate code

## Program Flow

1.  Start PWM
2.  Increase CCR gradually
3.  LED brightness increases
4.  Decrease CCR gradually
5.  LED brightness decreases

## Important Code

Start PWM:

``` c
HAL_TIM_PWM_Start(&htim2, TIM_CHANNEL_1);
```

Fade In:

``` c
for(x=0;x<1000;x++)
{
__HAL_TIM_SET_COMPARE(
&htim2,
TIM_CHANNEL_1,
x);

HAL_Delay(1);
}
```

Fade Out:

``` c
for(x=1000;x>0;x--)
{
__HAL_TIM_SET_COMPARE(
&htim2,
TIM_CHANNEL_1,
x);

HAL_Delay(1);
}
```

## Observation Table

  CCR    Duty Cycle   Brightness
  ------ ------------ ------------
  0      0%           
  250    25%          
  500    50%          
  750    75%          
  1000   100%         

## Result

PWM signal generation was successfully implemented using TIM2. LED
brightness changed smoothly according to duty cycle variation,
demonstrating PWM-based intensity control.

## Conclusion

This experiment demonstrates practical implementation of PWM using STM32
timers and its application in brightness control.
