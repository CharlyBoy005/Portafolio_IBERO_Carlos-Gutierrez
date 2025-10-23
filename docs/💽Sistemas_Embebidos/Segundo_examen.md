# Segundo examen parcial




<iframe width="560" height="315" src="https://www.youtube.com/embed/sVFG92QeUgw?si=6kIGSMI7zqA-Jy2o" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>


```bash

#include <stdio.h>
#include "pico/stdlib.h"
#include "hardware/pwm.h"
#include <string>

#define UART_ID uart0
#define BAUD_RATE 115200
#define UART_TX_PIN 0 
#define UART_RX_PIN 1

#define SERVO_PIN 2
const uint BTN1 = 4; // Botones del modo step
const uint BTN2 = 5;
const uint BTN3 = 3; // Botón para cambiar entre modos

using namespace std;

// Lista global donde se guardan las posiciones del servo, con esto podemos usar la lista en cualquier función
int valores_guardados[3] = {0, 0, 0};

// Variables compartidas entre el programa principal y la interrupción
volatile int modo_actual = 1;     // 1: write, 2: continuo, 3: step
volatile bool ciclo_activo = false;

//Función: borrar la lista de valores
void borrar_lista() {
    for (int i = 0; i < 3; i++) valores_guardados[i] = 0;
    printf("Lista borrada.\n");
}

//convertir un ángulo (0–180) a un valor de PWM 
uint16_t angle_to_level(uint16_t angle) {
    // Convierte un ángulo en microsegundos para el pulso del servo
    float pulse_us = 1000.0f + (angle * 1000.0f / 180.0f);
    // Convierte el tiempo a un nivel proporcional al periodo 
    return (uint16_t)((pulse_us / 20000.0f) * 65535);
}

//interrupción, sirve para cambiar entre los modos 1, 2 y 3
void cambiar_modo(uint gpio, uint32_t events) {
    if (gpio == BTN3) {
        modo_actual++;
        if (modo_actual > 3) modo_actual = 1; // Regresa a modo 1 si pasa del 3
        ciclo_activo = (modo_actual == 2);    // El modo continuo activa un ciclo
        printf("\nAhora estás en el modo %d\n", modo_actual);
    }
}

int main() {
    stdio_init_all();
    sleep_ms(2000);

    printf("Bienvenido, actualmente te encuentras en el modo de entrenamiento, presiona el boton para moverte a otro modo\n");
    printf("En este modo puedes escribir(write), remplazar(replace), y borrar(clear) 3 posiciones del servo\n");
    printf("Escribe alguno de los comandos para hacer alguna accion\n");

    //iniciar UART(comunicación serial entre Pico y PC)
    uart_init(UART_ID, BAUD_RATE);
    gpio_set_function(UART_TX_PIN, GPIO_FUNC_UART);
    gpio_set_function(UART_RX_PIN, GPIO_FUNC_UART);
    uart_set_format(UART_ID, 8, 1, UART_PARITY_NONE);
    uart_set_fifo_enabled(UART_ID, true);

    //iniciar PWM para controlar el servo
    gpio_set_function(SERVO_PIN, GPIO_FUNC_PWM);
    uint slice = pwm_gpio_to_slice_num(SERVO_PIN);   // Cada pin PWM pertenece a un "slice"
    uint chan  = pwm_gpio_to_channel(SERVO_PIN);     // Canal A o B
    pwm_set_wrap(slice, 65535);                      // Valor máximo del contador PWM
    float f_clk = 125000000.0f;                      // Frecuencia base del reloj (125 MHz)
    float div = f_clk / (50.0f * 65536.0f);          // Calcula divisor para 50Hz (servo)
    pwm_set_clkdiv(slice, div);
    pwm_set_enabled(slice, true);

    // --- Configurar los botones ---
    gpio_init(BTN1); gpio_set_dir(BTN1, GPIO_IN); gpio_pull_up(BTN1);
    gpio_init(BTN2); gpio_set_dir(BTN2, GPIO_IN); gpio_pull_up(BTN2);
    gpio_init(BTN3); gpio_set_dir(BTN3, GPIO_IN); gpio_pull_up(BTN3);

    // habilitar interrupción para BTN3
    // "&" se usa para pasar la dirección de la función "cambiar_modo, nos permite cambiar el valor de la variable directamente"
    // así la Pico sabe qué función ejecutar cuando se presiona el botón
    gpio_set_irq_enabled_with_callback(BTN3, GPIO_IRQ_EDGE_FALL, true, &cambiar_modo);

    // Variables locales de uso general
    string mensaje_usb = "", mensaje_uart = "";
    int modo_index = 0;      
    bool btn1_presionado = false, btn2_presionado = false;

    //Bucle principal
    while (true) {

        // Leer texto que llega desde la consola USB 
        int ch = getchar_timeout_us(0);
        if (ch != PICO_ERROR_TIMEOUT) {
            if (ch == '\n' || ch == '\r') { // Enter presionado
                if (!mensaje_usb.empty()) {
                    string comando = mensaje_usb;

                    // WRITE
                    if (comando == "write" || comando == "escribir" || comando == "Write" || comando == "Escribir") {
                        printf("Ingresa 3 valores separados por comas (ej: 0,90,130):\n");
                        string entrada_valores = "";
                        while (true) {
                            int c2 = getchar_timeout_us(0);
                            if (c2 != PICO_ERROR_TIMEOUT) {
                                if (c2 == '\n' || c2 == '\r') break;
                                entrada_valores += (char)c2;
                            }
                        }

                        // Procesar los valores separados por comas
                        int i = 0; string temp = ""; bool error = false; int num_comas = 0;
                        for (char c : entrada_valores) {
                            if (c == ',') {
                                num_comas++;
                                if (i < 3) {
                                    int val = stoi(temp); // convierte un string a número entero "string to integrer"
                                    if (val < 0 || val > 180) { error = true; break; }
                                    valores_guardados[i] = val;
                                    temp = ""; i++;
                                }
                            } else temp += c;
                        }
                        // Último valor después de la última coma
                        if (!error && !temp.empty() && i < 3) {
                            int val = stoi(temp); 
                            if (val < 0 || val > 180) error = true;
                            else valores_guardados[i] = val;
                            i++;
                        }

                        //Validaciones (errores)
                        if (error) printf("Que paso papito, dije valores de 0 a 180\n");
                        else if (i != 3 || num_comas != 2) printf("Ponte pilas, recuerda que son 3 valores\n");
                        else printf("Valores guardados: %d, %d, %d\n", valores_guardados[0], valores_guardados[1], valores_guardados[2]);
                    } 
                    //borrado de lista ---
                    else if (comando == "clear" || comando == "borrar" || comando == "Clear" || comando == "Borrar") {
                        borrar_lista();
                    }
                    //reemplazar un valor específico ---
                    else if (comando == "replace" || comando == "reemplazar" || comando == "Replace" || comando == "Reemplazar") {
                        printf("Formato: Replace:posicion,valor (ej: Replace:1,130)\n");
                        string entrada_replace = "";
                        while (true) {
                            int c2 = getchar_timeout_us(0);
                            if (c2 != PICO_ERROR_TIMEOUT) {
                                if (c2 == '\n' || c2 == '\r') break;
                                entrada_replace += (char)c2;
                            }
                        }

                        // Separar posición y valor
                        int pos = -1, val = -1; string temp=""; bool sep=false;
                        for (char c : entrada_replace) {
                            if (c == ',' && !sep) {
                                pos = stoi(temp)-1; temp=""; sep=true;
                            } else temp+=c;
                        }
                        if (sep && !temp.empty()) val = stoi(temp);

                        // Validaciones(error)
                        if (pos<0 || pos>2) printf("Error: posición inválida\n");
                        else if (val<0 || val>180) printf("Error: valor inválido\n");
                        else {
                            valores_guardados[pos]=val;
                            printf("Valor reemplazado: pos%d = %d\n", pos+1, val);
                            printf("Lista: %d, %d, %d\n", valores_guardados[0], valores_guardados[1], valores_guardados[2]);
                        }
                    }
                    mensaje_usb="";
                }
            } else mensaje_usb += (char)ch;
        }

        // STEP 
        bool b1 = gpio_get(BTN1)==0;
        bool b2 = gpio_get(BTN2)==0; 
        if (modo_actual==3) {
            if (b1 && !btn1_presionado) {
                if (modo_index>0) modo_index--;
            pwm_set_chan_level(slice, chan, angle_to_level(valores_guardados[modo_index]));
            printf("Servo a %d°\n", valores_guardados[modo_index]);
                btn1_presionado=true;
            } else if (!b1) btn1_presionado=false;

            if (b2 && !btn2_presionado) {
                if (modo_index<2) modo_index++;
            pwm_set_chan_level(slice, chan, angle_to_level(valores_guardados[modo_index]));
                printf("Servo a %d°\n", valores_guardados[modo_index]);
                btn2_presionado=true;
            } else if (!b2) btn2_presionado=false;
        }

        // CONTINUO 
        if (modo_actual==2 && ciclo_activo) {
            bool vacia = (valores_guardados[0]==0 && valores_guardados[1]==0 && valores_guardados[2]==0);
            if (vacia) {
                printf("Error: no hay lista de valores\n");
                sleep_ms(1500);
            } else {
                for (int i=0;i<3;i++) {
                 pwm_set_chan_level(slice, chan, angle_to_level(valores_guardados[i]));
                    printf("pos%d: %d\n", i+1, valores_guardados[i]);
                    for (int t=0;t<15;t++) { // Espera total de 1.5 segundos
                        sleep_ms(100);
                        if (!ciclo_activo) break;  // Si se cambia de modo, salir
                    }
                    if (!ciclo_activo) break;
                }
            }
        }

        sleep_ms(10); 
    }
}

```

