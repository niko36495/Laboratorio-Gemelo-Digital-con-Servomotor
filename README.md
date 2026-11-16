# Laboratorio-Gemelo-Digital-con-Servomotor

Un carro virtual en Unity cuyo **velocímetro, ruedas y carretera** se controlan con un **servomotor real** conectado a un Arduino. Se puede controlar de dos formas, y las dos se mantienen sincronizadas:

- **Físico:** girando un potenciómetro (la perilla).
- **Digital:** moviendo un slider en la pantalla de Unity.

Cuando mueves una de las dos, el servo se mueve, y el carro en pantalla acelera o frena.

---

## 1. ¿Cómo funciona? (explicado fácil)

Piensa en el juego como un **carro con acelerador doble**:

| Parte | Qué hace |
|---|---|
| **Potenciómetro** | La perilla física. Girarla = pisar el acelerador. |
| **Servomotor** | Un motor que se coloca en un ángulo entre **0° y 180°**. Aquí el ángulo representa la velocidad: 0° = parado, 180° = velocidad máxima. |
| **Arduino** | El "cerebro" de la parte física. Lee la perilla, mueve el servo y se comunica con el computador por el cable USB. |
| **Unity** | El juego en el computador. Muestra el carro, la carretera en movimiento y el velocímetro. |

### El recorrido de la información

```
 Gira la perilla ──► Arduino ──► mueve el servo
                        │
                        └──► avisa a Unity ("el ángulo es 90") ──► el carro va a 90 km/h

 Mueves el slider en Unity ──► Unity avisa al Arduino ("A:120") ──► el servo va a 120°
```

El Arduino y Unity se hablan por el cable USB con mensajes de texto muy cortos:

| Quién habla | Mensaje | Significa |
|---|---|---|
| Unity → Arduino | `A:120` | "Pon el servo en 120 grados" (viene del slider) |
| Arduino → Unity | `S:120,DIG` | "El servo está en 120°, y el control actual es el **digital**" |
| Arduino → Unity | `S:75,POT` | "El servo está en 75°, y el control actual es el **potenciómetro**" |

### ¿Quién manda, la perilla o el slider?

- Si mueves el **slider** en Unity, el control pasa a **digital** y la perilla se ignora.
- Si luego **giras la perilla** de forma clara (un movimiento sostenido, no un roce), el control **vuelve a ser físico**.

Esto evita que la perilla "pelee" con el slider por un pequeño temblor de la lectura.

### Qué se ve en Unity

- **Velocímetro:** la aguja y el texto ("90 km/h") siguen al slider.
- **Ruedas:** giran más rápido mientras más velocidad.
- **Carretera:** hay 5 tramos que se mueven hacia atrás. Cuando uno sale de la pantalla, salta al final de la fila, así que la carretera parece infinita.
- **Slider:** va de 0 a 180. Siempre refleja la posición real del servo.

---

## 2. Qué necesitas

### Hardware
- 1 placa Arduino (Uno o similar) y su cable USB
- 1 servomotor (tipo SG90 o similar)
- 1 potenciómetro (por ejemplo 10 kΩ)
- Cables jumper (y opcionalmente una protoboard)

