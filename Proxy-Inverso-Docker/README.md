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

- **Creación de carpetas web y proxy** ![Creación de carpetas](imagenes/creamos-las-carpetas.png)
![1- creamos las carpetas web y proxy en docker](https://github.com/user-attachments/assets/254b7661-f742-45e4-8f57-d61ea31fcfb7)


- **Configuración de Proxy** ![Default.conf Proxy](imagenes/default-conf-proxy.png)
![default conf de proxy](https://github.com/user-attachments/assets/b12c3a44-c212-4990-a507-cf02f380722d)


- **Dockerfile del Proxy** ![Dockerfile Proxy](imagenes/dockerfile-proxy.png)
![dockerfile de proxy](https://github.com/user-attachments/assets/8ac86449-ed03-44af-9811-0523d517dcb8)


- **Configuración del Servidor Web** ![Default.conf Web](imagenes/default-conf-web.png)

![default conf de web](https://github.com/user-attachments/assets/94f316ae-3ddb-4c27-9758-5c7fb6955f5d)

- **Dockerfile del Servidor Web** ![Dockerfile Web](imagenes/dockerfile-web.png)

![dockerfile de web](https://github.com/user-attachments/assets/54d675aa-6546-4eb0-9b6c-6df21c0fcfa0)

- **Index.html del Servidor Web** ![Index.html](imagenes/index-html-web.png)

![index html de web](https://github.com/user-attachments/assets/094766f3-4cfb-4b61-b8b1-556ed8f8a379)

- **Docker-Compose** ![Docker Compose](imagenes/docker-compose-yml.png)

![docker-compose yml a](https://github.com/user-attachments/assets/813c4e39-51cd-44ac-bb81-69075a7cd6c9)

- **Arranque de contenedores** ![Arranque](imagenes/arrancamos-los-contenedores.png)

![arrancamos los contenedores de docker](https://github.com/user-attachments/assets/f5748d56-86e1-439c-b3f6-dc601eb1e189)

- **Cabecera funcionando (X-friend: Jorge)** ![Cabecera funcionando](imagenes/cabecera-funcionando.png)

![cabecera instalada y funcionando con nombre Jorge](https://github.com/user-attachments/assets/4020fcf7-c424-4a63-ba98-ed16792d5719)

- **Servidor Web activo en Docker** ![Servidor Web activo](imagenes/servidor-web-docker.png)

![servidor web de docker funcionando](https://github.com/user-attachments/assets/44efee28-1f20-4bac-95ac-f8778f591f77)

- **Últimos logs del Proxy** ![Logs Proxy](imagenes/logs-proxy.png)

![ultimos 20 logs de proxy](https://github.com/user-attachments/assets/ad2d9a1f-eaeb-4201-b363-685687b5f932)

- **Últimos 20 logs del Servidor Web** ![Logs Web](imagenes/logs-web.png)

![ultimos 20 logs de web](https://github.com/user-attachments/assets/985c75df-e62e-47ca-8cbc-082c36e1fde4)

- **Funcionamiento correcto del Servidor Web** ![Funcionamiento correcto](imagenes/funcionamiento-correcto.png)
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
