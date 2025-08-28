# Tarea 2

## Objetivo

Realizar las siguientes tareas usando codigos que ocupen logica y mascaras, no se pueden poner todas las combinaciones:

1. **Contador binario 4 bits**:En cuentro leds debe mostrarse cad segundo la representacion binaria del 0 al 15.
2. **Barrido de leds**:Correr un “1” por cuatro LEDs P0..P3 y regresar (0→1→2→3→2→1).
3. **Secuencia en código Gray**

## Contador binario de 4 bits

<iframe width="560" height="315" 
src="https://www.youtube.com/embed/GmPpirOaW_k" 
title="YouTube video player" 
frameborder="0" 
allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" 
allowfullscreen>
</iframe>


```bash
#include "pico/stdlib.h"
#include "hardware/structs/sio.h"

int main() {
    const uint LEDS[4] = {0, 1, 2, 3};

    for (int i = 0; i < 4; i++) {
        gpio_init(LEDS[i]);
        gpio_set_dir(LEDS[i], true);
    }

    while (true) {
        for (int count = 0; count < 16; count++) {
            for (int i = 0; i < 4; i++) {
                if (count & (1 << i))
                    sio_hw->gpio_set = (1u << LEDS[i]);
                else
                    sio_hw->gpio_clr = (1u << LEDS[i]);
            }
            sleep_ms(1000);
        }
    }
}
```


## Barrido de leds

<iframe width="560" height="315"
  src="https://www.youtube.com/embed/xgS2Xh3UevE"
  title="YouTube video player"
  frameborder="0"
  allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture"
  allowfullscreen>
</iframe>




```bash

#include "pico/stdlib.h"

#define NUM_LEDS 4
#define DELAY_MS 150

int main() {
    const uint LED_PINS[NUM_LEDS] = {0, 1, 2, 3};

    for (int i = 0; i < NUM_LEDS; i++) {
        gpio_init(LED_PINS[i]);
        gpio_set_dir(LED_PINS[i], GPIO_OUT);
    }

    int current_led = 0;
    int direction = 1;

    while (true) {
        gpio_put(LED_PINS[current_led], 1);
        sleep_ms(DELAY_MS);
        gpio_put(LED_PINS[current_led], 0);

        current_led += direction;

        if (current_led == 0 || current_led == NUM_LEDS - 1) {
            direction *= -1;
        }
    }
}

```
## Secuencia en código Gray

<iframe width="560" height="315"
  src="https://www.youtube.com/embed/5QyGnysfwJc"
  title="YouTube video player"
  frameborder="0"
  allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture"
  allowfullscreen>
</iframe>


```bash

#include "pico/stdlib.h"
#include <stdint.h>

#define NUM_LEDS 4
#define DELAY_MS 1000

uint8_t binary_to_gray(uint8_t n) {
    return n ^ (n >> 1);
}

int main() {
    const uint LED_PINS[NUM_LEDS] = {0, 1, 2, 3};

    for (int i = 0; i < NUM_LEDS; i++) {
        gpio_init(LED_PINS[i]);
        gpio_set_dir(LED_PINS[i], GPIO_OUT);
    }

    uint8_t counter = 0;

    while (true) {
        uint8_t gray_value = binary_to_gray(counter);

        for (int i = 0; i < NUM_LEDS; i++) {
            bool bit_value = (gray_value >> i) & 1;
            gpio_put(LED_PINS[i], bit_value);
        }

        sleep_ms(DELAY_MS);

        counter = (counter + 1) % 16;
    }
}
```