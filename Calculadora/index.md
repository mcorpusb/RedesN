# Segmentación de redes, subnetting IPv4 y VLSM

**Nivel:** CCNA · Cisco Networking Academy · 1.º ASIR  
**Área:** Direccionamiento IPv4 · Segmentación de red · División de subredes · VLSM  
**Versión:** Tema completo de estudio — abril 2026

---

## Objetivos de aprendizaje

Al finalizar este tema serás capaz de:

1. Comprender qué es una dirección IPv4, cómo se estructura y por qué se usan máscaras de subred.  
2. Explicar por qué se segmenta una red y qué ventajas aporta.  
3. Dominar la conversión entre decimal y binario aplicada al direccionamiento.  
4. Calcular dirección de red, broadcast y rango de hosts válidos a partir de cualquier IP y máscara.  
5. Realizar subnetting clásico (subredes iguales) paso a paso.  
6. Aplicar el método del tamaño del bloque para resolver ejercicios de forma rápida y segura.  
7. Diseñar un esquema de direccionamiento con VLSM para redes con necesidades desiguales.  
8. Detectar y evitar los errores más frecuentes en subnetting.  
9. Resolver ejercicios progresivos desde nivel básico hasta diseño completo.

---

## Mapa general del tema

```
┌─────────────────────────────────────────────────────────────────┐
│                   SEGMENTACIÓN Y SUBNETTING                     │
├─────────────┬───────────────────┬───────────────────────────────┤
│  CONCEPTOS  │  SUBNETTING       │  VLSM                        │
│  PREVIOS    │  CLÁSICO          │                               │
│             │  (subredes iguales)│  (subredes de distinto tamaño)│
├─────────────┼───────────────────┼───────────────────────────────┤
│ · IPv4      │ · Bits prestados  │ · Ordenar de mayor a menor   │
│ · Binario   │ · Nueva máscara   │ · Asignar bloque justo       │
│ · Máscara   │ · Tamaño de       │ · Evitar solapamientos       │
│ · Prefijo     │   bloque          │ · Verificar resultado        │
│ · Red/Host  │ · Subredes        │                               │
│ · Broadcast │ · Hosts válidos   │                               │
└─────────────┴───────────────────┴───────────────────────────────┘
        │                │                      │
        ▼                ▼                      ▼
  Ejercicios       Ejercicios            Ejercicios
  de comprensión   de cálculo            de diseño
```

---

# PARTE I — FUNDAMENTOS

---

## 1. Introducción: ¿qué es una red IPv4 y por qué se segmenta?

### 1.1. ¿Qué es una dirección IPv4?

**Definición clara.**  
Una dirección IPv4 (Internet Protocol versión 4) es un identificador numérico de 32 bits que se asigna a cada interfaz de red de un dispositivo para que pueda comunicarse dentro de una red IP.

**Explicación intuitiva.**  
Piensa en una dirección IPv4 como en la dirección postal de una casa. Así como una carta necesita una dirección de destino para llegar al buzón correcto, un paquete de datos necesita una dirección IP de destino para llegar al dispositivo correcto.

**Explicación técnica.**  
Esos 32 bits se dividen en cuatro grupos de 8 bits (llamados **octetos**), separados por puntos y escritos en decimal para facilitar la lectura humana.

```
Binario:    11000000 . 10101000 . 00000001 . 00001010
Decimal:       192   .   168    .    1     .    10
```

Cada octeto puede tomar valores de 0 a 255, porque $2^8 = 256$ valores posibles (de `00000000` a `11111111`).

**Ejemplo cotidiano.**  
Imagina un edificio grande, como un centro comercial, donde cada tienda tiene un número. La dirección `Calle Mayor 15, Local 3B` identifica exactamente una tienda. Una dirección IPv4 como `192.168.1.10` identifica exactamente un dispositivo en la red.

**Ejemplo técnico.**  
Un PC conectado a una LAN corporativa podría tener la dirección `10.0.5.23` con máscara `255.255.255.0`. Esto significa que pertenece a la red `10.0.5.0` y es el host número 23 dentro de esa red.

**Error típico.**  
Pensar que la dirección IP identifica al dispositivo (al ordenador). No es así: la IP identifica a la **interfaz de red**. Un router con tres interfaces tiene tres direcciones IP distintas.

**Mini resumen.**  
Una dirección IPv4 son 32 bits divididos en 4 octetos que identifican, de forma única, una interfaz dentro de una red IP.

---

### 1.2. ¿Qué significa segmentar una red?

**Definición clara.**  
Segmentar una red significa dividir una red grande en redes más pequeñas llamadas **subredes** (subnets), de modo que cada subred funcione como una red independiente, con su propia dirección de red, su broadcast y su rango de hosts.

**Explicación intuitiva.**  
Imagina una gran nave industrial abierta donde trabajan 200 personas. Todo el mundo oye lo que dice todo el mundo, el ruido es constante y resulta imposible concentrarse. Si dividimos esa nave en oficinas separadas con tabiques, cada equipo trabaja en su espacio, solo oye lo que le concierne, y la comunicación mejora enormemente.

En términos de redes, esto significa que cada «oficina» es una subred. Dentro de cada subred, los dispositivos se comunican directamente. Para comunicarse con otra subred, los datos pasan por un **router** (que sería la puerta que conecta las oficinas).

**Explicación técnica.**  
Segmentar consiste en tomar una red original (por ejemplo `172.16.0.0/16`) y crear dentro de ella redes más pequeñas (por ejemplo `172.16.1.0/24`, `172.16.2.0/24`, etc.) ajustando la máscara de subred para «pedir prestados» bits de la parte de host.

**Error típico.**  
Creer que las subredes se crean «gratis», sin perder nada. Al segmentar, cada subred pierde dos direcciones: la de red y la de broadcast. Cuantas más subredes se crean, más direcciones se «gastan» en esas funciones de gestión.

**Mini resumen.**  
Segmentar = dividir una red grande en redes más pequeñas, para organizar, controlar y optimizar el tráfico.

---

### 1.3. ¿Por qué no usar siempre una sola red grande?

A primera vista, una red única parece más sencilla. ¿Por qué complicarse con subredes? Hay razones muy importantes:

| Problema de una red grande               | Solución con segmentación                         |
|------------------------------------------|---------------------------------------------------|
| Todo el tráfico de broadcast llega a todos los dispositivos | Cada subred tiene su propio dominio de broadcast |
| Un virus o ataque afecta a toda la red   | El problema queda contenido en la subred afectada |
| No se puede aplicar políticas diferentes a distintos grupos | Cada subred puede tener sus propias reglas de filtrado (ACLs) |
| Es difícil identificar dónde está un dispositivo | Las subredes se pueden asignar por ubicación, departamento, etc. |
| Se desperdician direcciones IP            | Con VLSM se asigna exactamente el bloque necesario |
| La red es lenta por exceso de tráfico    | Subredes más pequeñas reducen la congestión       |

---

## 2. ¿Por qué se segmenta una red? — Razones en detalle

### 2.1. Organización lógica

Así como una empresa organiza a su personal por departamentos (Ventas, Contabilidad, IT, Dirección), una red segmentada agrupa los dispositivos por función, ubicación o necesidad.

> **Analogía:** Piensa en una biblioteca. Aunque todo pertenece al mismo edificio, los libros están organizados por secciones: literatura, ciencia, historia… Si quieres un libro de ciencia, vas directamente a esa sección, no recorres toda la biblioteca.
>
> Técnicamente, esto corresponde a que los dispositivos de un departamento (por ejemplo, Contabilidad) estén en la subred `192.168.10.0/24`, mientras que los de IT estén en la `192.168.20.0/24`.

### 2.2. Reducción del dominio de broadcast

Cada subred define un **dominio de broadcast**: el conjunto de dispositivos que reciben un mensaje broadcast. En una red sin segmentar, un broadcast llega a todos los nodos. Si hay 500 dispositivos, todos procesan cada broadcast que se envíe, aunque no les concierna.

Con subredes de, por ejemplo, 60 hosts cada una, el broadcast solo afecta a esos 60 dispositivos.

```
Sin segmentar:                     Con segmentar:
┌──────────────────────────┐       ┌─────────┐ ┌─────────┐ ┌─────────┐
│  500 dispositivos        │       │ 60 hosts│ │ 60 hosts│ │ 60 hosts│
│  TODOS reciben TODOS     │       │ Subred A│ │ Subred B│ │ Subred C│
│  los broadcasts          │       └─────────┘ └─────────┘ └─────────┘
└──────────────────────────┘       Broadcast A no llega a B ni a C
```

### 2.3. Seguridad

Si un atacante compromete un equipo en la subred de invitados, el daño queda contenido. Un firewall o un router con ACLs entre subredes puede impedir que el ataque se extienda a la subred de servidores.

### 2.4. Eficiencia y rendimiento

Menos tráfico innecesario = más ancho de banda disponible para las comunicaciones que realmente importan.

### 2.5. Escalabilidad

Una red bien segmentada crece con orden. Añadir un nuevo departamento es tan sencillo como crear una nueva subred y conectarla al router.

### 2.6. Mejor aprovechamiento de direcciones (VLSM)

Si una sede tiene 200 hosts y otra tiene 10, no tiene sentido asignar la misma cantidad de direcciones a ambas. Con VLSM, cada subred recibe exactamente el tamaño que necesita.

### 2.7. Tabla resumen — Razones para segmentar

| Razón                   | Beneficio principal                              |
|-------------------------|--------------------------------------------------|
| Organización            | Agrupación lógica por departamento/ubicación     |
| Broadcast               | Menos tráfico innecesario                        |
| Seguridad               | Contención de amenazas                           |
| Rendimiento             | Más ancho de banda útil                          |
| Escalabilidad           | Crecimiento ordenado                             |
| Eficiencia de direcciones | Uso óptimo del espacio IPv4                    |
| Control                 | Aplicación de políticas por subred               |

---

## 3. Conceptos previos imprescindibles

Antes de hacer subnetting, necesitas dominar estas piezas. Vamos una a una, con calma.

### 3.1. Dirección IPv4 — estructura interna

**Definición clara.**  
Una dirección IPv4 es un número de 32 bits que se escribe como cuatro números decimales separados por puntos, donde cada número (octeto) representa 8 bits.

**Esquema: Estructura de una dirección IPv4**

```
    Octeto 1     Octeto 2     Octeto 3     Octeto 4
  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐
  │ 8 bits   │.│ 8 bits   │.│ 8 bits   │.│ 8 bits   │
  └──────────┘ └──────────┘ └──────────┘ └──────────┘
  192          . 168         . 1           . 10
  11000000     . 10101000    . 00000001    . 00001010

  ◄──────────────── 32 bits en total ────────────────►
```

**Tabla: Valor de cada bit en un octeto**

| Posición (bit) |  7  |  6  |  5  |  4  |  3  |  2  |  1  |  0  |
|:--------------:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|
| **Valor**      | 128 |  64 |  32 |  16 |   8 |   4 |   2 |   1 |

Cada bit puede valer 0 o 1. El valor del octeto es la **suma** de los valores de los bits que están a 1.

**Ejemplo técnico.**  
El octeto `11000000` en decimal:

$$128 + 64 + 0 + 0 + 0 + 0 + 0 + 0 = 192$$

El octeto `10101000` en decimal:

$$128 + 0 + 32 + 0 + 8 + 0 + 0 + 0 = 168$$

### 3.2. Conversión binario ↔ decimal — método rápido

Para convertir de **decimal a binario**, se restan los valores de la tabla de derecha a izquierda:

**Ejemplo: convertir 200 a binario.**

| Peso       | 128 | 64 | 32 | 16 | 8 | 4 | 2 | 1 |
|:----------:|:---:|:--:|:--:|:--:|:-:|:-:|:-:|:-:|
| ¿Cabe?     | Sí  | Sí | No | No | Sí| No| No| No|
| Bit        | 1   | 1  | 0  | 0  | 1 | 0 | 0 | 0 |
| Resto      | 72  | 8  | 8  | 8  | 0 | 0 | 0 | 0 |

$200 - 128 = 72 \to 72 - 64 = 8 \to 8 - 8 = 0$  
Resultado: `11001000`

**Verificación:** $128 + 64 + 8 = 200$ ✓

**Error típico.**  
Olvidar que el octeto siempre tiene 8 bits. Si el resultado es `1010`, **no** se escribe así: hay que rellenar con ceros a la izquierda → `00001010`.

### 3.3. Máscara de subred

**Definición clara.**  
La máscara de subred es un número de 32 bits que indica qué parte de la dirección IP corresponde a la **red** y qué parte corresponde al **host**. Los bits a 1 identifican la parte de red; los bits a 0 identifican la parte de host.

