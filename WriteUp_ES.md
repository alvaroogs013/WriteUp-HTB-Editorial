<h1>Enumeración</h1>
<p>Empezamos haciendo ping a la IP objetivo 10.10.11.20</p>
<br>
<center>
    <img src="https://github.com/alvaroogs013/WriteUp-HTB-Editorial/assets/131161276/08362971-cead-4896-a56e-fe40e75960f0" alt="Ping a la IP objetivo">
</center>
<br>

<p>A continuación, realizamos un escaneo simple con nmap para determinar qué puertos están abiertos en la máquina objetivo:</p>
<pre><code>sudo nmap -p- --open -sV -sS -n 10.10.11.20 --min-rate 5000</code></pre>
<p>Esto reporta los siguientes puertos abiertos:</p>
<ul>
    <li>Puerto 22 ejecutando OpenSSH</li>
    <li>Puerto 80 ejecutando el servidor web Nginx (http)</li>
</ul>
<br>
<center>
    <img src="https://github.com/alvaroogs013/WriteUp-HTB-Editorial/assets/131161276/4b2effdb-f0b9-40c9-a839-c58bf90c92d9" alt="Resultados del escaneo con nmap">
</center>
<br>

<h1>Comprobando el servidor web</h1>
<p>Abramos el navegador y verifiquemos el servidor web:</p>
<br>
<center>
    <img src="https://github.com/alvaroogs013/WriteUp-HTB-Editorial/assets/131161276/9acff6bd-4f3d-4575-9021-a44b7bf6c1f4" alt="Verificando el servidor web">
</center>
<br>

<p>Podemos ver que la IP objetivo resuelve su nombre en editorial.htb, por lo que añadimos esto a nuestro archivo local /etc/hosts para una resolución más rápida:</p>
<br>
<center>
    <img src="https://github.com/alvaroogs013/WriteUp-HTB-Editorial/assets/131161276/02037fad-5f58-4e75-9aae-173b4b99cb8b" alt="Actualizando /etc/hosts">
</center>
<br>

<h1>Explotación</h1>
<p>Recargando el servidor, podemos ver el sitio web editorial.htb. Vamos a explorar para encontrar algo interesante...</p>
<br>
<center>
    <img src="https://github.com/alvaroogs013/WriteUp-HTB-Editorial/assets/131161276/587b4d5d-c407-4c62-86ba-5c8316af0c06" alt="Sitio web editorial.htb">
</center>
<br>

<p>Hay un directorio editorial.htb/upload que nos permite subir URLs e imágenes. Utilizamos Burp Suite para inspeccionar cómo el servidor maneja esta solicitud. Si ingresamos una URL en el campo book URL y enviamos la solicitud usando Burp Suite Repeater, el servidor responde con un estado 200 OK, indicando una vulnerabilidad SSRF.</p>
<br>
<center>
    <img src="https://github.com/alvaroogs013/WriteUp-HTB-Editorial/assets/131161276/94fc8347-b13b-4f78-87db-834ab0d47af9" alt="Burp Suite - Vulnerabilidad SSRF">
</center>
<br>

<p>Conociendo esta brecha de seguridad, podemos intentar enviar la IP local (127.0.0.1) en el puerto 5000 (un puerto común para exploits SSRF). Alternativamente, podemos verificar todos los puertos (1-65535) para respuestas inusuales del servidor. Para demostrar, enviamos la solicitud a Burp Suite Intruder y lanzamos una carga útil para verificar la respuesta del servidor en diferentes puertos:</p>
<br>
<center>
    <img src="https://github.com/alvaroogs013/WriteUp-HTB-Editorial/assets/131161276/4aa1d854-2fc9-4241-a09c-b495bb0eb21d" alt="Burp Suite Intruder - Verificando puertos">
</center>
<br>

<p>Probemos esta lista de puertos: 1, 2, 5, 6, 80, 443, 5000, 8080</p>
<br>
<center>
    <img src="https://github.com/alvaroogs013/WriteUp-HTB-Editorial/assets/131161276/63a2620a-bc9f-4dd4-8881-5ce8121aed9c" alt="Lista de puertos">
</center>
<br>

