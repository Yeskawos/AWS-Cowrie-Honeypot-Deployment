# 🍯Proyecto Blue Team: Despliegue de Honeypot (Cowrie) en AWS

**Autor:** Córdoba

**Fecha:** 23/06/2026

**Categoría:** Blue Team / Active Defense / Cloud Security

**Objetivo:** Desplegar un honeypot SSH interactivo en infraestructura AWS para la recolección de inteligencia de amenazas, implementando bastionado de servicios, gestión de sockets en Linux y redirección NAT silenciosa.

## 1. Aprovisionamiento de Infraestructura Cloud (AWS EC2)
El proyecto comenzó con la creación y configuración de la máquina virtual que actuaría como servidor cebo en la nube pública.

<img width="1064" height="897" alt="configuracion-nombre-server" src="https://github.com/user-attachments/assets/936d24ec-5ede-4e08-b42b-d4fb827f60de" />


* **Configuración de la Instancia:** Se localizó el servicio EC2, definiendo el nombre del servidor y seleccionando una instancia de la capa gratuita (t3.micro) con Ubuntu 24.04.

Evidencias:
<img width="1069" height="239" alt="buscamos-ec2" src="https://github.com/user-attachments/assets/0eefe542-cd94-41cf-893b-1b84f6744094" />
<img width="1064" height="275" alt="configuracion-instancia-tipo" src="https://github.com/user-attachments/assets/10480eca-9083-4c8d-bd98-c42363f727be" />
<img width="1061" height="904" alt="configuracion-red-almacenamiento" src="https://github.com/user-attachments/assets/0b980c08-7306-496b-8d70-dc53da0e15df" />


* **Seguridad y Red:** Se generó un nuevo par de claves criptográficas (.pem) para el acceso inicial y se configuraron las reglas básicas del Grupo de Seguridad perimetral, junto con el almacenamiento.

Evidencias:

<img width="599" height="577" alt="creacion claves privadas" src="https://github.com/user-attachments/assets/8d50430b-b776-42fd-bb4d-49ae4d47d063" />
<img width="1073" height="219" alt="configuracion-par-claves" src="https://github.com/user-attachments/assets/027eb1da-58e5-4648-a241-c42824800d43" />

* **Despliegue:** Se lanzó la instancia y se verificó su estado de ejecución (Running), obteniendo la IP pública necesaria para la conexión.

Evidencias:

<img width="1627" height="633" alt="lanzamos-instancia" src="https://github.com/user-attachments/assets/ffdaf1e5-e4ed-415d-9482-07766feb2f76" />
<img width="1637" height="570" alt="pestaña-seguridad-aws" src="https://github.com/user-attachments/assets/1e83ad78-d8d7-43b2-b2a1-1615a1fe99af" />
<img width="1900" height="187" alt="instancia-running-se-ve-ip-publica" src="https://github.com/user-attachments/assets/363a977b-62d8-4584-844c-a4d996b9831f" />

## 2. Preparación del Sistema y Dependencias
Una vez levantado el servidor, procedimos a establecer la primera conexión e instalar la arquitectura base necesaria para soportar el entorno virtualizado de Python.

* **Conexión Inicial y Actualización:** Nos conectamos vía PowerShell usando la clave .pem y descargamos todas las dependencias del sistema operativo (Python, virtualenv, librerías de compilación).

Evidencias:

<img width="703" height="647" alt="configuracion-pem-powershell-y-conexion-ssh" src="https://github.com/user-attachments/assets/b5253f33-a3f6-4ffc-99be-136dabe4c44a" />
<img width="844" height="94" alt="descargando-dependencias" src="https://github.com/user-attachments/assets/e4132364-f9b8-4a81-ae86-5dd1e8a6718b" />

* **Creación del Usuario de Servicio:** Por seguridad, se creó un usuario sin privilegios llamado cowrie para aislar la ejecución del honeypot del resto del sistema operativo.

Evidencia:

<img width="389" height="203" alt="creamos-user-cowrie" src="https://github.com/user-attachments/assets/4819bb18-834c-46aa-a53f-09d8df313890" />

## 3. Instalación y Configuración del Motor Cowrie
Trabajando bajo el nuevo usuario restringido, construimos la "jaula de cristal" que simularía el sistema operativo falso.

* **Clonación y Entorno Virtual:** Descargamos el código fuente desde GitHub y creamos un entorno aislado de Python para evitar conflictos con las librerías nativas de Ubuntu.

Evidencias:

<img width="594" height="205" alt="clonamos-cowrie-git" src="https://github.com/user-attachments/assets/f892dccd-a30a-403a-b4f1-a6359a0c2e94" />
<img width="838" height="120" alt="creamos-entorno-aislado" src="https://github.com/user-attachments/assets/89b6ac12-61af-43ac-bbeb-7d8855815fee" />

* **Instalación de Requisitos:** Instalamos las dependencias específicas de Cowrie mediante pip dentro de la burbuja virtual.

Evidencias:

<img width="721" height="213" alt="instalamos-requiriments" src="https://github.com/user-attachments/assets/2f94551d-97c4-46cd-983d-38ce44653004" />
<img width="581" height="438" alt="pip-install-e" src="https://github.com/user-attachments/assets/b4b39558-2a2d-4cbb-a205-db70c9d4a63e" />

* **Configuración del Cebo:** Copiamos el archivo de configuración base y modificamos el hostname para hacer el servidor más atractivo a los atacantes. Finalmente, arrancamos el motor y verificamos mediante netstat (o ss) que el proceso de Python estaba a la escucha en el puerto aislado 2222.