**Explicación intuitiva.**  
Imagina que tienes la dirección de una casa: «Calle Mayor 15, Piso 3, Puerta B». ¿Dónde termina la dirección de la «calle» y dónde empieza la del «piso»? Necesitas una convención que separe ambas partes. La máscara hace exactamente eso con la dirección IP: marca la frontera entre la parte que identifica la red y la parte que identifica al host.

> Llevado al subnetting, si la máscara tiene 24 bits a 1, los primeros 24 bits de la IP son la red y los últimos 8 bits son el host.

**Esquema: Máscara de subred separando red y host**

```
IP:       192.168.1.10
          11000000.10101000.00000001.00001010

Máscara:  255.255.255.0
          11111111.11111111.11111111.00000000
          ◄──── Parte de red ────►◄─ Host ─►
                 (24 bits)          (8 bits)
```

**Regla fundamental.**  
En una máscara válida, todos los bits a 1 están a la izquierda (consecutivos) y todos los bits a 0 están a la derecha. **Nunca** se mezclan.

```
✓ Válida:   11111111.11111111.11110000.00000000   (255.255.240.0)
✗ Inválida: 11111111.11111111.10101010.00000000   (bits mezclados)
```

### 3.4. Prefijo CIDR (notación con barra)

**Definición clara.**  
El prefijo CIDR (Classless Inter-Domain Routing) expresa la máscara de manera compacta: un número después de una barra (`/`) que indica cuántos bits a 1 tiene la máscara.

**Tabla: Equivalencias más usadas entre máscara y prefijo CIDR**

| Prefijo | Máscara decimal      | Máscara binaria (último octeto relevante) | Hosts utilizables |
|:-------:|:--------------------:|:-----------------------------------------:|:-----------------:|
| /8      | 255.0.0.0            | 11111111.00000000.00000000.00000000       | 16 777 214        |
| /16     | 255.255.0.0          | 11111111.11111111.00000000.00000000       | 65 534            |
| /24     | 255.255.255.0        | 11111111.11111111.11111111.00000000       | 254               |
| /25     | 255.255.255.128      | 11111111.11111111.11111111.10000000       | 126               |
| /26     | 255.255.255.192      | 11111111.11111111.11111111.11000000       | 62                |
| /27     | 255.255.255.224      | 11111111.11111111.11111111.11100000       | 30                |
| /28     | 255.255.255.240      | 11111111.11111111.11111111.11110000       | 14                |
| /29     | 255.255.255.248      | 11111111.11111111.11111111.11111000       | 6                 |
| /30     | 255.255.255.252      | 11111111.11111111.11111111.11111100       | 2                 |
| /31     | 255.255.255.254      | 11111111.11111111.11111111.11111110       | 2 (punto a punto) |
| /32     | 255.255.255.255      | 11111111.11111111.11111111.11111111       | 1 (host route)    |

**Error típico.**  
Confundir la máscara `/25` con «25 hosts». El `/25` indica 25 bits de red, no 25 hosts. Los hosts se calculan con los bits restantes: $2^{(32-25)} - 2 = 2^7 - 2 = 126$ hosts.

### 3.5. Bits de red y bits de host

**Definición clara.**  
- **Bits de red:** los bits de la dirección IP que están «protegidos» por la máscara (donde la máscara tiene 1). Identifican la red y la subred.  
- **Bits de host:** los bits restantes (donde la máscara tiene 0). Identifican al dispositivo individual dentro de esa red.

**Fórmulas clave:**

$$\text{Bits de host} = 32 - \text{prefijo}$$

$$\text{Número de hosts utilizables} = 2^{\text{bits de host}} - 2$$

Se restan 2 porque dos direcciones están reservadas:
- La dirección con todos los bits de host a **0** → **dirección de red**
- La dirección con todos los bits de host a **1** → **dirección de broadcast**

**Ejemplo cotidiano.**  
Piensa en un aparcamiento con 3 plantas (red) y 50 plazas por planta (hosts). El número de planta identifica el «grupo»; el número de plaza identifica el sitio concreto. Si cambias la distribución y haces 6 plantas con 25 plazas cada una, estás «tomando prestados» bits de las plazas para crear más plantas.

> En términos de redes: si tienes una red `/24` (1 red con 254 hosts) y quieres 2 subredes, tomas 1 bit prestado de la parte de host. Ahora tienes `/25`: 2 subredes con 126 hosts cada una.

### 3.6. Dirección de red

**Definición clara.**  
Es la primera dirección de una subred. Tiene todos los bits de host a 0. Identifica a la subred en su conjunto. **No se puede asignar a ningún dispositivo.**

**Ejemplo técnico.**  
IP: `192.168.1.130/26`

```
IP:       192.168.1. 10|000010
Máscara:  255.255.255.11|000000   (/26 → 26 bits de red, 6 bits de host)

Bits de host puestos a 0:
Red:      192.168.1. 10|000000  →  192.168.1.128
```

La dirección de red es `192.168.1.128`.

### 3.7. Dirección de broadcast

**Definición clara.**  
Es la última dirección de una subred. Tiene todos los bits de host a 1. Un mensaje enviado a esta dirección llega a **todos** los dispositivos de la subred. **Tampoco se puede asignar a ningún dispositivo.**

**Ejemplo técnico (continuando).**  

```
Bits de host puestos a 1:
Broadcast: 192.168.1. 10|111111  →  192.168.1.191
```

La dirección de broadcast es `192.168.1.191`.

### 3.8. Rango de hosts válidos

**Definición clara.**  
El rango de hosts válidos va desde la dirección de red + 1 (primer host) hasta la dirección de broadcast − 1 (último host).

**Ejemplo técnico (continuando).**

| Elemento         | Dirección         |
|:-----------------|:------------------|
| Dirección de red | 192.168.1.128     |
| Primer host      | 192.168.1.129     |
| Último host      | 192.168.1.190     |
| Broadcast        | 192.168.1.191     |
| Hosts válidos    | 62                |

$$2^6 - 2 = 64 - 2 = 62 \text{ hosts}$$

### 3.9. Tabla comparativa: dirección de red vs broadcast vs host válido

| Concepto          | Bits de host      | ¿Se puede asignar a un dispositivo? | Función                           |
|:------------------|:-----------------:|:------------------------------------:|:----------------------------------|
| Dirección de red  | Todos a 0         | **No**                               | Identifica la subred              |
| Primer host       | ...00001          | **Sí**                               | Primera IP asignable              |
| Último host       | ...11110          | **Sí**                               | Última IP asignable               |
| Broadcast         | Todos a 1         | **No**                               | Envío a todos los hosts de la red |

**Error típico.**  
Confundir la dirección de red con el primer host, o el broadcast con el último host. Son cosas **distintas**: la red y el broadcast están **reservados** y no se asignan a ninguna interfaz de un equipo final.

---

## 4. Relación entre binario y subnetting

### 4.1. ¿Por qué importan los bits?

El subnetting funciona enteramente a nivel de bits. Cuando escribimos una máscara como `255.255.255.192`, lo que realmente importa es su forma binaria: `11111111.11111111.11111111.11000000`. Son 26 unos y 6 ceros.

Esos **26 unos** definen la parte de red.  
Esos **6 ceros** definen la parte de host.

Toda la lógica del subnetting se reduce a **dónde está la frontera** entre los 1 y los 0 de la máscara.

### 4.2. ¿Cómo se interpreta una máscara?

La máscara es como un sello que se superpone a la dirección IP para revelar dos zonas:

```
Dirección IP:   172  .  16   .  10   . 65
Binario:        10101100.00010000.00001010.01000001

Máscara /24:    11111111.11111111.11111111.00000000
                ├─── RED (fija) ────────┤├─ HOST ─┤
```

**Operación AND bit a bit.**  
Para hallar la dirección de red, se hace una operación **AND** entre la IP y la máscara:

```
IP:       10101100.00010000.00001010.01000001
Máscara:  11111111.11111111.11111111.00000000
          ────────────────────────────────────
AND:      10101100.00010000.00001010.00000000  →  172.16.10.0
```

La dirección de red es `172.16.10.0`.

### 4.3. ¿Qué significa «pedir bits prestados»?

**Explicación intuitiva.**  
Imagina un almacén con estantes. Originalmente tienes 8 estantes (bits de host) y con ellos puedes almacenar 254 cajas (hosts). Pero necesitas dividir el almacén en secciones independientes. Para eso, «tomas prestado» algún estante: lo usas para marcar la sección, no para almacenar cajas. Ahora tienes menos espacio por sección, pero más secciones.

> Llevado al subnetting: si tienes un `/24` (8 bits de host) y «pides prestados» 2 bits, pasas a `/26` (6 bits de host). Ahora tienes $2^2 = 4$ subredes, cada una con $2^6 - 2 = 62$ hosts.

**Esquema: Pedir bits prestados**

```
Red original /24:
  Red:  ████████.████████.████████ | Host: ○○○○○○○○
  Máscara: /24                       254 hosts

Tomamos 1 bit prestado → /25:
  Red:  ████████.████████.████████.█ | Host: ○○○○○○○
  Máscara: /25                          126 hosts × 2 subredes

Tomamos 2 bits prestados → /26:
  Red:  ████████.████████.████████.██ | Host: ○○○○○○
  Máscara: /26                           62 hosts × 4 subredes

Tomamos 3 bits prestados → /27:
  Red:  ████████.████████.████████.███ | Host: ○○○○○
  Máscara: /27                            30 hosts × 8 subredes

  █ = bit de red (fijo)      ○ = bit de host (variable)
```

### 4.4. Tabla: Bits prestados, subredes, hosts (partiendo de /24)

| Bits prestados | Nuevo prefijo | N.º subredes | Bits de host | Hosts/subred | Máscara           |
|:--------------:|:-------------:|:------------:|:------------:|:------------:|:-----------------:|
| 0              | /24           | 1            | 8            | 254          | 255.255.255.0     |
| 1              | /25           | 2            | 7            | 126          | 255.255.255.128   |
| 2              | /26           | 4            | 6            | 62           | 255.255.255.192   |
| 3              | /27           | 8            | 5            | 30           | 255.255.255.224   |
| 4              | /28           | 16           | 4            | 14           | 255.255.255.240   |
| 5              | /29           | 32           | 3            | 6            | 255.255.255.248   |
| 6              | /30           | 64           | 2            | 2            | 255.255.255.252   |

**Fórmulas clave:**

$$\text{Número de subredes} = 2^{s}$$

donde $s$ = número de bits prestados.

$$\text{Número de hosts por subred} = 2^{h} - 2$$

donde $h$ = número de bits de host restantes ($h = 32 - \text{nuevo prefijo}$).

**Error típico.**  
Confundir el número de subredes con el número de hosts. Para subredes se usa $2^s$ (sin restar); para hosts se usa $2^h - 2$ (restando la dirección de red y el broadcast).

---

# PARTE II — SUBNETTING CLÁSICO

---

## 5. Método general para resolver subnetting (subredes iguales)

### Procedimiento paso a paso

Este es el algoritmo que debes seguir siempre que te pidan dividir una red en subredes iguales:

```
┌──────────────────────────────────────────────────────┐
│ PASO 1. Identificar la red base y su máscara/prefijo │
├──────────────────────────────────────────────────────┤
│ PASO 2. Determinar el requisito:                     │
│         ¿Cuántas subredes? ○ ¿Cuántos hosts/subred?  │
├──────────────────────────────────────────────────────┤
│ PASO 3. Calcular los bits que se necesitan prestar   │
├──────────────────────────────────────────────────────┤
│ PASO 4. Obtener la nueva máscara / prefijo           │
├──────────────────────────────────────────────────────┤
│ PASO 5. Calcular el tamaño del bloque (salto)        │
├──────────────────────────────────────────────────────┤
│ PASO 6. Enumerar las subredes                        │
├──────────────────────────────────────────────────────┤
│ PASO 7. Para cada subred, determinar:                │
│         · Dirección de red                           │
│         · Primer host                                │
│         · Último host                                │
│         · Broadcast                                  │
└──────────────────────────────────────────────────────┘
```

Ahora vamos a explicar cada paso con detalle.

---

### PASO 1. Identificar la red base

Antes de dividir, necesitas saber **qué** estás dividiendo.

**Dato:** Te dan `192.168.1.0/24`.

Esto te dice:
- Red base: `192.168.1.0`
- Máscara: `255.255.255.0` (`/24`)
- Bits de host originales: $32 - 24 = 8$
- Hosts posibles (sin dividir): $2^8 - 2 = 254$

### PASO 2. Determinar el requisito

Hay dos formas habituales de plantear el problema:

**Caso A. «Necesito X subredes»** → Calcula cuántos bits prestar para obtener al menos X subredes.

