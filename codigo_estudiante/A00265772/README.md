
### TALLER
**Universidad ICESI**  
**Curso:** Sistemas Distribuidos  

****
Estudiante | Código
--- | --- | ---
Dylan Torres | A00265772
****

#PASOS DE CREACIÓN Y DESPLIEGUE DEL MIRROR

Creamos una máquina virtual por medio de Vagrant la cual sera nuestro mirror. Nuestro Vagrantfile tendra la siguiente configuración:

```
# -*- mode: ruby -*-
# vi: set ft=ruby :

VAGRANTFILE_API_VERSION = "2"

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
  config.ssh.insert_key = false
  config.vm.define :mirror do |mir|
    mir.vm.box = "trusty"
    mir.vm.network "private_network", ip: "192.168.33.15"
    mir.vm.network "public_network", bridge: "enp5s0", ip: "192.168.131.93"
    mir.vm.provider :virtualbox do |vb|
      vb.customize ["modifyvm", :id, "--memory", "1024","--cpus", "1", "--name", "mirror" ]
    end
  end
end
```

**Luego debemos ejecutar los siguientes comandos para crear nuestro mirror:**

Primero Generamos la llave:

```
gpg --gen-key
```

Generamos una entropía para agilizar la creación de la llave:

```
cat /dev/urandom
```

Importamos el keyring:

```
gpg --no-default-keyring --keyring /usr/share/keyrings/ubuntu-archive-keyring.gpg --export | gpg --no-default-keyring --keyring trustedkeys.gpg --import
```

Para ejecutar el siguiente comando es necesario tener instalado aptly. Para ello seguimos los pasos de instalación de la siguiente fuente:

```
https://www.aptly.info/download/
```

Selección de los paquetes que se instalaran (postgresql) y luego procedemos a actualizar el mirror:

```
aptly mirror create -architectures=amd64 -filter='Priority (required) | Priority (important) | Priority (standard) | postgresql' -filter-with-deps xenial-main-postgresql http://mirror.upb.edu.co/ubuntu/ xenial main
```

```
aptly mirror update xenial-main-postgresql
```

Creamos un snaphost y luego lo publicamos e ingresamos una passphrase:

```
aptly snapshot create xenial-snapshot-postgresql from mirror xenial-main-postgresql
```

```
aptly publish snapshot xenial-snapshot-postgresql
```

Luego exportamos la llave:

```
gpg --export --armor > my_key.pub
```

La copiamos por fuera del equipo mirror desde el equipo anfitrión

```
scp vagrant@ip_solo_anfitrion:/tmp/my_key.pub .

```









