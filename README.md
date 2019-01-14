# Jenkins en Ubuntu

[[TOC]]

## Inicializar el Vagrantfile con para ubuntu/trusty64
``` bash
$ vagrant init ubuntu/trusty64 -f -m
```

## Lanar la máquina

``` bash
$ vagrant up
```

## Acceder a la máquina vía ssh
``` bash
$ vagrant ssh
```
## Ejecutar comando dentro de la máquina virtual
``` bash
$ vagrant ssh -c "curl http://localhost:8080"
```
Éste ejemplo ejecuta el comando **curl** dentro de la máquina virtual

## Ver si java está instalado
``` bash
$ type -p java
```

## Mostar las actualizaciones de un archivo
``` bash
$ vagrant ssh -c "tail -f /var/log/jenkins/jenkins.log"
```

## Estructura de Directorio para los Roles

```
site.yml
webservers.yml
fooservers.yml
roles/
   common/
     tasks/
     handlers/
     files/
     templates/
     vars/
     defaults/
     meta/
   webservers/
     tasks/
     defaults/
     meta/
```
[Documentación oficial](https://docs.ansible.com/ansible/latest/user_guide/playbooks_reuse_roles.html)

## Trabajar con Roles

Para trabajar con Roles, se recomienda la estructura que se muestra a continuación

``` bash
.
├── provision
│   ├── install.yml
│   └── roles
│       ├── common
│       │   ├── defaults
│       │   │   └── main.yml
│       │   └── tasks
│       │       └── main.yml
│       ├── java
│       │   ├── defaults
│       │   │   └── main.yml
│       │   └── tasks
│       │       └── main.yml
│       ├── jenkins
│       │   ├── defaults
│       │   │   └── main.yml
│       │   ├── metas
│       │   │   └── main.yml
│       │   ├── tasks
│       │   │   └── main.yml
│       │   └── templates
│       │       └── main.yml
│       └── nginx
│           ├── defaults
│           │   └── main.yml
│           └── tasks
│               └── main.yml
├── README.md
└── Vagrantfile

```

## ./provision/install.yml

En este archivo se indican todos los roles que están disponibles para ser utilizados

``` yml
# ./provision/install.yml
---
- hosts: all

  roles:
    - { role: common, tags: common }
    - { role: java, tags: java }
    - { role: jenkins, tags: jenkins }
    - { role: nginx, tags: nginx }
```

##Depenencias
Las dependencias se establecen en el archivo ``metas/main.yml`` del proyecto que depende de otro, por ejemplo

En Vagrantfile se debe listar los tags asignados a los Roles que se desea utilizar para el servidor en cuestion.

## Templates

Los templates son archivos que se pueden utilizar para generar más de un archvivos nuevo en el servidor reemplazando la o las variables que se indican.

Los templates están escritos utilizando el formato de archivo ***jinja2 (.j2)*** cuya extensión debe ser ....

```jinja2
upstream app_server {
    server 127.0.0.1:{{ port_number }} fail_timeout=0;
}
 
server {
    listen 80;
    listen [::]:80 default ipv6only=on;
    server_name {{ site_name }};
 
    location / {
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header Host $http_host;
        proxy_redirect off;
 
        if (!-f $request_filename) {
            proxy_pass http://app_server;
            break;
        }
    }
}

```