**Caso B. «Necesito al menos Y hosts por subred»** → Calcula cuántos bits de host necesitas para que quepan al menos Y hosts.

> ⚠️ **Cuidado:** en algunos ejercicios te dan ambos datos y debes comprobar que la solución satisfaga los dos.

### PASO 3. Calcular los bits a prestar

**Si el requisito es un número de subredes ($n$):**  
Busca el menor $s$ tal que $2^s \geq n$.

| Subredes necesarias | Bits a prestar ($s$) | Subredes obtenidas |
|:-------------------:|:-------------------:|:------------------:|
| 2                   | 1                   | 2                  |
| 3–4                 | 2                   | 4                  |
| 5–8                 | 3                   | 8                  |
| 9–16                | 4                   | 16                 |
| 17–32               | 5                   | 32                 |
| 33–64               | 6                   | 64                 |

**Si el requisito es un número de hosts ($n$):**  
Busca el menor $h$ tal que $2^h - 2 \geq n$.

| Hosts necesarios | Bits de host ($h$) | Hosts obtenidos |
|:----------------:|:-----------------:|:---------------:|
| 1–2              | 2                 | 2               |
| 3–6              | 3                 | 6               |
| 7–14             | 4                 | 14              |
| 15–30            | 5                 | 30              |
| 31–62            | 6                 | 62              |
| 63–126           | 7                 | 126             |
| 127–254          | 8                 | 254             |

### PASO 4. Obtener la nueva máscara

$$\text{Nuevo prefijo} = \text{Prefijo original} + s$$

**Ejemplo.** Red original `/24`, necesitamos 4 subredes → $s = 2$ → Nuevo prefijo = $24 + 2 = /26$.

Máscara en decimal de `/26`: `255.255.255.192`

¿Cómo se obtiene la máscara decimal del último octeto?  
$11000000_2 = 128 + 64 = 192$

### PASO 5. Calcular el tamaño del bloque

El **tamaño del bloque** (también llamado **salto**, **incremento** o **block size**) indica la distancia entre una subred y la siguiente.

$$\text{Tamaño del bloque} = 2^h$$

donde $h$ = bits de host en la nueva máscara.

**Truco rápido.** También se puede calcular como:

$$\text{Tamaño del bloque} = 256 - \text{valor del último octeto significativo de la máscara}$$

**Ejemplo con /26:**
- Máscara: `255.255.255.192`
- Último octeto significativo: 192
- Tamaño del bloque: $256 - 192 = 64$
- O bien: $h = 32 - 26 = 6$, por tanto $2^6 = 64$ ✓

### PASO 6. Enumerar las subredes

Partiendo de la dirección de red base, se van sumando bloques:

Para `192.168.1.0/26` (bloque = 64):

| Subred | Dirección de red  |
|:------:|:-----------------:|
| 1.ª    | 192.168.1.0       |
| 2.ª    | 192.168.1.64      |
| 3.ª    | 192.168.1.128     |
| 4.ª    | 192.168.1.192     |

### PASO 7. Completar la tabla de cada subred

| Subred | Dir. de red     | Primer host     | Último host     | Broadcast       | Hosts |
|:------:|:---------------:|:---------------:|:---------------:|:---------------:|:-----:|
| 1      | 192.168.1.0     | 192.168.1.1     | 192.168.1.62    | 192.168.1.63    | 62    |
| 2      | 192.168.1.64    | 192.168.1.65    | 192.168.1.126   | 192.168.1.127   | 62    |
| 3      | 192.168.1.128   | 192.168.1.129   | 192.168.1.190   | 192.168.1.191   | 62    |
| 4      | 192.168.1.192   | 192.168.1.193   | 192.168.1.254   | 192.168.1.255   | 62    |

**¿Cómo se calcula el broadcast?**  
Broadcast = dirección de red de la **siguiente** subred − 1.

Para la subred 3: siguiente subred comienza en `.192`, luego broadcast = $.192 - 1 = .191$ ✓

---

## 6. Método del tamaño del bloque — Técnica práctica

### 6.1. ¿Qué es el tamaño del bloque?

**Definición clara.**  
El tamaño del bloque es la cantidad total de direcciones que contiene cada subred (incluyendo la dirección de red y el broadcast). Es la distancia entre el inicio de una subred y el inicio de la siguiente.

**Explicación intuitiva.**  
Piensa en un aparcamiento que se divide en zonas de 64 plazas. La zona 1 ocupa las plazas 0 a 63, la zona 2 las plazas 64 a 127, la zona 3 las plazas 128 a 191, etc. El «bloque» es 64: cada 64 plazas empieza una nueva zona.

> En términos de redes, cada «zona» es una subred. El tamaño del bloque te dice cuántas direcciones IP ocupa cada subred y dónde empieza la siguiente.

### 6.2. Fórmulas

$$\text{Tamaño del bloque} = 2^{h} = 256 - m$$

donde:
- $h$ = bits de host (bits a 0 en la máscara).
- $m$ = valor decimal del octeto de la máscara donde ocurre la segmentación.

### 6.3. ¿Cómo usar el bloque para resolver rápido?

**Técnica en 3 pasos:**

1. **Calcula el bloque:** $256 - m$.
2. **Lista los múltiplos del bloque** desde 0: $0, \text{bloque}, 2 \times \text{bloque}, 3 \times \text{bloque}, \ldots$ hasta llegar a 256.
3. **Para cualquier IP**, identifica entre qué dos múltiplos cae:
   - El múltiplo inferior es la **dirección de red**.
   - El múltiplo superior $- 1$ es el **broadcast**.

**Ejemplo práctico.**  
¿A qué subred pertenece `192.168.1.200/27`?

1. Bloque: $256 - 224 = 32$ (porque `/27` → máscara `255.255.255.224`).
2. Múltiplos de 32: 0, 32, 64, 96, 128, 160, **192**, **224**, 256.
3. 200 cae entre 192 y 224.
   - **Red:** `192.168.1.192`
   - **Broadcast:** `192.168.1.223` (224 − 1)
   - **Primer host:** `192.168.1.193`
   - **Último host:** `192.168.1.222`

**Mini resumen.**  
El método del bloque es la forma más rápida de resolver ejercicios de subnetting. Funciona siempre, con cualquier máscara.

---

## 7. Interpretación de direcciones IPv4 — Ejercicios guiados

### 7.1. Dada una IP y máscara, encontrar la red

**Problema:** ¿Cuál es la dirección de red de `10.45.100.200/20`?

**Resolución.**

1. `/20` → 20 bits de red, 12 bits de host.
2. La máscara es `255.255.240.0` (¿cómo se obtiene? El tercer octeto tiene los primeros 4 bits a 1: $11110000 = 240$).
3. El bloque en el tercer octeto: $256 - 240 = 16$.
4. El tercer octeto de la IP es 100. Múltiplos de 16: 0, 16, 32, 48, 64, 80, **96**, **112**...
5. 100 cae entre 96 y 112. La red comienza en 96.
6. **Dirección de red:** `10.45.96.0`

### 7.2. Encontrar el broadcast

**Continuación.** ¿Cuál es el broadcast de `10.45.100.200/20`?

La siguiente subred comienza en `10.45.112.0`. El broadcast es la dirección justo antes:
- **Broadcast:** `10.45.111.255`

### 7.3. Primer y último host

| Dato             | Valor             |
|:-----------------|:------------------|
| Dirección de red | 10.45.96.0        |
| Primer host      | 10.45.96.1        |
| Último host      | 10.45.111.254     |
| Broadcast        | 10.45.111.255     |
| Hosts válidos    | $2^{12} - 2 = 4094$ |

### 7.4. ¿Están dos IP en la misma subred?

**Problema.** ¿Están `192.168.5.130` y `192.168.5.200` en la misma subred si la máscara es `/26`?

**Resolución.**
1. Bloque: $256 - 192 = 64$.
2. Múltiplos de 64: 0, 64, **128**, **192**, 256.
3. 130 cae entre 128 y 192 → subred `192.168.5.128`.
4. 200 cae entre 192 y 256 → subred `192.168.5.192`.
5. **No están en la misma subred.**

---

## 8. Subnetting con distintos prefijos de partida

### 8.1. Subnetting desde una red /24

Es el caso más habitual en ejercicios de CCNA y ASIR. Has visto los ejemplos anteriores. La segmentación ocurre en el **cuarto octeto**.

**Ejemplo:** `192.168.10.0/24` dividida en 8 subredes → `/27`, bloque = 32.

| Subred | Dir. red          | Primer host       | Último host       | Broadcast         |
|:------:|:-----------------:|:-----------------:|:-----------------:|:-----------------:|
| 1      | 192.168.10.0      | 192.168.10.1      | 192.168.10.30     | 192.168.10.31     |
| 2      | 192.168.10.32     | 192.168.10.33     | 192.168.10.62     | 192.168.10.63     |
| 3      | 192.168.10.64     | 192.168.10.65     | 192.168.10.94     | 192.168.10.95     |
| 4      | 192.168.10.96     | 192.168.10.97     | 192.168.10.126    | 192.168.10.127    |
| 5      | 192.168.10.128    | 192.168.10.129    | 192.168.10.158    | 192.168.10.159    |
| 6      | 192.168.10.160    | 192.168.10.161    | 192.168.10.190    | 192.168.10.191    |
| 7      | 192.168.10.192    | 192.168.10.193    | 192.168.10.222    | 192.168.10.223    |
| 8      | 192.168.10.224    | 192.168.10.225    | 192.168.10.254    | 192.168.10.255    |

### 8.2. Subnetting desde una red /16

Cuando partes de una `/16`, tienes 16 bits de host. La segmentación puede ocurrir en el **tercer octeto** (o en combinación del tercero y cuarto).

**Ejemplo:** `172.16.0.0/16` dividida en 256 subredes.

- Bits a prestar: $2^s \geq 256 \to s = 8$.
- Nuevo prefijo: $16 + 8 = /24$.
- Bloque en el tercer octeto: $256 - 255 = 1$. Cada subred ocupa un valor del tercer octeto.

| Subred | Dir. de red       | Rango                                   | Broadcast         |
|:------:|:-----------------:|:---------------------------------------:|:-----------------:|
| 1      | 172.16.0.0/24     | 172.16.0.1 – 172.16.0.254              | 172.16.0.255      |
| 2      | 172.16.1.0/24     | 172.16.1.1 – 172.16.1.254              | 172.16.1.255      |
| 3      | 172.16.2.0/24     | 172.16.2.1 – 172.16.2.254              | 172.16.2.255      |
| ...    | ...               | ...                                     | ...               |
| 256    | 172.16.255.0/24   | 172.16.255.1 – 172.16.255.254          | 172.16.255.255    |

**Si necesitáramos solo 16 subredes:**  
- $s = 4$, nuevo prefijo $= /20$.
- Bloque en el tercer octeto: $256 - 240 = 16$.
- Subredes: 172.16.0.0/20, 172.16.16.0/20, 172.16.32.0/20, ..., 172.16.240.0/20.
- Hosts por subred: $2^{12} - 2 = 4094$.

### 8.3. Subnetting desde una red /8

**Ejemplo:** `10.0.0.0/8` dividida en 512 subredes.

- Bits a prestar: $2^s \geq 512 \to s = 9$.
- Nuevo prefijo: $8 + 9 = /17$.
- Hosts por subred: $2^{15} - 2 = 32\,766$.
- La segmentación ocurre en el **segundo y tercer octeto**.

Las primeras subredes serían:

| Subred | Dir. de red           | Broadcast             |
|:------:|:---------------------:|:---------------------:|
| 1      | 10.0.0.0/17           | 10.0.127.255          |
| 2      | 10.0.128.0/17         | 10.0.255.255          |
| 3      | 10.1.0.0/17           | 10.1.127.255          |
| 4      | 10.1.128.0/17         | 10.1.255.255          |
| ...    | ...                   | ...                   |

**Cómo se entiende:** el segundo octeto + el bit más alto del tercer octeto forman la parte de subred. El tercer octeto se «parte por la mitad»: 0–127 es una subred, 128–255 es la siguiente.

---

# PARTE III — VLSM

---

## 9. VLSM: Variable Length Subnet Masking

### 9.1. ¿Qué es VLSM?

**Definición clara.**  
VLSM (enmascaramiento de subred de longitud variable) es una técnica que permite crear subredes de diferentes tamaños a partir de una misma red, asignando a cada subred exactamente el número de direcciones que necesita.

**Explicación intuitiva.**  
Imagina que tienes un terreno grande y necesitas repartirlo entre varios vecinos. Uno necesita 500 m², otro 200 m², otro 50 m² y otro solo 10 m². Si divides el terreno en parcelas iguales de 500 m², desperdiciarás muchísimo espacio para los vecinos que solo necesitan 10 m². Lo inteligente es cortar parcelas de diferentes tamaños: una grande, una mediana, una pequeña, una diminuta. Eso es VLSM.