Evidencias:

<img width="739" height="126" alt="cp-src-courie-y-ls-del-nuevo-etc" src="https://github.com/user-attachments/assets/64f85047-0025-41a2-9d39-59c4e8adb6fb" />
<img width="633" height="689" alt="cambiando-hostname-cowrie" src="https://github.com/user-attachments/assets/87289f15-4a2e-474c-be04-2e925af7a52d" />
<img width="836" height="343" alt="start-cowrie-y-netstat" src="https://github.com/user-attachments/assets/0d5d0791-a5a2-4a02-b35d-01122f609a25" />

## 4. Bastionado (Hardening) y Evasión de Bloqueos
Para asegurar que los atacantes cayeran en la trampa sin perder el control remoto del servidor, implementamos una estrategia de doble cerradura.

* **Firewall Perimetral (AWS):** Modificamos el Security Group en AWS para permitir tráfico entrante por nuestro nuevo puerto secreto de administración (22222).

Evidencias:

<img width="1672" height="485" alt="grupo de seguridad" src="https://github.com/user-attachments/assets/2a623e69-8cde-4095-a5b8-99e30eb88f82" />
<img width="1665" height="424" alt="editar-reglas-entrada" src="https://github.com/user-attachments/assets/115bc755-2adf-4d8b-8a1d-e18bacdbee7e" />

* **Modificación del Servicio SSH:** Editamos el archivo /etc/ssh/sshd_config para indicarle al servicio que abriera temporalmente ambos puertos (22 y 22222).

Evidencia:

<img width="810" height="369" alt="modificamos-ssh-conf" src="https://github.com/user-attachments/assets/c2089cd3-43a8-4353-b76c-4a9ff13d8076" />

* **Troubleshooting de Arquitectura (Systemd Sockets):** Nos enfrentamos a un bloqueo causado por el mecanismo moderno de Ubuntu, el cual forzaba la escucha exclusiva en el puerto 22. Deshabilitamos este "recepcionista" y reactivamos el servicio SSH tradicional.

Evidencias:

<img width="726" height="91" alt="funcionando tupln" src="https://github.com/user-attachments/assets/59ae5311-e210-4f3f-8c4a-4517b8b812cd" />
<img width="776" height="126" alt="deshabilitamos-ssh-socket-reiniciamos servicio y eso" src="https://github.com/user-attachments/assets/4f9ce0f4-e4f3-4407-be19-6b341f6c8164" />

## 5. Enrutamiento Ofensivo y Monitorización SOC
Con el control asegurado en el puerto 22222, cerramos la puerta principal y tendimos la trampa final en la red.

* **Liberación del Puerto y Redirección:** Actualizamos el sshd_config para eliminar el puerto 22 y reiniciamos el servicio. Aplicamos una regla de iptables en la tabla NAT (PREROUTING) para redirigir todo el tráfico entrante del puerto 22 hacia la jaula de Cowrie (puerto 2222).

Evidencias:

<img width="412" height="406" alt="volvemos a actualizar sshd config y quitamos port 22" src="https://github.com/user-attachments/assets/3f2c900c-5a1c-4d27-95ac-47ba90d0c6fc" />
<img width="768" height="62" alt="reiniciamos ssh de nuevo y cambiamos iptables" src="https://github.com/user-attachments/assets/5fe053da-44b9-46e8-897b-52beaf5146a3" />

* **Simulación de Ataque (Red Teaming):** Desde una máquina externa, lanzamos un asalto intentando conectarnos como root. Tuvimos que purgar el registro de llaves locales de Windows (ssh-keygen -R) debido a la alerta de Man-in-the-Middle generada por el honeypot al inyectar su llave criptográfica falsa.

Evidencias:

<img width="648" height="234" alt="nos conectamos desde la maquina atacante" src="https://github.com/user-attachments/assets/5d575d47-2766-4c92-9cd7-2b2e9cd5a20e" />
<img width="595" height="663" alt="keygen y conexion del atacante exitosa al honeypot" src="https://github.com/user-attachments/assets/58ca3738-264b-4357-a0f0-a11ce27c11bd" />

* **Vigilancia Activa (Blue Teaming):** Desde nuestra consola segura de administrador, ejecutamos tail -f sobre el archivo de log en crudo del honeypot, observando en tiempo real las pulsaciones y comandos inyectados por el atacante dentro del entorno simulado.

Evidencia:

<img width="941" height="177" alt="ponemos camara" src="https://github.com/user-attachments/assets/822515be-a1b6-42ad-934d-79978cf4bf88" />

## ✅ Conclusiones
El despliegue de infraestructura defensiva activa revela lecciones críticas sobre arquitectura y redes:

* **La trampa de los Firewalls Dobles: Manejar simultáneamente Security Groups perimetrales y cortafuegos internos (UFW) puede provocar bloqueos ciegos; delegar la seguridad al perímetro es vital en entornos Cloud.**

* **Evolución del Kernel (Systemd): Los servicios modernos de Linux a menudo delegan la gestión de puertos a sockets (ssh.socket), lo que requiere técnicas de bastionado actualizadas para evitar la denegación de servicio autoinducida.**

* **El poder de IPTables: La manipulación transparente del tráfico NAT permite desviar ataques hacia jaulas aisladas sin que el agresor detecte anomalías en el escaneo de puertos inicial, generando inteligencia de alto valor.**
