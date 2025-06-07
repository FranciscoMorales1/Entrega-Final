

## Configuración de red Ad-Hoc en Linux usando Ubuntu desde booteable

### Introducción

En este experimento se utilizó una distribución Ubuntu Linux ejecutada desde un medio **booteable** (memoria USB). Esta metodología garantiza que todos los nodos participantes del experimento inicien en un entorno limpio, estandarizado y libre de configuraciones previas que puedan interferir en la comunicación ad-hoc. La elección de Ubuntu se debe a su compatibilidad con una amplia variedad de hardware y su facilidad para gestionar interfaces inalámbricas, así como su soporte para herramientas necesarias como `iwconfig`, `ip`, y el protocolo BATMAN (`batctl`).

---

### 1. Booteable Ubuntu

Cada nodo fue iniciado desde una memoria USB previamente configurada con la imagen ISO de Ubuntu. Esto permitió disponer de un entorno homogéneo y evitar conflictos relacionados con controladores de red instalados en el sistema base. No fue necesario instalar Ubuntu en el disco duro; el sistema fue completamente funcional en modo "live".

---

### 2. Arreglar la interfaz inalámbrica (dejarla libre)

Antes de configurar la red ad-hoc, es necesario desactivar cualquier proceso o servicio que esté utilizando la interfaz inalámbrica (en este caso `wlp2s0`). Esto se hace con el siguiente comando:

```bash
sudo ip link set wlp2s0 down
```

Una vez desactivada, la interfaz puede ser configurada manualmente para funcionar en modo ad-hoc.

---

### 3. Configuración de red Ad-Hoc (comandos utilizados)

A continuación se muestran los comandos ejecutados secuencialmente para establecer una red ad-hoc con BATMAN (Better Approach To Mobile Adhoc Networking) como protocolo de enrutamiento.

```bash
# Apagar la interfaz
sudo ip link set wlp2s0 down

# Cambiar el modo a ad-hoc
sudo iwconfig wlp2s0 mode ad-hoc

# Asignar un nombre a la red (ESSID)
sudo iwconfig wlp2s0 essid meshnet

# Seleccionar el canal de frecuencia
sudo iwconfig wlp2s0 channel 1

# Ajustar el tamaño de la unidad de transmisión máxima
sudo ip link set wlp2s0 mtu 1428

# Activar nuevamente la interfaz
sudo ip link set wlp2s0 up

# Agregar la interfaz física a batman-adv
sudo batctl if add wlp2s0

# Activar la interfaz virtual de batman
sudo ip link set bat0 up

# Asignar una dirección IP a la interfaz virtual
sudo ip addr add 10.0.0.1/24 dev bat0
```

**Verificación de nodos vecinos**
Se puede comprobar la visibilidad de los vecinos usando:

```bash
sudo batctl n
```

**Ejemplo de salida:**

```
[B.A.T.M.A.N. adv 2024.2, MainIF/MAC: wlp2s0/44:1c:a8:9e:89:bb (bat0/ea:ef:49:7b:44:28 BATMAN_IV)]
IF        Neighbor              last-seen
wlp2s0    60:dd:8e:b6:dc:f8     0.874s
wlp2s0    60:dd:8e:b4:99:57     0.846s
```

---

### 4. Pruebas de conectividad

Se realizaron pruebas de `ping` entre nodos de la red, asignando direcciones IP en la subred `10.0.0.0/24` por interfaz `bat0`.

#### Resultado del `ping` hacia nodo 10.0.0.2 (al inicio sin éxito)

```bash
ping 10.0.0.2
```

```
Destination Host Unreachable
```

#### Posteriormente, conectividad exitosa:

```bash
PING 10.0.0.2 (10.0.0.2) 56(84) bytes of data.
64 bytes from 10.0.0.2: icmp_seq=1 ttl=64 time=8.97 ms
64 bytes from 10.0.0.2: icmp_seq=2 ttl=64 time=20.7 ms
...
rtt min/avg/max/mdev = 0.797/3.41/20.7/5.55 ms
```

#### Ping a nodo 10.0.0.3:

```bash
64 bytes from 10.0.0.3: icmp_seq=1 ttl=64 time=59.2 ms
...
rtt min/avg/max/mdev = 0.989/13.821/59.196/20.091 ms
```

#### Ping a nodo 10.0.0.4:

```bash
64 bytes from 10.0.0.4: icmp_seq=1 ttl=64 time=1.03 ms
...
rtt min/avg/max/mdev = 0.809/3.445/9.952/3.329 ms
```


