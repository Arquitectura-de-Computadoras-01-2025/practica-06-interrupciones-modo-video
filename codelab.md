<!-- ===== DANGER ZONE: DON'T TOUCH IT ======-->
id: codelab-deployment
status: Published
<!-- ======================================= -->

summary: Práctica de laboratorio 06 - interrupciones modo video
authors: Ing. Gabriela Reynosa, Erika Paz, Kevin Escobar
categories: Ensamblador, Laboratorio, Modo Video

# Laboratorio 06 - Interrupciones de Modo Video

---

## Objetivos

- Utilizar interrupciones del BIOS para manejar modos de video y control de píxeles.
- Dibujar formas geométricas básicas ajustando coordenadas y colores en programas de ensamblador.
- Desarrollar y depurar algoritmos para la creación de gráficos simples en ensamblador.

---

## Introducción al modo video

### ➡ ¿Qué es el modo de video?

El modo de video en una computadora define cómo se muestra la información gráfica en el monitor. En términos de ensamblador y programación a bajo nivel, cambiar el modo de video significa ajustar cómo el hardware de la computadora maneja la presentación de los gráficos y texto en la pantalla. Estos modos pueden variar en términos de resolución, número de colores y la forma en que se organiza la información en la memoria de video.

### ➡ Uso de los Registros AH y AL en la Interrupción INT 10H

Para controlar el modo de video, utilizamos la interrupción del BIOS `INT 10H`. Los registros `AH` y `AL` son fundamentales en este proceso:

- **Registro AH**: Se usa para especificar la función del BIOS que queremos utilizar. Para configurar el modo de video, se coloca el valor `00H` en AH.
- **Registro AL**: Se usa para especificar el modo de video deseado.

### ➡ Selección del Modo de Video

Para configurar el modo de video deseado, es fundamental entender cómo utilizar el registro `AL` junto con la interrupción `INT 10H` del BIOS. El valor asignado a `AL` determina las características del despliegue gráfico, incluyendo resolución, cantidad de colores y formato de pantalla. A continuación, se detallan algunos de los modos más comunes y sus características:

- **Modo 04H**: Este es un modo gráfico que ofrece una resolución de 320x200 píxeles con una paleta de 4 colores. Es adecuado para aplicaciones que no requieren una alta densidad de colores pero sí una resolución moderada.
- **Modo 0DH**: Similar al modo 04H en términos de resolución (320x200 píxeles), pero ofrece una paleta ampliada de 16 colores, lo que permite una mayor flexibilidad en la representación gráfica.
- **Modo 0EH**: Este modo incrementa la resolución a 640x200 píxeles, manteniendo la paleta de 16 colores. Es ideal para detalles finos en gráficos debido a su mayor resolución horizontal.
- **Modo 10H**: Con una resolución de 640x350 píxeles y 16 colores, este modo es superior en términos de claridad y detalle, adecuado para aplicaciones gráficas más complejas.
- **Modo 11H**: Ofrece una alta resolución de 640x480 píxeles pero limita la paleta a solo 2 colores. Este modo es útil para aplicaciones que requieren alta resolución pero no colores, como ciertas aplicaciones de diseño CAD o texto.
- **Modo 12H**: Combina la alta resolución de 640x480 píxeles con una paleta de 16 colores, lo que lo hace adecuado para gráficos de alta calidad que también necesitan un buen manejo de color.
- **Modo 13H**: Este es un modo gráfico que destaca por su combinación de 320x200 píxeles en una impresionante paleta de 256 colores. Es especialmente popular para juegos y aplicaciones multimedia que requieren una amplia gama de colores y una resolución moderada.

### ➡ Ejemplo de Implementación en Ensamblador

Para activar un modo de video, se deben cargar los valores apropiados en los registros `AH` y `AL`, y luego ejecutar la interrupción `INT 10H`. Por ejemplo, para establecer el modo 12H, que ofrece una resolución de 640x480 con 16 colores, el código sería:

```nasm
IniciarModoVideo:
  MOV     AH, 0h      ; Especifica la función de configuración de modo de video
  MOV     AL, 12h     ; Define el modo de video 640x480 píxeles, 16 colores
  INT     10h         ; Llama a la interrupción del BIOS para cambiar el modo de video
  RET                 ; Retorna del procedimiento
```

