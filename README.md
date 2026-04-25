# Resumen Módulos 3, 4, 5 y 6 – Curso de Ciberseguridad Cisco

---

## Módulo 3: Reconocimiento y análisis de vulnerabilidades

### Reconocimiento pasivo

El reconocimiento pasivo consiste en recopilar información sobre una organización sin tocar directamente sus sistemas. Básicamente usas lo que ya está público en internet: registros DNS, redes sociales, certificados SSL, metadatos de archivos, etc. Todo esto cae dentro de lo que se conoce como OSINT (Open Source Intelligence). La idea es que el objetivo nunca se entere de que lo estás investigando.

### Reconocimiento activo

Aquí ya interactúas directamente con los sistemas. Se escanean puertos, se identifican servicios corriendo en servidores, versiones de software, sistemas operativos, etc.

La herramienta principal es **Nmap**. En las prácticas de laboratorio lo usamos bastante y hay varias cosas que aprendí:

- Al escanear **localhost (127.0.0.1)** en un contenedor Docker, todos los puertos aparecen cerrados porque no hay servicios levantados. Esto es normal en ese entorno, no es que el sistema esté súper protegido, simplemente no hay nada corriendo.
- Los parámetros de velocidad importan mucho: un escaneo lento (**-T0 o -T1**) lo usa un atacante para pasar desapercibido, mientras que uno rápido (**-T4 o -T5**) da resultados rápido pero es más fácil de detectar.
- Con **-sV** y **--version-intensity 9** se intenta sacar las versiones exactas de los servicios, pero si no hay puertos abiertos no sirve de nada.
- Los scripts **NSE** de Nmap pueden extraer info útil como títulos de páginas web, cabeceras del servidor y métodos HTTP permitidos. Es peligroso cuando un servidor tiene habilitados métodos como PUT o DELETE porque un atacante podría subir archivos maliciosos o borrar contenido.
- El estado **filtered** en Nmap significa que algo (un firewall, un router) está bloqueando las solicitudes y Nmap no puede saber si el puerto está abierto o cerrado.

También se pueden analizar paquetes con **Wireshark** o crear paquetes personalizados con **Scapy** para hacer pruebas más específicas.

### Análisis de vulnerabilidades

El análisis de vulnerabilidades se hace con herramientas automáticas que revisan configuraciones y versiones de software buscando fallas conocidas. El problema es que pueden dar **falsos positivos** (marcar algo como vulnerable cuando no lo es) o no detectar algunas cosas.

En el laboratorio usamos **Lynis** para hacer una auditoría del sistema. Los resultados fueron:
- 1 advertencia principal: klogd no estaba corriendo, lo que significa que algunos mensajes del kernel no se registran
- Varias sugerencias como instalar fail2ban, mejorar las políticas de contraseñas con PAM, habilitar auditd y usar herramientas de verificación de integridad
- El **Hardening Index** fue de 56 sobre 100, o sea nivel medio. Para mejorarlo habría que instalar herramientas de monitoreo, configurar políticas de contraseñas más fuertes y habilitar auditorías

También usamos **rkhunter** y **chkrootkit** para buscar rootkits. Revisaron más de 461 rootkits conocidos y no encontraron nada. Las únicas advertencias eran por el uso de overlayfs (normal en contenedores Docker) que genera falsos positivos.

Con **netstat -tupan** verificamos las conexiones de red activas y no había ninguna, lo que tiene sentido en un contenedor limpio — no hay servicios escuchando y eso reduce la superficie de ataque.

También probamos **ClamAV** como antivirus. Se actualizó bien con freshclam (más de 3.6 millones de firmas), pero el escaneo falló porque la ruta /home/root no existe en el sistema. El motor funciona correctamente, solo fue un error de ruta.

---

## Módulo 4: Ingeniería social

### Qué es la ingeniería social

La ingeniería social es cuando el atacante no ataca el sistema directamente, sino que engaña a las personas para que le den acceso o información. Aprovecha la confianza, el miedo, la urgencia o simplemente el descuido humano. Muchas veces es más fácil engañar a alguien que hackear un servidor.

### Tipos de ataques

**Phishing** — correos falsos que parecen venir de bancos o empresas para robar contraseñas o datos personales. Es el ataque más común de ingeniería social.

