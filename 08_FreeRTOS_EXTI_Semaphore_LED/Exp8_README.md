# Experiment 8 — External Interrupt Handling with Binary Semaphore

Never do heavy work inside an ISR. Use it to signal a task, and let the task handle the rest.

---

## Objective

Configure an **EXTI interrupt** on the onboard user button (PC13) and use a **Binary Semaphore** to synchronize an LED control task — implementing the industry-standard **Deferred Interrupt Processing** pattern.

---

## Components

| Component | Qty |
|-----------|-----|
| STM32 Nucleo-F446RE | 1 |
| USB Type-A to Mini-B cable | 1 |

---

## Background — Deferred Interrupt Processing

### Naive Approach (heavy work inside ISR)

```
Button pressed
    │
    ▼
ISR fires → blinks LED 5 times (2.5 seconds of HAL_Delay inside ISR!)
              ↑
              BLOCKS ALL OTHER INTERRUPTS for this entire duration
```

### Correct RTOS Approach (this experiment)

```
Button pressed
    │
    ▼
ISR fires → osSemaphoreRelease()   ← one fast operation, done in microseconds
    │
    ▼
Scheduler unblocks LED_Control task
    │
    ▼
LED_Control task → acquires semaphore → blinks LED 5 times
    (runs in task context — HAL_Delay is safe here)
```

### Complete Sequence

```
USER        BUTTON HW      EXTI/NVIC      SEMAPHORE      LED_Control TASK
 │               │               │               │               │
 │── press ──►   │               │               │               │
 │           falling             │               │               │
 │           edge  ──────────►   │               │               │
 │                           IRQ fires           │               │
 │                               │── Release ───►│  count: 0→1   │
 │                               │               │  task unblocks│
 │                               │               │◄── Acquire ───│  count: 1→0
 │                               │               │               │
 │                               │               │          blink × 5
 │                               │               │         (10 toggles
 │                               │               │          × 250 ms)
 │                               │               │               │
 │                               │               │          blocks again
```

---

## CubeMX Configuration

### Step 1 — GPIO

| Pin | Label | Mode |
|-----|-------|------|
| PA5 | LED LD2 | GPIO Output |
| PC13 | USER Button | GPIO_EXTI13 |

### Step 2 — EXTI and Interrupt Settings

1. **GPIO → PC13** → Trigger detection: **Falling Edge**
2. **GPIO → NVIC tab** → enable `EXTI line[15:10] interrupt`
3. **System Core → NVIC** → set EXTI[15:10] preemption priority to **7**

> FreeRTOS requires that any ISR calling FreeRTOS API functions have its NVIC priority set to `configLIBRARY_MAX_SYSCALL_INTERRUPT_PRIORITY` or higher (numerically ≥ 5). Priority 7 satisfies this.

### Step 3 — System

| Setting | Value |
|---------|-------|
| RCC | BYPASS |
| HCLK | 84 MHz |
| SYS → Debug | Trace Asynchronous Sw |
| SYS → Timebase | TIM6 |

### Step 4 — FreeRTOS (CMSIS_V2)

- **Tasks & Queues** → rename the default task to `LED_Control`
- **Timers & Semaphores** → add a Binary Semaphore:

| Field | Value |
|-------|-------|
| Semaphore Name | `myBinarySem01` |
| Initial Count | `1` *(manually corrected to `0` in code — see below)* |

### Step 5 — SWV / Printf

- Enable `printf` support under **C/C++ Build → Settings**
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

### LED Control Task

```c
void StartDefaultTask(void *argument) {
    uint8_t i;
    for (;;) {
        /* Block here until the ISR releases the semaphore */
        if (osSemaphoreAcquire(myBinarySem01Handle, 100) == osOK) {
            printf("Inside LEDControl Task\n");
            i = 0;
            while (i < 10) {           // 10 toggles = 5 complete blinks
                HAL_GPIO_TogglePin(GPIOA, GPIO_PIN_5);
                HAL_Delay(250);        // 250 ms per toggle
                i++;
            }
        }
        /* Loop back and block again — dormant until next button press */
    }
}
```

### Button ISR Callback

```c
/* Called automatically by HAL when EXTI triggers on PC13 */
void HAL_GPIO_EXTI_Callback(uint16_t GPIO_Pin) {
    osSemaphoreRelease(myBinarySem01Handle);  // wake the LED task
}
```

### Critical — Fix the Semaphore Initial Count

CubeMX auto-generates the semaphore with an initial count of `1`, which causes the task to run immediately at startup without any button press. Change the second argument to `0`:

```c
// Auto-generated (incorrect — task runs at startup without a press):
myBinarySem01Handle = osSemaphoreNew(1, 1, &myBinarySem01_attributes);

// Required (task starts BLOCKED, only unblocks on button press):
myBinarySem01Handle = osSemaphoreNew(1, 0, &myBinarySem01_attributes);
//                                      ↑
//                               initial count = 0 → starts blocked
```

> **This change must be reapplied every time CubeMX regenerates the code.**

---

## Steps to Run

1. Build → Debug → switch to the debug perspective
2. Open **Window → Show View → SWV ITM Data Console**
3. Enable **Port 0** → click **Start Trace ●** → click **Resume ▶**
4. The application starts — `LED_Control` is BLOCKED and the console is silent
5. Press the USER button on the board
6. LED blinks 5 times; `"Inside LEDControl Task"` appears in the console
7. Task returns to BLOCKED state — waiting for the next press

---

## Observation Table

| S.No | Query | Response |
|------|-------|----------|
| 1 | State of LED_Control task at program start | BLOCKED |
| 2 | State immediately after button press | READY → RUNNING |
| 3 | Does LED_Control run continuously after the press? | No — blocks again |
| 4 | Priority level of the EXTI interrupt line | |

---

## Bonus Experiment

Replace the task body with this variant and observe the difference:

```c
for (;;) {
    osSemaphoreAcquire(myBinarySem01Handle, 100);  // no osOK check
    i = 0;
    while (i < 10) {
        HAL_GPIO_TogglePin(GPIOA, GPIO_PIN_5);
        osDelay(250);
        i++;
    }
    printf("Inside LEDControl Task\n");
}
```

**Question:** What changes when the 100 ms timeout expires without a button press?

---

## Reflection Questions

1. Why doesn't `LED_Control` keep running indefinitely after the button is pressed once?
2. What is a breakpoint and how can it help verify task state transitions in this experiment?
3. Walk through the full `osSemaphoreAcquire` execution path — what happens at each step?
4. How does behavior change if the `if (... == osOK)` guard is removed from the task?

---

## Result

The Binary Semaphore correctly synchronized the LED blinking task with the hardware button press event. The ISR performed only a single fast `osSemaphoreRelease()` call, while all time-intensive LED operations were safely deferred to the `LED_Control` task context — a clean implementation of the Deferred Interrupt Processing pattern.