<p>El puerto 5000 da la longitud más corta (222), lo que sugiere un directorio anterior. Verifiquemos esta ruta con Burp Suite Repeater:</p>
<br>
<center>
    <img src="https://github.com/alvaroogs013/WriteUp-HTB-Editorial/assets/131161276/a5e3c986-c081-4fcf-9912-81a013a68e9c" alt="Burp Suite Repeater - Ruta de directorio">
</center>
<br>

<p>Intentamos obtener el contenido de esta ruta. Si usamos una solicitud POST, obtenemos una respuesta encriptada, así que probamos con una solicitud GET:</p>
<br>
<center>
    <img src="https://github.com/alvaroogs013/WriteUp-HTB-Editorial/assets/131161276/33c8ccc8-6923-4107-9d23-c0e32ade964a" alt="Solicitud GET">
</center>
<br>

<p>En el directorio static/uploads, encontramos un archivo JSON que contiene información sobre varias API utilizadas por el servidor web. La API /api/latest/metadata/messages/authors llama nuestra atención. Repetimos el mismo proceso con esta ruta:</p>
<br>
<center>
    <img src="https://github.com/alvaroogs013/WriteUp-HTB-Editorial/assets/131161276/cdde3c9f-dae9-4d34-b583-34b2bd2e46b5" alt="Archivo JSON - Información de API">
</center>
<br>

<p>Finalmente, obtenemos una respuesta con información sensible sobre los datos de esta API. Usamos estas credenciales para iniciar una sesión SSH:</p>
<br>
<center>
    <img src="https://miro.medium.com/v2/resize:fit:720/format:webp/1*-lbTLf929CaUAHaWWoLQbQ.png">
</center>
<center>
    <img src="https://github.com/alvaroogs013/WriteUp-HTB-Editorial/assets/131161276/cde3b765-7f5a-4afd-a658-cf89af3f7d1d" alt="Inicio de sesión SSH">
</center>
<br>

<p>Podemos iniciar sesión en SSH con las credenciales obtenidas. Capturamos la bandera del usuario ejecutando <code>cat user.txt</code>. Primer objetivo completado, ahora vamos por la bandera de root.</p>

<p>Navegamos a los directorios del usuario dev y encontramos un directorio .git. Veamos qué contiene:</p>
<br>
<center>
    <img src="https://github.com/alvaroogs013/WriteUp-HTB-Editorial/assets/131161276/a815b606-687e-4f9d-a933-84db4dbdd1d9" alt="Directorio .git">
</center>
<br>

<p>Hay varios directorios dentro del repositorio git. Verificamos el directorio de logs para ver si hay información sobre otro usuario con permisos elevados. La segunda entrada del log sugiere un archivo con la información de creación de la API editorial. Usamos el comando <code>git show</code> y el ID del log para ver su contenido:</p>
<br>
<center>
    <img src="https://github.com/alvaroogs013/WriteUp-HTB-Editorial/assets/131161276/635ca00b-1685-4352-b021-ab04c1b8cd62" alt="Comando git show">
</center>
<br>

<p>Encontramos un nuevo usuario en el sistema con una contraseña. Iniciamos sesión en SSH con este nuevo usuario:</p>
<br>
<center>
    <img src="https://github.com/alvaroogs013/WriteUp-HTB-Editorial/assets/131161276/dd4ef326-f2bd-448d-88ca-17dd0bc666de" alt="Inicio de sesión SSH - Nuevo usuario">
</center>
<br>

<p>No hay ningún archivo interesante y no somos root, así que ejecutamos <code>sudo -l</code> para verificar qué permisos tenemos con este usuario:</p>
<br>
<center>
    <img src="https://github.com/alvaroogs013/WriteUp-HTB-Editorial/assets/131161276/9870d3b8-d78d-4ea0-b984-969c6cd83909" alt="Comando sudo -l">
</center>
<br>

<p>Tenemos permisos de root para ejecutar un script de Python que nos permite copiar archivos. Explotamos este script para copiar el directorio /root/root.txt a nuestro /home/root.txt:</p>
<pre><code>sudo /usr/bin/python3 /opt/internal_apps/clone_changes/clone_prod_change.py "ext::sh -c 'cat /root