**Vishing** — lo mismo que phishing pero por llamada telefónica. El atacante se hace pasar por soporte técnico, el banco, etc.

**Smishing** — igual pero usando mensajes de texto (SMS). Te llega un mensaje con un link malicioso.

### Ataques físicos

No todo es digital. También existen ataques presenciales:

- **Tailgating**: entrar a un área restringida pegándote a alguien que sí tiene acceso, como colarte detrás de alguien que abre una puerta con tarjeta
- **Dumpster diving**: buscar información útil en la basura — documentos, post-its con contraseñas, discos duros sin borrar
- **Shoulder surfing**: mirar por encima del hombro de alguien para ver su contraseña o información en pantalla

### Herramientas

Para pruebas de seguridad se usan herramientas que simulan estos ataques:
- **Social-Engineer Toolkit (SET)**: permite crear campañas de phishing, clonar sitios web y otras pruebas de ingeniería social
- **BeEF**: se usa para explotar vulnerabilidades en navegadores web

---

## Módulo 5: Explotación de redes cableadas e inalámbricas

### Ataques en redes cableadas

En este módulo vimos cómo se pueden explotar los protocolos que usamos todos los días en redes locales. Lo primero que se estudió fue la **resolución de nombres en Windows** — básicamente cuando el DNS falla, Windows usa otros protocolos como LLMNR y NetBIOS para buscar nombres en la red, y ahí es donde un atacante puede meterse con herramientas como Responder para capturar credenciales.

También vimos los ataques a **SMB**, que es el protocolo que usa Windows para compartir archivos e impresoras. Con enum4linux se puede sacar bastante info de un sistema: usuarios, carpetas compartidas, políticas de contraseñas, etc.

El **envenenamiento de DNS** es otro ataque importante. La idea es meter registros falsos en la caché del servidor DNS para que cuando alguien busque un sitio legítimo, lo mande a uno falso controlado por el atacante.

Otros protocolos vulnerables que vimos:
- **SNMP**: muchas veces viene con las community strings por defecto (public/private) y eso permite sacar información de la configuración de red
- **SMTP**: con comandos como VRFY se pueden enumerar usuarios válidos en un servidor de correo
- **FTP**: login anónimo, todo va en texto plano, fácil de hacer fuerza bruta

Algo que me pareció muy interesante fue el ataque **Pass-the-Hash**. No necesitas saber la contraseña, solo el hash NTLM, y con eso ya puedes autenticarte en otros servicios. Mimikatz es la herramienta principal para esto.

Sobre **Kerberos**, vimos el Kerberoasting que consiste en pedir tickets de servicio y luego crackearlos offline. También los ataques a LDAP y cómo se pueden combinar con otras técnicas.

Los **ataques Man-in-the-Middle** (o "en ruta" como les dicen ahora) se hacen con ARP spoofing usando Ettercap. Te pones en medio de la comunicación y puedes ver todo el tráfico.

Otros temas que cubrimos:
- **DoS/DDoS**: saturar recursos para tumbar un servicio
- **Bypass de NAC**: saltarse controles de acceso a la red con MAC spoofing
- **VLAN hopping**: acceder a VLANs que no te corresponden usando switch spoofing o double tagging
- **DHCP starvation**: agotar todas las IPs del servidor DHCP y luego montar uno falso para convertirte en gateway

### Ataques inalámbricos

En la parte inalámbrica vimos varios ataques. Los más relevantes para mí fueron:

El **Evil Twin** es básicamente clonar el nombre de una red WiFi legítima para que la gente se conecte a tu AP falso. Combinado con un **ataque de desautenticación** (mandar tramas deauth para desconectar a todos) es bastante efectivo.

También están los ataques a la **lista de redes preferidas (PNL)**. Tu celular o laptop siempre está buscando las redes a las que se ha conectado antes, y un atacante puede responder a esas solicitudes.

Sobre cifrado, vimos por qué **WEP es inseguro** — los vectores de inicialización son muy cortos y se repiten, así que con suficiente tráfico capturado se puede sacar la clave. La recomendación es usar WPA2-AES o WPA3.