### Software
- [Arduino IDE](https://www.arduino.cc/en/software)
- [Unity Hub](https://unity.com/download) con el editor de Unity que use el proyecto (ver `ProjectSettings/ProjectVersion.txt`). El proyecto usa el render pipeline URP.
- Windows (el puerto por defecto es `COM3`). En Mac/Linux funciona, pero el nombre del puerto cambia (ver más abajo).

---

## 3. Conexiones del circuito

| Componente | Pin del componente | Pin del Arduino |
|---|---|---|
| Potenciómetro | Pata izquierda | `5V` |
| Potenciómetro | Pata del centro | `A0` |
| Potenciómetro | Pata derecha | `GND` |
| Servomotor | Cable rojo | `5V` |
| Servomotor | Cable café/negro | `GND` |
| Servomotor | Cable naranja/amarillo (señal) | `D9` |

> Si el servo se reinicia o tiembla, alimenta el servo con una fuente externa de 5 V y une su GND con el GND del Arduino.

---

## 4. Estructura del repositorio

```
.
├── firmware/
│   └── servomotor/
│       └── servomotor.ino      # Código del Arduino
├── UnityProject/               # Proyecto de Unity
│   ├── Assets/
│   │   ├── ArduinoSerialController.cs   # Comunicación con el Arduino
│   │   ├── ControlVelocidad.cs          # Velocímetro, ruedas y velocidad
│   │   ├── MovimientoCarretera.cs       # Carretera infinita
│   │   └── Scenes/SampleScene.unity     # Escena principal
│   ├── Packages/
│   └── ProjectSettings/
├── .gitignore
└── README.md
```

---

## 5. Cómo ejecutarlo paso a paso

### Paso 1: Cargar el firmware en el Arduino
1. Abre `firmware/servomotor/servomotor.ino` con el Arduino IDE.
2. Conecta el Arduino por USB.
3. En **Herramientas → Placa** elige tu modelo (ej. *Arduino Uno*).
4. En **Herramientas → Puerto** elige el puerto del Arduino y **anota su nombre** (ej. `COM3`).
5. Presiona **Subir** (la flecha →).
6. Gira la perilla: el servo debe moverse. Si se mueve, el Arduino está listo.

### Paso 2: Cerrar el Monitor Serie
Si abriste el *Monitor Serie* del Arduino IDE, **ciérralo**. Solo un programa a la vez puede usar el puerto, y Unity lo necesita.

### Paso 3: Abrir el proyecto en Unity
1. Abre **Unity Hub → Add → Add project from disk**.
2. Selecciona la carpeta `UnityProject`.
3. Ábrelo con la versión de Unity indicada. La primera vez tarda varios minutos en importar.
4. En la ventana **Project**, abre `Assets/Scenes/SampleScene`. **Si no abres la escena, verás solo un cielo vacío.**

### Paso 4: Configurar el puerto
1. En la ventana **Hierarchy** selecciona el objeto `ComunicacionArduino`.
2. En el **Inspector**, en el componente *Arduino Serial Controller*, cambia **Puerto** por el que anotaste en el paso 1 (ej. `COM5`). La velocidad debe quedar en `9600`.

### Paso 5: ¡Jugar!
1. Presiona **Play** ▶ en Unity.
2. En la consola debe aparecer `Arduino conectado en COMx`.
3. Gira la perilla o mueve el slider. El servo, el velocímetro, las ruedas y la carretera deben moverse juntos.

---

## 6. Problemas comunes

| Problema | Causa probable | Solución |
|---|---|---|
| Solo veo cielo y suelo vacío | No abriste la escena | Abre `Assets/Scenes/SampleScene` |
| `Error conectando Arduino: ... no existe / access denied` | Puerto equivocado o en uso | Revisa el nombre del puerto y cierra el Monitor Serie del Arduino IDE |
| Error de compilación: `System.IO.Ports` no existe | Unity usa la API .NET Standard | **Edit → Project Settings → Player → Other Settings → Api Compatibility Level** → elige **.NET Framework** |
| Todo funciona pero la carretera no se mueve | Referencias vacías en el Inspector | Revisa que `ControlCarretera` tenga los 5 tramos asignados |
| El servo tiembla o se reinicia | Poca corriente | Usa una fuente externa de 5 V para el servo |
| Mac/Linux: no encuentro el puerto | Los puertos se llaman distinto | Usa algo como `/dev/ttyUSB0` o `/dev/cu.usbmodemXXXX` |
| Warning `FindObjectOfType is obsolete` | Es solo un aviso del paquete de UI | Se puede ignorar, no es un error |

---

## 7. Créditos

Assets de terceros usados en el proyecto (revisa la licencia de cada uno):

- Designersoup Low Poly Car Pack
- KayKit Forest Nature Pack (licencia incluida en su carpeta)
- Props Tools Fencing (licencia incluida en su carpeta)
- In Game UI Assets
- Texturas de asfalto, pasto y cielo (HDRI)
