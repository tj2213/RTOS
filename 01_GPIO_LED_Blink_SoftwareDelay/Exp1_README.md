# Experiment 1 — GPIO Digital Output: LED Blink via Software Delay

> **Platform:** STM32 Nucleo-F446RE &nbsp;|&nbsp; **IDE:** STM32CubeIDE &nbsp;|&nbsp; **HAL:** STM32F4xx HAL

---

## Objective

Configure a GPIO pin on the **STM32F446RE** as a digital output and verify LED blinking behaviour using software-based delay routines.

---

## Components

| Component | Details |
|-----------|---------|
| Microcontroller | STM32 Nucleo-F446RE |
| Cable | USB Type-A to Mini-B |
| IDE | STM32CubeIDE (includes CubeMX) |

---

## Background

### GPIO Modes

The STM32F446RE exposes GPIO ports **GPIOA – GPIOH**, each pin configurable in software to one of four modes:

| Mode | Description |
|------|-------------|
| Input | Reads an incoming digital signal |
| Output | Drives a digital HIGH or LOW signal |
| Alternate Function | Routed to an on-chip peripheral (UART, SPI, I2C, …) |
| Analog | Connected to internal ADC/DAC circuits |

### Why PA5?

On the Nucleo-F446RE, the green user LED **LD2** is wired to Arduino header pin **D13**, which maps to microcontroller pin **PA5**. No external wiring is needed — configuring PA5 as an output drives LD2 directly.

### Push-Pull Output Mode

PA5 is configured in **push-pull** mode, which actively controls both output states:

- Drive **HIGH** (~3.3 V) → LED **ON**
- Drive **LOW** (0 V) → LED **OFF**

This differs from open-drain mode, which can only pull the line low and relies on an external pull-up resistor for the high state.

### HAL Functions Used

| Function | Purpose |
|----------|---------|
| `HAL_GPIO_TogglePin(GPIOx, GPIO_Pin)` | Inverts the current logic level of the specified pin |
| `HAL_Delay(ms)` | Blocks execution for the given number of milliseconds (SysTick-based) |

---

## CubeMX Configuration

### 1 — MCU Selection
1. Open STM32CubeMX → **Access to MCU Selector**
2. Search for and select **STM32F446RETx**
3. Click **Start Project**

### 2 — GPIO (PA5)
Navigate to **System Core → GPIO** and apply the following settings:

| Parameter | Value |
|-----------|-------|
| Pin | PA5 |
| GPIO Mode | Output Push Pull |
| Pull-up / Pull-down | No pull-up, no pull-down |
| Maximum Output Speed | Low |
| User Label | `LD2` *(optional)* |

### 3 — Clock Source (RCC)
- **System Core → RCC**
- High Speed Clock (HSE): **Bypass Clock Source**  
  *(Uses the external clock signal supplied by the ST-Link interface.)*

### 4 — Clock Configuration

| Domain | Setting | Frequency |
|--------|---------|-----------|
| PLL Source | HSE | — |
| SYSCLK | PLL | 180 MHz |
| AHB (HCLK) | ÷1 | 180 MHz |
| APB1 | ÷4 | 45 MHz |
| APB2 | ÷2 | 90 MHz |

### 5 — Project Manager
- Set **Project Name** (e.g., `Experiment_1`)
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
    HAL_GPIO_TogglePin(GPIOA, GPIO_PIN_5);   // Toggle LED LD2 on PA5
    HAL_Delay(500);                           // Wait 500 ms

    /* USER CODE END WHILE */
}
```

> ⚠️ **Important:** Always write code between `USER CODE BEGIN` and `USER CODE END` markers. Anything outside these blocks is **overwritten** on the next CubeMX code regeneration.

---

## Build & Flash

1. Connect the Nucleo board via USB.
2. Click **Build** (hammer icon) and confirm **0 errors, 0 warnings**.
3. Click **Debug** → click **Switch** when the perspective-switch dialog appears.
4. Click **Resume (▶)** to start execution.
5. **LD2** should begin blinking at the configured rate.

---

## Observations

The `HAL_Delay()` value was varied across trials to observe the effect on blink speed:

| # | GPIO Pin | Mode | Delay (ms) | Blink Period | Observation |
|---|----------|------|-----------|--------------|-------------|
| 1 | PA5 | Output, PP, No PU/PD | 100 | ~0.2 s | Very fast — barely perceptible |
| 2 | PA5 | Output, PP, No PU/PD | 200 | ~0.4 s | Moderate and clearly visible |
| 3 | PA5 | Output, PP, No PU/PD | 500 | ~1.0 s | Comfortable, distinct ON/OFF |
| 4 | PA5 | Output, PP, No PU/PD | 1000 | ~2.0 s | Slow — each phase clearly distinct |

> **Note:** Blink period ≈ 2 × delay, because each complete ON/OFF cycle consists of two consecutive delay intervals.

---

## Result

GPIO pin **PA5** on the STM32F446RE Nucleo board was successfully configured as a push-pull digital output via STM32CubeIDE. Onboard LED **LD2** blinked correctly at every configured rate, and altering `HAL_Delay()` produced a proportional, clearly visible change in blink frequency — validating correct GPIO and system clock configuration.

---

## Key Takeaways

- GPIO pins are **multi-functional** — input, output, alternate function, or analog is purely a software choice.
- **Push-pull mode** actively drives both logic levels, making it ideal for direct LED drive with no additional components.
- `HAL_Delay()` is a **blocking call** — the CPU stalls entirely while waiting and cannot handle any other task. This is a fundamental limitation of bare-metal super-loop programs.
- The **accuracy of `HAL_Delay()`** depends on correct SYSCLK configuration; a misconfigured clock yields incorrect delays.
- The **super loop (`while(1)`)** is the simplest embedded program structure: linear, single-task, running indefinitely.
- **Blink period = 2 × delay** — one ON phase and one OFF phase each consume a full delay interval.

---

## Project Structure

```
01_GPIO_LED_Blink_SoftwareDelay/
├── Core/
│   ├── Inc/
│   │   └── main.h
│   └── Src/
│       └── main.c              ← User code goes here (while loop)
├── Drivers/
│   └── STM32F4xx_HAL_Driver/
├── Experiment_1.ioc            ← CubeMX pin & clock configuration
└── README.md
```