> En términos de redes: en lugar de hacer 4 subredes /26 (62 hosts cada una), haces una /25 (126 hosts), una /26 (62 hosts), una /28 (14 hosts) y una /30 (2 hosts). Cada subred tiene la máscara justa para cubrir sus necesidades.

### 9.2. ¿Por qué se usa VLSM?

**Tabla: Subnetting clásico vs VLSM**

| Característica              | Subnetting clásico          | VLSM                              |
|:----------------------------|:---------------------------|:-----------------------------------|
| Tamaño de subredes          | Todas iguales              | Cada una del tamaño que necesite   |
| Aprovechamiento de IPs      | Malo si las necesidades varían | Óptimo                          |
| Complejidad                 | Menor                      | Mayor                              |
| Flexibilidad                | Limitada                   | Alta                               |
| ¿Se usa en la práctica?     | Poco                       | **Sí, siempre**                   |
| Soporte de protocolos       | Todos                      | Requiere protocolos classless (OSPF, EIGRP, RIPv2) |

**Ejemplo numérico de la diferencia.**

Red asignada: `192.168.1.0/24` (254 hosts).

Necesidades:
- LAN A: 100 hosts
- LAN B: 50 hosts  
- LAN C: 25 hosts
- Enlace WAN: 2 hosts

**Con subnetting clásico** (todas iguales): si usamos /25 (126 hosts), solo obtenemos 2 subredes → **no alcanzan**. Si usamos /26 (62 hosts), obtenemos 4 subredes → pero **no caben 100 hosts** en ninguna.

¡Problema! No hay una máscara única que satisfaga todas las necesidades.

**Con VLSM:**

| Red   | Necesidad  | Prefijo asignado | Hosts disponibles | Desperdicio |
|:------|:-----------|:----------------:|:-----------------:|:-----------:|
| LAN A | 100 hosts  | /25              | 126               | 26          |
| LAN B | 50 hosts   | /26              | 62                | 12          |
| LAN C | 25 hosts   | /27              | 30                | 5           |
| WAN   | 2 hosts    | /30              | 2                 | 0           |
| **Total** |        |                  | **220**           | **43**      |

Con VLSM usamos 220 direcciones de las 256 disponibles. Sin VLSM, no habría habido solución posible con una sola /24.

### 9.3. Método paso a paso para VLSM

```
┌────────────────────────────────────────────────────────────────┐
│ PASO 1. Listar todas las subredes necesarias con su número    │
│         de hosts requerido.                                   │
├────────────────────────────────────────────────────────────────┤
│ PASO 2. ORDENAR de MAYOR a MENOR número de hosts.             │
│         (Esto es obligatorio para evitar solapamientos.)      │
├────────────────────────────────────────────────────────────────┤
│ PASO 3. Para cada subred (empezando por la mayor):            │
│   a) Calcular la máscara mínima que cubra la necesidad.       │
│   b) Asignar el bloque empezando en la primera dirección      │
│      disponible (que sea múltiplo del tamaño del bloque).     │
│   c) Calcular dir. de red, primer host, último host,          │
│      broadcast.                                               │
│   d) Marcar ese rango como ocupado.                           │
├────────────────────────────────────────────────────────────────┤
│ PASO 4. Verificar que no haya solapamientos.                  │
│ PASO 5. Verificar que todo quepa dentro de la red original.   │
└────────────────────────────────────────────────────────────────┘
```

### 9.4. Ejemplo completo de VLSM

**Enunciado.** La red asignada es `192.168.10.0/24`. Debes crear subredes para:

| Subred       | Hosts necesarios |
|:-------------|:----------------:|
| Ventas       | 100              |
| Producción   | 50               |
| Administración | 20             |
| IT           | 10               |
| Enlace R1-R2 | 2               |
| Enlace R2-R3 | 2               |

**PASO 1 y 2. Ordenar de mayor a menor.**

| Prioridad | Subred         | Hosts necesarios |
|:---------:|:---------------|:----------------:|
| 1         | Ventas         | 100              |
| 2         | Producción     | 50               |
| 3         | Administración | 20               |
| 4         | IT             | 10               |
| 5         | Enlace R1-R2   | 2                |
| 6         | Enlace R2-R3   | 2                |

**PASO 3. Asignar bloques.**

**Subred 1: Ventas (100 hosts)**  
- Necesito al menos 100 hosts → $2^h - 2 \geq 100$ → $h = 7$ ($2^7 - 2 = 126$ ✓).
- Prefijo: $/32 - 7 = /25$.
- Bloque: $2^7 = 128$ direcciones.
- Primera dirección disponible: `192.168.10.0` (es múltiplo de 128 → ✓).

| Dato             | Valor                |
|:-----------------|:---------------------|
| Dirección de red | 192.168.10.0/25      |
| Primer host      | 192.168.10.1         |
| Último host      | 192.168.10.126       |
| Broadcast        | 192.168.10.127       |
| Hosts            | 126                  |

Siguiente dirección disponible: `192.168.10.128`.

---

**Subred 2: Producción (50 hosts)**  
- $2^h - 2 \geq 50$ → $h = 6$ ($2^6 - 2 = 62$ ✓).
- Prefijo: `/26`. Bloque: 64.
- Dirección disponible: `192.168.10.128` (es múltiplo de 64 → ✓).

| Dato             | Valor                |
|:-----------------|:---------------------|
| Dirección de red | 192.168.10.128/26    |
| Primer host      | 192.168.10.129       |
| Último host      | 192.168.10.190       |
| Broadcast        | 192.168.10.191       |
| Hosts            | 62                   |

Siguiente disponible: `192.168.10.192`.

---

**Subred 3: Administración (20 hosts)**  
- $2^h - 2 \geq 20$ → $h = 5$ ($2^5 - 2 = 30$ ✓).
- Prefijo: `/27`. Bloque: 32.
- Dirección disponible: `192.168.10.192` (es múltiplo de 32 → ✓).

| Dato             | Valor                |
|:-----------------|:---------------------|
| Dirección de red | 192.168.10.192/27    |
| Primer host      | 192.168.10.193       |
| Último host      | 192.168.10.222       |
| Broadcast        | 192.168.10.223       |
| Hosts            | 30                   |

Siguiente disponible: `192.168.10.224`.

---

**Subred 4: IT (10 hosts)**  
- $2^h - 2 \geq 10$ → $h = 4$ ($2^4 - 2 = 14$ ✓).
- Prefijo: `/28`. Bloque: 16.
- Dirección disponible: `192.168.10.224` (es múltiplo de 16 → ✓).

| Dato             | Valor                |
|:-----------------|:---------------------|
| Dirección de red | 192.168.10.224/28    |
| Primer host      | 192.168.10.225       |
| Último host      | 192.168.10.238       |
| Broadcast        | 192.168.10.239       |
| Hosts            | 14                   |

Siguiente disponible: `192.168.10.240`.

---

**Subred 5: Enlace R1-R2 (2 hosts)**  
- $2^h - 2 \geq 2$ → $h = 2$ ($2^2 - 2 = 2$ ✓).
- Prefijo: `/30`. Bloque: 4.
- Dirección disponible: `192.168.10.240` (es múltiplo de 4 → ✓).

| Dato             | Valor                |
|:-----------------|:---------------------|
| Dirección de red | 192.168.10.240/30    |
| Primer host      | 192.168.10.241       |
| Último host      | 192.168.10.242       |
| Broadcast        | 192.168.10.243       |
| Hosts            | 2                    |

Siguiente disponible: `192.168.10.244`.

---

**Subred 6: Enlace R2-R3 (2 hosts)**  
- Prefijo: `/30`. Bloque: 4.
- Dirección disponible: `192.168.10.244` (es múltiplo de 4 → ✓).

| Dato             | Valor                |
|:-----------------|:---------------------|
| Dirección de red | 192.168.10.244/30    |
| Primer host      | 192.168.10.245       |
| Último host      | 192.168.10.246       |
| Broadcast        | 192.168.10.247       |
| Hosts            | 2                    |

Siguiente disponible: `192.168.10.248`.

---

**PASO 4 y 5. Verificación.**

```
Red original: 192.168.10.0 ──────────────────────────────── 192.168.10.255
              ├── Ventas /25 ──────────┤
              .0                     .127
                                      ├── Producción /26 ──┤
                                      .128               .191
                                                          ├── Admin /27 ──┤
                                                          .192          .223
                                                                        ├── IT /28 ┤
                                                                        .224     .239
                                                                                 ├/30┤├/30┤
                                                                                 .240 .244
                                                                                 .243 .247

Espacio libre: 192.168.10.248 – 192.168.10.255 (8 direcciones sin usar)
```

✅ No hay solapamientos.  
✅ Todo cabe dentro de la /24 original.  
✅ Cada subred tiene las direcciones suficientes.

**Resumen final del diseño VLSM:**

| Subred         | Dirección de red     | Máscara             | Rango de hosts                    | Broadcast         |
|:---------------|:--------------------:|:-------------------:|:---------------------------------:|:-----------------:|
| Ventas         | 192.168.10.0/25      | 255.255.255.128     | .1 – .126                        | .127              |
| Producción     | 192.168.10.128/26    | 255.255.255.192     | .129 – .190                      | .191              |
| Administración | 192.168.10.192/27    | 255.255.255.224     | .193 – .222                      | .223              |
| IT             | 192.168.10.224/28    | 255.255.255.240     | .225 – .238                      | .239              |
| Enlace R1-R2   | 192.168.10.240/30    | 255.255.255.252     | .241 – .242                      | .243              |
| Enlace R2-R3   | 192.168.10.244/30    | 255.255.255.252     | .245 – .246                      | .247              |

### 9.5. Reglas de oro del VLSM

> **Regla 1.** Siempre ordena las subredes de **mayor a menor** número de hosts antes de asignar.
>
> **Regla 2.** La dirección de inicio de cada bloque debe ser **múltiplo** del tamaño de ese bloque.
>
> **Regla 3.** Después de asignar cada subred, la siguiente dirección disponible es broadcast + 1.
>
> **Regla 4.** Verifica siempre que no hay **solapamientos** y que todo cabe dentro de la red original.
>
> **Regla 5.** Los enlaces punto a punto (WAN) entre routers usan `/30` (2 hosts) o `/31` (en implementaciones modernas).

---

# PARTE IV — CASOS PRÁCTICOS REALISTAS

---

## 10. Casos reales de diseño

### 10.1. Caso 1: Empresa con varios departamentos

**Escenario.** Una empresa mediana ocupa un solo edificio y dispone de la red `172.16.50.0/24`. Tiene los siguientes departamentos:

| Departamento    | Dispositivos |
|:----------------|:------------:|
| Comercial       | 55           |
| Desarrollo      | 28           |
| RRHH            | 12           |
| Dirección       | 5            |
| Enlace al ISP   | 2            |

**Solución con VLSM** (ordenado de mayor a menor):

| Prioridad | Departamento | Hosts | Prefijo | Bloque | Dir. de red         | Rango hosts         | Broadcast       |
|:---------:|:-------------|:-----:|:-------:|:------:|:-------------------:|:-------------------:|:---------------:|
| 1         | Comercial    | 55    | /26     | 64     | 172.16.50.0/26      | .1 – .62            | .63             |
| 2         | Desarrollo   | 28    | /27     | 32     | 172.16.50.64/27     | .65 – .94           | .95             |
| 3         | RRHH         | 12    | /28     | 16     | 172.16.50.96/28     | .97 – .110          | .111            |
| 4         | Dirección    | 5     | /29     | 8      | 172.16.50.112/29    | .113 – .118         | .119            |
| 5         | Enlace ISP   | 2     | /30     | 4      | 172.16.50.120/30    | .121 – .122         | .123            |

Direcciones usadas: 0 a 123 (124 de 256). Quedan 132 direcciones libres para crecimiento futuro.

### 10.2. Caso 2: Instituto con aulas y profesorado

**Escenario.** Un IES dispone de la red `10.10.0.0/16` y necesita estas subredes:

| Subred             | Hosts necesarios |
|:-------------------|:----------------:|
| Aula 1 (informática) | 30            |
| Aula 2 (informática) | 30            |
| Aula 3 (informática) | 30            |
| Sala profesores    | 15               |
| Secretaría         | 10               |
| Dirección          | 5                |
| Servidores         | 4                |
| WiFi alumnado      | 200              |
| WiFi profesorado   | 50               |
| Enlace R1-R2       | 2                |

**Solución VLSM** (ordenando de mayor a menor):