En Bluetooth vimos **Bluejacking** (mandar mensajes no solicitados) y **Bluesnarfing** (robar datos del dispositivo). También ataques a BLE que se usa mucho en IoT.

Algo que me quedó claro es el concepto de **encadenamiento de exploits**: en la vida real un atacante no usa un solo ataque, va combinando varios para ir escalando privilegios y moviéndose dentro de la red.

---

## Módulo 6: Explotación de aplicaciones web

### Lo básico: HTTP y OWASP

Antes de entrar a los ataques, el módulo repasa cómo funciona HTTP (métodos, códigos de estado, cabeceras) y cómo se manejan las sesiones web con cookies y tokens. Es importante entender esto porque casi todos los ataques web se basan en manipular estas cosas.

El **OWASP Top 10** es como la biblia de las vulnerabilidades web. Es una lista que clasifica las fallas más comunes y críticas. Casi todo lo que vimos en este módulo está ahí.

### Inyección

La **inyección SQL** es probablemente el ataque web más conocido. Metes código SQL en los campos de entrada y si la aplicación no valida bien, puedes sacar datos de la base de datos, modificarlos o hasta borrarlos. Hay varios tipos: union-based, error-based, blind, etc.

La **inyección de comandos** es parecida pero en vez de SQL inyectas comandos del sistema operativo. Si la app usa funciones como system() o exec() sin sanitizar, puedes ejecutar lo que quieras en el servidor.

También vimos **inyección LDAP** que es lo mismo pero contra directorios LDAP, útil sobre todo en entornos con Active Directory.

La defensa para todo esto es la misma: validar y sanitizar las entradas, usar consultas parametrizadas y aplicar el principio de mínimo privilegio.

### Autenticación

El **secuestro de sesión** es cuando robas el token o cookie de sesión de alguien y te haces pasar por esa persona. Se puede hacer por sniffing, por XSS o prediciendo tokens débiles.

Los **ataques de redirección** aprovechan parámetros de redirect que no están validados para mandar al usuario a un sitio malicioso después del login.

Y las **credenciales por defecto** siguen siendo un problema enorme. Muchos equipos y aplicaciones vienen con admin/admin o root/root y nadie los cambia.

### Autorización

Aquí vimos dos cosas principales:

La **contaminación de parámetros (HPP)** que es mandar el mismo parámetro varias veces en la solicitud HTTP para confundir al servidor.

Y el **IDOR** (Insecure Direct Object Reference) que es simplemente cambiar un ID en la URL para acceder a datos de otro usuario. Por ejemplo cambiar `?id=100` por `?id=101`. Parece muy simple pero es sorprendentemente común.

### XSS (Cross-Site Scripting)

Hay dos tipos principales:
- **Reflejado**: el script malicioso va en la URL y se ejecuta cuando la víctima hace clic en un link
- **Almacenado**: el script se guarda en el servidor (en un comentario por ejemplo) y se ejecuta cada vez que alguien ve esa página. Este es más peligroso

Para defenderse: sanitizar entradas, codificar salidas, usar Content Security Policy y poner las cookies con HttpOnly y Secure.

### CSRF y SSRF

El **CSRF** engaña al navegador del usuario para que haga solicitudes que no quería hacer en un sitio donde ya está logueado. La defensa son los tokens anti-CSRF y la cookie SameSite.

El **SSRF** hace que el servidor haga solicitudes a recursos internos que normalmente no serían accesibles desde afuera.

### Configuraciones incorrectas

El **directory traversal** usa `../` para salirse del directorio web y leer archivos del sistema como /etc/passwd.

La **manipulación de cookies** es modificar los valores de las cookies cuando la aplicación confía en ellas sin validar del lado del servidor.

### Inclusión de archivos

- **LFI (Local)**: la app incluye archivos locales basándose en input del usuario. Se pueden leer archivos sensibles y en algunos casos ejecutar código
- **RFI (Remote)**: igual pero incluyendo un archivo de un servidor externo del atacante. Más peligroso porque da ejecución de código directa

### Malas prácticas de programación

Esta sección recopila errores comunes de los desarrolladores: dejar comentarios con info sensible en el código fuente, mensajes de error que dan demasiada información (stack traces, rutas, versiones), credenciales hardcodeadas, condiciones de carrera, APIs sin autenticación ni rate limiting, campos hidden en formularios que se pueden manipular fácil, y no firmar el código.

