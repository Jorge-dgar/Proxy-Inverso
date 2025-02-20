#Primero se muestra el README de la práctica usando vagrantfile y seguidamente el README realizada con Docker.

# Práctica de Proxy Inverso y Cabeceras

## Descripción
Esta práctica consiste en la configuración de un entorno con Vagrant que despliega dos máquinas virtuales en Debian Bullseye 64: un servidor proxy inverso y un servidor web. Se utilizan Nginx y cabeceras personalizadas para gestionar las peticiones y redirigirlas correctamente.

## Estructura del Proyecto

- `Vagrantfile`: Configura y provisiona las máquinas virtuales.
- `proxy_default/`: Contiene la configuración del proxy inverso.
  - `default`: Archivo de configuración de Nginx para el proxy inverso.
  - `hosts`: Archivo de hosts para la resolución de nombres.
- `web_default/`: Contiene la configuración del servidor web.
  - `default`: Archivo de configuración de Nginx para el servidor web.
  - `index.html`: Página web de prueba.
- Imágenes de comprobación: Capturas de pantalla de pruebas realizadas.

## Configuración de las Máquinas Virtuales

### Vagrantfile

```ruby
Vagrant.configure("2") do |config| 
  config.vm.box = "debian/bullseye64"

  config.vm.provider "virtualbox" do |vb| 
    vb.memory = "256" 
  end

  config.vm.provision "shell", inline: <<-SHELL 
    sudo apt-get update && apt-get install -y nginx 
    sudo apt install -y curl 
  SHELL

  config.vm.define "proxy" do |proxy| 
    proxy.vm.hostname = "www.example.test" 
    proxy.vm.network "private_network", ip: "192.168.57.10"

    proxy.vm.provision "shell", inline: <<-SHELL 
      sudo cp /vagrant/proxy_default/default /etc/nginx/sites-available/default 
      sudo cp /vagrant/proxy_default/hosts /etc/hosts 
    SHELL 
  end

  config.vm.define "web" do |web| 
    web.vm.hostname = "w1.example.test" 
    web.vm.network "private_network", ip: "192.168.57.11"

    web.vm.provision "shell", inline: <<-SHELL 
      sudo apt-get update && apt-get install -y nginx 
      sudo apt install -y curl 
      sudo cp /vagrant/web_default/index.html /var/www/html/index.html 
      sudo cp /vagrant/web_default/default /etc/nginx/sites-available/default 
    SHELL 
  end 
end
```

### Proxy Inverso (proxy_default/default)

```nginx
server { 
    listen 80; 
    listen [::]:80;
    
    server_name example.test www.example.test;
    
    location / {
        proxy_pass http://192.168.57.11:8080;
        add_header X-friend jorge;
    }
}
```

### Servidor Web (web_default/default)

```nginx
server { 
    listen 8080; 
    listen [::]:8080;

    server_name w1; 
    root /var/www/html; 
    index index.html index.html;

    location / { 
        add_header Host w1.example.test; 
        try_files $uri $uri/ =404; 
    } 
}
```

### Archivo index.html (web_default/index.html)

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>example.test</title>
</head>
<body>
    <h1>example.test</h1>
    <h2>Bienvenido</h2>
    <p>Servidor w1</p>
