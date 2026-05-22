# Experiment 2 — GPIO Digital Input: Push Button Controlled LED Toggle

> **Platform:** STM32 Nucleo-F446RE &nbsp;|&nbsp; **IDE:** STM32CubeIDE &nbsp;|&nbsp; **HAL:** STM32F4xx HAL

---

## Objective

Interface the onboard push button as a digital input and toggle the onboard LED state on every confirmed button press.

---

## Components

| Component | Details |
|-----------|---------|
| Microcontroller | STM32 Nucleo-F446RE |
| Cable | USB Type-A to Mini-B |
| IDE | STM32CubeIDE (includes CubeMX) |

---

## Background

This experiment extends Experiment 1 (GPIO output) by adding **GPIO input** — using the onboard user button to drive the onboard LED.

### GPIO as a Digital Input

When a pin is set to **input** mode, the MCU passively monitors the voltage on that pin and interprets it as logic `1` (HIGH) or `0` (LOW). Unlike output mode, the MCU does not drive the pin — it only reads it.

### Board Pin Assignments

| Function | Arduino Label | MCU Pin | Notes |
|----------|---------------|---------|-------|
| User LED (LD2) | D13 | **PA5** | Output, Push-Pull |
| User Button (B1) | — | **PC13** | Input, Active-LOW |

### Active-Low Button Logic

The onboard button **B1** is connected to **PC13**, which has a hardware pull-up resistor already fitted on the board:

- Button **released** → PC13 reads **HIGH (1)**
- Button **pressed** → PC13 reads **LOW (0)**

This is called **active-low** logic — the signal goes low when the event of interest (a press) occurs.

### Contact Bounce

Mechanical buttons don't produce a single clean transition. The metal contacts physically bounce several times before settling, generating a rapid burst of HIGH/LOW transitions. Left unhandled, the MCU can miscount one press as many.

A simple **software debounce** is used here: after detecting a press, a `HAL_Delay(200)` pause lets the signal settle before the next read.

### HAL Functions Used

| Function | Purpose |
|----------|---------|
| `HAL_GPIO_ReadPin(GPIOx, GPIO_Pin)` | Returns the current logic level of an input pin (`0` or `1`) |
| `HAL_GPIO_TogglePin(GPIOx, GPIO_Pin)` | Inverts the current state of a GPIO output pin |
| `HAL_Delay(ms)` | Blocking millisecond pause — used here for debounce |

---

## CubeMX Configuration

### 1 — MCU Selection
1. Open STM32CubeMX → **Access to MCU Selector**
2. Search for and select **STM32F446RETx**
3. Click **Start Project**

### 2 — GPIO Configuration

**PA5 — LED Output** (System Core → GPIO):

| Parameter | Value |
|-----------|-------|
| GPIO Mode | Output Push Pull |
| Pull-up / Pull-down | No pull-up, no pull-down |
| Maximum Output Speed | Low |
| User Label | `LED` *(optional)* |

**PC13 — Button Input** (System Core → GPIO):

| Parameter | Value |
|-----------|-------|
| GPIO Mode | Input mode |
| Pull-up / Pull-down | No pull-up, no pull-down |
| User Label | `BTN` *(optional)* |

> The Nucleo board already provides a hardware pull-up on PC13, so no internal pull-up is needed in CubeMX.

### 3 — Clock Source (RCC)
- **System Core → RCC**
- High Speed Clock (HSE): **Bypass Clock Source**

### 4 — Clock Configuration

| Domain | Setting | Frequency |
|--------|---------|-----------|
| PLL Source | HSE | — |
| SYSCLK | PLL | 180 MHz |
| AHB (HCLK) | ÷1 | 180 MHz |
| APB1 | ÷4 | 45 MHz |
| APB2 | ÷2 | 90 MHz |

### 5 — Project Manager
- Set **Project Name** (e.g., `Experiment_2`)
- Toolchain / IDE: **STM32CubeIDE**
- Click **Generate Code → Open Project**