| Subred                | Hosts | Prefijo | Dir. de red       | Rango                          | Broadcast        |
|:----------------------|:-----:|:-------:|:-----------------:|:------------------------------:|:----------------:|
| WiFi alumnado         | 200   | /24     | 10.10.0.0/24      | 10.10.0.1 – 10.10.0.254       | 10.10.0.255      |
| WiFi profesorado      | 50    | /26     | 10.10.1.0/26      | 10.10.1.1 – 10.10.1.62        | 10.10.1.63       |
| Aula 1                | 30    | /27     | 10.10.1.64/27     | 10.10.1.65 – 10.10.1.94       | 10.10.1.95       |
| Aula 2                | 30    | /27     | 10.10.1.96/27     | 10.10.1.97 – 10.10.1.126      | 10.10.1.127      |
| Aula 3                | 30    | /27     | 10.10.1.128/27    | 10.10.1.129 – 10.10.1.158     | 10.10.1.159      |
| Sala profesores       | 15    | /27     | 10.10.1.160/27    | 10.10.1.161 – 10.10.1.190     | 10.10.1.191      |
| Secretaría            | 10    | /28     | 10.10.1.192/28    | 10.10.1.193 – 10.10.1.206     | 10.10.1.207      |
| Dirección             | 5     | /29     | 10.10.1.208/29    | 10.10.1.209 – 10.10.1.214     | 10.10.1.215      |
| Servidores            | 4     | /29     | 10.10.1.216/29    | 10.10.1.217 – 10.10.1.222     | 10.10.1.223      |
| Enlace R1-R2          | 2     | /30     | 10.10.1.224/30    | 10.10.1.225 – 10.10.1.226     | 10.10.1.227      |

Observa cómo con una /16 (65 534 hosts posibles), hemos organizado elegantemente 10 subredes de diferentes tamaños.

### 10.3. Caso 3: Empresa con varias sedes y topología con routers

**Escenario.** Una empresa con dos sedes unidas por un enlace WAN. Red asignada: `192.168.20.0/24`.

```
     SEDE CENTRAL                        SEDE REMOTA
  ┌────────────────┐                  ┌────────────────┐
  │  LAN Central   │                  │  LAN Remota    │
  │  (60 hosts)    │──── Router A ──── Router B ────│  (25 hosts)    │
  │                │       │  enlace punto a punto   │                │
  └────────────────┘       │       (2 hosts)         └────────────────┘
                     ┌─────┴──────┐
                     │ DMZ        │
                     │ (5 hosts)  │
                     └────────────┘
```

**Solución VLSM:**

| Subred       | Hosts | Prefijo | Dir. de red          | Broadcast         |
|:-------------|:-----:|:-------:|:--------------------:|:-----------------:|
| LAN Central  | 60    | /26     | 192.168.20.0/26      | 192.168.20.63     |
| LAN Remota   | 25    | /27     | 192.168.20.64/27     | 192.168.20.95     |
| DMZ          | 5     | /29     | 192.168.20.96/29     | 192.168.20.103    |
| Enlace WAN   | 2     | /30     | 192.168.20.104/30    | 192.168.20.107    |

Direcciones usadas: 108 de 256. Espacio libre suficiente para crecimiento.

---

# PARTE V — ERRORES TÍPICOS, TRUCOS Y ESTRATEGIAS

---

## 11. Errores típicos del alumnado

### 11.1. Tabla de errores frecuentes y cómo evitarlos

| N.º | Error típico | Por qué ocurre | Cómo evitarlo |
|:---:|:-------------|:---------------|:--------------|
| 1 | Confundir la dirección de red con el primer host | Se olvida que la dir. de red (todos los bits de host a 0) está reservada | Primer host = dirección de red **+ 1** |
| 2 | Confundir el broadcast con el último host | Se olvida que el broadcast (todos los bits de host a 1) está reservado | Último host = broadcast **− 1** |
| 3 | Olvidar restar 2 al calcular hosts | Se calcula $2^h$ en lugar de $2^h - 2$ | Memorizar: se restan la red y el broadcast |
| 4 | Confundir n.º de subredes con n.º de hosts | Se usa la misma fórmula para ambos | Subredes = $2^s$ (sin restar). Hosts = $2^h - 2$ |
| 5 | Calcular mal el tamaño del bloque | Se resta al revés o se suma mal | Bloque = $256 - m$ (donde $m$ es el octeto de la máscara) |
| 6 | Usar una dirección de red como IP de host | Se asigna `.0` o `.128` como IP de un PC | Verificar que no sea dirección de red de ninguna subred |
| 7 | Hacer VLSM sin ordenar de mayor a menor | Se empieza por la subred más pequeña | **Siempre** ordenar primero por número de hosts descendente |
| 8 | Solapar subredes en VLSM | No se controla dónde termina una subred | Siguiente subred = broadcast anterior + 1 |
| 9 | Equivocarse con /26, /27, /28… | No se domina la tabla de prefijos | Estudiar la tabla hasta que sea automática |
| 10 | Empezar un bloque en una dirección que no es múltiplo | Se ignora la regla de alineamiento | La dir. de red debe ser múltiplo del tamaño del bloque |

### 11.2. Desarrollo de los errores más peligrosos

**Error 1: Confundir red con primer host.**

```
Mal:   "La red 192.168.1.0/24 tiene hosts de .0 a .254"
Bien:  "La red 192.168.1.0/24 tiene hosts de .1 a .254"

.0    es la DIRECCIÓN DE RED → no asignable
.255  es la DIRECCIÓN DE BROADCAST → no asignable
```

**Error 7: VLSM sin ordenar.**

```
Mal (empezar por la más pequeña):
  Enlace /30: 192.168.1.0/30   → ocupa .0 a .3
  LAN grande: 192.168.1.4/25   → .4 NO es múltiplo de 128 → ¡ERROR!

Bien (empezar por la más grande):
  LAN grande: 192.168.1.0/25   → ocupa .0 a .127  ✓
  Enlace /30: 192.168.1.128/30 → .128 es múltiplo de 4  ✓
```

**Error 8: Solapar subredes.**

```
Subred A: 192.168.1.0/26    → .0 a .63
Subred B: 192.168.1.32/27   → .32 a .63   ← ¡SOLAPAMIENTO!

La subred B empieza DENTRO de la subred A.
```

Para evitarlo: la siguiente subred siempre empieza en broadcast_anterior + 1.

---

## 12. Trucos y estrategias para el examen

### 12.1. Cómo detectar rápido el salto (bloque)

Memoriza la «escalera»:

| Prefijo | Bloque | Máscara (último octeto) |
|:-------:|:------:|:-----------------------:|
| /25     | 128    | 128                     |
| /26     | 64     | 192                     |
| /27     | 32     | 224                     |
| /28     | 16     | 240                     |
| /29     | 8      | 248                     |
| /30     | 4      | 252                     |

$$\text{Bloque} = 256 - \text{máscara}$$

**Truco:** El bloque siempre es una potencia de 2. Si te sale algo que no es potencia de 2, has cometido un error.

### 12.2. Cómo saber cuántos hosts caben

$$\text{Hosts} = 2^{32 - \text{prefijo}} - 2$$

**Truco rápido:** cuenta los bits a 0 de la máscara, calcula $2^n$ y resta 2.

### 12.3. Cómo verificar si una IP pertenece a una subred

1. Calcula el bloque.
2. Divide el octeto relevante de la IP entre el bloque (división entera).
3. Multiplica el resultado por el bloque → esa es la dirección de red.

**Ejemplo.** ¿`192.168.5.147` pertenece a `192.168.5.128/26`?

- Bloque de /26 = 64.
- $147 \div 64 = 2$ (entero) → $2 \times 64 = 128$.
- La IP pertenece a la subred `192.168.5.128`. ✓

### 12.4. Cómo organizar la resolución en papel

1. **Escribe siempre los datos** del enunciado (red, máscara, requisitos).
2. **Calcula el bloque** y anótalo.
3. **Dibuja la tabla** con columnas: Subred | Red | 1.er host | Último host | Broadcast.
4. **Rellena la tabla**, subred a subred.
5. **Verifica**: ¿el broadcast de una subred + 1 = dir. de red de la siguiente?

### 12.5. Cómo revisar si el resultado tiene sentido

Checklist final:

- [ ] ¿El número de hosts por subred es $2^h - 2$ (no $2^h$)?
- [ ] ¿La dirección de red tiene los bits de host a 0?
- [ ] ¿El broadcast tiene los bits de host a 1?
- [ ] ¿El primer host = red + 1?
- [ ] ¿El último host = broadcast − 1?
- [ ] ¿Las subredes no se solapan?
- [ ] ¿Todo cabe dentro de la red original?
- [ ] ¿El bloque es potencia de 2?

---

# PARTE VI — RESUMEN FINAL Y GLOSARIO

---

## 13. Resumen final

```
    ┌─────────────────────────────────────────────────┐
    │          SUBNETTING EN UNA PÁGINA               │
    ├─────────────────────────────────────────────────┤
    │                                                 │
    │  1. Una IP tiene 32 bits: parte red + parte host│
    │                                                 │
    │  2. La máscara marca la frontera entre ambas    │
    │                                                 │
    │  3. Subnetting = tomar bits de host para crear  │
    │     más subredes (bits prestados)               │
    │                                                 │
    │  4. Más bits prestados = más subredes pero      │
    │     menos hosts por subred                      │
    │                                                 │
    │  5. Fórmulas clave:                             │
    │     · Subredes = 2^s                            │
    │     · Hosts = 2^h − 2                           │
    │     · Bloque = 256 − máscara = 2^h              │
    │                                                 │
    │  6. Direcciones reservadas por subred:           │
    │     · Red (bits host a 0)                       │
    │     · Broadcast (bits host a 1)                 │
    │                                                 │
    │  7. VLSM = subredes de distinto tamaño          │
    │     · Ordenar mayor a menor                     │
    │     · Asignar bloque justo                      │
    │     · Verificar no solapar                      │
    │                                                 │
    │  8. Método del bloque: la herramienta estrella  │
    │     para resolver rápido                        │
    └─────────────────────────────────────────────────┘
```

---

## 14. Glosario

| Término                   | Definición                                                                                   |
|:--------------------------|:---------------------------------------------------------------------------------------------|
| **Dirección IPv4**        | Identificador de 32 bits asignado a una interfaz de red para la comunicación IP              |
| **Octeto**                | Cada uno de los 4 grupos de 8 bits de una dirección IPv4                                     |
| **Máscara de subred**     | Número de 32 bits que separa la parte de red de la parte de host en una IP                   |
| **Prefijo CIDR**          | Notación compacta que indica cuántos bits de la máscara están a 1 (ej: /24)                  |
| **Dirección de red**      | Primera dirección de una subred; bits de host a 0. No asignable a dispositivos               |
| **Broadcast**             | Última dirección de una subred; bits de host a 1. Envía a todos los hosts de la subred       |
| **Host válido**           | Cualquier dirección entre la de red y la de broadcast, asignable a un dispositivo            |
| **Subnetting**            | Proceso de dividir una red en subredes más pequeñas tomando bits de la parte de host         |
| **Bits prestados**        | Bits que se toman de la parte de host para ampliar la parte de red y crear subredes          |
| **Tamaño del bloque**     | Número total de direcciones de una subred; distancia entre el inicio de subredes consecutivas|
| **VLSM**                  | Variable Length Subnet Masking: subredes de distinto tamaño dentro de la misma red           |
| **Dominio de broadcast**  | Conjunto de dispositivos que reciben un mensaje broadcast                                    |
| **AND binario**           | Operación lógica que aplicada a IP y máscara da como resultado la dirección de red           |
| **CIDR**                  | Classless Inter-Domain Routing: sistema de direccionamiento sin clases                       |
| **Enlace punto a punto**  | Conexión directa entre dos routers; típicamente usa /30 o /31                                |

---

# PARTE VII — EJERCICIOS RESUELTOS

---

## 15. Ejercicios resueltos — Nivel 1: Comprensión básica

### Ejercicio 1.1. Identificar red, broadcast y hosts

**Enunciado.**  
Dada la dirección `192.168.5.75/24`, determina la dirección de red, el broadcast, el primer y último host válidos, y el número de hosts utilizables.

**Qué se pide.**  
Dirección de red, broadcast, primer host, último host, número de hosts.

**Datos disponibles.**  
- IP: `192.168.5.75`  
- Máscara: `/24` → `255.255.255.0`

**Conceptos implicados.**  
Dirección de red, broadcast, rango de hosts, fórmula $2^h - 2$.

**Resolución paso a paso.**