Este fragmento de código configura el sistema para utilizar el modo de video especificado, lo cual se reflejará inmediatamente en la pantalla. Cada modo tiene sus propias características y es importante elegir el más adecuado según las necesidades del programa que se está desarrollando.

---

## ¿Qué podemos hacer en el modo video?

### ➡ Colocar un carácter en la pantalla - Interrupción 10h / 09h

La función `09H` se emplea para escribir un carácter con un atributo de color específico en la pantalla, en la posición donde se encuentre el cursor. 

**Configuración de Registros**

Para utilizar esta función, necesitas configurar varios registros que especifican cómo se comportará la función:

- **AL**: Este registro debe contener el carácter ASCII que deseas mostrar en pantalla.
- **BH**: Este es el número de la página de video donde se mostrará el carácter. La mayoría de las veces se utiliza la página 0, pero otras páginas pueden ser seleccionadas dependiendo de la configuración del sistema.
- **BL**: Este registro almacena el atributo de color que se aplicará al carácter. Los valores de este registro determinan tanto el color del fondo como del frente, utilizando el esquema de colores descrito anteriormente.
- **CX**: Este es el número de veces que el carácter se repetirá en pantalla a partir de la posición del cursor.

Así se aplica esta función en un código de ensamblador:

```nasm
; Ejemplo para mostrar la letra 'A' en rojo repetida tres veces en la pantalla
MOV AH, 09H           ; Activar la función 09H para mostrar carácter
MOV AL, 'A'           ; Carácter ASCII que se va a mostrar
MOV BH, 00H           ; Seleccionar la página de video 0
MOV BL, 04H           ; Establecer el atributo de color a rojo
MOV CX, 003H          ; Repetir el carácter tres veces
INT 10H               ; Activar la interrupción del BIOS para modo video
```

### ➡ Encender un pixel - Interrupción 10h / 0Ch

La función `0Ch` se utiliza para establecer el color de un píxel específico en la pantalla en modo gráfico. Es importante notar que esta función requiere que el sistema esté en un modo gráfico compatible para funcionar correctamente.

**Configuración de Registros**

Para utilizar esta función, debes configurar los siguientes registros:

- **AH**: Este registro debe contener el valor `0Ch` que identifica la función de poner un píxel.
- **AL**: Este registro determina el color del píxel. El rango de colores disponibles dependerá del modo gráfico activo y la paleta de colores configurada.
- **BH**: Este es el número de la página de video donde se colocará el píxel. Esto permite manejar diferentes páginas de memoria de video, útil en aplicaciones que requieren doble buffer o múltiples vistas.
- **CX**: Este registro debe contener la coordenada X del píxel que se va a encender.
- **DX**: Este registro debe contener la coordenada Y del píxel que se va a encender.

```nasm
; Ejemplo para encender un píxel en la coordenada (160, 100) con el color 4 (rojo)
MOV   AH, 0Ch         ; Establecer la función de poner un píxel
MOV   AL, 04H         ; Color del píxel (rojo)
MOV   BH, 00H         ; Usar la página de video 0
MOV   CX, 160         ; Coordenada X del píxel
MOV   DX, 100         ; Coordenada Y del píxel
INT   10H             ; Llamar a la interrupción del BIOS para modo gráfico
```

Este código configurará un píxel rojo en la posición (160, 100) en la página de video 0. Es crucial asegurarse de que el sistema esté en un modo gráfico adecuado antes de ejecutar este código, ya que de lo contrario, la función no tendrá el efecto deseado.

### ➡ Leer el Color de un Píxel - Interrupción 10h / 0Dh

La función `0Dh` permite leer el color del píxel en una posición determinada en la pantalla, siempre que el sistema esté en un modo gráfico compatible. 

**Configuración de Registros**

Para utilizar esta función correctamente, debes configurar los siguientes registros antes de llamar a la interrupción:

- **AH**: Este registro debe contener el valor `0Dh`, que identifica la función de leer el color de un píxel.
- **BH**: Este es el número de la página de video de la cual se leerá el píxel. Esto permite acceder a diferentes páginas en memoria de video.
- **CX**: Este registro debe contener la coordenada X del píxel cuyo color se desea leer.
- **DX**: Este registro debe contener la coordenada Y del píxel cuyo color se desea leer.