### 6 — Post-Build Outputs
Right-click the project → **Properties → C/C++ Build → Settings → MCU Post Build Outputs**

- ✅ Convert to Binary File (`.bin`)
- ✅ Convert to Intel Hex File (`.hex`)

---

## Source Code

Open `Core/Src/main.c` and add the following inside the `while(1)` loop, **between the user code markers**:

```c
/* USER CODE BEGIN WHILE */
while (1)
{
    // PC13 is Active-LOW: reads 0 when button is pressed
    if (HAL_GPIO_ReadPin(GPIOC, GPIO_PIN_13) == 0)
    {
        HAL_GPIO_TogglePin(GPIOA, GPIO_PIN_5);   // Toggle LED on PA5
        HAL_Delay(200);                           // Debounce pause
    }
    /* USER CODE END WHILE */
}
```

> ⚠️ **Important:** Always write code between `USER CODE BEGIN` and `USER CODE END` markers. Anything outside these blocks is **overwritten** on the next CubeMX code regeneration.

### Program Flow

```
Loop begins
    │
    ▼
Read PC13
    │
    ├── HIGH (button released) ──→ do nothing → loop again
    │
    └── LOW (button pressed)
            │
            ▼
        Toggle PA5 (LED flips state)
            │
            ▼
        Wait 200 ms  ← debounce: let contacts settle
            │
            ▼
        Loop again
```

---

## Build & Flash

1. Connect the Nucleo board via USB.
2. Click **Build** (hammer icon) and confirm **0 errors, 0 warnings**.
3. Click **Debug** → click **Switch** when the perspective-switch dialog appears.
4. Click **Resume (▶)** to start execution.
5. Press the blue **B1 user button** — **LD2** should toggle with each press.

---

## Observations

### Button Press vs. LED State

| # | Button Action | PC13 State | LED Before | LED After | Remark |
|---|---------------|------------|------------|-----------|--------|
| 1 | No press (initial) | HIGH | OFF | OFF | No change |
| 2 | 1st press | LOW | OFF | ON | LED turns ON |
| 3 | Release | HIGH | ON | ON | LED latches ON |
| 4 | 2nd press | LOW | ON | OFF | LED turns OFF |
| 5 | 3rd press | LOW | OFF | ON | LED turns ON again |
| 6 | Rapid press | Bouncing | Varies | Toggles | May flicker at very high speed |

- The LED state is **latched** — it holds its value after the button is released.
- Every confirmed press triggers exactly one toggle.
- Very rapid presses can cause flickering if the 200 ms debounce window is insufficient.

---

## Result

For every valid button press, LED **LD2** on PA5 toggled precisely once and held its state after release. This confirms correct configuration of **PC13 as a digital input** and **PA5 as a digital output**, with functional software debounce via `HAL_Delay()`.

---

## Key Takeaways

- A microcontroller has no built-in concept of a "button press" — it only reads instantaneous logic levels. Detecting a press as an event is entirely the programmer's responsibility.
- **Active-low logic** is the norm in embedded hardware. Always check the board schematic to know the idle-state polarity of any input signal.
- **Contact bounce** is a physical inevitability. A single press can produce dozens of spurious transitions in microseconds. A delay-based debounce is the simplest fix; timer-based or state-machine approaches are more robust for production use.
- The LED here is **stateful** — unlike the periodic blink in Experiment 1, it remembers and holds its last condition. This is the basis of latches and toggles in embedded control.
- Combining GPIO input and output creates a **stimulus-response loop** — the foundational building block of all embedded control systems.

---

## Project Structure

```
02_PushButton_LED_Toggle/
├── Core/
│   ├── Inc/
│   │   └── main.h
│   └── Src/
│       └── main.c              ← User code goes here (while loop)
├── Drivers/
│   └── STM32F4xx_HAL_Driver/
├── Experiment_2.ioc            ← CubeMX pin & clock configuration
└── README.md
```
