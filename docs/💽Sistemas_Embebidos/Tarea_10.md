# Tarea 10

El objetivo es por medio de una simulación en WOKWI o en físico leer un sensor RTC (Real Time Clock) y un acelerómetro.Todo esto se debe de realizar utilizando el método de lectura y escritura de datos I2C.

<iframe width="560" height="315" src="https://www.youtube.com/embed/kkYVy-IQJkk?si=4M4nI3uvGmdHba7D" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

## Real Time Clock


```c
#include "pico/stdlib.h"
#include "hardware/i2c.h"
#include <stdio.h>

#define I2C_PORT i2c0
#define RTC_SDA 4
#define RTC_SCL 5
#define RTC_ADDR 0x68  // Dirección del DS1307 o DS3231

// Convierte de Decimal codificado en binario (BCD) a decimal
uint8_t bcd_to_dec(uint8_t val) {
    return ((val / 16) * 10) + (val % 16);
}

// Función para leer la hora del RTC
void rtc_read_time() {
    uint8_t buffer[3];
    uint8_t reg = 0x00;  // Registro inicial (segundos)

    // Escribir dirección inicial de lectura
    i2c_write_blocking(I2C_PORT, RTC_ADDR, &reg, 1, true);//No envía STOP, deja el bus abierto para leer inmediatamente después.
    // Leer segundos, minutos y horas
    i2c_read_blocking(I2C_PORT, RTC_ADDR, buffer, 3, false);// Ya terminé de enviar los datos, cierro la comunicación.

    uint8_t seconds = bcd_to_dec(buffer[0] & 0x7F);//Borra el bit 7, ya que ese no contiene información de los segundos
    uint8_t minutes = bcd_to_dec(buffer[1]);
    uint8_t hours = bcd_to_dec(buffer[2] & 0x3F);

    printf("Hora actual: %02d:%02d:%02d\n", hours, minutes, seconds);
}

int main() {
    stdio_init_all();

    // Inicializar I2C en los pines 0 (SDA) y 1 (SCL)
    i2c_init(I2C_PORT, 100 * 1000);// se comunican a 100KHZ (velocidad estándar)
    gpio_set_function(RTC_SDA, GPIO_FUNC_I2C);
    gpio_set_function(RTC_SCL, GPIO_FUNC_I2C);
    gpio_pull_up(RTC_SDA);// Activamos las resistencias de forma que esté en 1 constante
    gpio_pull_up(RTC_SCL);//de esta forma solo necesitamos bajara a GND no generar voltaje

    sleep_ms(500);
    printf("Lectura de hora desde RTC iniciada...\n");

    while (true) {// Permite que mande la lectura de tiempo cada 1 segundos
        rtc_read_time();
        sleep_ms(1000);
    }
}

```


## Acelerómetro

<iframe width="560" height="315" src="https://www.youtube.com/embed/aX4zsqDe68Y?si=39Dqx2rjbX-krDAR" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

```c

#include <stdio.h>
#include "pico/stdlib.h"
#include "hardware/i2c.h"

#define MPU6050_ADDR 0x68
#define SDA_PIN 8
#define SCL_PIN 9

int main() {
    stdio_init_all();
    i2c_init(i2c0, 400 * 1000);
    gpio_set_function(SDA_PIN, GPIO_FUNC_I2C);
    gpio_set_function(SCL_PIN, GPIO_FUNC_I2C);
    gpio_pull_up(SDA_PIN);
    gpio_pull_up(SCL_PIN);

    sleep_ms(1000);
    printf("MPU6050 básico iniciado...\n");

    // Despertar el sensor (quitar sleep mode)
    uint8_t wake[2] = {0x6B, 0x00};
    i2c_write_blocking(i2c0, MPU6050_ADDR, wake, 2, false);

    uint8_t reg = 0x3B;
    uint8_t data[14];
    int16_t ax, ay, az, gx, gy, gz;

    while (true) {
        // Leer 14 bytes desde el registro 0x3B (acelerómetro y giroscopio)
        i2c_write_blocking(i2c0, MPU6050_ADDR, &reg, 1, true);
        i2c_read_blocking(i2c0, MPU6050_ADDR, data, 14, false);

        ax = (data[0] << 8) | data[1];
        ay = (data[2] << 8) | data[3];
        az = (data[4] << 8) | data[5];
        gx = (data[8] << 8) | data[9];
        gy = (data[10] << 8) | data[11];
        gz = (data[12] << 8) | data[13];

        printf("A[X:%6d Y:%6d Z:%6d] | G[X:%6d Y:%6d Z:%6d]\n",
               ax, ay, az, gx, gy, gz);

        sleep_ms(500);
    }
}





```

## Diagrama de conexiones

![Diagrama de señal prefiltrado](../recursos/imgs/RTC.jpg)
