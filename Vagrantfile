Vagrant.configure("2") do |config|
  config.vm.box = "debian/bullseye64"

  config.vm.provider "virtualbox" do |vb|
    vb.memory = "256"
  end

  # Configuración del Proxy Inverso (proxy)
  config.vm.define "proxy" do |proxy|
    proxy.vm.hostname = "www.example.test"
    proxy.vm.network "private_network", ip: "192.168.57.10"

    proxy.vm.provision "shell", inline: <<-SHELL
      # Instalar Nginx y herramientas necesarias
      sudo apt update -y
      sudo apt install -y nginx curl
      sudo apt update && sudo apt install -y curl


      # Configurar el archivo /etc/hosts dentro de la VM
      echo "192.168.57.11 w1.example.test" | sudo tee -a /etc/hosts

      # Configurar Nginx como proxy inverso
      cat <<EOF | sudo tee /etc/nginx/sites-available/default
      server {
          listen 80;
          listen [::]:80;

          server_name example.test www.example.test;

          location / {
              proxy_pass http://w1.example.test:8080;
              proxy_set_header Host \$host;
              proxy_set_header X-Real-IP \$remote_addr;
              proxy_set_header X-Forwarded-For \$proxy_add_x_forwarded_for;
          }
      }
      EOF

      # Habilitar el sitio y reiniciar Nginx
      sudo ln -sf /etc/nginx/sites-available/default /etc/nginx/sites-enabled/
      sudo systemctl restart nginx
    SHELL
  end

  # Configuración del Servidor Web (w1)
  config.vm.define "web" do |web|
    web.vm.hostname = "w1.example.test"
    web.vm.network "private_network", ip: "192.168.57.11"

    web.vm.provision "shell", inline: <<-SHELL
      # Instalar Nginx y curl
      sudo apt update -y
      sudo apt install -y nginx curl
      sudo apt update && sudo apt install -y curl


      # Configurar el archivo /etc/hosts dentro de la VM
      echo "192.168.57.10 www.example.test" | sudo tee -a /etc/hosts

      # Configurar Nginx para escuchar en el puerto 8080
      cat <<EOF | sudo tee /etc/nginx/sites-available/default
      server {
          listen 8080;
          listen [::]:8080;

          server_name w1.example.test;
          root /var/www/html;
          index index.html index.htm;

          location / {
              try_files \$uri \$uri/ =404;
          }
      }
      EOF

      # Habilitar el sitio en Nginx
      sudo ln -sf /etc/nginx/sites-available/default /etc/nginx/sites-enabled/

      # Crear la página web de prueba
      cat <<EOF | sudo tee /var/www/html/index.html
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
      EOF

      # Reiniciar Nginx para aplicar los cambios
      sudo systemctl restart nginx
    SHELL
  end

  # Agregar entradas en el archivo hosts del HOST automáticamente
  host_os = RbConfig::CONFIG["host_os"]

  if host_os =~ /linux|darwin/ # Linux o macOS
    config.vm.provision "shell", inline: <<-SHELL
      echo "192.168.57.10 www.example.test" | sudo tee -a /etc/hosts
    SHELL
  elsif host_os =~ /mswin|mingw|cygwin/ # Windows
    config.vm.provision "shell", inline: <<-SHELL
      powershell.exe -Command "Add-Content -Path C:\\Windows\\System32\\drivers\\etc\\hosts -Value '192.168.57.10 www.example.test'"
    SHELL
  end
end