---

## Herramientas que se usaron

- **Nmap** — escaneo de puertos, detección de servicios y scripts NSE
- **Lynis** — auditoría de seguridad y hardening de sistemas Linux
- **rkhunter / chkrootkit** — detección de rootkits
- **ClamAV** — antivirus para Linux
- **Wireshark** — análisis de paquetes de red
- **Scapy** — creación de paquetes personalizados
- **SET (Social-Engineer Toolkit)** — simulación de ataques de ingeniería social
- **BeEF** — explotación de navegadores web
- **Responder** — captura de hashes LLMNR/NBT-NS
- **enum4linux** — enumeración SMB
- **Ettercap** — ataques MitM
- **Mimikatz** — extracción de credenciales y PtH
- **Hashcat / John the Ripper** — cracking de hashes
- **Burp Suite** — interceptar y analizar tráfico web
- **SQLMap** — automatizar inyección SQL
- **GVM** — escaneo de vulnerabilidades
- **Proxmark** — clonación RFID
- **Kismet** — escaneo de redes inalámbricas

## Conclusión

Lo que más me llevo de estos cuatro módulos es que un ataque real sigue un proceso: primero se hace reconocimiento (pasivo y activo), luego se buscan vulnerabilidades, se puede usar ingeniería social para obtener acceso inicial, y después se explotan las fallas encontradas en la red o en las aplicaciones web.

La seguridad tiene que ser por capas. No basta con un firewall o un antivirus, hay que proteger desde la red física hasta la lógica de la aplicación. También me quedó claro que muchos ataques se basan en lo mismo: falta de validación de entradas, configuraciones por defecto que nadie cambia, y sobre todo, el factor humano que sigue siendo el eslabón más débil.


Módulo 7: Seguridad en la Nube, Dispositivos Móviles e IoT
Ataques en la nube
Este módulo se enfoca en lo que pasa cuando el objetivo no es un servidor físico sino infraestructura en la nube (AWS, Azure, GCP). El cambio de mentalidad importante es que en la nube tú no controlas el hardware, controlas configuraciones — y ahí es donde están la mayoría de las fallas.
Los ataques más comunes que vimos:

Credenciales expuestas: keys de AWS o tokens que terminan en repos públicos de GitHub. Hay bots que escanean GitHub 24/7 buscando esto. Una vez que un atacante tiene una key con permisos amplios, básicamente ya entró.
Buckets S3 mal configurados: cubos de almacenamiento que se dejan públicos por error y exponen datos sensibles. Es uno de los errores más típicos.
IAM mal configurado: dar más permisos de los necesarios. Si una función Lambda tiene permisos de admin, cualquier vulnerabilidad en esa función se convierte en un compromiso total.
Metadata service abuse: en AWS, la URL 169.254.169.254 da acceso a credenciales temporales de la instancia. Si hay un SSRF en una app que corre en EC2, el atacante puede sacar esas credenciales desde ahí.
Account takeover: secuestrar la cuenta del proveedor cloud, normalmente por phishing al admin o por reutilización de contraseñas.

También vimos el modelo de responsabilidad compartida, que básicamente dice que el proveedor (AWS, por ejemplo) se encarga de la seguridad de la infraestructura, pero la seguridad en la nube (configuraciones, datos, accesos) es responsabilidad del cliente. Mucha gente no entiende esto y asume que AWS los protege de todo.
Ataques a dispositivos móviles
En móviles los ataques se dividen en Android e iOS, pero los vectores son parecidos:

Apps maliciosas: aplicaciones que parecen legítimas pero piden permisos excesivos o tienen funcionalidad oculta. En Android es más común porque se pueden instalar APKs fuera de la Play Store.
Jailbreak/Root: cuando el usuario rompe las restricciones del sistema operativo, también rompe muchas de las protecciones de seguridad.
Almacenamiento inseguro: muchas apps guardan tokens, contraseñas o datos sensibles sin cifrar en el dispositivo.
Comunicación insegura: apps que no usan HTTPS o que no validan correctamente los certificados (SSL pinning mal implementado o ausente).
Ingeniería inversa: con herramientas como APKTool o Frida se puede descompilar una app, modificarla y volverla a empaquetar. Esto se usa para saltar validaciones, ver cómo funcionan APIs internas, etc.