1. La máscara `/24` indica que los primeros 24 bits son red y los últimos 8 son host.
2. Dirección de red: ponemos los bits de host a 0 → el último octeto es 0 → `192.168.5.0`.
3. Broadcast: ponemos los bits de host a 1 → el último octeto es 255 → `192.168.5.255`.
4. Primer host: red + 1 = `192.168.5.1`.
5. Último host: broadcast − 1 = `192.168.5.254`.
6. Número de hosts: $2^8 - 2 = 254$.

**Resultado final.**

| Dato             | Valor            |
|:-----------------|:-----------------|
| Dirección de red | 192.168.5.0      |
| Primer host      | 192.168.5.1      |
| Último host      | 192.168.5.254    |
| Broadcast        | 192.168.5.255    |
| Hosts válidos    | 254              |

**Interpretación.**  
La IP `192.168.5.75` pertenece a la red `192.168.5.0/24`, que tiene capacidad para 254 dispositivos.

**Error típico.**  
Escribir que la red tiene 256 hosts. Se olvida restar las 2 direcciones reservadas (red y broadcast).

**Mini variante.**  
Si la máscara fuera `/25`, ¿cuál sería la red? Ayuda: calcula el bloque (128), y 75 cae entre 0 y 128 → red = `192.168.5.0/25`, broadcast = `192.168.5.127`.

---

### Ejercicio 1.2. Distinguir parte de red y parte de host

**Enunciado.**  
Para la dirección `10.200.15.130/16`, indica qué parte identifica la red y qué parte identifica al host.

**Qué se pide.**  
Separar la dirección en parte de red y parte de host.

**Datos disponibles.**  
- IP: `10.200.15.130`  
- Prefijo: `/16`

**Conceptos implicados.**  
Bits de red, bits de host, máscara como separador.

**Resolución paso a paso.**

1. `/16` significa que los primeros 16 bits (2 octetos) son red.
2. Parte de red: `10.200` (los primeros dos octetos).
3. Parte de host: `15.130` (los dos últimos octetos).
4. Dirección de red: `10.200.0.0`.
5. El host `15.130` identifica a este dispositivo concreto dentro de la red `10.200.0.0/16`.

**Resultado final.**
```
IP:     10 . 200 . 15  . 130
        ├─ RED ─┤  ├─ HOST ─┤
        16 bits     16 bits
```

**Interpretación.**  
Esta es una red grande (clase A privada) con $2^{16} - 2 = 65\,534$ hosts posibles.

**Error típico.**  
Pensar que `10` es la parte de red y `200.15.130` es la parte de host. Eso sería correcto solo con `/8`, no con `/16`.

---

### Ejercicio 1.3. Interpretar una máscara

**Enunciado.**  
¿Qué prefijo CIDR corresponde a la máscara `255.255.255.224`? ¿Cuántos hosts caben en una subred con esa máscara?

**Qué se pide.**  
Prefijo CIDR equivalente y número de hosts.

**Datos disponibles.**  
Máscara: `255.255.255.224`.

**Conceptos implicados.**  
Conversión máscara ↔ CIDR, fórmula de hosts.

**Resolución paso a paso.**

1. Los tres primeros octetos son 255 → 24 bits a 1.
2. El cuarto octeto es 224 → en binario: `11100000` → 3 bits a 1.
3. Total bits a 1: $24 + 3 = 27$.
4. **Prefijo:** `/27`.
5. Bits de host: $32 - 27 = 5$.
6. **Hosts:** $2^5 - 2 = 30$.

**Resultado final.**  
`/27`, 30 hosts por subred.

**Interpretación.**  
Con esta máscara se pueden crear subredes para grupos de hasta 30 dispositivos.

**Error típico.**  
Calcular $224$ como si fueran 4 bits a 1 (creer que $224 = 11110000$). En realidad, $11110000 = 240$, no 224.

---

## 16. Ejercicios resueltos — Nivel 2: Clasificación e identificación

### Ejercicio 2.1. ¿Están en la misma subred?

**Enunciado.**  
¿Las direcciones `192.168.1.100` y `192.168.1.200` pertenecen a la misma subred si la máscara es `/25`?

**Qué se pide.**  
Determinar si ambas IP están en la misma subred.

**Datos disponibles.**  
- IP-A: `192.168.1.100`  
- IP-B: `192.168.1.200`  
- Máscara: `/25`

**Conceptos implicados.**  
Bloque, dirección de red, pertenencia a subred.

**Resolución paso a paso.**

1. `/25` → máscara `255.255.255.128` → bloque = $256 - 128 = 128$.
2. Múltiplos de 128 en el cuarto octeto: **0**, **128**, 256.
3. IP-A: 100 está entre 0 y 128 → subred `192.168.1.0/25`.
4. IP-B: 200 está entre 128 y 256 → subred `192.168.1.128/25`.
5. **No están en la misma subred.**

**Resultado final.**  
`192.168.1.100` está en `192.168.1.0/25`. `192.168.1.200` está en `192.168.1.128/25`. **Son subredes diferentes.**

**Interpretación.**  
Aunque las dos IP comparten los tres primeros octetos, la máscara `/25` divide el cuarto octeto en dos bloques de 128, y cada IP cae en un bloque distinto. Necesitarían un router para comunicarse.

**Error típico.**  
Asumir que si los tres primeros octetos son iguales, están en la misma red. Con máscaras más largas que /24, el cuarto octeto también participa en la segmentación.

**Mini variante.**  
Con máscara `/24`, ¿estarían en la misma red? Sí, porque `/24` no divide el cuarto octeto.

---

### Ejercicio 2.2. ¿Es una dirección de red, de broadcast o de host?

**Enunciado.**  
Clasifica cada una de las siguientes direcciones como dirección de red, broadcast o host válido, sabiendo que la máscara es `/28`:

a) `172.16.5.0`  
b) `172.16.5.15`  
c) `172.16.5.10`  
d) `172.16.5.16`  
e) `172.16.5.31`  
f) `172.16.5.20`

**Qué se pide.**  
Clasificar cada dirección.

**Datos disponibles.**  
Máscara `/28` → bloque = $256 - 240 = 16$.

**Resolución paso a paso.**

Múltiplos de 16: 0, 16, 32, 48, 64, 80, 96, 112, 128, 144, 160, 176, 192, 208, 224, 240, 256.

| Dirección    | ¿Múltiplo de 16? | ¿Múltiplo − 1? | Clasificación      |
|:-------------|:-----------------:|:---------------:|:--------------------|
| 172.16.5.0   | Sí (0)            | —               | **Dirección de red** |
| 172.16.5.15  | —                 | Sí (16 − 1)    | **Broadcast**        |
| 172.16.5.10  | No                | No              | **Host válido**      |
| 172.16.5.16  | Sí (16)           | —               | **Dirección de red** |
| 172.16.5.31  | —                 | Sí (32 − 1)    | **Broadcast**        |
| 172.16.5.20  | No                | No              | **Host válido**      |

**Interpretación.**  
Con bloque 16, las direcciones de red son: 0, 16, 32, 48, …  
Los broadcast son: 15, 31, 47, 63, …  
Todo lo demás son hosts válidos.

**Error típico.**  
Pensar que `172.16.5.0` siempre es una dirección de red solo porque termina en 0. Con máscara `/25`, por ejemplo, `172.16.5.0` sería efectivamente una dirección de red, pero `172.16.5.128` también lo sería (no solo las que terminan en 0).

---

## 17. Ejercicios resueltos — Nivel 3: Cálculo guiado

### Ejercicio 3.1. Dividir 192.168.1.0/24 en 2 subredes

**Enunciado.**  
Divide la red `192.168.1.0/24` en 2 subredes iguales.

**Qué se pide.**  
Tabla completa con dir. de red, primer host, último host, broadcast y número de hosts.

**Datos disponibles.**  
Red: `192.168.1.0/24`. Subredes necesarias: 2.

**Conceptos implicados.**  
Bits prestados, nueva máscara, tamaño del bloque.

**Resolución paso a paso.**

1. Necesito 2 subredes → $2^s \geq 2$ → $s = 1$.
2. Nuevo prefijo: $24 + 1 = /25$.
3. Máscara: `255.255.255.128`.
4. Bits de host: $32 - 25 = 7$.
5. Hosts por subred: $2^7 - 2 = 126$.
6. Bloque: $256 - 128 = 128$.
7. Subredes: empiezan en 0, 128.

**Resultado final.**

| Subred | Dir. de red     | Primer host     | Último host     | Broadcast       | Hosts |
|:------:|:---------------:|:---------------:|:---------------:|:---------------:|:-----:|
| 1      | 192.168.1.0     | 192.168.1.1     | 192.168.1.126   | 192.168.1.127   | 126   |
| 2      | 192.168.1.128   | 192.168.1.129   | 192.168.1.254   | 192.168.1.255   | 126   |

**Interpretación.**  
Hemos dividido una red de 254 hosts en dos redes de 126 hosts cada una. Se han «perdido» 2 direcciones adicionales (una de red y una de broadcast por subred extra).

**Error típico.**  
Pensar que el primer host de la segunda subred es `.128`. No: `.128` es la dirección de red de la segunda subred; el primer host es `.129`.

---

### Ejercicio 3.2. Dividir 192.168.1.0/24 en 4 subredes

**Enunciado.**  
Divide la red `192.168.1.0/24` en 4 subredes iguales.

**Resolución paso a paso.**

1. $2^s \geq 4$ → $s = 2$.
2. Nuevo prefijo: `/26`. Máscara: `255.255.255.192`.
3. Bloque: $256 - 192 = 64$.
4. Hosts: $2^6 - 2 = 62$.

**Resultado final.**

| Subred | Dir. de red     | Primer host     | Último host     | Broadcast       | Hosts |
|:------:|:---------------:|:---------------:|:---------------:|:---------------:|:-----:|
| 1      | 192.168.1.0     | 192.168.1.1     | 192.168.1.62    | 192.168.1.63    | 62    |
| 2      | 192.168.1.64    | 192.168.1.65    | 192.168.1.126   | 192.168.1.127   | 62    |
| 3      | 192.168.1.128   | 192.168.1.129   | 192.168.1.190   | 192.168.1.191   | 62    |
| 4      | 192.168.1.192   | 192.168.1.193   | 192.168.1.254   | 192.168.1.255   | 62    |

**Error típico.**  
Calcular que el bloque es 63 (restando 1 extra). El bloque es 64, no 63. El último host está en la posición bloque − 2 respecto al inicio.

---

### Ejercicio 3.3. Dividir 192.168.1.0/24 en 8 subredes

**Enunciado.**  
Divide la red `192.168.1.0/24` en 8 subredes iguales.

**Resolución paso a paso.**

1. $2^s \geq 8$ → $s = 3$.
2. Nuevo prefijo: `/27`. Máscara: `255.255.255.224`.
3. Bloque: $256 - 224 = 32$.
4. Hosts: $2^5 - 2 = 30$.

**Resultado final.**

| Subred | Dir. de red     | Primer host     | Último host     | Broadcast       | Hosts |
|:------:|:---------------:|:---------------:|:---------------:|:---------------:|:-----:|
| 1      | 192.168.1.0     | 192.168.1.1     | 192.168.1.30    | 192.168.1.31    | 30    |
| 2      | 192.168.1.32    | 192.168.1.33    | 192.168.1.62    | 192.168.1.63    | 30    |
| 3      | 192.168.1.64    | 192.168.1.65    | 192.168.1.94    | 192.168.1.95    | 30    |
| 4      | 192.168.1.96    | 192.168.1.97    | 192.168.1.126   | 192.168.1.127   | 30    |
| 5      | 192.168.1.128   | 192.168.1.129   | 192.168.1.158   | 192.168.1.159   | 30    |
| 6      | 192.168.1.160   | 192.168.1.161   | 192.168.1.190   | 192.168.1.191   | 30    |
| 7      | 192.168.1.192   | 192.168.1.193   | 192.168.1.222   | 192.168.1.223   | 30    |
| 8      | 192.168.1.224   | 192.168.1.225   | 192.168.1.254   | 192.168.1.255   | 30    |

---

### Ejercicio 3.4. Hallar red, broadcast y rango de una IP dada

**Enunciado.**  
Dada la IP `172.20.45.100/21`, determina red, broadcast, primer host, último host y número de hosts.

**Qué se pide.**  
Todos los datos de la subred a la que pertenece esta IP.

**Datos disponibles.**  
IP: `172.20.45.100`, prefijo `/21`.

**Resolución paso a paso.**

