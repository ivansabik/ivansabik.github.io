---
title: "Ejecutar scripts node.js al iniciar ubuntu"
date: 2015-08-04
template: article.jade
---

Node.js es famoso por sus callback y su enfoque orientado eventos lo que se presta para usarlo para ejecutar scripts o programitas desarrollados con Javascript. Recientemente quize saber cuándo publicarían la convocatoria para el premio de la Antonio Minzoni de la CNSF y pensé que sería mejor tener un servicio que avisar por correo electrónico si se publicó la página con la convocatoria.

Otra cosa interesante de Node.js es el extenso número de librerías que existen disponibles a través de npm además que es muy sencillo configurar las dependecias. 

### Requisitos

Node.js y npm instalados, la forma más fácil en ubuntu es:

```bash
$ sudo apt-get install nodejs
$ sudo apt-get install npm
```

Luego las dependencias con npm:

```bash
$ npm install nodemailer
$ npm install nodemailer-smtp-transport
$ npm install request
```

Para saber si ya se publicó la convocatoria, se puede usar un programa que se ejecute periódicamente o al iniciar la compu (que es este caso). Para esto voy a usar [node-startup](https://github.com/chovy/node-startup/blob/master/init.d/node-app). 

Primero hay que crear el archivo ```/etc/init.d/alerta-cnsf``` y copiar el contenido del template de node-app. Este archivo contendrá un código sh (shell, bash) que ejecutará el script en Node.js al inicio. El mío quedó [algo así](https://gist.github.com/ivansabik/cd70ca34e82403c66781), los programines en javascript se guardarán en "home/scripts-startup":

```bash
#!/bin/sh

NODE_ENV="production"
PORT="3000"
APP_DIR="~/scripts-startup"
NODE_APP="cnsf.js"
...
...
```
Luego hay que actualizar con:

```
$ update-rc.d node-startup defaults
```

### El ~~script de java~~ javascript

En convocatorias anteriores, las publicaciones y los resultados están en:

 - http://www.cnsf.gob.mx/Eventos/Paginas/Premios_2012.aspx
 - http://www.cnsf.gob.mx/Eventos/Paginas/Premios_2013.aspx
 - http://www.cnsf.gob.mx/Eventos/Paginas/Premios_2014.aspx
 
Entonces la idea es hacer una petición HTTP a http://www.cnsf.gob.mx/Eventos/Paginas/Premios_2015.aspx esperando que ahí se publique. Esto se puede hacer por medio de las repuestas HTTP comunes (200 para ok y 400 para no encontrado) Entonces hay que crear el archivo "~/scripts-startup/cnsf.js".

#### ~/scripts-startup/cnsf.js
<script src="https://gist.github.com/ivansabik/bc4ed7f2cc4ad118eb8d.js"></script>

#### Probando que funciona

Listo, si todo salió bien al reiniciar, se ejecutará el javascript pero podemos ver si funciona ejecutándolo como cualquier otro programa de Node.js:

```bash
$ node ~/scripts-startup/cnsf.js
-------------------------
Tue Apr 07 2015 03:11:08 GMT-0500 (CDT)
Iniciando cnsf.js...
Intento 1
Tue Apr 07 2015 03:11:08 GMT-0500 (CDT)
Pagina http://www.cnsf.gob.mx/Eventos/Paginas/Premios_2015.aspx responde "404"
```

Y para probar que funciona el envío del correo se cambia temporalmente esta línea:

```javascript
if(res.statusCode === 200) {
``` 

Por:

```javascript
if(res.statusCode) {
``` 
Y al volver a correr el script:

```bash
$ node ~/scripts-startup/cnsf.js
-------------------------
Tue Apr 07 2015 03:22:04 GMT-0500 (CDT)
Iniciando cnsf.js...
Intento 1
Enviando mail
Enviado!
Respuesta "250 2.0.0 OK 1428394926 c41sm5809909yhc.53 - gsmtp"
```

![Mail de alerta](mail-alerta-cnsf.png)

¡Y sí son las tres de la mañana entonces ya no voy a corregir el "Convocatorioa"!
