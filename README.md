# 🦈 Guía Esencial de Wireshark para Blue Team 🛡️ (Principiantes)

¡Hola! 👋 Bienvenido/a a esta guía de referencia rápida ("cheat sheet") de **Wireshark**, la herramienta indispensable para cualquier profesional del análisis de redes. Este documento está optimizado para **principiantes** que dan sus primeros pasos en el **Blue Team** (ciberseguridad defensiva).

> **¿Por qué Wireshark es Clave en Blue Team?**
> Wireshark te da **visibilidad** sobre el tráfico de tu red. Es fundamental para:
> * Investigar alertas de seguridad (IDS/IPS, SIEM).
> * Diagnosticar problemas de conectividad y rendimiento.
> * Comprender las fases de un ataque (reconocimiento, C2, exfiltración).
> * Verificar políticas de firewall.
> * En resumen: **defender tus activos digitales con conocimiento.**

---

## 🔍 Anatomía de la Interfaz de Wireshark

Al iniciar Wireshark y cargar una captura, estas son las áreas principales que encontrarás:
*(Una captura de pantalla aquí es altamente recomendada 👇)*

1.  **Barra de Menús y Herramientas:** Acceso a todas las funciones y opciones.
2.  **Barra de Filtros de Visualización:** ¡Tu centro de comandos! Aquí aplicas filtros para encontrar lo relevante.
3.  **Lista de Paquetes:** El corazón de Wireshark. Cada fila representa un paquete capturado.
4.  **Detalles del Paquete:** Desglose jerárquico (por capas OSI/TCP) del paquete seleccionado.
5.  **Bytes del Paquete:** Contenido crudo del paquete en hexadecimal y ASCII.

**Columnas Típicas en la Lista de Paquetes:**

| Columna    | Descripción                                                      |
| :--------- | :--------------------------------------------------------------- |
| **No.** | #️⃣ Identificador secuencial único del paquete en esta captura. |
| **Time** | 🕒 Timestamp relativo al inicio de la captura (o absoluto).   |
| **Source** | ➡️ Dirección Origen (IP, MAC, etc.).                            |
| **Dest.** | ⬅️ Dirección Destino (IP, MAC, etc.).                           |
| **Protocol**| 📜 El protocolo de más alto nivel identificado (HTTP, DNS, TCP...). |
| **Length** | 📏 Longitud total del paquete en bytes.                          |
| **Info** | ℹ️ Resumen contextual útil (Flags TCP, URL, consulta DNS...).    |

---

## 🔬 Filtrado: El Arte de Encontrar la Aguja en el Pajar

Wireshark puede capturar gigabytes de datos rápidamente. Sin filtros, analizar es casi imposible.

> **Filtros de Captura vs. Filtros de Visualización: La Diferencia Clave**
>
> * **Filtros de Captura:** Se configuran *antes* de iniciar la captura. Definen qué paquetes se guardarán en el archivo `.pcapng`. Son **menos flexibles** una vez iniciada la captura y usan una **sintaxis diferente** (basada en BPF, ej: `host 1.2.3.4`, `port 80`, `tcp and not port 22`). Útiles para descartar ruido masivo en capturas largas.
> * **Filtros de Visualización:** Se aplican *sobre la marcha* a los paquetes ya capturados (o al abrir un archivo). Permiten **mostrar/ocultar paquetes dinámicamente** sin modificar la captura original. Son **extremadamente potentes y flexibles**, y son los que **usarás el 95% del tiempo para analizar**. **Esta guía se centra en ellos.**
> *(Una captura mostrando la barra de filtros en acción es ideal aquí 👇)*

### Operadores para Filtros de Visualización

Construye tus filtros combinando **campos** (ej: `ip.addr`, `tcp.port`, `dns.qry.name`) con **operadores**:

**Operadores de Comparación:**

| Operador   | Ejemplo                   | Descripción                                    |
| :--------- | :------------------------ | :--------------------------------------------- |
| `==`       | `ip.addr == 1.1.1.1`      | Igual a (valor exacto)                         |
| `!=`       | `tcp.port != 80`          | Diferente de                                   |
| `>`        | `frame.len > 1000`        | Mayor que                                      |
| `<`        | `udp.port < 1024`         | Menor que                                      |
| `>=`       | `ip.ttl >= 64`            | Mayor o igual que                              |
| `<=`       | `tcp.window_size <= 14k`  | Menor o igual que                              |
| `contains` | `http contains "admin"`   | El campo (o paquete) contiene la subcadena (case-insensitive por defecto) |
| `matches`  | `dns.qry.name matches "\\d{3}\\.com"` | El campo coincide con una Expresión Regular (PCRE) |

