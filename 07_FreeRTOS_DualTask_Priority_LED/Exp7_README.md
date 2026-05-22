# Experiment 7 — FreeRTOS Dual Tasks & Priority-Based Preemptive Scheduling

Two tasks. Same delay interval. Different priority levels. Completely different outcomes.

---

## Objective

Configure two FreeRTOS tasks with adjustable priority levels and observe how the **preemptive scheduler** distributes CPU time — verified through LED blink behavior and SWV ITM Data Console output.

---

## Components

| Component | Qty |
|-----------|-----|
| STM32 Nucleo-F446RE | 1 |
| External LED | 1 |
| 220Ω Resistor | 1 |
| Breadboard + jumper wires | — |
| USB Type-A to Mini-B cable | 1 |

**External LED wiring:**
```
PA6 ──── [220Ω] ──── LED(+) ──── LED(−) ──── GND
```

---

## Background — FreeRTOS Priority Scheduling

### Equal Priority → Round-Robin Time Sharing

When both tasks share the same priority and each calls `osDelay(500)`, the scheduler alternates between them evenly:

```
Time:    0ms      500ms    1000ms   1500ms
Task_1:  ▓▓░░░░░░░░▓▓░░░░░░░░▓▓░░░░░░░░   LED1 blinks @ 1 Hz
Task_2:  ░░▓▓░░░░░░░░▓▓░░░░░░░░▓▓░░░░░░   LED2 blinks @ 1 Hz
```

### Unequal Priority → Preemption

The higher-priority task claims CPU first. The lower-priority task only runs during blocking windows:

```
Time:    0ms      500ms    1000ms
Task_1:  ▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓   HIGH — dominates CPU
Task_2:  ░░░░░▓░░░░░▓░░░░░▓░░░░░▓   LOW  — runs infrequently
```

### Extreme Gap → Task Starvation

```
Task_HIGH (Realtime): ████████████████████████████████
Task_LOW  (Low):      ░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░   may never run
```

---

## CubeMX Configuration

### Step 1 — GPIO

| Pin | Label | Mode |
|-----|-------|------|
| PA5 | LED_1 (onboard LD2) | GPIO Output |
| PA6 | LED_2 (external) | GPIO Output |

### Step 2 — System

| Setting | Value |
|---------|-------|
| RCC | BYPASS Clock Source |
| HCLK | 84 MHz |
| SYS → Debug | Trace Asynchronous Sw |
| SYS → Timebase | TIM6 |

### Step 3 — FreeRTOS (CMSIS_V2)

Go to **Tasks & Queues** and create two tasks:

| Field | Task 1 | Task 2 |
|-------|--------|--------|
| Task Name | `LED_1` | `LED_2` |
| Entry Function | `Task1_function` | `StartLED_2` |
| Priority | *(vary per experiment row)* | *(vary per experiment row)* |
| Stack Size | 128 words | 128 words |

### Step 4 — SWV / Printf

- Enable `printf` float support under **C/C++ Build → Settings**
- In the Debug configuration, enable **SWV** with Core Clock = 84 MHz

---

## Source Code

### ITM Output Redirect (`main.c`)

```c
/* USER CODE BEGIN Includes */
#include <stdio.h>
/* USER CODE END Includes */

/* USER CODE BEGIN 0 */
int _write(int file, char *ptr, int len) {
    for (int i = 0; i < len; i++) {
        ITM_SendChar(*ptr++);
    }
    return len;
}
/* USER CODE END 0 */
```

### Task Functions

```c
/* Task 1 — toggles LED on PA5 */
void Task1_function(void *argument) {
    for (;;) {
        HAL_GPIO_TogglePin(GPIOA, GPIO_PIN_5);
        printf("Task_1 Executing for LED Toggle\n");
        osDelay(500);
    }
}

/* Task 2 — toggles LED on PA6 */
void StartLED_2(void *argument) {
    for (;;) {
        HAL_GPIO_TogglePin(GPIOA, GPIO_PIN_6);
        printf("Task_2 Executing for LED Toggle\n");
        osDelay(500);
    }
}
```

---

## Observation Table

Change priorities in CubeMX → regenerate code → re-add task bodies → observe and record:

| S.No | Task_1 Priority | Task_2 Priority | LED_1 Rate | LED_2 Rate | SWV Console Pattern | Remarks |
|------|----------------|----------------|-----------|-----------|---------------------|---------|
| 1 | `osPriorityLow` | `osPriorityLow` | | | | Equal share |
| 2 | `osPriorityNormal` | `osPriorityLow` | | | | Minor imbalance |
| 3 | `osPriorityHigh` | `osPriorityNormal` | | | | Noticeable difference |
| 4 | `osPriorityRealtime` | `osPriorityNormal` | | | | Risk of starvation |
| 5 | `osPriorityError` | `osPriorityLow` | | | | Extreme scenario |

### Expected SWV Patterns

**Equal priorities** — regular alternation:
```
Task_1 Executing for LED Toggle
Task_2 Executing for LED Toggle
Task_1 Executing for LED Toggle
Task_2 Executing for LED Toggle
```

**High vs Low** — Task_2 appears rarely:
```
Task_1 Executing for LED Toggle
Task_1 Executing for LED Toggle
Task_1 Executing for LED Toggle
Task_2 Executing for LED Toggle
```

---

## Steps to Run

1. Set task priorities in CubeMX → **Generate Code**
2. Re-add the task function bodies (regeneration overwrites them)
3. Connect the Nucleo board → **Build** → **Debug**
4. Open **Window → Show View → SWV ITM Data Console**
5. Enable **Port 0** → click **Start Trace ●** → click **Resume ▶**
6. Observe LED blink rates and console message frequency
7. Repeat for every row in the observation table

---

## Suggested Extensions

| Idea | How to Implement |
|------|-----------------|
| Make Task_2 dominate | Assign Task_2 a higher priority than Task_1 |
| Demonstrate starvation | Task_1 → `osPriorityRealtime`, Task_2 → `osPriorityLow` |
| Add a third task | Create `LED_3` on another GPIO with an intermediate priority |
| Remove `osDelay` from Task_1 | Task_1 spins forever, completely starving Task_2 |

---

## Reflection Questions

1. How does the scheduler divide CPU time between two equal-priority tasks that both call `osDelay(500)`?
2. What happens to LED behavior when one task has no `osDelay()` at all?
3. How does this RTOS dual-task approach compare to a super loop managing two LEDs?
4. Why is SWV ITM tracing preferable to UART or GPIO debugging for verifying scheduler behavior?

---

## Result

Two FreeRTOS tasks were implemented using the CMSIS-RTOS v2 interface with configurable priorities. Priority-based preemptive scheduling was clearly demonstrated through differential LED blink rates and SWV ITM console output, confirming that higher-priority tasks monopolize CPU access and can starve lower-priority tasks when no blocking calls are present.