</body>
</html>
```

## Proceso de Instalación y Configuración
1. Instalar [Vagrant](https://www.vagrantup.com/) y [VirtualBox](https://www.virtualbox.org/).
2. Clonar este repositorio.
3. En la terminal, ejecutar:
   ```sh
   vagrant up
   ```
4. Para acceder a la máquina proxy:
   ```sh
   vagrant ssh proxy
   ```
5. Para acceder a la máquina web:
   ```sh
   vagrant ssh web
   ```

## Comprobaciones Realizadas
- **Comprobaciones de peticiones de proxy**: Se verifica que las peticiones pasan a través del proxy inverso.

  ![comprobaciones de peticiones con proxy](https://github.com/user-attachments/assets/eb65c3c0-fc9f-4b5e-9472-0c084ecc1297)

- **Comprobación accedemos a la página desde web**: Se accede directamente al servidor web.
![comprobación accedemos a la página desde web](https://github.com/user-attachments/assets/54778ab5-6544-44af-800c-e8946b5aac37)

  
- **Accedemos también desde la IP 192.168.57.10**: Se confirma el acceso vía proxy.
![accedemos también desde la ip 192 168 57 10](https://github.com/user-attachments/assets/a28ef644-d5b2-4f71-b260-24e7e9dda54f)

  
- **Añadimos la línea para que salga nuestro nombre en la cabecera de proxy**: Se verifica la cabecera personalizada `X-friend: jorge`.

  ![añadimos la línea para que salga nuestro nombre en la cabecera](https://github.com/user-attachments/assets/80ef22d2-59b6-4351-bb5c-9f685c04465e)

- **El proxy nos redirige al servidor web para que escuche en el puerto 8080**: Se confirma la redirección correcta.

  ![el proxy nos redirige al servidor web para que escuche en el puerto 8080](https://github.com/user-attachments/assets/1d6eba76-1f95-4ab1-b2f9-a1561475a605)

- **Añadimos la cabecera de web**: Se verifica que el servidor web tiene la cabecera `Host: w1.example.test`.

  ![añadimos la cabecera en web](https://github.com/user-attachments/assets/6f5ab7af-c97d-44a9-8944-7293ccc7a5c1)

- **Comprobaciones de las peticiones con web**: Se analiza la respuesta del servidor web.

  ![comprobaciones de las peticiones con web](https://github.com/user-attachments/assets/d47cd0b2-a67e-4271-95b0-96e0746d9e72)

- **IP bien en 200 HTML**: Se verifica el código de estado 200.

  ![ip bien 200](https://github.com/user-attachments/assets/5f2369af-bdb8-4b96-a2bf-f74b6c2a7693)

- **Desactivamos la caché**: Se eliminan respuestas almacenadas para obtener datos actualizados.

  ![desactivamos la cache](https://github.com/user-attachments/assets/32ef940d-3d85-4a1a-8380-ec170dc40105)

- **Nuestro nombre puesto en cabecera de web**: Se confirma la cabecera personalizada en la respuesta del servidor web.

  ![nombre puesto en cabecera del navegador](https://github.com/user-attachments/assets/7eb7196a-00e8-4c1d-b2b8-cd7177c513b3)

- **Cambiado en la cabecera de web**: Se prueba la modificación de cabeceras.

  ![cambiado en la cabecera de web](https://github.com/user-attachments/assets/5adf3771-96f7-4a7c-b8dd-826f69cbe794)


## Notas
- Si hay problemas con la red, reiniciar VirtualBox y Vagrant.
- Para modificar configuraciones, actualizar los archivos en `proxy_default/` y `web_default/` y ejecutar:
  ```sh
  vagrant reload --provision
  ```



**PRÁCTICA CON DOCKER


# Práctica de Proxy Inverso y Cabeceras con Docker

## Descripción
Esta práctica consiste en la configuración de un entorno con Docker, donde se implementa un proxy inverso y un servidor web utilizando contenedores. Se utilizan cabeceras personalizadas para gestionar las peticiones y demostrar la correcta redirección a través del proxy.

## Estructura del Proyecto

- `docker/` - Carpeta contenedora del entorno Docker
  - `proxy/` - Configuración del proxy inverso
    - `Dockerfile` - Archivo para construir la imagen del proxy
    - `default.conf` - Configuración de Nginx para el proxy
  - `web/` - Configuración del servidor web
    - `Dockerfile` - Archivo para construir la imagen del servidor web
    - `default.conf` - Configuración de Nginx para el servidor web
    - `index.html` - Página web de prueba
  - `docker-compose.yml` - Archivo para definir y gestionar los servicios

## Configuración de los Contenedores

### `proxy/Dockerfile`

```dockerfile
FROM nginx:latest
COPY default.conf /etc/nginx/conf.d/default.conf
```

### `proxy/default.conf`

```nginx
server {
    listen 80;
    server_name example.test www.example.test;
    
    location / {
        proxy_pass http://web:8080;
        add_header X-friend Jorge;
    }
}
```

### `web/Dockerfile`

```dockerfile
FROM nginx:latest
COPY default.conf /etc/nginx/conf.d/default.conf
COPY index.html /usr/share/nginx/html/index.html
```

### `web/default.conf`

```nginx
server {
    listen 8080;
    server_name w1;
    root /usr/share/nginx/html;
    index index.html;

    location / {
        add_header Host w1.example.test;
        try_files $uri $uri/ =404;
    }
}
```

### `web/index.html`

```html
<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <title>Servidor Web</title>
</head>
<body>
    <h1>Servidor Web</h1>
    <p>Este es el servidor web en Docker.</p>
