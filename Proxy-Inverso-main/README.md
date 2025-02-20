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