1. `/21` → 21 bits de red, 11 bits de host.
2. Máscara: los primeros 2 octetos = 255.255. El tercer octeto tiene 5 bits a 1: `11111000` = 248.
3. Máscara completa: `255.255.248.0`.
4. Bloque en el tercer octeto: $256 - 248 = 8$.
5. Tercer octeto de la IP = 45. Múltiplos de 8: 0, 8, 16, 24, 32, **40**, **48**, ...
6. 45 está entre 40 y 48 → la red comienza en 40.
7. **Dir. de red:** `172.20.40.0`.
8. Siguiente subred: `172.20.48.0` → **Broadcast:** `172.20.47.255`.
9. Primer host: `172.20.40.1`.
10. Último host: `172.20.47.254`.
11. Hosts: $2^{11} - 2 = 2046$.

**Resultado final.**

| Dato             | Valor            |
|:-----------------|:-----------------|
| Dirección de red | 172.20.40.0      |
| Primer host      | 172.20.40.1      |
| Último host      | 172.20.47.254    |
| Broadcast        | 172.20.47.255    |
| Hosts válidos    | 2 046            |

**Interpretación.**  
Esta es una subred de una red de clase B privada, con espacio para más de 2 000 dispositivos. El rango abarca los octetos terceros del 40 al 47 (8 valores), cada uno con 256 direcciones → $8 \times 256 = 2048$ direcciones totales, de las cuales 2 046 son hosts utilizables.

**Error típico.**  
Olvidar que cuando el bloque afecta al tercer octeto, el cuarto octeto varía de 0 a 255 completo dentro de cada subred.

---

### Ejercicio 3.5. ¿Cuántos hosts admite una subred /28?

**Enunciado.**  
¿Cuántos hosts utilizables tiene una subred con prefijo /28?

**Resolución paso a paso.**

1. Bits de host: $32 - 28 = 4$.
2. Hosts: $2^4 - 2 = 14$.

**Resultado:** 14 hosts utilizables por subred.

---

### Ejercicio 3.6. ¿Cuántas subredes se obtienen al tomar 5 bits prestados de una /24?

**Enunciado.**  
Partiendo de una red /24, si se toman 5 bits prestados, ¿cuántas subredes se generan?

**Resolución.**

$$2^5 = 32 \text{ subredes}$$

Nuevo prefijo: $24 + 5 = /29$. Hosts por subred: $2^3 - 2 = 6$.

---

### Ejercicio 3.7. Decidir el prefijo a partir de subredes necesarias

**Enunciado.**  
Partiendo de `10.0.0.0/8`, necesitas al menos 1 000 subredes. ¿Qué prefijo debes usar?

**Resolución paso a paso.**

1. $2^s \geq 1000$.
2. $2^{10} = 1024 \geq 1000$ ✓. Con $s = 10$.
3. Nuevo prefijo: $8 + 10 = /18$.
4. Hosts por subred: $2^{14} - 2 = 16\,382$.

**Resultado:** Prefijo `/18`, que genera 1 024 subredes de 16 382 hosts cada una.

---

### Ejercicio 3.8. Decidir el prefijo a partir de hosts necesarios

**Enunciado.**  
Necesitas subredes de al menos 500 hosts. ¿Qué prefijo debes usar?

**Resolución paso a paso.**

1. $2^h - 2 \geq 500$.
2. $2^9 - 2 = 510 \geq 500$ ✓. Con $h = 9$.
3. Prefijo: $32 - 9 = /23$.
4. Máscara: `255.255.254.0`.

**Resultado:** Prefijo `/23`, que permite 510 hosts por subred.

**Error típico.**  
Usar $h = 8$ ($2^8 - 2 = 254$), que es insuficiente para 500 hosts.

---

## 18. Ejercicios resueltos — Nivel 4: Análisis intermedio

### Ejercicio 4.1. Elegir la máscara adecuada

**Enunciado.**  
Una oficina necesita una subred para 45 equipos. ¿Cuál es la máscara más ajustada que cubre esa necesidad?

**Qué se pide.**  
La máscara con menor desperdicio que aloje al menos 45 hosts.

**Resolución paso a paso.**

1. $2^h - 2 \geq 45$.
2. $2^5 - 2 = 30$ → insuficiente (30 < 45).
3. $2^6 - 2 = 62$ → **suficiente** (62 ≥ 45).
4. Prefijo: $32 - 6 = /26$.
5. Máscara: `255.255.255.192`.

**Resultado:** `/26` (255.255.255.192), con 62 hosts, desperdiciando solo 17 direcciones.

**Comparación de opciones:**

| Prefijo | Hosts | ¿Suficiente? | Desperdicio |
|:-------:|:-----:|:------------:|:-----------:|
| /27     | 30    | No           | —           |
| /26     | 62    | **Sí** ✓     | 17          |
| /25     | 126   | Sí           | 81          |

Elegimos `/26` porque es la que menos desperdicia cumpliendo el requisito.

---

### Ejercicio 4.2. Detectar errores en una propuesta de subnetting

**Enunciado.**  
Un compañero ha diseñado el siguiente esquema de subnetting para la red `192.168.50.0/24`. Detecta los errores.

| Subred | Dir. de red         | Primer host       | Último host       | Broadcast         | Máscara |
|:------:|:-------------------:|:-----------------:|:-----------------:|:-----------------:|:-------:|
| A      | 192.168.50.0/26     | 192.168.50.0      | 192.168.50.62     | 192.168.50.63     | /26     |
| B      | 192.168.50.64/26    | 192.168.50.65     | 192.168.50.127    | 192.168.50.127    | /26     |
| C      | 192.168.50.100/27   | 192.168.50.101    | 192.168.50.130    | 192.168.50.131    | /27     |

**Errores encontrados:**

1. **Subred A — Primer host mal.** El primer host es `.0`; debería ser `.1`. La dirección `.0` es la dirección de red y no es asignable.

2. **Subred B — Broadcast y último host coinciden.** El broadcast es `.127` y el último host también es `.127`. Error: el último host debería ser `.126` (broadcast − 1).

3. **Subred C — Solapamiento.** La subred C empieza en `.100`, pero la subred B va de `.64` a `.127`. La dirección `.100` ya pertenece a la subred B. ¡Solapamiento grave!

4. **Subred C — Dirección de red inválida.** `.100` no es múltiplo de 32 (bloque de /27). Los múltiplos de 32 son: 0, 32, 64, 96, **128**, 160… La subred debería empezar en `.128`, no en `.100`.

5. **Subred C — Rango y broadcast incorrectos.** Con /27 empezando en `.128`: rango sería `.129` a `.158`, broadcast `.159`.

---

## 19. Ejercicios resueltos — Nivel 5: Síntesis y diseño

### Ejercicio 5.1. Diseño VLSM para un centro educativo

**Enunciado.**  
Eres el administrador de red de un instituto. Te asignan la red `172.20.100.0/23` (dos /24 consecutivas, es decir, 510 hosts disponibles). Necesitas crear subredes para:

| Subred              | Hosts necesarios |
|:--------------------|:----------------:|
| Aula informática A  | 60               |
| Aula informática B  | 60               |
| WiFi alumnado       | 200              |
| WiFi profesorado    | 30               |
| Administración      | 20               |
| Sala de servidores  | 10               |
| Enlace R1-Switch L3 | 2                |

**Qué se pide.**  
Diseño completo con VLSM. Tabla de direccionamiento. Verificación de no solapamiento.

**Datos disponibles.**  
- Red: `172.20.100.0/23`
- Rango total: `172.20.100.0` a `172.20.101.255` (512 direcciones, 510 hosts)
- Total hosts necesarios: $60 + 60 + 200 + 30 + 20 + 10 + 2 = 382$. Cabe en la /23.

**Conceptos implicados.**  
VLSM, ordenación de mayor a menor, cálculo de bloques, verificación de alineamiento.

**Resolución paso a paso.**

**Paso 1. Ordenar de mayor a menor:**

| Prioridad | Subred                | Hosts |
|:---------:|:----------------------|:-----:|
| 1         | WiFi alumnado         | 200   |
| 2         | Aula informática A    | 60    |
| 3         | Aula informática B    | 60    |
| 4         | WiFi profesorado      | 30    |
| 5         | Administración        | 20    |
| 6         | Sala de servidores    | 10    |
| 7         | Enlace R1-Switch L3   | 2     |

**Paso 2. Asignar bloques:**

**WiFi alumnado (200 hosts):**
- $2^8 - 2 = 254 \geq 200$ → $h = 8$, prefijo `/24`, bloque 256.
- Inicio: `172.20.100.0` (múltiplo de 256 → ✓).

| Dir. de red        | Rango                              | Broadcast         |
|:------------------:|:----------------------------------:|:-----------------:|
| 172.20.100.0/24    | 172.20.100.1 – 172.20.100.254     | 172.20.100.255    |

Siguiente disponible: `172.20.101.0`.

**Aula informática A (60 hosts):**
- $2^6 - 2 = 62 \geq 60$ → $h = 6$, prefijo `/26`, bloque 64.
- Inicio: `172.20.101.0` (múltiplo de 64 → ✓).

| Dir. de red        | Rango                              | Broadcast         |
|:------------------:|:----------------------------------:|:-----------------:|
| 172.20.101.0/26    | 172.20.101.1 – 172.20.101.62      | 172.20.101.63     |

Siguiente disponible: `172.20.101.64`.

**Aula informática B (60 hosts):**
- Misma máscara: `/26`, bloque 64.
- Inicio: `172.20.101.64` (múltiplo de 64 → ✓).

| Dir. de red        | Rango                              | Broadcast         |
|:------------------:|:----------------------------------:|:-----------------:|
| 172.20.101.64/26   | 172.20.101.65 – 172.20.101.126    | 172.20.101.127    |

Siguiente disponible: `172.20.101.128`.

**WiFi profesorado (30 hosts):**
- $2^5 - 2 = 30 \geq 30$ → $h = 5$, prefijo `/27`, bloque 32.
- Inicio: `172.20.101.128` (múltiplo de 32 → ✓).

| Dir. de red          | Rango                              | Broadcast         |
|:--------------------:|:----------------------------------:|:-----------------:|
| 172.20.101.128/27    | 172.20.101.129 – 172.20.101.158   | 172.20.101.159    |

Siguiente disponible: `172.20.101.160`.

**Administración (20 hosts):**
- $2^5 - 2 = 30 \geq 20$ → $h = 5$, prefijo `/27`, bloque 32.
- Inicio: `172.20.101.160` (múltiplo de 32 → ✓).

| Dir. de red          | Rango                              | Broadcast         |
|:--------------------:|:----------------------------------:|:-----------------:|
| 172.20.101.160/27    | 172.20.101.161 – 172.20.101.190   | 172.20.101.191    |

Siguiente disponible: `172.20.101.192`.

**Sala de servidores (10 hosts):**
- $2^4 - 2 = 14 \geq 10$ → $h = 4$, prefijo `/28`, bloque 16.
- Inicio: `172.20.101.192` (múltiplo de 16 → ✓).

| Dir. de red          | Rango                              | Broadcast         |
|:--------------------:|:----------------------------------:|:-----------------:|
| 172.20.101.192/28    | 172.20.101.193 – 172.20.101.206   | 172.20.101.207    |

Siguiente disponible: `172.20.101.208`.

**Enlace R1-Switch L3 (2 hosts):**
- $2^2 - 2 = 2$ → $h = 2$, prefijo `/30`, bloque 4.
- Inicio: `172.20.101.208` (múltiplo de 4 → ✓).

| Dir. de red          | Rango                              | Broadcast         |
|:--------------------:|:----------------------------------:|:-----------------:|
| 172.20.101.208/30    | 172.20.101.209 – 172.20.101.210   | 172.20.101.211    |

**Paso 3. Verificación visual:**

```
172.20.100.0 ──────────────────────────────────────── 172.20.101.255
├─── WiFi alumnado /24 ───────────────────────────┤
100.0                                          100.255
                                                  ├── AulaA /26──┤
                                                  101.0       101.63
                                                               ├── AulaB /26──┤
                                                               101.64      101.127
                                                                            ├─Prof/27─┤
                                                                            .128    .159
                                                                                   ├Adm/27┤
                                                                                   .160 .191
                                                                                        ├Srv/28┤
                                                                                        .192 .207
                                                                                             ├/30┤
                                                                                             .208.211
Libre: 172.20.101.212 – 172.20.101.255 (44 direcciones)
```

✅ Sin solapamientos.  
✅ Todo cabe dentro de la /23 original.  
✅ Quedan 44 direcciones libres para crecimiento futuro.

**Resultado final — Tabla resumen:**