El OWASP Mobile Top 10 es el equivalente del OWASP Top 10 pero específico para móviles. Cubre cosas como uso indebido de plataformas, almacenamiento inseguro, autenticación débil, etc.
IoT (Internet de las Cosas)
El IoT es probablemente el área más vulnerable de todas. La razón es simple: muchos dispositivos IoT se diseñan pensando en costos y funcionalidad, no en seguridad. Cosas como cámaras IP, termostatos, electrodomésticos inteligentes, equipos médicos, etc.
Problemas típicos:

Credenciales por defecto que nadie cambia (admin/admin, root/root). Esto es exactamente cómo se construyó la botnet Mirai, que tumbó medio internet en 2016.
Firmware sin actualizaciones: muchos fabricantes ni siquiera publican parches, y si los publican, los usuarios no los aplican.
Protocolos inseguros: muchos dispositivos usan MQTT, CoAP o protocolos propietarios sin cifrado.
Interfaces de gestión expuestas: paneles web accesibles desde internet sin autenticación robusta.
Falta de cifrado en comunicaciones: la información viaja en texto plano entre el dispositivo y la nube del fabricante.

También vimos ataques a sistemas especializados como SCADA y entornos industriales (OT), donde las consecuencias de un ataque pueden ser físicas — apagar una planta, dañar maquinaria, etc.

Módulo 8: Realización de técnicas posteriores a la explotación
Este módulo es el "qué hago después de que ya entré". Conseguir el acceso inicial es solo el primer paso — los atacantes serios después se establecen, se mueven lateralmente y mantienen el acceso a largo plazo.
Establecer un punto de apoyo (foothold)
Una vez que tienes acceso a una máquina, lo primero es asegurarte de que ese acceso no se va a perder. Algunas técnicas:

Reverse shells: en vez de que tú te conectes a la víctima (lo que sería bloqueado por firewalls), haces que la víctima se conecte a ti. Herramientas como netcat, Metasploit o scripts en Python/Bash sirven para esto.
Bind shells: lo contrario, abrir un puerto en la víctima al que tú te conectas. Más fácil de detectar pero útil en algunos escenarios.
Web shells: subir un archivo (PHP, ASP, JSP) a un servidor web comprometido que te da una interfaz para ejecutar comandos. Cosas como c99 o WSO son clásicos.

Mantener persistencia
La persistencia es asegurarte de seguir teniendo acceso aunque la máquina se reinicie o el usuario cambie su contraseña. Métodos típicos:

Tareas programadas (cron en Linux, scheduled tasks en Windows) que ejecutan tu payload cada cierto tiempo.
Servicios maliciosos: crear un servicio que arranque con el sistema.
Modificación del registro en Windows (claves Run, RunOnce) para ejecutar algo al iniciar sesión.
Backdoors en archivos legítimos: modificar binarios o scripts que se ejecutan normalmente.
Cuentas nuevas: crear usuarios con privilegios para tener un acceso "limpio" si se descubre el original.
Llaves SSH: agregar tu llave pública al authorized_keys del servidor.

Escalada de privilegios
Casi nunca entras como administrador. Lo normal es entrar como un usuario con pocos permisos y de ahí escalar a root/SYSTEM.
En Linux:

Buscar binarios SUID mal configurados (con find / -perm -u=s -type f 2>/dev/null)
Explotar el archivo sudoers mal configurado
Aprovechar tareas cron con permisos incorrectos
Explotar kernel exploits si la versión es vulnerable
Herramientas como LinPEAS o linux-exploit-suggester automatizan mucho esto

En Windows:

Buscar servicios con permisos débiles
Token impersonation (con Mimikatz, por ejemplo)
DLL hijacking
Unquoted service paths
Herramientas como WinPEAS o PowerUp ayudan a enumerar todo esto

Movimiento lateral
Una vez que tienes privilegios en una máquina, casi siempre quieres llegar a otras. Eso es movimiento lateral.