**Operadores Lógicos:**

| Operador     | Ejemplo                                       | Descripción                                       |
| :----------- | :-------------------------------------------- | :------------------------------------------------ |
| `&&` o `and` | `ip.src == 10.1.1.5 and tcp.dstport == 443`   | **Y Lógico:** Ambas condiciones deben ser ciertas. |
| `||` o `or`  | `udp.port == 53 or tcp.port == 53`            | **O Lógico:** Al menos una condición debe ser cierta. |
| `!` o `not`  | `not arp`                                     | **NO Lógico:** Invierte la condición siguiente.    |
| `^^` o `xor` | `tcp.flags.syn == 1 ^^ tcp.flags.ack == 1`    | **O Exclusivo:** Una condición sí, la otra no.      |
| `(...)`      | `(ip.addr == 1.1.1.1 and tcp) or dns`         | **Paréntesis:** Para agrupar y controlar precedencia. |

---

## 📡 Modos de Captura

* **Modo Promiscuo:** (Activado por defecto en Ethernet cableada) La interfaz de red captura *todo* el tráfico que físicamente llega a ella, incluso si no está destinado a su dirección MAC. Esencial para monitorizar un segmento de red (su efectividad depende de la topología de red, especialmente de los switches).
* **Modo Monitor:** (Específico para interfaces Wi-Fi, soportado nativamente en Linux/macOS) Permite capturar *todo* el tráfico 802.11 en un canal específico (incluyendo tramas de gestión y control) sin necesidad de estar asociado a una red (AP). Clave para auditorías y análisis de seguridad Wi-Fi.

---

## 🔥 ¡Filtros Esenciales para Blue Team! 🔥

Este es tu kit de inicio de filtros de visualización para análisis defensivo:

| Propósito                               | Filtro de Ejemplo                               | Notas Defensivas 🛡️                                                            |
| :-------------------------------------- | :---------------------------------------------- | :------------------------------------------------------------------------------ |
| **IP Específica (Origen o Destino)** | `ip.addr == 1.2.3.4`                            | **Fundamental.** Aislar actividad de IP sospechosa, servidor crítico, IOC.      |
| **IP Origen** | `ip.src == 10.0.0.50`                           | Analizar todo el tráfico *originado* por un host específico.                  |
| **IP Destino** | `ip.dst == 8.8.8.8`                             | Analizar todo el tráfico *destinado* a un host específico (ej. servidor externo). |
| **Subred Completa** | `ip.addr == 192.168.1.0/24`                     | Enfocarse en el tráfico dentro (o hacia/desde) un segmento de red local.      |
| **Excluir IP (Eliminar Ruido)** | `not ip.addr == 192.168.1.1`                    | Ignorar tráfico conocido y benigno (tu máquina, scanners autorizados...).      |
| **Puerto TCP Específico** | `tcp.port == 443`                             | Aislar tráfico de un servicio (HTTPS, RDP/3389, SSH/22, etc.).                 |
| **Puerto UDP Específico** | `udp.port == 53`                              | Aislar tráfico DNS (53), NTP (123), SNMP (161/162), etc.                    |
| **Puerto Destino Específico** | `tcp.dstport == 3389`                           | Buscar intentos de conexión *entrantes* a un servicio (RDP en este caso).       |
| **Combinar IP y Puerto** | `ip.addr == 1.2.3.4 and tcp.port == 22`         | **Muy potente.** Investigar conexiones SSH hacia/desde una IP en particular. |
| **Protocolos Clave (Filtrar por Nombre)**| `http` / `dns` / `icmp` / `arp` / `smb` / `dhcp`| Centrarse en protocolos específicos para análisis (C2, exfiltración, escaneos...). |
| **Buscar Texto en Payload (General)** | `frame contains "password"`                     | ⚠️ **Precaución:** Puede ser muy lento en capturas grandes. Útil para encontrar datos sensibles no cifrados o indicadores específicos. |
| **TCP SYN (Inicio de Conexión)** | `tcp.flags.syn == 1 and tcp.flags.ack == 0`     | **Clave para detectar escaneos.** Identifica paquetes de inicio de conexión.     |
| **TCP RST (Reset/Rechazo)** | `tcp.flags.reset == 1`                          | Indica conexiones rechazadas/abortadas. Puede señalar escaneos, firewalls.      |
| **DNS Queries (Consultas de Nombres)** | `dns.flags.response == 0`                       | Ver qué nombres de dominio están intentando resolver los hosts.                 |
| **DNS Queries por Nombre Específico** | `dns.qry.name contains "malware-domain"`        | Buscar consultas a dominios maliciosos conocidos (IOC).                         |
| **HTTP Requests (GET/POST...)** | `http.request`                                  | Ver todas las solicitudes HTTP (URLs, métodos...). Útil para análisis de C2 web. |
| **Tráfico Broadcast (Red Local)** | `eth.dst == ff:ff:ff:ff:ff:ff`                  | Analizar tráfico enviado a todos en el segmento (ARP, DHCP Discover...).      |