</body>
</html>
```

### `docker-compose.yml`

```yaml
version: '3'
services:
  proxy:
    build: ./proxy
    ports:
      - "80:80"
    depends_on:
      - web
  web:
    build: ./web
    ports:
      - "8080:8080"
```

## Puesta en Marcha

1. Clonar este repositorio y moverse a la carpeta `docker/`:
   ```sh
   git clone https://github.com/Jorge-dgar/Proxy-Inverso.git
   cd Proxy-Inverso/docker
   ```

2. Construir y levantar los contenedores:
   ```sh
   docker-compose up -d --build
   ```

3. Verificar los contenedores en ejecución:
   ```sh
   docker ps
   ```

## Comprobaciones Realizadas

- **Creación de carpetas web y proxy**
  
![1- creamos las carpetas web y proxy en docker](https://github.com/user-attachments/assets/254b7661-f742-45e4-8f57-d61ea31fcfb7)


- **Configuración de Proxy**
  
![default conf de proxy](https://github.com/user-attachments/assets/b12c3a44-c212-4990-a507-cf02f380722d)


- **Dockerfile del Proxy**
  
![dockerfile de proxy](https://github.com/user-attachments/assets/8ac86449-ed03-44af-9811-0523d517dcb8)


- **Configuración del Servidor Web**

![default conf de web](https://github.com/user-attachments/assets/94f316ae-3ddb-4c27-9758-5c7fb6955f5d)

- **Dockerfile del Servidor Web**

![dockerfile de web](https://github.com/user-attachments/assets/54d675aa-6546-4eb0-9b6c-6df21c0fcfa0)

- **Index.html del Servidor Web**

![index html de web](https://github.com/user-attachments/assets/094766f3-4cfb-4b61-b8b1-556ed8f8a379)

- **Docker-Compose**

![docker-compose yml a](https://github.com/user-attachments/assets/813c4e39-51cd-44ac-bb81-69075a7cd6c9)

- **Arranque de contenedores**

![arrancamos los contenedores de docker](https://github.com/user-attachments/assets/f5748d56-86e1-439c-b3f6-dc601eb1e189)

- **Cabecera funcionando (X-friend: Jorge)**

![cabecera instalada y funcionando con nombre Jorge](https://github.com/user-attachments/assets/4020fcf7-c424-4a63-ba98-ed16792d5719)

- **Servidor Web activo en Docker**

![servidor web de docker funcionando](https://github.com/user-attachments/assets/44efee28-1f20-4bac-95ac-f8778f591f77)

- **Últimos logs del Proxy**

![ultimos 20 logs de proxy](https://github.com/user-attachments/assets/ad2d9a1f-eaeb-4201-b363-685687b5f932)

- **Últimos 20 logs del Servidor Web**

![ultimos 20 logs de web](https://github.com/user-attachments/assets/985c75df-e62e-47ca-8cbc-082c36e1fde4)

- **Funcionamiento correcto del Servidor Web**
  
![funcionamiento correcto del servidor web](https://github.com/user-attachments/assets/8da098c9-5782-4d81-a5dd-3bab82e9e90f)

## Notas
- Si hay problemas con los puertos en uso, detener contenedores en conflicto con:
  ```sh
  docker stop $(docker ps -q)
  ```
- Para ver los logs de proxy:
  ```sh
  docker logs $(docker ps -q --filter "name=proxy")
  ```
- Para ver los logs del servidor web:
  ```sh
  docker logs $(docker ps -q --filter "name=web")
  ```
