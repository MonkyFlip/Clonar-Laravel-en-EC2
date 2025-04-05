# Clonar-Laravel-en-EC2
Repositorio creado para clonar un proyecto de Laravel desde un repositorio de GitHub e implementarlo en una instancia EC2 de AWS.  

## Configuración y despliegue de Laravel en AWS EC2 con Amazon Linux 2023 y Apache

## Solución a errores comunes
# amazon-linux-extras: command not found
sudo dnf install -y php-cli php-mbstring php-xml php-curl php-zip php-pdo php-fpm php-mysqlnd unzip

# Unable to find a match: php-mysql
sudo dnf install -y php-mysqlnd

# apt: command not found (Usa dnf en lugar de apt)
sudo dnf update -y

## Pasos para la instalación

# 1. Actualizar paquetes
sudo dnf update -y

# 2. Instalar Apache
sudo dnf install -y httpd

# 3. Habilitar y arrancar Apache
sudo systemctl enable --now httpd

# 4. Verificar que Apache está corriendo
sudo systemctl status httpd

## Configurar Apache para Laravel
# Asegúrate de que la carpeta de configuración de Apache existe antes de editar el archivo:
sudo mkdir -p /etc/httpd/conf.d

sudo nano /etc/httpd/conf.d/laravel.conf

# Añadir la siguiente configuración en el archivo:
<VirtualHost *:80>
    ServerAdmin webmaster@localhost
    DocumentRoot "/var/www/laravel/public"
    <Directory "/var/www/laravel/public">
        AllowOverride All
        Require all granted
    </Directory>
    ErrorLog "/var/log/httpd/laravel-error.log"
    CustomLog "/var/log/httpd/laravel-access.log" common
</VirtualHost>

# Aplicar los cambios en Apache
sudo systemctl restart httpd

## Clonar Laravel desde GitHub

# 1. Instalar Git
sudo dnf install -y git

# 2. Descargar Composer
curl -sS https://getcomposer.org/installer | php
sudo mv composer.phar /usr/local/bin/composer

# 3. Crear la carpeta de instalación si no existe
sudo mkdir -p /var/www

# 4. Clonar el proyecto desde GitHub
cd /var/www
sudo git clone https://github.com/usuario/repositorio-laravel.git laravel

# 5. Instalar dependencias con Composer
cd /var/www/laravel
sudo composer install

## Configurar permisos
sudo chown -R apache:apache /var/www/laravel
sudo chmod -R 775 /var/www/laravel/storage /var/www/laravel/bootstrap/cache

## Configurar el archivo `.env`

# Opción 1: Copiar el archivo de ejemplo y generar la clave de la aplicación
cp .env.example .env
php artisan key:generate

# Opción 2: Crear el archivo manualmente y copiar el contenido desde la máquina local
sudo nano /var/www/laravel/.env
# Luego, copia y pega el contenido de tu archivo `.env` local en este archivo.

## Configurar un certificado SSL autofirmado para HTTPS

# 1. Instalar mod_ssl en Apache
sudo dnf install -y mod_ssl

# 2. Crear la carpeta de certificados si no existe
sudo mkdir -p /etc/pki/tls/certs
sudo mkdir -p /etc/pki/tls/private

# 3. Generar el certificado autofirmado y la clave privada
sudo openssl req -new -x509 -days 365 -nodes -out /etc/pki/tls/certs/laravel-cert.pem -keyout /etc/pki/tls/private/laravel-key.pem

# Durante la generación del certificado, se te pedirá llenar un formulario con los siguientes campos:
 - Country Name (2 letter code) [XX]: MX   # Código del país, por ejemplo MX para México.
 - State or Province Name (full name) []: Estado de México   # Escribe el estado donde se encuentra el servidor.
 - Locality Name (eg, city) []: Xonacatlán   # La ciudad donde se hospeda el servidor.
 - Organization Name (eg, company) []: MiEmpresa   # Nombre de tu empresa o sitio.
 - Organizational Unit Name (eg, section) []: IT   # Área de tu organización, puede ser "IT" o "Desarrollo".
 - Common Name (e.g. server FQDN or YOUR name) []: tu-dominio.com   # Escribe el dominio o la IP pública del servidor.
 - Email Address []: admin@tu-dominio.com   # Correo de contacto.

# Puedes dejar en blanco los campos opcionales y presionar "Enter" para continuar.

# 4. Configurar Apache para usar el certificado
sudo nano /etc/httpd/conf.d/ssl.conf

# Si el archivo no existe, créalo con:
sudo touch /etc/httpd/conf.d/ssl.conf

# Busca y modifica estas líneas:
SSLCertificateFile /etc/pki/tls/certs/laravel-cert.pem

SSLCertificateKeyFile /etc/pki/tls/private/laravel-key.pem

# 5. Reiniciar Apache para aplicar cambios
sudo systemctl restart httpd

## Acceder a Laravel
# Abre en tu navegador: http://tu-ip-publica (para HTTP) o https://tu-ip-publica (para HTTPS)

Créditos originales a "richie20502".