| Subred              | Dir. de red          | Prefijo | Máscara          | Rango hosts               | Broadcast       | Hosts |
|:--------------------|:--------------------:|:-------:|:----------------:|:-------------------------:|:---------------:|:-----:|
| WiFi alumnado       | 172.20.100.0         | /24     | 255.255.255.0    | 100.1 – 100.254           | 100.255         | 254   |
| Aula A              | 172.20.101.0         | /26     | 255.255.255.192  | 101.1 – 101.62            | 101.63          | 62    |
| Aula B              | 172.20.101.64        | /26     | 255.255.255.192  | 101.65 – 101.126          | 101.127         | 62    |
| WiFi profesorado    | 172.20.101.128       | /27     | 255.255.255.224  | 101.129 – 101.158         | 101.159         | 30    |
| Administración      | 172.20.101.160       | /27     | 255.255.255.224  | 101.161 – 101.190         | 101.191         | 30    |
| Servidores          | 172.20.101.192       | /28     | 255.255.255.240  | 101.193 – 101.206         | 101.207         | 14    |
| Enlace R1-SwL3      | 172.20.101.208       | /30     | 255.255.255.252  | 101.209 – 101.210         | 101.211         | 2     |

**Error típico.**  
No comprobar que la subred WiFi alumnado (254 hosts) más las demás subredes caben en la /23 (510 hosts). Con 382 hosts necesarios y 510 disponibles, hay margen suficiente.

---

# PARTE VIII — TABLAS COMPARATIVAS OBLIGATORIAS

---

## 20. Tablas comparativas de referencia

### Tabla 1. Dirección de red vs broadcast vs host válido

| Concepto          | Bits de host      | Ejemplo (/26, red .128)  | ¿Asignable? | Función                    |
|:------------------|:-----------------:|:------------------------:|:-----------:|:---------------------------|
| Dirección de red  | Todos a 0         | 192.168.1.128            | No          | Identifica la subred       |
| Primer host       | 000001            | 192.168.1.129            | Sí          | Primera IP asignable       |
| Último host       | 111110            | 192.168.1.190            | Sí          | Última IP asignable        |
| Broadcast         | Todos a 1         | 192.168.1.191            | No          | Difusión a toda la subred  |

### Tabla 2. Máscara decimal vs prefijo CIDR (completa para el 4.º octeto)

| Prefijo | Máscara (4.º octeto) | Binario     | Bloque | Subredes (desde /24) | Hosts |
|:-------:|:--------------------:|:-----------:|:------:|:--------------------:|:-----:|
| /24     | .0                   | 00000000    | 256    | 1                    | 254   |
| /25     | .128                 | 10000000    | 128    | 2                    | 126   |
| /26     | .192                 | 11000000    | 64     | 4                    | 62    |
| /27     | .224                 | 11100000    | 32     | 8                    | 30    |
| /28     | .240                 | 11110000    | 16     | 16                   | 14    |
| /29     | .248                 | 11111000    | 8      | 32                   | 6     |
| /30     | .252                 | 11111100    | 4      | 64                   | 2     |
| /31     | .254                 | 11111110    | 2      | 128                  | 2*    |
| /32     | .255                 | 11111111    | 1      | 256                  | 1*    |

(*) /31 y /32 son casos especiales: /31 para enlaces punto a punto (RFC 3021), /32 para host routes.

### Tabla 3. Número de bits prestados vs número de subredes

| Bits prestados ($s$) | Subredes ($2^s$) |
|:--------------------:|:----------------:|
| 1                    | 2                |
| 2                    | 4                |
| 3                    | 8                |
| 4                    | 16               |
| 5                    | 32               |
| 6                    | 64               |
| 7                    | 128              |
| 8                    | 256              |
| 9                    | 512              |
| 10                   | 1 024            |

### Tabla 4. Número de bits de host vs hosts utilizables

| Bits de host ($h$) | Total direcciones ($2^h$) | Hosts utilizables ($2^h - 2$) |
|:------------------:|:-------------------------:|:-----------------------------:|
| 2                  | 4                         | 2                             |
| 3                  | 8                         | 6                             |
| 4                  | 16                        | 14                            |
| 5                  | 32                        | 30                            |
| 6                  | 64                        | 62                            |
| 7                  | 128                       | 126                           |
| 8                  | 256                       | 254                           |
| 9                  | 512                       | 510                           |
| 10                 | 1 024                     | 1 022                         |
| 11                 | 2 048                     | 2 046                         |
| 12                 | 4 096                     | 4 094                         |
| 16                 | 65 536                    | 65 534                        |
| 24                 | 16 777 216                | 16 777 214                    |

### Tabla 5. Subnetting clásico vs VLSM

| Aspecto                    | Subnetting clásico            | VLSM                               |
|:---------------------------|:-----------------------------|:-------------------------------------|
| Tamaño de subredes         | Todas iguales                | Cada una diferente                   |
| Máscara                    | Una sola para todas          | Diferente por subred                 |
| Desperdicio de IPs         | Alto si las necesidades varían | Mínimo                             |
| Complejidad de diseño      | Baja                         | Media-alta                           |
| Dificultad de gestión      | Baja                         | Media                                |
| Flexibilidad               | Baja                         | Alta                                 |
| Uso real en producción     | Raro                         | **Estándar**                        |
| Protocolo de enrutamiento  | Cualquiera                   | Requiere classless (OSPF, EIGRP…)    |
| Procedimiento              | Directo                      | Requiere ordenar y verificar         |

### Tabla 6. Errores típicos y cómo evitarlos

| Error                                       | Consecuencia                     | Prevención                              |
|:--------------------------------------------|:---------------------------------|:----------------------------------------|
| Asignar la dir. de red a un host            | El host no funciona correctamente | Primer host = dir. red + 1              |
| Asignar el broadcast a un host              | Conflictos de tráfico            | Último host = broadcast − 1             |
| Calcular $2^h$ en lugar de $2^h - 2$        | Creer que caben más hosts        | Siempre restar 2                        |
| No ordenar en VLSM                          | Bloques desalineados             | Ordenar de mayor a menor siempre        |
| Solapar subredes                             | Dispositivos inalcanzables       | Siguiente = broadcast anterior + 1      |
| Empezar en dirección no alineada            | Subred inválida                  | Dir. red debe ser múltiplo del bloque   |
| Confundir subredes con hosts                | Cálculo completamente erróneo    | Subredes = $2^s$, hosts = $2^h - 2$    |
| No verificar al final                        | Errores no detectados            | Usar checklist de revisión              |

---

# PARTE IX — EJERCICIOS PROPUESTOS

---

## 21. Ejercicios propuestos sin resolver

### Nivel básico

**Ejercicio P1.** Dada la IP `10.0.5.200/8`, determina: dirección de red, broadcast, primer host, último host y número de hosts utilizables.  
*Pista: con /8, los tres últimos octetos son host.*

**Ejercicio P2.** Convierte la máscara `255.255.252.0` a notación CIDR. ¿Cuántos bits de host tiene?  
*Pista: 252 en binario es 11111100.*

**Ejercicio P3.** ¿Cuántos hosts utilizables tiene una subred /29?  
*Pista: usa la fórmula $2^h - 2$ con $h = 32 - 29$.*

**Ejercicio P4.** Dada `192.168.100.45/27`, encuentra la dirección de red y el broadcast.  
*Pista: bloque de /27 = 32. ¿Entre qué múltiplos de 32 cae 45?*

---

### Nivel intermedio

**Ejercicio P5.** Divide la red `192.168.30.0/24` en 16 subredes iguales. Completa una tabla con: dirección de red, primer host, último host, broadcast y número de hosts para las primeras 4 subredes.

**Ejercicio P6.** ¿Las IP `172.16.45.100/20` y `172.16.50.200/20` están en la misma subred? Justifica tu respuesta.  
*Pista: el bloque en /20 afecta al tercer octeto. Bloque = 256 − 240 = 16.*

**Ejercicio P7.** Un departamento necesita exactamente 100 hosts. ¿Cuál es la máscara más ajustada? ¿Cuántos hosts sobrantes quedan?

**Ejercicio P8.** Partiendo de `10.10.0.0/16`, necesitas crear 500 subredes. ¿Qué prefijo usarías? ¿Cuántos hosts por subred tendrías?

---

### Nivel avanzado

**Ejercicio P9.** Diseña un esquema VLSM para la red `192.168.80.0/24` que satisfaga:

| Subred         | Hosts necesarios |
|:---------------|:----------------:|
| Marketing      | 50               |
| Ventas         | 30               |
| Contabilidad   | 12               |
| Gerencia       | 6                |
| Enlace WAN 1   | 2                |
| Enlace WAN 2   | 2                |

*Pista: ordena de mayor a menor. Marketing necesita /26 (62 hosts). Empieza por ella.*

**Ejercicio P10.** Un ISP te asigna `203.0.113.0/24`. Tienes 4 clientes con estas necesidades:

- Cliente A: 100 dispositivos  
- Cliente B: 50 dispositivos  
- Cliente C: 20 dispositivos  
- Cliente D: 10 dispositivos  

Diseña un esquema VLSM. ¿Cabe todo en la /24? Si no cabe, justifica por qué.

*Pista: suma los hosts mínimos necesarios y compáralos con los 254 disponibles.*

**Ejercicio P11.** Detecta los errores en este diseño VLSM para `172.16.0.0/24`:

| Subred | Dir. de red        | Máscara              |
|:------:|:------------------:|:--------------------:|
| A      | 172.16.0.0/25      | 255.255.255.128      |
| B      | 172.16.0.100/26    | 255.255.255.192      |
| C      | 172.16.0.192/27    | 255.255.255.224      |

*Pista: ¿es `.100` un inicio válido para un bloque /26?*

---

### Ejercicio propuesto completo de VLSM

**Ejercicio P12.** Eres el responsable de red de una empresa con tres sedes. La red asignada es `10.50.0.0/16`. Diseña el esquema de direccionamiento completo:

| Sede    | Subred           | Hosts necesarios |
|:--------|:-----------------|:----------------:|
| Central | Producción       | 500              |
| Central | Oficinas         | 200              |
| Central | Servidores       | 20               |
| Remota A| LAN              | 100              |
| Remota A| WiFi             | 50               |
| Remota B| LAN              | 30               |
| —       | Enlace Central-A | 2                |
| —       | Enlace Central-B | 2                |
| —       | Enlace A-B       | 2                |

*Pista: comienza por la subred de 500 hosts (necesita /23, bloque 512). Comprueba que 10.50.0.0 es múltiplo de 512 ÷ 256 en el tercer octeto.*

---

## 22. Pistas y soluciones resumidas

| Ejercicio | Resultado clave |
|:---------:|:----------------|
| P1        | Red: `10.0.0.0`, Broadcast: `10.255.255.255`, Hosts: 16 777 214 |
| P2        | `/22`, 10 bits de host |
| P3        | 6 hosts |
| P4        | Red: `192.168.100.32`, Broadcast: `192.168.100.63` |
| P5        | Prefijo /28, bloque 16, 14 hosts/subred |
| P6        | 45 cae en subred .32, 50 cae en subred .48 → **no** están en la misma subred |
| P7        | /25 (126 hosts), sobran 26 |
| P8        | /25 (9 bits prestados = 512 subredes), 126 hosts/subred |
| P9        | Mkt: .0/26, Ventas: .64/27, Cont: .96/28, Ger: .112/29, WAN1: .120/30, WAN2: .124/30 |
| P10       | A: .0/25 (126h), B: .128/26 (62h), C: .192/27 (30h), D: .224/28 (14h). Total usado: 232 de 256. Sí cabe. |
| P11       | Error: .100 no es múltiplo de 64 (bloque de /26). Debería empezar en .128. Además, .100 está dentro de la subred A (.0–.127). Solapamiento. |
| P12       | Producción: 10.50.0.0/23, Oficinas: 10.50.2.0/24, Remota A LAN: 10.50.3.0/25, WiFi A: 10.50.3.128/26, Remota B: 10.50.3.192/27, Servidores: 10.50.3.224/27, Enlaces: 10.50.4.0/30, .4/30, .8/30 |

---

# PARTE X — CIERRE

---

## 23. Reflexión final

El subnetting no es un tema que se aprenda leyendo una vez. Es un tema que se aprende **haciendo ejercicios**, uno tras otro, hasta que el procedimiento se automatiza.

Lo que debes llevar grabado:

1. **La máscara es la clave.** Todo el subnetting gira alrededor de dónde está la frontera entre red y host.

2. **El bloque lo resuelve todo.** Aprende a calcular $256 - m$ y a listar múltiplos. Con eso puedes resolver cualquier ejercicio.

3. **Siempre resta 2.** Dirección de red y broadcast no cuentan como hosts.

4. **En VLSM, ordena primero.** De mayor a menor. Sin excepciones.

5. **Verifica al final.** ¿No hay solapamientos? ¿Cabe en la red original? ¿Las direcciones son correctas?

Si dominas estas cinco ideas y has practicado con los ejercicios de este tema, el subnetting dejará de ser un problema y se convertirá en una herramienta que usarás con seguridad y criterio.

---

*Fin del tema.*