```nasm
; Ejemplo para leer el color del píxel en la posición (160, 100)
MOV   AH, 0Dh         ; Establecer la función de leer el color del píxel
MOV   BH, 00H         ; Seleccionar la página de video 0
MOV   CX, 160         ; Coordenada X del píxel
MOV   DX, 100         ; Coordenada Y del píxel
INT   10H             ; Llamar a la interrupción del BIOS para modo gráfico
; El color del píxel se almacenará en AL después de ejecutar esta interrupción
```

Este código leerá el color del píxel situado en (160, 100) de la página de video 0. El valor del color leído se almacenará en el registro `AL` tras la ejecución de la interrupción.

### ➡ Dibujar una línea - Interrupción 10h / 0Ch

Para crear gráficos más complejos en ensamblador, a menudo es necesario dibujar formas básicas como líneas. Utilizando la función `0Ch` de la interrupción `INT 10H`, se puede diseñar un procedimiento para dibujar una línea horizontal en la pantalla. Aquí se demostrará cómo se puede implementar esto en código de ensamblador, aprovechando el ciclo de repetición para colocar píxeles consecutivos en una línea horizontal.

**Descripción del Proceso**

El proceso para dibujar una línea horizontal implica encender píxeles sucesivos a lo largo de una coordenada Y constante, cambiando solamente la coordenada X. Esto se realiza mediante un bucle que incrementa la posición X de cada píxel hasta alcanzar el final de la línea.

**Configuración de Registros para el Procedimiento `DibujarLinea`**

Antes de ejecutar el código para dibujar la línea, es crucial asegurarse de que el modo gráfico esté activado y que el color deseado esté configurado correctamente. Los registros necesarios son:

- **AH**: Debe contener `0Ch` para la función de encendido de píxel.
- **AL**: Define el color del píxel.
- **BH**: Número de la página de video.
- **CX**: Coordenada X del píxel.
- **DX**: Coordenada Y del píxel (constante para una línea horizontal).

```nasm
MOV SI, 0d            ; Inicializa el índice para la coordenada X
CALL DibujarLinea     ; Llama al procedimiento para encender el primer píxel

INT 20H               ; Termina el programa

DibujarLinea:         ; Subrutina para dibujar una línea
  MOV   AH, 0Ch       ; Función para encender píxel
  MOV   AL, 010b      ; Color del píxel en binario (por ejemplo, 2 en binario)
  MOV   BH, 0         ; Página de video 0
  MOV   CX, SI        ; Coordenada X del píxel
  MOV   DX, 300d      ; Coordenada Y constante para la línea horizontal
  INT   10h           ; Interrupción del BIOS para modo gráfico

  INC   SI            ; Incrementa la coordenada X
  CMP   SI, 320d      ; Compara si se ha alcanzado el final de la línea (320 píxeles)
  JNE   DibujarLinea  ; Si no es el final, se repite el proceso

  RET                 ; Retorna de la subrutina
```

Este código dibujará una línea horizontal a lo largo de la coordenada Y = 300, desde X = 0 hasta X = 319, en el color especificado por el valor en `AL`.

Este código requiere que el sistema esté en un modo gráfico compatible antes de su ejecución. De no estarlo, los píxeles no se mostrarán correctamente.

---

## Ejemplo práctico

Este ejemplo en ensamblador demuestra cómo dibujar un rectángulo en modo gráfico usando la interrupción `INT 10H` y cómo hacer que el programa espere una tecla antes de terminar. La explicación detallada se centra en cómo se implementa el dibujo del rectángulo, paso a paso.

**Configuración Inicial y Estructura del Programa**

El programa comienza con la declaración inicial y la configuración del punto de inicio:

```nasm
org 100h
section .text
```

- `org 100h`: Establece el inicio del código, adecuado para programas `.COM` en DOS.
- `section .text`: Inicia la sección de código del programa.

**Configuración de Variables para el Rectángulo**

Antes de dibujar, se establecen las coordenadas iniciales del rectángulo:

```nasm
setup:
  MOV SI, 90d ; Columna inicial
  MOV DI, 70d ; Fila inicial
```

- `SI` almacena la columna inicial para comenzar el dibujo.
- `DI` almacena la fila inicial.

**Procedimiento Principal**

El procedimiento principal del programa organiza la ejecución de funciones esenciales:

