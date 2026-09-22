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

## 📊 Tabla de Resultados

| Puerto | Servicio | Versión | Riesgo inicial | Qué revisaría desde Blue Team |
| :---: | :---: | :---: | :--- | :--- |
| **`21/tcp`** | FTP | vsftpd (Ubuntu) | **Medio-Alto:** Autenticación y transferencia de ficheros en texto plano por defecto. | Comprobar que no admita conexiones anónimas (`anonymous_enable=NO`), revisar eventos en `/var/log/vsftpd.log` y planificar su reemplazo por SFTP/SCP. |
| **`22/tcp`** | SSH | OpenSSH (Ubuntu Linux) | **Medio:** Interfaz de administración remota expuesta a intentos de fuerza bruta o credenciales comprometidas. | Auditar `/var/log/auth.log` en busca de intentos fallidos, asegurar que `PermitRootLogin` esté en `no` y exigir autenticación obligatoria mediante par de claves. |
| **`80/tcp`** | HTTP | Apache httpd 2.4.x | **Bajo-Medio:** Tráfico web no cifrado; potencial exposición de estructura de directorios y versiones de software. | Inspeccionar `/var/log/apache2/access.log` y `error.log`, ocultar cabeceras (`ServerSignature Off` y `ServerTokens Prod`) y forzar redirección a HTTPS (puerto 443). |

---

##  Conclusión de Seguridad Defensiva

* **Qué servicios deberían revisarse:**
  El servicio FTP (`vsftpd`) en el puerto 21 es la prioridad de revisión, ya que no cifra credenciales en tránsito y su funcionalidad puede ser asumida directamente por SSH a través de SFTP.
* **Si están expuestos:**
  Todos los puertos analizados se encuentran accesibles a nivel de red sin filtros locales. Es necesario habilitar el firewall de host (`ufw`) y restringir el acceso al puerto 22 exclusivamente a IPs o subredes de administración.
* **Si la versión está actualizada:**
  Las versiones identificadas mediante las sondas de Nmap deben contrastarse con los repositorios oficiales de la distribución mediante `apt list --upgradable` para aplicar parches de seguridad ante posibles vulnerabilidades conocidas (CVEs).
* **Si generan logs:**
  Los tres servicios generan trazas de auditoría en el sistema operativo:
  * Los intentos de acceso por SSH y sudo quedan registrados en `/var/log/auth.log`.
  * Las peticiones web entrantes se almacenan en `/var/log/apache2/access.log` y `error.log`.
  * La actividad de subida, descarga y autenticación FTP se guarda en `/var/log/vsftpd.log`.
* **Qué miraría como primer análisis:**
  Ante una sospecha de escaneo o intrusión, el primer triaje consistiría en consultar `/var/log/auth.log` para buscar patrones repetitivos de fallos de login desde una misma IP, listar los sockets activos con `ss -tuln` para verificar qué procesos siguen en escucha y aplicar un bloqueo preventivo temporal mediante reglas de `iptables` o `ufw`.
