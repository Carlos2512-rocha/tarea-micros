[Uploading README.md…]()
# tarea-micros# ESP32-WROOM-32 — Guía Técnica

Documentación de referencia sobre el módulo ESP32-WROOM-32: definición, arquitectura, características de hardware y comparativa entre programarlo en C (ESP-IDF/Arduino) y en MicroPython.

## Tabla de Contenidos

- [1. Definición de la placa ESP32](#1-definición-de-la-placa-esp32)
- [2. Estructura y Arquitectura](#2-estructura-y-arquitectura)
- [3. Características Generales](#3-características-generales)
- [4. Conexiones y Distribución de Pines](#4-conexiones-y-distribución-de-pines)
- [5. ADC (Conversor Analógico-Digital)](#5-adc-conversor-analógico-digital)
- [6. PWM (Modulación por Ancho de Pulso)](#6-pwm-modulación-por-ancho-de-pulso)
- [7. DAC (Conversor Digital-Analógico)](#7-dac-conversor-digital-analógico)
- [8. Programación en C vs MicroPython](#8-programación-en-c-vs-micropython)
- [9. Conclusión](#9-conclusión)
- [10. Referencias](#10-referencias)

---

## 1. Definición de la placa ESP32

El **ESP32-WROOM-32** es un módulo de microcontrolador desarrollado por **Espressif Systems**, basado en el chip **ESP32**. Integra conectividad **Wi-Fi (802.11 b/g/n)** y **Bluetooth (Clásico + BLE)** en un solo circuito integrado (SoC — *System on Chip*), lo que lo convierte en una solución compacta y de bajo costo para proyectos de Internet de las Cosas (IoT), automatización, robótica y sistemas embebidos en general.

A diferencia de placas orientadas únicamente al control (como el ESP8266), el ESP32 combina procesamiento de alto rendimiento, periféricos analógicos y digitales, y radios inalámbricas integradas, funcionando como una plataforma completa para aplicaciones conectadas.

El módulo WROOM-32 es una de las variantes de encapsulado del chip ESP32, que incluye:

- El chip ESP32 (SoC).
- Memoria flash SPI (típicamente 4 MB).
- Antena impresa en PCB (o conector externo, según la versión).
- Circuitos de blindaje (shielding) para reducir interferencias.

---

## 2. Estructura y Arquitectura

### 2.1 Arquitectura del procesador

El ESP32 utiliza un procesador **Tensilica Xtensa LX6**, disponible en versión de **doble núcleo (dual-core)** o núcleo único, según el modelo:

- **Dual-core:** dos núcleos (`PRO_CPU` y `APP_CPU`) que operan hasta **240 MHz**.
  - `PRO_CPU` (Protocol CPU): normalmente gestiona la conectividad Wi-Fi/Bluetooth.
  - `APP_CPU` (Application CPU): ejecuta la lógica de la aplicación del usuario.
- Arquitectura **Harvard** de 32 bits, con instrucciones RISC.

### 2.2 Bloques funcionales internos

| Bloque | Función |
|---|---|
| CPU Xtensa LX6 (x2) | Procesamiento principal |
| ULP (Ultra Low Power Co-processor) | Ejecuta tareas simples en modo *deep sleep* |
| SRAM interna (~520 KB) | Memoria de trabajo |
| ROM interna | Bootloader y funciones base |
| Controlador Wi-Fi 802.11 b/g/n | Conectividad inalámbrica local |
| Controlador Bluetooth v4.2 (Clásico + BLE) | Conectividad de corto alcance |
| Periféricos (ADC, DAC, PWM, SPI, I2C, UART, I2S, CAN) | Interacción con el mundo físico |
| Controlador de memoria flash externa (SPI) | Almacenamiento de programa y datos |
| RTC (Real-Time Clock) + dominio de bajo consumo | Gestión de energía y modos de suspensión |

### 2.3 Jerarquía de memoria

- **Flash externa (SPI):** almacena el firmware (típicamente 4 MB en el WROOM-32).
- **SRAM (~520 KB):** dividida entre datos, instrucciones y caché.
- **RTC SRAM/RTC memory:** memoria persistente durante *deep sleep*, útil para mantener variables entre ciclos de bajo consumo.

### 2.4 Modos de energía

El ESP32 soporta múltiples estados de energía que permiten optimizar el consumo:

- **Active mode:** CPU y radios funcionando a plena capacidad.
- **Modem-sleep:** CPU activa, radios apagadas.
- **Light-sleep:** CPU pausada, mantiene el estado.
- **Deep-sleep:** solo el coprocesador ULP y la RTC permanecen activos (consumo en el orden de microamperios).

---

## 3. Características Generales

| Característica | Especificación |
|---|---|
| CPU | Xtensa LX6 dual-core, hasta 240 MHz |
| Memoria SRAM | ~520 KB |
| Memoria Flash | 4 MB (típico, externa vía SPI) |
| Conectividad Wi-Fi | 802.11 b/g/n (2.4 GHz) |
| Conectividad Bluetooth | v4.2 BR/EDR y BLE |
| Voltaje de operación | 2.2 V – 3.6 V (típico 3.3 V) |
| Alimentación de la placa (dev board) | 5 V vía USB / VIN |
| GPIOs | Hasta 34 pines digitales |
| ADC | 2 unidades SAR, hasta 18 canales, 12 bits |
| DAC | 2 canales, 8 bits |
| PWM | Hasta 16 canales independientes (LEDC) |
| Interfaces de comunicación | SPI, I2C, I2S, UART, CAN (TWAI) |
| Sensor táctil capacitivo | 10 pines |
| Sensor de temperatura interno | Sí (en algunos revisions) |
| Coprocesador de bajo consumo | ULP |
| Seguridad | Cifrado flash, arranque seguro (Secure Boot), aceleración criptográfica (AES, SHA, RSA) |

---

## 4. Conexiones y Distribución de Pines

El módulo WROOM-32 expone entre **30 y 38 pines** físicos (según la placa de desarrollo, por ejemplo DevKit V1 de 30 pines), que incluyen alimentación, tierra y pines de propósito general (GPIO).

### 4.1 Pines de alimentación

| Pin | Descripción |
|---|---|
| `VIN` / `5V` | Entrada de alimentación externa (5 V) |
| `3V3` | Salida regulada de 3.3 V |
| `GND` | Tierra (múltiples pines) |
| `EN` | Habilitación/reset del chip (activo en alto) |

### 4.2 Consideraciones importantes de GPIO

- **Pines solo de entrada:** GPIO 34, 35, 36 (VP), 39 (VN) — no tienen resistencias pull-up/pull-down internas ni pueden usarse como salida.
- **Pines de arranque (strapping pins):** GPIO 0, 2, 12, 15 — su estado durante el arranque determina el modo de boot; deben usarse con precaución.
- **Pines reservados para la flash SPI:** GPIO 6 a 11 — **no deben utilizarse** en módulos con flash integrada (como el WROOM-32), ya que están conectados internamente a la memoria flash.
- **GPIO 1 y 3:** usados para `TX0`/`RX0` (UART0), compartidos con el puerto serie de programación/depuración.

### 4.3 Interfaces de comunicación disponibles

| Interfaz | Pines típicos (configurables por software) |
|---|---|
| UART | UART0 (TX0/RX0), UART1, UART2 |
| I2C | SDA / SCL (asignables a casi cualquier GPIO) |
| SPI | VSPI y HSPI (MOSI, MISO, SCK, CS) |
| I2S | Audio digital |
| CAN (TWAI) | Bus CAN para comunicación industrial/automotriz |

> **Nota:** una de las grandes ventajas del ESP32 es el **GPIO matrix**, que permite remapear por software la mayoría de las funciones (I2C, SPI, UART, PWM) a casi cualquier pin GPIO disponible.

---

## 5. ADC (Conversor Analógico-Digital)

- El ESP32 cuenta con **2 controladores ADC** de tipo **SAR (Successive Approximation Register)**:
  - **ADC1:** 8 canales, disponible en `GPIO32`–`GPIO39`. Funciona simultáneamente con Wi-Fi.
  - **ADC2:** 10 canales, disponible en varios GPIO (0, 2, 4, 12–15, 25–27). **No puede usarse mientras el Wi-Fi está activo**, ya que comparte recursos con el radio.
- **Resolución:** configurable de 9 a 12 bits (valores 0–4095 en 12 bits).
- **Rango de voltaje de entrada:** depende de la atenuación configurada (típicamente hasta ~3.3 V con atenuación de 11 dB, aunque la linealidad real es menor, por lo que se recomienda calibración).
- **Limitaciones conocidas:** el ADC del ESP32 presenta cierta no linealidad y ruido; para lecturas de precisión se recomienda promediar muestras o aplicar las funciones de calibración de fábrica (`esp_adc_cal`).

---

## 6. PWM (Modulación por Ancho de Pulso)

El ESP32 no tiene pines PWM dedicados por hardware fijo; en su lugar utiliza el periférico **LEDC (LED Control)**, que genera señales PWM mediante software/hardware configurable en casi cualquier GPIO de salida.

- **Canales disponibles:** hasta 16 canales PWM independientes.
- **Resolución:** configurable hasta 16 bits (con relación inversa entre resolución y frecuencia máxima).
- **Frecuencia:** ajustable, típicamente desde unos Hz hasta cientos de kHz según la resolución elegida.
- **Usos comunes:** control de brillo de LEDs, control de velocidad de motores, generación de señales de control para servomotores.
- También existe el periférico **MCPWM** (Motor Control PWM), orientado específicamente a control de motores con funciones avanzadas (detección de fallas, sincronización, captura).

---

## 7. DAC (Conversor Digital-Analógico)

- El ESP32 incluye **2 canales DAC** de **8 bits** de resolución:
  - `DAC1` → `GPIO25`
  - `DAC2` → `GPIO26`
- **Rango de salida:** de 0 V a ~3.3 V (Vref), en pasos de 8 bits (256 niveles).
- **Aplicaciones típicas:** generación de señales analógicas simples, síntesis de audio básica, generación de formas de onda (junto con temporizadores o DMA).
- **Limitación:** la resolución de 8 bits es baja comparada con DACs dedicados externos (como el MCP4725 de 12 bits), por lo que para audio o señales de precisión suele recurrirse a un DAC externo vía I2C/SPI.

---

## 8. Programación en C vs MicroPython

### 8.1 Programar en C (ESP-IDF / Arduino Framework)

**Ventajas:**
- **Máximo rendimiento:** acceso directo al hardware, sin capa de interpretación, aprovechando toda la velocidad del CPU (240 MHz).
- **Menor consumo de memoria y energía:** ideal para aplicaciones con recursos limitados o que requieren modos de bajo consumo optimizados (deep-sleep, ULP).
- **Control total del hardware:** acceso a todos los periféricos, registros e interrupciones de bajo nivel.
- **Uso de FreeRTOS nativo:** el ESP-IDF se basa en FreeRTOS, permitiendo multitarea real y control fino de prioridades entre núcleos.
- **Ecosistema maduro:** el framework oficial de Espressif (ESP-IDF) y la compatibilidad con Arduino IDE ofrecen amplia documentación y librerías.
- **Ideal para producción:** mayor estabilidad y previsibilidad en aplicaciones comerciales o industriales.

**Desventajas:**
- **Curva de aprendizaje más alta:** requiere conocimientos de C/C++, gestión de memoria y, en ESP-IDF, del sistema de compilación (CMake, `menuconfig`).
- **Desarrollo más lento:** el ciclo de compilar → flashear → probar es más largo que en un intérprete.
- **Depuración más compleja:** los errores de bajo nivel (punteros, desbordamientos de memoria) pueden ser difíciles de rastrear.
- **Prototipado menos ágil:** no es la mejor opción para pruebas rápidas de concepto.

### 8.2 Programar en MicroPython

**Ventajas:**
- **Rapidez de desarrollo:** sintaxis Python de alto nivel, ideal para prototipado rápido y proyectos educativos.
- **Curva de aprendizaje baja:** accesible para principiantes o quienes ya conocen Python.
- **Interacción en tiempo real (REPL):** permite probar código línea por línea directamente en la placa sin recompilar ni volver a flashear todo el firmware.
- **Código más legible y conciso:** menos líneas para lograr la misma funcionalidad que en C.
- **Buena para IoT rápido:** útil en proyectos donde el tiempo de desarrollo importa más que el rendimiento máximo.

**Desventajas:**
- **Menor rendimiento:** al ser interpretado, es notablemente más lento que el código C compilado, especialmente en tareas intensivas (procesamiento de señales, control en tiempo real estricto).
- **Mayor consumo de memoria RAM:** el intérprete y el *garbage collector* consumen recursos, limitando la complejidad de las aplicaciones en placas con poca RAM.
- **Menor control de bajo nivel:** no todas las funciones avanzadas del hardware (interrupciones muy precisas, DMA, ciertos periféricos) están expuestas o tan optimizadas como en C.
- **Latencia e imprecisión temporal:** el *garbage collector* puede introducir pausas impredecibles, problemático en aplicaciones de temporización crítica (ej. generación de señales precisas).
- **Ecosistema más limitado:** menos librerías especializadas comparado con el ecosistema C/Arduino/ESP-IDF, aunque en constante crecimiento.

### 8.3 Tabla comparativa resumida

| Criterio | C (ESP-IDF / Arduino) | MicroPython |
|---|---|---|
| Rendimiento | Alto | Medio-bajo |
| Consumo de RAM | Bajo | Alto |
| Velocidad de desarrollo | Lenta | Rápida |
| Curva de aprendizaje | Alta | Baja |
| Control de hardware | Total | Limitado |
| Depuración interactiva (REPL) | No | Sí |
| Adecuado para producción industrial | Sí | Limitado |
| Adecuado para prototipado/educación | Limitado | Sí |

---

## 9. Conclusión

El **ESP32-WROOM-32** es una plataforma versátil que combina procesamiento dual-core, conectividad inalámbrica integrada y un amplio conjunto de periféricos analógicos y digitales (ADC, DAC, PWM), lo que la hace idónea tanto para proyectos de IoT como para sistemas embebidos más exigentes. La elección entre **C** y **MicroPython** dependerá del objetivo del proyecto: **C** resulta más adecuado cuando se requiere máximo rendimiento, eficiencia energética y control de bajo nivel (aplicaciones de producción o tiempo real), mientras que **MicroPython** es preferible para prototipado rápido, aprendizaje y proyectos donde la velocidad de desarrollo prima sobre el rendimiento absoluto.

---

## 10. Referencias

- Espressif Systems — *ESP32 Series Datasheet*.
- Espressif Systems — *ESP32 Technical Reference Manual*.
- Documentación oficial de ESP-IDF: https://docs.espressif.com/projects/esp-idf/
- Documentación oficial de MicroPython para ESP32: https://docs.micropython.org/en/latest/esp32/quickref.html
