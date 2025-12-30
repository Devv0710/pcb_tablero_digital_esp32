Este documento detalla las especificaciones técnicas y decisiones de diseño de hardware para la PCB de control de un tablero marcador digital. El sistema está basado en el microcontrolador **ESP8266 (Wemos D1 Mini)** y está diseñado para gestionar tiras LED direccionables de alta potencia y periféricos inductivos.

> **[IMAGEN 1: VISTA GENERAL DE LA PCB]**
><img width="1724" height="930" alt="pcb" src="https://github.com/user-attachments/assets/2158bd64-9dce-497a-bd41-dcd96c4173a4" />

## 1. Arquitectura de Distribución de Potencia

El diseño gestiona dos rieles de alimentación independientes para evitar que el ruido de los actuadores y la carga de los LEDs interfieran con la estabilidad del microcontrolador.

> **[IMAGEN 2: DIAGRAMA DE BLOQUES DE POTENCIA U ORDENACIÓN FÍSICA]**
><img width="959" height="589" alt="image" src="https://github.com/user-attachments/assets/acb6e9d6-3b1b-4b04-b0bd-65cd88f5a659" />
><img width="1205" height="648" alt="image" src="https://github.com/user-attachments/assets/fdfcc7ea-0814-46f9-b4a3-64e5bfceacdd" />

### Rail de Potencia Principal (5V)
* **Carga de LEDs:** El sistema controla 13 dígitos. El consumo máximo teórico es de **7.6A** (blanco al 100% de brillo), con un consumo típico de **400mA** por dígito en operación estándar.
* **Plano de Cobre:** Para soportar esta corriente sin caídas de tensión significativas (IR Drop), se utilizó un **plano de potencia de cobre** en la capa superior (Top Layer).
* **Protección:** Un fusible de acción rápida de **10A** (**F1**) protege la integridad de las pistas y componentes ante cortocircuitos en las tiras externas.

### Rail de Periféricos (12V)
* **Función:** Alimentación exclusiva para una bocina externa de alta potencia.
* **Consumo:** Optimizado para cargas menores a **1A**.
* **Control Electrónico:** Implementado mediante un MOSFET N-Channel **AO3400A**. 
* **Protección Inductiva:** Se incluyó un diodo de libre circulación (**1N4007**) en paralelo con la carga para suprimir los picos de fuerza contraelectromotriz al conmutar la bocina.  

## 2. Diseño de la PCB y Stackup

La placa se diseñó en **2 capas** con una estrategia de baja impedancia para manejar las altas corrientes involucradas.

### Capas y Planos (Copper Pours)

* **Capa Inferior (Bottom Layer - GND):** Utilizada íntegramente como un plano de masa continuo.
* **Capa Superior (Top Layer - 5V):** Aloja las señales y un plano de alimentación robusto para los 5V.
  
> **[IMAGEN 3: VISUALIZACIÓN DE LOS PLANOS DE COBRE EN KICAD]**
><img width="802" height="615" alt="image" src="https://github.com/user-attachments/assets/68f93425-73c9-466e-9730-78b39033c33f" />
><img width="761" height="603" alt="image" src="https://github.com/user-attachments/assets/a47c4b75-0bcb-4482-8a64-533467bbf0c9" />

### Filtrado de Transitorios
Se implementó una red de desacoplo masiva con una capacitancia total cercana a los **10,000 uF** (incluyendo capacitores electrolíticos de **2200uF** y **1000uF**) para estabilizar el riel de 5V ante los cambios rápidos de estado de los LEDs.
## 3. Integridad de Señal y Lógica de Control

Para asegurar la comunicación fiable con los 13 dígitos de LEDs **WS2813**, se implementaron las siguientes etapas:

### Conversión de Niveles Lógicos (Level Shifting)
Dado que el ESP8266 opera a 3.3V y los LEDs requieren un umbral lógico cercano a 5V para evitar errores de datos, se incorporó un buffer de alta velocidad **74AHCT244**. Este componente asegura que los pulsos de datos tengan flancos de subida y bajada limpios y con la amplitud necesaria (5V).

### Terminación de Línea
Cada salida de datos hacia los dígitos incluye una resistencia serie de **330 Ohm**. Esto ayuda a:
* Reducir el *ringing* y las reflexiones en los cables de conexión.
* Proteger los pines de salida del buffer ante transitorios.

## 4. Selección de Componentes y Justificación

| Componente | Función | Justificación Técnica |
| :--- | :--- | :--- |
| **WS2813** | LEDs RGB | Posee un pin de **Backup Data**. Si un LED falla, la señal no se interrumpe. |
| **AO3400A** | MOSFET de Control | Baja resistencia $R_{DS(on)}$ para minimizar pérdidas térmicas en el control de la bocina (Carga < 1A). |
| **74AHCT244** | Level Shifter | Buffer de grado industrial para compatibilidad entre lógica CMOS de 3.3V y TTL de 5V. |

### Especificaciones de los LEDs (según Datasheet)
* **Corriente Quiescente:** < 0.6mA.
* **Intensidad Lumínica:** Rojo (380-600 mcd), Verde (1050-1500 mcd), Azul (270-400 mcd) a 16mA.
* **Longitud de onda:** Rojo (623nm), Verde (520nm), Azul (471nm).
