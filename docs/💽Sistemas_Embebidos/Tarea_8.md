# Tarea 8

Por medio de comunicación entre dos Pico 2 , lograr accionar leds con botones.

## Código 1 (echo)


```bash

#include "pico/stdlib.h"
#include <stdio.h>

#define UART_ID uart1
#define BAUD_RATE 115200
#define TX_PIN 0
#define RX_PIN 1



int main() {
    stdio_init_all();


    gpio_set_function(TX_PIN, GPIO_FUNC_UART);
    gpio_set_function(RX_PIN, GPIO_FUNC_UART);

    uart_init(UART_ID, BAUD_RATE);
    uart_set_format(UART_ID, 8, 1, UART_PARITY_NONE);

    
    sleep_ms(2000); // tiempo para enumeración USB

    while (getchar_timeout_us(0) != PICO_ERROR_TIMEOUT) {
    // solo leer y descartar cualquier carácter residual
}

    printf("\n[Pico USB] Conexión lista. Escribe algo y Enter.\n");

    while (true) {
        int ch = getchar_timeout_us(0);
        if (ch != PICO_ERROR_TIMEOUT) {
            printf("Eco: %c\n", (char)ch);
            uart_putc(UART_ID, (char)ch); 
        }
        sleep_ms(10);
    }
}

```

## Código 2 (envia y recibe)

```bash
#include "pico/stdlib.h"
#include <stdio.h>

#define UART_ID uart1
#define BAUD_RATE 115200
#define TX_PIN 0
#define RX_PIN 1



int main() {
    stdio_init_all();


    gpio_set_function(TX_PIN, GPIO_FUNC_UART);
    gpio_set_function(RX_PIN, GPIO_FUNC_UART);

    uart_init(UART_ID, BAUD_RATE);
    uart_set_format(UART_ID, 8, 1, UART_PARITY_NONE);

    
    sleep_ms(2000); // tiempo para enumeración USB

    while (getchar_timeout_us(0) != PICO_ERROR_TIMEOUT) {
    // solo leer y descartar cualquier carácter residual
}

    printf("\n[Pico USB] Conexión lista. Escribe algo y Enter.\n");

    while (true) {
        int ch = getchar_timeout_us(0);
        if (ch != PICO_ERROR_TIMEOUT) {
            printf("Eco: %c\n", (char)ch);
            uart_putc(UART_ID, (char)ch); 
        }
  
        if (uart_is_readable(UART_ID)) {
            char c = uart_getc(UART_ID);
            printf("%c", c);

        }
        sleep_ms(10);
    }
}

```

## Codigo 3 (Pines)

```bash
#include "pico/stdlib.h"
#include <stdio.h>

#define UART_ID uart0
#define BAUD_RATE 115200
#define TX_PIN 0
#define RX_PIN 1
#define BTN 3
#define LED 2



int main() {

    
    stdio_init_all();


    gpio_init(LED);
    gpio_set_dir(LED, true);

    gpio_init(BTN);
    gpio_set_dir(BTN, false);
    gpio_pull_up(BTN);

    gpio_set_function(TX_PIN, GPIO_FUNC_UART);
    gpio_set_function(RX_PIN, GPIO_FUNC_UART);

    uart_init(UART_ID, BAUD_RATE);
    uart_set_format(UART_ID, 8, 1, UART_PARITY_NONE);


    sleep_ms(2000); // tiempo para enumeración USB

    while (getchar_timeout_us(0) != PICO_ERROR_TIMEOUT) {
    // solo leer y descartar cualquier carácter residual
}

    printf("\n[Pico USB] Conexión lista. Escribe algo y Enter.\n");

    while (true) {
        
        if (uart_is_readable(UART_ID)) {
            char c = uart_getc(UART_ID);
            printf("%c", c);

            if (c == '1') {
                gpio_put(LED, 1);  
                printf("LED encendido!\n");
            } 
            
            else {
                gpio_put(LED, 0);  

            }
         }

        bool pressed = !gpio_get(BTN);  
        if (pressed) {
            printf("Botón presionado!\n");
            uart_putc(UART_ID, '1');
            sleep_ms(300); 

        
        }
         sleep_ms(10);
    }
}

```


![Diagrama de señal prefiltrado](/../recursos/imgs/Diagrama_com.jpg)