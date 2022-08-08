# Guía para subir un proyecto de Laravel con Nginx en un servidor Ubuntu.

## Creación de una base de datos de MySQL y un usuario.

Ya instalamos MySQL, pero debemos crear una base de datos y un usuario para que lo use nuestra app.

Para comenzar, inicie sesión en la cuenta root de MySQL (administrativa) emitiendo este comando (tenga en cuenta que este no es el usuario root de su servidor):

```
mysql -u root -p
```

Una vez dentro de MySQL, se le solicitará la contraseña que configuró para la cuenta root de MySQL.

**Nota:** Si no puede acceder a su base de datos MySQL a través de root, como usuario sudo puede actualizar la contraseña de su usuario root iniciando sesión en la base de datos de esta forma:

```
sudo mysql -u root
```

Una vez que reciba la instrucción de MySQL, puede actualizar la contraseña del usuario root. Aquí, sustituya ***new_password*** por una contraseña segura de su elección.

```sql
ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'new_password';
```

Ahora puede escribir EXIT; y puede volver a iniciar sesión en la base de datos a través de la contraseña con el siguiente comando:

```
mysql -u root -p
```

En mysql, puede crear una base de datos exclusiva para que la app la controle. Puede ponerle el nombre que quiera. Cree la base de datos escribiendo lo siguiente:

```sql
CREATE DATABASE name_db DEFAULT CHARACTER SET utf8 COLLATE utf8_general_ci;
```
**Nota:** Cada instrucción de MySQL debe terminar en punto y coma (;). Asegúrese de que esto no falte si experimenta problemas.

A continuación, crearemos una cuenta de usuario separada de MySQL que usaremos exclusivamente para realizar operaciones en nuestra nueva base de datos. Crear bases de datos y cuentas específicas puede ayudarnos desde el punto de vista de administración y seguridad.

Crearemos esta cuenta, configuraremos una contraseña y concederemos acceso a la base de datos que hemos creado. Podemos hacerlo escribiendo el siguiente comando. Recuerde elegir una contraseña segura aquí para su usuario de base de datos donde tenemos `password`:

```sql
CREATE USER 'user_name'@'%' IDENTIFIED WITH mysql_native_password BY 'password';
```

A continuación, deje saber a la base de datos que nuestro usuario debería tener acceso completo a la base de datos que configuramos:

```sql
GRANT ALL ON name_db.* TO 'user_name'@'%';
```

Ahora tiene una base de datos y una cuenta de usuario, creadas específicamente para WordPress. Debemos eliminar los privilegios de modo que la instancia actual de MySQL sepa sobre los cambios recientes que hemos realizado:

```sql
FLUSH PRIVILEGES;

EXIT;
```

## Descargar el proyecto de GIT

Nos dirigimos a var/www

```
cd var/www
```

Y clonamos el repositorio

```
git clone url_repository
```

## Configuramos el proyecto Laravel

Luego creamos el archivo .env en la raiz del proyecto y le colocamos todas las constantes necesarias, incluida la conexión con la base de datos.

Ahora generaremos la key que necesita Laravel
```
php artisan key:generate
```

Laravel necesita que la carpeta storage y bootstrap/cache tengan los permisos necesarios.

Asi que cambiamos los permisos de la carpeta storage
```bash
sudo chown -R www-data.www-data /var/www/name_project/storage/
```

Y la carpeta bootstrap/cache

```bash
sudo chown -R www-data.www-data /var/www/name_project/bootstrap/cache/
```

## Ejecutar migraciones

Debemos ejecutar las migraciones para que se creen las tablas.

```bash
php artisan migrate
```

## Configuración Nginx para laravel

Ahora crearemos un archivo para configurar nginx en nuestro proyecto

```bash
sudo nano etc/nginx/sites-available/academiadecumplimiento.com
```

Y copiamos la configuración que encontramos en el sitio oficial de Laravel
https://laravel.com/docs/9.x/deployment

Y cambiamos las constantes `server_name` y `root`

Ahora creamos un enlace simbolico de nuestro archivo de sites-available

```bash
sudo ln -s /etc/nginx/sites-available/academiadecumplimiento.com /etc/nginx/sites-enabled/
```

Ahora verificamos que la sintaxis de la configuración de nginx se encuentre bien y sin errores.

```bash
sudo nginx -t
```

Ahora recargamos nginx

```bash
sudo systemctl reload nginx
```
o
```bash
sudo service nginx reload
```

Nuestro proyecto ya deberia de estar funcionando, pero nos falta algo, configurar el dominio e instalar el certificado SSL.

## Configurar dominio

Para este paso, debemos ingresar a nuestro proveedor de dominio e ingresar en los DNS un nuevo registro:

Tipo: `A record` 

Host: `@` (si necesitamos un subdominio, lo colocamos, ejemplo: app, que vendria siendo app.dominio.com)

Valor: `ip_server`

Y listo, ahora solo deberiamos esperar que los DNS se extiendan por todo el internet.

## Instalar certificado SSL

Instalamos la herramienta necesaria para los certificados.

```bash
sudo apt install certbot python3-certbot-nginx
```

Y ahora instalamos el certificado a nuestro dominio
```bash
sudo certbot --nginx -d ejemplo.com -d www.ejemplo.com
```

Luego nos pedirá nuestro correo para notificarnos cuando el certificado vaya a expirar, despues aceptamos los terminos y condiciones y por ultimo escribimos N para no compartir nuestro correo.