> **💡 Consejo Profesional:** La verdadera potencia reside en **combinar** estos filtros usando `and`, `or`, `not` y paréntesis `()` para aislar exactamente el tráfico que necesitas investigar.
> Ejemplo: `(ip.src == 10.1.2.3 and tcp.dstport == 443) and not ip.dst == 1.1.1.1`

---

## ⌨️ Atajos de Teclado Imprescindibles

* `Ctrl + E`: Iniciar / Detener la captura activa.
* `Ctrl + R`: Reiniciar la captura activa.
* `Ctrl + F`: Buscar texto dentro de los datos de los paquetes.
* `Ctrl + O`: Abrir un archivo de captura (`.pcapng`, `.pcap`...).
* `Ctrl + S`: Guardar la captura actual (o los paquetes seleccionados).
* `Ctrl + Q`: Salir de Wireshark.
* `Ctrl + Flecha Abajo` (o `F8`): Ir al siguiente paquete en la lista.
* `Ctrl + Flecha Arriba` (o `F7`): Ir al paquete anterior en la lista.
* `Ctrl + G`: Ir a un número de paquete específico.
* `Ctrl + M`: Marcar/Desmarcar el paquete seleccionado.

---

## ⚙️ Barra de Herramientas: Iconos Clave

* **🦈 Aleta Azul:** Iniciar una nueva captura (usando la última configuración).
* **🟥 Cuadrado Rojo:** Detener la captura activa.
* **🦈 Aleta Gris:** Reiniciar la captura activa (Detener y volver a Iniciar).
* **📂 Carpeta Abierta:** Abrir un archivo de captura existente.
* **💾 Disquete:** Guardar la captura actual.
* **🔍 Lupa:** Encontrar paquetes basados en criterios (filtros, texto...).
* **⬅️ / ➡️ Flechas:** Navegar por el historial de selección de paquetes.
* **🎨 Paleta Colores:** Activar/Desactivar las reglas de coloreado de paquetes (ayuda visual crucial).

---

## ⬇️ Instalación y Disponibilidad

* **Descarga Oficial (Todas las plataformas):** [https://www.wireshark.org/download.html](https://www.wireshark.org/download.html)
* > **💡 Nota para Pentesters/Analistas:** Wireshark viene **preinstalado y listo para usar** en la mayoría de distribuciones Linux enfocadas en ciberseguridad, como **Kali Linux** y **Parrot Security OS**.

---

## ❗ Consideraciones Éticas Fundamentales

> **⚠️ ¡ADVERTENCIA SERIA!**
> La captura y análisis de tráfico de red puede interceptar comunicaciones privadas y datos sensibles. Realizar estas acciones sin la **autorización explícita y por escrito** del propietario de la red y/o sistemas implicados es **ilegal** en la mayoría de jurisdicciones y constituye una **grave violación ética y de privacidad.**
> **Utiliza Wireshark únicamente de forma legal, ética y responsable dentro de los límites de tu autorización.**

---

## ❓ Contribuciones y Mejoras

¿Has encontrado un error, una errata o tienes alguna sugerencia para mejorar esta guía? ¡Excelente! Por favor, abre un **Issue** detallado en este repositorio. El objetivo es que este recurso sea lo más útil posible para la comunidad.
