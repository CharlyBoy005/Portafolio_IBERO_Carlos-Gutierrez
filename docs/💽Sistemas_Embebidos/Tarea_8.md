# Tarea 8

1 - Por medio de comunicación entre dos Pico 2 , lograr accionar leds con botones.
2 - Por medio de la consola lograr prender leds conectados a las Pico 2.
3 - Elaborar un Hanshake.

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

## Prendido de leds por medio de boton físico

<iframe width="560" height="315" src="https://www.youtube.com/embed/maq5isjeKiM?si=75wQQwwbAhVU0S0J" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>


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





## Prendido de leds por medio de consola

<iframe width="560" height="315" src="https://www.youtube.com/embed/tMJrOc3qSj8?si=PrEbZyXIMHp8E2JV" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>


```bash


#include <stdio.h>
#include "pico/stdlib.h"
#include <string>

#define UART_ID uart0
#define BAUD_RATE 115200
#define UART_TX_PIN 0
#define UART_RX_PIN 1
#define LED_PIN 2

using namespace std;

int main() {
    stdio_init_all(); // Inicializa USB serial
    sleep_ms(2000);

    printf("\n[Pico listo] Comunicación UART iniciando...\n");

    // Inicializa UART antes de asignar pines
    uart_init(UART_ID, BAUD_RATE);
    gpio_set_function(UART_TX_PIN, GPIO_FUNC_UART);
    gpio_set_function(UART_RX_PIN, GPIO_FUNC_UART);
    uart_set_format(UART_ID, 8, 1, UART_PARITY_NONE);
    uart_set_fifo_enabled(UART_ID, true);

    // Configuración del LED
    gpio_init(LED_PIN);
    gpio_set_dir(LED_PIN, GPIO_OUT);
    gpio_put(LED_PIN, 0);

    string mensaje_usb = "";
    string mensaje_uart = "";

    while (true) {
        //Escritura mensaje
        int ch = getchar_timeout_us(0); // no bloqueante
        if (ch != PICO_ERROR_TIMEOUT) {
            if (ch == '\n' || ch == '\r') {
                if (!mensaje_usb.empty()) {
                    uart_puts(UART_ID, (mensaje_usb + "\n").c_str());
                    printf("Mensaje enviado: %s\n", mensaje_usb.c_str());
                    mensaje_usb = "";
                }
            } else {
                mensaje_usb += (char)ch;
            }
        }

        //Lectura en UART
        while (uart_is_readable(UART_ID)) {
            char ch_uart = uart_getc(UART_ID);

            if (ch_uart == '\n' || ch_uart == '\r') {
                if (!mensaje_uart.empty()) {
                    printf("Mensaje recibido: %s\n", mensaje_uart.c_str());

                    // Comparar información
                    if (mensaje_uart == "on" || mensaje_uart == "ON") {
                        gpio_put(LED_PIN, 1);
                        printf("LED encendido\n");
                    } else if (mensaje_uart == "off" || mensaje_uart == "OFF") {
                        gpio_put(LED_PIN, 0);
                        printf("LED apagado\n");
                    } else {
                        printf("Comando desconocido.\n");
                    }

                    mensaje_uart = "";
                }
            } else {
                mensaje_uart += ch_uart;
            }
        }

        sleep_ms(10);
    }
}
```

# Handshake

<iframe width="560" height="315" src="https://www.youtube.com/embed/vJIdFFVWSVY?si=WbIGB0pVrfPuJHTt" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

## Código maestro