Pass-the-Hash: ya lo vimos en el módulo 5, pero aquí toma sentido completo. Con el hash de un admin de dominio puedes autenticarte en cualquier máquina del dominio.
Pass-the-Ticket: similar pero con tickets de Kerberos.
PsExec / WMI / WinRM: ejecutar comandos remotamente en otras máquinas del dominio.
RDP: conectarse por escritorio remoto si tienes credenciales.
SSH pivoting: usar una máquina comprometida como puente para llegar a otras redes.

Evitar detección
Todo esto no sirve si te detectan rápido. Las técnicas anti-forenses incluyen:

Borrar logs: limpiar /var/log/ en Linux o los Event Logs en Windows.
Timestomping: modificar las fechas de los archivos para que no parezcan recientes.
Living off the land (LOTL): usar herramientas que ya están en el sistema (PowerShell, certutil, bitsadmin, etc.) en vez de subir binarios nuevos. Así no levantas alarmas en el antivirus.
Ofuscación: codificar payloads en base64, comprimir scripts, usar nombres aleatorios.
Evadir EDRs: técnicas más avanzadas para saltarse los sistemas de detección modernos.

Enumeración interna
Una vez dentro, se hace una enumeración mucho más profunda que la del reconocimiento inicial: usuarios del dominio, grupos, políticas, recursos compartidos, máquinas, servidores de bases de datos, etc. Herramientas como BloodHound mapean toda la red de Active Directory y muestran rutas de ataque que no son obvias a simple vista.

Módulo 9: Informes y comunicación
Este módulo cambia el chip completamente. Ya no es sobre cómo hackear, es sobre cómo contar lo que hackeaste de una forma que sea útil para el cliente. Algo que me quedó claro es que un pentest mal reportado es básicamente un pentest perdido — si nadie entiende los hallazgos, nadie los va a arreglar.
Componentes de un informe
Un informe profesional de pentesting suele tener:

Resumen ejecutivo: para los gerentes y la directiva. Sin tecnicismos. Explica el riesgo en términos de negocio (¿qué pasaría si esto se explota? ¿cuánto puede costar?). Una página máximo.
Alcance y metodología: qué se probó, qué no, cuándo, con qué herramientas y bajo qué reglas.
Hallazgos detallados: cada vulnerabilidad encontrada, con su descripción técnica, evidencia (capturas, comandos), nivel de riesgo y recomendación de mitigación.
Calificación de severidad: usando estándares como CVSS (Common Vulnerability Scoring System) que da una puntuación de 0 a 10 según el impacto.
Recomendaciones: qué hacer para arreglar cada problema, idealmente priorizadas por riesgo.
Conclusiones: postura general de seguridad de la organización.
Anexos: logs, listados completos, evidencia adicional.

Análisis de hallazgos
Un buen informe no solo lista vulnerabilidades. Las analiza. Por ejemplo, encontrar un SQL injection es un hallazgo, pero analizarlo sería decir: "esta vulnerabilidad permite acceder a la base de datos de clientes, que contiene 50.000 registros con información personal, lo que viola la ley de protección de datos y puede resultar en multas".
La priorización importa mucho. No todo es crítico. Hay que distinguir entre:

Crítico: explotación inmediata con impacto grave (ej. RCE en servidor de producción)
Alto: vulnerabilidad seria pero requiere algunas condiciones
Medio: explotable pero con impacto limitado
Bajo: información o configuraciones débiles que ayudarían a otros ataques
Informativo: cosas que no son vulnerabilidades pero conviene mencionar

Comunicación durante el pentest
La comunicación no empieza ni termina con el informe. Durante el pentest hay que:

Tener un contacto claro del lado del cliente para emergencias.
Reportar inmediatamente si encuentras algo crítico (datos expuestos en producción, brechas activas, etc.) — no esperes al informe final.
Mantener al cliente informado del progreso, sobre todo en pentests largos.
Documentar todo mientras ocurre, no después. Capturas, comandos, fechas, hashes — todo.

Actividades post-entrega
Después de entregar el informe el trabajo no se acaba:

