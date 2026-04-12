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