```bash
#include <stdio.h>
#include "pico/stdlib.h"
#include <string>

#define UART_ID uart0
#define BAUD_RATE 115200
#define UART_TX_PIN 0
#define UART_RX_PIN 1
#define LED_PIN 2

using namespace std;

int main() {
    stdio_init_all();
    sleep_ms(2000);

    printf("\n[Pico A listo] Escribe 'conectar' para iniciar handshake.\n");

    uart_init(UART_ID, BAUD_RATE);
    gpio_set_function(UART_TX_PIN, GPIO_FUNC_UART);
    gpio_set_function(UART_RX_PIN, GPIO_FUNC_UART);
    uart_set_format(UART_ID, 8, 1, UART_PARITY_NONE);
    uart_set_fifo_enabled(UART_ID, true);

    gpio_init(LED_PIN);
    gpio_set_dir(LED_PIN, GPIO_OUT);
    gpio_put(LED_PIN, 0);

    string usb_msg = "";
    string uart_msg = "";
    bool conectado = false;

    while (true) {
        // Leer comandos desde USB
        int ch = getchar_timeout_us(0);
        if (ch != PICO_ERROR_TIMEOUT) {
            if (ch == '\n' || ch == '\r') {
                if (!usb_msg.empty()) {
                    uart_puts(UART_ID, (usb_msg + "\n").c_str());
                    printf("[Enviado por UART]: %s\n", usb_msg.c_str());
                    usb_msg = "";
                }
            } else usb_msg += (char)ch;
        }

        // Leer mensajes por UART
        while (uart_is_readable(UART_ID)) {
            char c = uart_getc(UART_ID);
            if (c == '\n' || c == '\r') {
                if (!uart_msg.empty()) {
                    printf("[Recibido]: %s\n", uart_msg.c_str());

                    if (!conectado) {
                        if (uart_msg == "ok") {
                            printf("Pico B respondió OK.\n");
                        } else if (uart_msg == "conectado") {
                            printf("Conexión establecida \n");
                            conectado = true;
                        } else {
                            printf("[Error] Mensaje inesperado durante handshake.\n");
                        }
                    } else {
                        if (uart_msg == "on" || uart_msg == "ON") {
                            gpio_put(LED_PIN, 1);
                            printf("[LED] Encendido (por comando remoto)\n");
                        } else if (uart_msg == "off" || uart_msg == "OFF") {
                            gpio_put(LED_PIN, 0);
                            printf("[LED] Apagado (por comando remoto)\n");
                        } else {
                            printf("Error: Comando desconocido tras conexión.\n");
                        }
                    }
                    uart_msg = "";
                }
            } else uart_msg += c;
        }

        sleep_ms(10);
    }
}

```

## Código esclavo
```bash
#include <stdio.h>
#include "pico/stdlib.h"
#include <string>

#define UART_ID uart0
#define BAUD_RATE 115200
#define UART_TX_PIN 0
#define UART_RX_PIN 1
#define LED_PIN 2

using namespace std;

int main() {
    stdio_init_all();
    sleep_ms(2000);

    printf("\nPico B listo, Esperando handshake desde el otro dispositivo...\n");

    uart_init(UART_ID, BAUD_RATE);
    gpio_set_function(UART_TX_PIN, GPIO_FUNC_UART);
    gpio_set_function(UART_RX_PIN, GPIO_FUNC_UART);
    uart_set_format(UART_ID, 8, 1, UART_PARITY_NONE);
    uart_set_fifo_enabled(UART_ID, true);

    gpio_init(LED_PIN);
    gpio_set_dir(LED_PIN, GPIO_OUT);
    gpio_put(LED_PIN, 0);

    string uart_msg = "";
    string usb_msg = "";
    bool conectado = false;

    while (true) {
        // Leer mensajes entrantes por UART
        while (uart_is_readable(UART_ID)) {
            char c = uart_getc(UART_ID);
            if (c == '\n' || c == '\r') {
                if (!uart_msg.empty()) {
                    printf("Recibido: %s\n", uart_msg.c_str());

                    if (!conectado) {
                        if (uart_msg == "conectar") {
                            uart_puts(UART_ID, "ok\n");
                            sleep_ms(300);
                            uart_puts(UART_ID, "conectado\n");
                            conectado = true;
                            printf("Conexión establecida\n");
                        } else {
                            uart_puts(UART_ID, "error\n");
                            printf("Error, mensaje inesperado durante handshake.\n");
                        }
                    } else {
                        if (uart_msg == "on" || uart_msg == "ON") {
                            gpio_put(LED_PIN, 1);
                            printf("LED encendido\n");
                        } else if (uart_msg == "off" || uart_msg == "OFF") {
                            gpio_put(LED_PIN, 0);
                            printf("LED apagado\n");
                        } else {
                            printf("[Error] Comando desconocido tras conexión.\n");
                        }
                    }
                    uart_msg = "";
                }
            } else uart_msg += c;
        }

    
        int ch = getchar_timeout_us(0);
        if (ch != PICO_ERROR_TIMEOUT) {
            if (ch == '\n' || ch == '\r') {
                if (!usb_msg.empty()) {
                    uart_puts(UART_ID, (usb_msg + "\n").c_str());
                    printf("[Enviado por UART]: %s\n", usb_msg.c_str());
                    usb_msg = "";
                }
            } else usb_msg += (char)ch;
        }

        sleep_ms(10);
    }
}
```

## Recomendaciones importantes

En el cmake agergar:

target_link_libraries(título de tu documento
        pico_stdlib
        hardware_uart//Esto es importante agregar
        )
Y mantener esto así:

pico_enable_stdio_uart(Examen_2 0)
pico_enable_stdio_usb(Examen_2 1)