```nasm
main:
  CALL IniciarModoVideo
  CALL DibujarRectangulo
  CALL EsperarTecla

  INT 20H
```

- `CALL IniciarModoVideo`: Establece el modo gráfico adecuado.
- `CALL DibujarRectangulo`: Dibuja el rectángulo.
- `CALL EsperarTecla`: Pausa la ejecución hasta que se presione una tecla.
- `INT 20H`: Termina el programa.

**Iniciar Modo Gráfico**

Configura el modo de video para gráficos:

```nasm
IniciarModoVideo:
  MOV AH, 0h
  MOV AL, 12h      ; Modo gráfico 640x480, 16 colores
  INT 10h
  RET
```

- Se selecciona un modo gráfico que soporta la manipulación directa de píxeles.

**Dibujar el Rectángulo: Detalles**

El proceso para dibujar el rectángulo se maneja con un bucle que enciende cada píxel en las coordenadas especificadas:

```nasm
DibujarRectangulo:
  MOV AH, 0Ch           ; Función del BIOS para poner un píxel
  MOV AL, 04h           ; Color del píxel (rojo)
  MOV BH, 0             ; Página de video 0
  MOV CX, SI            ; Coordenada X
  MOV DX, DI            ; Coordenada Y
  INT 10h               ; Enciende el píxel

  INC SI                ; Incrementa la columna
  CMP SI, 190d          ; Compara la columna actual con el límite de 190
  JNE DibujarRectangulo ; Continúa en la misma fila si no se alcanza el límite

  ; Al alcanzar el límite de la fila, prepara la siguiente fila
  INC DI                ; Incrementa la fila
  MOV SI, 90d           ; Reinicia la columna al inicio para la nueva fila

  CMP DI, 120d          ; Compara la fila actual con el límite de 120
  JNE DibujarRectangulo ; Si no se alcanza el límite, continúa dibujando la fila
  RET                   ; Termina la función cuando el rectángulo está completo
```

- **Control de Columnas y Filas**: Utiliza `SI` para controlar la posición horizontal (columnas) y `DI` para la posición vertical (filas).
- **Bucles Anidados**: Implementa un bucle interno para las columnas que se resetea cada vez que se completa una fila, y un bucle externo para las filas.
- **Condicionales**: Usa instrucciones de salto condicional (`JNE`) para determinar cuándo una fila o columna ha alcanzado su límite.

**Esperar una Tecla**

La ejecución se detiene hasta que el usuario interviene:

```nasm
EsperarTecla:
  MOV AH, 00h
  INT 16h        ; Lee una tecla del teclado
  RET
```

- `INT 16h` espera y captura una tecla, lo cual es útil para pausar la ejecución y observar el resultado gráfico antes de cerrar el programa.

---

## Indicaciones de entrega

- La entrega se realizará a través de GitHub Classroom, en el repositorio asignado para las [prácticas de laboratorio](https://classroom.github.com/a/p3Yq-RKA).
- Crear una carpeta llamada "**`Laboratorio-06`**" dentro del repositorio. Esta carpeta será el contenedor para los archivos de esta práctica.
- Dentro de la carpeta "**`Laboratorio-06`**", crear un archivo llamado **`desarrollo.asm`**. En este archivo, colocar todos los ejemplos y ejercicios desarrollados durante la práctica de laboratorio.
- Crear un segundo archivo llamado **`tarea.asm`** dentro de la carpeta "**`Laboratorio-06`**". Este archivo debe contener la solución a la tarea propuesta.

```
└── Laboratorio-06
    ├── desarrollo.asm
    └── tarea.asm
```

- Realizar dos commits separados:
    - **Primer Commit:** Subir el archivo **`desarrollo.asm`** una vez completado el desarrollo durante la práctica.
    - **Segundo Commit:** Subir el archivo **`tarea.asm`** una vez completada la tarea propuesta.
- Copiar el enlace de su repositorio en el entregable llamado “[Nota] Laboratorio 04 - Instrucciones de salto y comparación” correspondiente a la práctica de laboratorio que está colocado en el e-campus.

---

## Rúbrica

| **Criterios** | **Porcentaje** |
| --- | --- |
| Desarrollo | 50% |
| Tarea | 40% |
| Ayuda necesitada | 10% |
| **Total** | **100%** |