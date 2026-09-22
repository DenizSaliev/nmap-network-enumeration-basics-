#  Fundamentos de Enumeración de Redes y Escaneo con Nmap

##  Objetivo del Laboratorio
El propósito de este laboratorio es practicar técnicas esenciales de reconocimiento y enumeración de red mediante Nmap, analizando el descubrimiento de hosts, puertos abiertos y servicios en ejecución. La práctica está orientada desde una perspectiva de ciberseguridad defensiva (Blue Team) para comprender la visibilidad de los activos en red y evaluar la superficie de exposición.

##  Entorno y Máquinas Implicadas
* **Máquina Auditora (Escáner):** Ubuntu Linux (192.168.52.10).
* **Máquina Objetivo (Target):** Clon de Ubuntu Linux (192.168.52.138) con servicios estándar de red en ejecución (Apache, OpenSSH, vsftpd).
* **Segmento de Red:** Red privada local aislada mediante adaptador virtual (Host-Only).
* **Entorno Controlado:** Todas las pruebas y escaneos documentados en este proyecto han sido realizados exclusivamente dentro de un entorno virtualizado local, controlado y debidamente autorizado con fines estrictamente académicos.

---
##  Explicación de Resultados

1. **Descubrimiento de Hosts Activos (`-sn`):**
   * El barrido ICMP/ARP sobre el segmento local confirmó la dirección IP asignada a la máquina objetivo dentro de la red privada, permitiendo descartar las direcciones no asignadas o inactivas sin saturar la red ni generar alertas en servicios de host.

2. **Comparativa entre Escaneo Rápido (`-F`) y Completo (`-p-`):**
   * **Escaneo rápido (`-sS -F`):** Analizó únicamente los 100 puertos más habituales definidos por Nmap. La ejecución tomó menos de 2 segundos, localizando de inmediato los servicios estándar habituales (SSH, HTTP y FTP).
   * **Escaneo exhaustivo (`-p-`):** Requirió un tiempo significativamente mayor al enviar paquetes a los 65.535 puertos TCP posibles. Esta prueba demuestra la diferencia operativa entre un triaje inicial ágil y una auditoría completa orientada a descubrir servicios ocultos o configurados en puertos no estándar (por ejemplo, puertos de administración por encima del 1024).

3. **Detección de Servicios y Versiones (`-sV`):**
   * La consulta específica sobre los puertos abiertos permitió interactuar con las cabeceras de aplicación (*banners*), extrayendo la compilación exacta del software en ejecución en lugar de asumir el servicio por su número de puerto por defecto. Esto resulta indispensable para consultar bases de datos de vulnerabilidades (CVEs) asociadas a versiones específicas.

---

##  Tabla de Puertos Detectados

| Puerto / Protocolo | Estado | Servicio Identificado | Versión Detectada |
| :---: | :---: | :---: | :---: |
| **`21/tcp`** | Open | FTP | vsftpd (versión en Ubuntu) |
| **`22/tcp`** | Open | SSH | OpenSSH (Ubuntu Linux) |
| **`80/tcp`** | Open | HTTP | Apache httpd 2.4.x |


---

##  Conclusión de Seguridad

1. **Reducción de la Superficie de Ataque (*Attack Surface*):**
   * Cada puerto en estado `Open` implica un servicio cargado en la memoria del sistema operativo y expuesto a peticiones externas. Desde una perspectiva de seguridad defensiva (Blue Team), es imperativo aplicar el principio de mínimo privilegio desinstalando o deteniendo servicios prescindibles para la operativa del host.

2. **Importancia del Inventariado Preciso de Versiones:**
   * La enumeración con Nmap no es únicamente una fase previa a una intrusión ofensiva; constituye la herramienta primaria para auditorías internas de cumplimiento y gestión de vulnerabilidades. Conocer la versión exacta del servicio permite a un administrador parchear de forma proactiva software vulnerable antes de que pueda ser explotado.

3. **Visibilidad Interna y Configuración de Firewalls:**
   * El hecho de que todos los puertos respondieran sin restricciones resalta la ausencia o permisividad de un cortafuegos local de host (`ufw` / `iptables`). En un entorno corporativo endurecido (*hardened*), los puertos de administración como SSH deben estar filtrados mediante reglas restrictivas que solo permitan accesos desde direcciones IP o subredes de gestión específicas.
