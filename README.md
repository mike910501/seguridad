Primer punto - Como estoy ejecutando KliLinux en un contenedor de Docker, me sale que todos los puertos están abiertos, lo cual, es anomalo.

                El escaneo realizado con Nmap sobre la dirección 127.0.0.1 (localhost) mostró que el host está activo, pero no se detectaron 
                puertos abiertos entre los 1000 puertos analizados. Todos los puertos se encuentran cerrados, por lo que no se identificaron 
                servicios ni versiones de software asociadas. Además, el sistema no pudo determinar con precisión el sistema operativo, ya que 
                no había suficiente información disponible debido a la ausencia de puertos abiertos. La distancia de red fue de 0 saltos, 
                lo que indica que el escaneo se realizó sobre la misma máquina.

                El escaneo de vulnerabilidades realizado con Nmap sobre 127.0.0.1 (localhost) mostró que el host está activo, pero no se detectaron 
                puertos abiertos ni servicios en ejecución, por lo que no se encontraron vulnerabilidades. Este tipo de análisis es útil porque permite 
                identificar servicios expuestos y posibles fallas de seguridad en un sistema.

                Escaneo lento (por ejemplo -T0 o -T1):
                Un atacante lo usa para evitar ser detectado por sistemas de seguridad o monitoreo de red. Envía los paquetes muy despacio, 
                lo que hace que el escaneo sea más difícil de identificar, pero tarda mucho más tiempo.

                Escaneo rápido (por ejemplo -T4 o -T5):
                Se utiliza cuando el atacante quiere obtener resultados rápidamente. El escaneo envía muchos paquetes en poco tiempo, 
                lo que permite descubrir puertos y servicios más rápido, pero aumenta la probabilidad de que sea detectado por sistemas 
                de seguridad.
                
                El escaneo realizado con Nmap sobre la dirección 127.0.0.1 (localhost) mostró que el host está activo, pero todos los puertos 
                analizados (22, 25, 80, 110, 143, 443 y 8080) se encuentran en estado closed, lo que significa que el sistema está accesible, 
                pero no hay servicios escuchando en esos puertos. En Nmap, el estado filtered indica que el escáner no puede determinar si el
                puerto está abierto porque algún dispositivo o mecanismo de seguridad está bloqueando o filtrando los paquetes. Esto normalmente 
                ocurre cuando existe un firewall, un router o un sistema de seguridad de red que impide que las solicitudes lleguen al puerto o
                bloquea la respuesta. Este tipo de protección es común en redes para evitar accesos no autorizados y ocultar servicios internos.

                Se ejecutó un escaneo de detección de versiones con Nmap utilizando el parámetro -sV y una intensidad alta (--version-intensity 9) 
                sobre la dirección 127.0.0.1. Sin embargo, el resultado fue similar al escaneo -sV normal, ya que no se detectaron puertos abiertos 
                entre los 1000 analizados. Debido a que no hay servicios activos escuchando en el sistema, Nmap no pudo obtener información adicional 
                como versiones de software o banners de servicios. Por esta razón, no se identificaron datos como versiones de servidores web, 
                configuraciones de SSH u otros servicios.

                Nmap, utilizando scripts NSE sobre el puerto 8080, pudo extraer información del servidor web como el título de la página, la cabecera del 
                servidor y los métodos HTTP permitidos (por ejemplo GET y HEAD). Esta información ayuda a identificar qué tipo de servicio web está corriendo 
                y cómo responde a distintas solicitudes. Es peligroso que un atacante conozca métodos como PUT o DELETE, porque podrían permitir subir, 
                modificar o eliminar archivos en el servidor, lo que facilitaría ataques como la carga de archivos maliciosos o la manipulación del contenido 
                del sitio web.

Segundo Punto

                Durante la auditoría del sistema realizada con Lynis, se identificó 1 warning y varias sugerencias de seguridad. La advertencia 
                principal indica que klogd no está ejecutándose, lo que podría provocar que algunos mensajes del kernel no se registren en los archivos 
                de log. Entre las recomendaciones aparecen instalar herramientas de seguridad adicionales como fail2ban, mejorar la configuración de 
                contraseñas mediante módulos PAM, habilitar sistemas de registro y auditoría como auditd, y utilizar herramientas de verificación de 
                integridad de archivos. La sugerencia más crítica es mejorar el sistema de registro y monitoreo de seguridad, ya que actualmente no hay 
                un sistema de logs completo activo. Finalmente, Lynis asignó un Hardening Index de 56, lo que indica un nivel de seguridad medio; este 
                puntaje podría mejorarse instalando herramientas de monitoreo, configurando políticas de contraseñas más fuertes, habilitando auditorías 
                del sistema y aplicando configuraciones adicionales de hardening.

Tercer Punto
                El análisis del sistema realizado con rkhunter y chkrootkit muestra que no se encontraron rootkits ni archivos maliciosos conocidos en el 
                sistema. En el escaneo de rkhunter se revisaron más de 461 rootkits, y todos aparecen como Not found, aunque se generó una advertencia relacionada 
                con posibles comprobaciones adicionales del Suckit Rootkit y con la ausencia de un servicio de registro del sistema (logging). 
                Por su parte, chkrootkit también indica que la mayoría de los binarios del sistema no están infectados y no se detectaron rootkits 
                conocidos; las advertencias mostradas se deben principalmente al uso de overlayfs en el entorno (común en contenedores o laboratorios), 
                lo que puede provocar falsos positivos. En general, los resultados indican que el sistema parece limpio, aunque se recomienda revisar los 
                logs y mantener herramientas de seguridad y monitoreo activas para mejorar la detección de amenazas.

                El comando netstat con la opción -tupan muestra las conexiones de red activas, los puertos abiertos y los procesos asociados. 
                En este caso, la salida solo muestra el encabezado y no aparecen conexiones ni puertos en estado LISTEN o ESTABLISHED, lo que indica que 
                actualmente no hay servicios de red activos ni conexiones abiertas en el sistema. Esto significa que ningún programa está escuchando en puertos 
                TCP o UDP, lo cual reduce la superficie de ataque del sistema desde la red.

                Se realizó la actualización y prueba del antivirus ClamAV en el sistema Kali Linux utilizando la herramienta freshclam para descargar las 
                últimas bases de datos de firmas de malware. Durante el proceso se actualizaron correctamente las bases daily, main y bytecode, alcanzando 
                más de 3.6 millones de firmas de virus. Posteriormente se ejecutó un escaneo con clamscan, pero no se analizaron archivos porque la ruta 
                especificada (/home/root) no existe en el sistema, por lo que el antivirus no encontró directorios para revisar. Aun así, el resultado muestra 
                que el motor antivirus funciona correctamente y está actualizado, lo cual es importante para detectar y prevenir amenazas de malware en el 
                sistema.