Reunión de cierre: presentar los hallazgos al cliente, explicar lo más importante en persona.
Re-test: volver a probar después de que el cliente arregle las cosas, para confirmar que las correcciones funcionan.
Lecciones aprendidas: documentar internamente qué funcionó y qué no para mejorar futuros pentests.
Limpieza: eliminar herramientas, accesos, cuentas creadas durante el pentest. No dejar puertas abiertas.

Aspectos legales y éticos
Algo que se enfatiza mucho es la importancia del contrato y autorización por escrito. Sin un documento firmado que defina claramente el alcance, lo que estás haciendo deja de ser pentesting y empieza a ser un delito. La regla básica es: si no está autorizado, no lo toques.

Módulo 10: Herramientas y análisis de código
Este último módulo es como la caja de herramientas y un poco de programación aplicada al pentesting.
Conceptos básicos de scripting
No necesitas ser desarrollador para hacer pentesting, pero sí conviene saber leer y escribir scripts. Los lenguajes más útiles:

Python: probablemente el más usado. Sirve para casi todo: automatizar escaneos, escribir exploits, parsear salidas, manipular paquetes con Scapy, interactuar con APIs.
Bash: indispensable para Linux. Encadenar comandos, automatizar tareas repetitivas, post-explotación.
PowerShell: el equivalente en Windows. Muy potente para movimiento lateral y enumeración en entornos Active Directory. Frameworks como PowerSploit o Empire se basan en esto.
Ruby: lo usa Metasploit internamente.
JavaScript: necesario para entender XSS y ataques del lado del cliente.

Análisis de código de explotación
Una habilidad importante es saber leer un exploit antes de ejecutarlo. Hay varias razones:

Seguridad: muchos "exploits" en internet en realidad son backdoors disfrazados que comprometen tu propia máquina cuando los ejecutas.
Adaptación: rara vez un exploit funciona tal cual. Casi siempre hay que ajustar IPs, puertos, payloads, encoding, etc.
Aprendizaje: entender cómo funciona un exploit te enseña sobre la vulnerabilidad subyacente.

Cuando se analiza código de explotación se busca:

Qué vulnerabilidad explota (CVE específico)
Cómo construye el payload
Qué condiciones requiere
Si tiene comportamiento sospechoso o conexiones a hosts desconocidos

Categorías de herramientas
El módulo organiza las herramientas por categoría según en qué fase del pentest se usan:
Reconocimiento y OSINT: Maltego, Recon-ng, theHarvester, Shodan, Spiderfoot.
Escaneo de red: Nmap, Masscan, Zmap.
Análisis de vulnerabilidades: Nessus, OpenVAS/GVM, Nikto, OWASP ZAP.
Explotación: Metasploit Framework, ExploitDB, searchsploit.
Aplicaciones web: Burp Suite, OWASP ZAP, SQLMap, Gobuster, ffuf, dirb.
Cracking de contraseñas: John the Ripper, Hashcat, Hydra, Medusa.
Inalámbrico: Aircrack-ng, Kismet, Wifite, Reaver.
Post-explotación: Mimikatz, BloodHound, Empire, PowerSploit, LinPEAS, WinPEAS.
Análisis forense y debugging: Wireshark, Ghidra, IDA, radare2.
Frameworks completos: Cobalt Strike (comercial), Metasploit (open source), Sliver.
Distribuciones especializadas
En vez de instalar todo a mano, hay distribuciones de Linux que ya vienen con todo el arsenal listo:

Kali Linux: la más popular. Mantenida por Offensive Security.
Parrot OS: alternativa más ligera y con enfoque en privacidad.
BlackArch: basada en Arch, con muchísimas herramientas.

Para defensa también hay distribuciones como Security Onion o REMnux (esta última específica para análisis de malware).
Conclusión personal del módulo
Lo que entendí es que las herramientas son solo eso, herramientas. No te hacen pentester por sí solas. Lo importante es entender qué hace cada una, cuándo usarla y cómo interpretar lo que te devuelve. Una persona que solo aprende a apretar botones en Metasploit no va a llegar muy lejos cuando algo no funcione, porque no va a entender por qué.
También me quedó claro que el scripting es lo que separa a un pentester básico de uno bueno. Saber automatizar tus propios procesos, escribir tus propios exploits o adaptar los existentes es lo que marca la diferencia.