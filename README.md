# PRIMER-PARCIAL-SEGURIDAD-DE-REDES

[![Video de Demostración en YouTube](https://img.shields.io/badge/YouTube-Demostración%20en%20Video-red?style=for-the-badge&logo=youtube)](https://youtu.be/3wrf4X_yae0)

> **Enlace directo al video:** [https://youtu.be/3wrf4X_yae0](https://youtu.be/3wrf4X_yae0)

---

## 1. Información General y Propósito

* **Estudiante:** Manuel Cruz Messón  
* **Matrícula:** 2025-0689  
* **Propósito:** Diseñar, implementar y auditar una infraestructura de red corporativa segura multi-sitio. Se integra un firewall perimetral **FortiGate (FortiOS 7.0.x)** administrado al 100% por interfaz gráfica (GUI) como núcleo central y gateway inter-VLAN, conmutadores **Cisco IOSvL2** para segmentación de Capa 2 y hardening de puertos, servidores Linux, y un router **Cisco IOS** en sucursal remota enlazado mediante un túnel **VPN IPsec Site-to-Site** con políticas de acceso y registro de auditoría.

---

## 2. Diagrama de la Topología

![Topología de Red](https://raw.githubusercontent.com/labcruzmesson-cyber/PRIMER-PARCIAL-SEGURIDAD-DE-REDES/refs/heads/main/IMAGENES/topologia.png)

### Plan de Direccionamiento (Basado en Matrícula 2025-0689)

| Dispositivo / Zona | Interfaz | Dirección IP / Máscara | Gateway | Descripción |
| :--- | :--- | :--- | :--- | :--- |
| **FortiGate (HQ)** | `port1` | `192.168.145.130/24` | `192.168.145.2` | WAN / Acceso GUI / Endpoint VPN |
| **FortiGate (HQ)** | `port2.10` | `172.25.10.1/24` | N/A | Gateway VLAN 10 (DHCP .10 a .100) |
| **FortiGate (HQ)** | `port2.20` | `172.25.20.1/24` | N/A | Gateway VLAN 20 (Administración) |
| **FortiGate (HQ)** | `port3` | `172.25.68.1/28` | N/A | Gateway DMZ Servidores |
| **Switch SW-1** | `Gi0/0` | Trunk 802.1Q | N/A | Enlace troncal hacia FortiGate (`port2`) |
| **Switch SW-2** | `Gi0/0` - `Gi0/2` | Acceso (VLAN 30) | N/A | Conmutador de Capa 2 para DMZ Servidores |
| **Router Sucursal** | `Gi0/0` | `192.168.145.30/24` | `192.168.145.2` | WAN Sucursal / Endpoint VPN |
| **Router Sucursal** | `Gi0/1` | `10.25.89.1/24` | N/A | Gateway LAN Sucursal |
| **PC10 (Usuarios)** | `eth0` | `172.25.10.10/24` (DHCP)| `172.25.10.1` | Cliente VLAN 10 con salida a Internet |
| **PC20 (Admin)** | `eth0` | `172.25.20.10/24` (Estática)| `172.25.20.1`| Acceso de gestión administrativa (SSH) |
| **Web Server** | `eth0` | `172.25.68.2/28` | `172.25.68.1` | Apache2 HTTP (Puerto 80) |
| **DB Server** | `eth0` | `172.25.68.3/28` | `172.25.68.1` | MariaDB (Puerto 3306) |
| **PC-SUC (Sucursal)**| `eth0` | `10.25.89.10/24` | `10.25.89.1` | Cliente remoto que accede a Web por VPN |

---

## 3. Evidencias de Cumplimiento por Requisito

### Requisito 1: VLAN por Defecto e Interfaces sin Uso (1 pt)
Se modificó la VLAN nativa por defecto a la VLAN 999 en el enlace troncal 802.1Q y se apagaron administrativamente todos los puertos sin utilizar en los conmutadores.

* **Demostración de Trunk y VLAN nativa 999:**
  ![VLAN Nativa en Trunk](https://raw.githubusercontent.com/labcruzmesson-cyber/PRIMER-PARCIAL-SEGURIDAD-DE-REDES/refs/heads/main/IMAGENES/req1_trunk_nativa.png)

* **Demostración de interfaces apagadas (Administratively Down):**
  ![Puertos sin uso apagados](https://raw.githubusercontent.com/labcruzmesson-cyber/PRIMER-PARCIAL-SEGURIDAD-DE-REDES/refs/heads/main/IMAGENES/req1_puertos_down.png)

---

### Requisito 2: Salida a Internet y NAT (1 pt)
La red de usuarios (VLAN 10) obtiene direccionamiento mediante el servidor DHCP configurado en la GUI del FortiGate y sale hacia Internet a través de la política con NAT hacia la interfaz WAN.

* **Ping exitoso a 8.8.8.8 desde PC10:**
* 
  ![Ping a Internet](https://raw.githubusercontent.com/labcruzmesson-cyber/PRIMER-PARCIAL-SEGURIDAD-DE-REDES/refs/heads/main/IMAGENES/req2_ping_internet..png)

* **Traceroute funcional hacia Internet evidenciando el salto en FortiGate (172.25.10.1):**
  ![Traceroute a Internet](https://raw.githubusercontent.com/labcruzmesson-cyber/PRIMER-PARCIAL-SEGURIDAD-DE-REDES/refs/heads/main/IMAGENES/req2_traceroute_internet.png)

---

### Requisito 3: SSH Restringido a la VLAN 20 con Registro (1 pt)
Se implementaron políticas en el FortiGate donde únicamente el segmento administrativo (VLAN 20) tiene permitido el tráfico SSH hacia los servidores. Cualquier intento desde la VLAN 10 u otros orígenes es denegado y registrado.

* **Acceso SSH exitoso desde PC20:**
  ![SSH Exitoso desde VLAN 20](https://raw.githubusercontent.com/labcruzmesson-cyber/PRIMER-PARCIAL-SEGURIDAD-DE-REDES/refs/heads/main/IMAGENES/req3_ssh_permitido_vlan20.png)

* **Acceso SSH rechazado desde PC10:**
* 
  ![SSH Denegado desde VLAN 10](https://raw.githubusercontent.com/labcruzmesson-cyber/PRIMER-PARCIAL-SEGURIDAD-DE-REDES/refs/heads/main/IMAGENES/req3_ssh_denegado_vlan10.png)

* **Registro en el FortiGate (Forward Traffic Log) de la denegación:**
  ![Log SSH Denegado FortiGate](https://raw.githubusercontent.com/labcruzmesson-cyber/PRIMER-PARCIAL-SEGURIDAD-DE-REDES/refs/heads/main/IMAGENES/req3_log_ssh_denegado.png)

---

### Requisito 4: Bloqueo Explícito Usuarios → DB (3306) con Registro (1 pt)
Política explícita en el FortiGate que bloquea el tráfico originado en la VLAN 10 con destino a la IP del Servidor de Base de Datos (`172.25.68.3`) en el puerto 3306 (MySQL/MariaDB), generando su respectivo log de violación.

* **Conexión rechazada con netcat desde PC10 al puerto 3306:**
  ![Bloqueo MySQL](https://raw.githubusercontent.com/labcruzmesson-cyber/PRIMER-PARCIAL-SEGURIDAD-DE-REDES/refs/heads/main/IMAGENES/req4_bloqueo_mysql_pc10.png)

* **Log del FortiGate evidenciando el bloqueo explícito:**
  ![Log Bloqueo MySQL](https://raw.githubusercontent.com/labcruzmesson-cyber/PRIMER-PARCIAL-SEGURIDAD-DE-REDES/refs/heads/main/IMAGENES/req4_log_mysql_fortigate.png)

---

### Requisito 5: Servidores Operativos (1 pt)
Validación de los servicios base en los servidores de la DMZ:

* **Servidor Web (`172.25.68.2`) respondiendo solicitudes HTTP por el puerto 80:**
  ![Servicio Apache Web](https://raw.githubusercontent.com/labcruzmesson-cyber/PRIMER-PARCIAL-SEGURIDAD-DE-REDES/refs/heads/main/IMAGENES/req5_web_apache.png)

* **Servidor de Base de Datos (`172.25.68.3`) con MariaDB escuchando en `0.0.0.0:3306`:**
  ![Servicio MariaDB](https://raw.githubusercontent.com/labcruzmesson-cyber/PRIMER-PARCIAL-SEGURIDAD-DE-REDES/refs/heads/main/IMAGENES/req5_db_mariadb.png)

---

### Requisito 6: VPN IPsec FortiGate ↔ Cisco (3 pts)
Túnel Site-to-Site negociado en IKEv1 / IPsec (DES-SHA1) entre el FortiGate y el router Cisco. El tráfico de la PC de sucursal fluye de forma segura hacia el Web Server a través de la VPN y se comprueba el corte total del servicio al deshabilitar el túnel.

* **Túnel activo en el Router Cisco (`UP-ACTIVE`):**
  ![Sesión IPsec en Cisco](https://raw.githubusercontent.com/labcruzmesson-cyber/PRIMER-PARCIAL-SEGURIDAD-DE-REDES/refs/heads/main/IMAGENES/req6_vpn_session_cisco.png)

* **Túnel activo en la interfaz gráfica del FortiGate:**
  ![Túnel en GUI FortiGate](https://raw.githubusercontent.com/labcruzmesson-cyber/PRIMER-PARCIAL-SEGURIDAD-DE-REDES/refs/heads/main/IMAGENES/req6_vpn_gui_fortigate.png)

* **Petición HTTP funcional vía VPN desde PC-SUC:**
* 
  ![Petición Web vía VPN](https://raw.githubusercontent.com/labcruzmesson-cyber/PRIMER-PARCIAL-SEGURIDAD-DE-REDES/refs/heads/main/IMAGENES/req6_vpn_curl_exitoso.png)

* **Pérdida de conectividad (Timeout) al bajar el Crypto Map en Cisco:**
  ![Pérdida de servicio VPN Down](https://raw.githubusercontent.com/labcruzmesson-cyber/PRIMER-PARCIAL-SEGURIDAD-DE-REDES/refs/heads/main/IMAGENES/req6_vpn_corte_timeout.png)

---
### Requisito 7: Direccionamiento, VLANs y Hostnames (1 pt)
* Esquema de subredes derivado de la matrícula `2025-0689`.
* Segmentación con trunk 802.1Q en el switch hacia las subinterfaces del FortiGate.
* Red de servidores contenida estrictamente en un bloque `/28` (`172.25.68.0/28`).
* Hostnames homologados y visibles en los prompts de cada equipo (`FGT-HQ`, `SW-1`, `SW-2`, `R-SUCURSAL`, `WEB`, `DB`).

## ⚠️ Declaración de Uso de Inteligencia Artificial

En cumplimiento con las buenas prácticas de integridad académica y transparencia profesional:

* **Herramientas utilizadas:** Modelos de lenguaje e inteligencia artificial generativa asistieron en la formulación de consultas técnicas, estructuración de la documentación y depuración de sintaxis.
* **Alcance del uso:** La IA se empleó exclusivamente como herramienta de apoyo, consulta y optimización de formato.
* **Autoría y validación:** El diseño topológico, la implementación en entorno virtualizado, la configuración de los equipos (Cisco IOS y FortiOS), la resolución de problemas de enrutamiento/VPN y la verificación funcional de todos los requerimientos fueron realizados, auditados y demostrados íntegramente por el autor del proyecto.
