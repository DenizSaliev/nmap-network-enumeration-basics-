---

##  Tabla de Puertos Detectados

| Puerto / Protocolo | Estado | Servicio Identificado | Versión Detectada | Observaciones Técnicas / Riesgo Defensivo |
| :---: | :---: | :---: | :---: | :--- |
| **`21/tcp`** | Open | FTP | vsftpd (versión en Ubuntu) | Transmisión de credenciales y datos en texto plano. Se debe evaluar su sustitución por SFTP/SCP. |
| **`22/tcp`** | Open | SSH | OpenSSH (Ubuntu Linux) | Acceso a consola remota cifrada. Requiere auditar que no permita login directo de `root` ni contraseñas débiles. |
| **`80/tcp`** | Open | HTTP | Apache httpd 2.4.x | Servidor web sin capa de cifrado TLS. Superficie de exposición habitual que requiere control de permisos en directorios web. |

*(Nota: Si en tu escaneo completo `-p-` detectaste algún puerto adicional como `53`, `443` o `3306`, añádelo a esta tabla con su respectiva versión y estado).*

---

##  Conclusión de Seguridad

1. **Reducción de la Superficie de Ataque (*Attack Surface*):**
   * Cada puerto en estado `Open` implica un servicio cargado en la memoria del sistema operativo y expuesto a peticiones externas. Desde una perspectiva de seguridad defensiva (Blue Team), es imperativo aplicar el principio de mínimo privilegio desinstalando o deteniendo servicios prescindibles para la operativa del host.

2. **Importancia del Inventariado Preciso de Versiones:**
   * La enumeración con Nmap no es únicamente una fase previa a una intrusión ofensiva; constituye la herramienta primaria para auditorías internas de cumplimiento y gestión de vulnerabilidades. Conocer la versión exacta del servicio permite a un administrador parchear de forma proactiva software vulnerable antes de que pueda ser explotado.

3. **Visibilidad Interna y Configuración de Firewalls:**
   * El hecho de que todos los puertos respondieran sin restricciones resalta la ausencia o permisividad de un cortafuegos local de host (`ufw` / `iptables`). En un entorno corporativo endurecido (*hardened*), los puertos de administración como SSH deben estar filtrados mediante reglas restrictivas que solo permitan accesos desde direcciones IP o subredes de gestión específicas.
