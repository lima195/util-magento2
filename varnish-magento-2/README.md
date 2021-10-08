# VARNISH - MAGENTO 2

Olá, nesse repositório mostrarei como instalar e configurar o varnish no EC2 da AWS para Magento 2.
https://devdocs.magento.com/guides/v2.3/config-guide/varnish/use-varnish-cache.html

# Arquivos

Arquivos de configuração para `centos 7 na AWS - EC2`
versão: `varnish-4.0.5`
```
├── centos
│   ├── etc
│   │   └── varnish
│   │       ├── default.vcl
│   │       └── varnish.params
│   ├── info
│   ├── lib
│   │   └── systemd
│   │       └── system
│   │           └── varnish.service
│   └── nginx.conf
```

Arquivos de configuração para `ubuntu` na Digital Ocean
versão: `varnish-5.2.1`
```
├── ubuntu
│   ├── etc
│   │   ├── default
│   │   │   └── varnish
│   │   └── varnish
│   │       └── default.vcl
│   ├── info
│   ├── lib
│   │   └── systemd
│   │       └── system
│   │           └── varnish.service
│   ├── magento.conf
│   ├── nginx.conf
│   ├── nginx-withoutvarnish-example.conf
│   └── usr
│       └── share
│           └── varnish
└───────── varnishreload
```

## Instalação (Centos - AWS)

1 - atualiza os pacotes
```
yum -y update;
```

2 - Instala o varnish
```
yum -y install varnish;
```

3 - Confere a versão do varnish
```
varnishd -V;
```

deve vir algo como:
> varnishd (varnish-4.0.5 revision 07eff4c29)

> Copyright (c) 2006 Verdens Gang AS

> Copyright (c) 2006-2014 Varnish Software AS

4 - Vá até o nginx e faça backup do arquivo de configuração

5 - Mude a porta do nginx de 80 para 8080 no arquivo principal do nginx e depois no do site (faça backup)

6 - Edite o arquivo do nginx para ficar nesse padrão:
(Lembrando que a ordem de cada linha pode ser importante)

```
upstream fastcgi_backend {
    server  unix:/var/run/magento.sock;
}

server {
    listen 8080;
    server_name my-website.com;

    set $MAGE_ROOT /opt/magento/www;
    include /opt/magento/www/nginx.conf.sample;
}

server {
    server_name my-website.com;
    
    listen 443 ssl http2;
    listen [::]:443 ssl http2;
    ssl_protocols TLSv1.2;

    ssl_certificate /etc/letsencrypt/live/my-website.com/fullchain.pem; # managed by Certbot
    ssl_certificate_key /etc/letsencrypt/live/my-website.com/privkey.pem; # managed by Certbot
    include /etc/letsencrypt/options-ssl-nginx.conf; # managed by Certbot
    ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem; # managed by Certbot

    proxy_max_temp_file_size 0;
    keepalive_timeout 300s;

    location / {
        proxy_pass http://127.0.0.1:80;
        proxy_set_header Host $http_host;
        proxy_set_header X-Forwarded-Host $http_host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header Ssl-Offloaded "1";
        proxy_set_header      X-Forwarded-Proto https;
        proxy_set_header      X-Forwarded-Port 443;
        proxy_redirect http://my-website.com:8080/ /;
        proxy_http_version 1.1;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

7 - Desliga o nginx e liga o varnish
```
systemctl stop nginx
systemctl enable varnish
systemctl start varnish
systemctl status varnish
```
8 - Substitua os arquivos desse repositório para o servidor:
(Que estão na pasta centos desse repositório)

> /etc/varnish/default.vcl
https://github.com/lima195/varnish-magento-2/blob/master/centos/etc/varnish/default.vcl

> /etc/varnish/varnish.params
https://github.com/lima195/varnish-magento-2/blob/master/centos/etc/varnish/varnish.params

> /lib/systemd/system/varnish.service
https://github.com/lima195/varnish-magento-2/blob/master/centos/lib/systemd/system/varnish.service

9 - Altere o arquivo `/etc/varnish/default.vcl` para colocar o ip e domínio da loja.
(Esse passo é importante porque se não fizer isso, o varnish não aceitará o request do tipo `PURGE` vindo do magento que serve para limpar o cache sincronizadamente com o magento)
Insira o domínio em `Host`:
```
probe site_probe {
     .request =
          "HEAD /health_check.php HTTP/1.1"
          "Host: my-website.com"
          "Connection: close";
          .interval  = 5s; # check the health of each backend every 5 seconds
          .timeout   = 3s;  # timing out after 1 second by default.
          .window    = 5;  # If 3 out of the last 5 polls succeeded the backend is considered healthy, otherwise it will be marked as sick
      .threshold = 3;
}
```
E insira o domínio e ip público e privado em:
```
acl purge {
    "127.0.0.1";
    "my-website.com";
}
```
 

10 - Reinicie o varnish

```
systemctl stop varnish
systemctl start varnish
systemctl daemon-reload # Esse comando é necessário quando faz alteração nos arquivos do varnish
systemctl stop varnish
systemctl start varnish
```

11 - Confere se está na porta correta:

```
ss -tlpn| grep varnishd
```

Deve retornar algo assim:

```
  LISTEN   0         128                 0.0.0.0:80               0.0.0.0:*        users:(("varnishd",pid=27233,fd=6))                                            
  LISTEN   0         10                127.0.0.1:6082             0.0.0.0:*        users:(("varnishd",pid=27231,fd=5))                                            
  LISTEN   0         128                    [::]:80                  [::]:*        users:(("varnishd",pid=27233,fd=7)) 
```

12 - Restarta todos os serviços:

```
service varnish reload; 
service php71-php-fpm reload; 
service nginx reload;
```

13 - Vá até a pasta do magento com o usuário do magento:
(Esses comandos habilitam o varnish no magento)
```
bin/magento config:set system/full_page_cache/caching_application 2;
bin/magento config:set system/full_page_cache/varnish/access_list localhost;
bin/magento config:set system/full_page_cache/varnish/backend_host localhost;
bin/magento config:set system/full_page_cache/varnish/backend_port 8080;
bin/magento setup:config:set --http-cache-hosts=127.0.0.1:80;
bin/magento c:c;
```


## Desligando/Desinstalando o Varnish
1 - Desligue o varnish
```
service varnish stop; 
```

2 - Volte os backups do nginx

3 - Vá para o usuário e a pasta do magento e desative a configuração do varnish

```
bin/magento config:set system/full_page_cache/caching_application 2;
bin/magento c:c;
```

4 - Restarte os serviços:
```
service php71-php-fpm reload; 
service nginx reload;
```

## Possíveis problemas e soluções

1 - Erro 502 em algumas ocasiões como muitas alterações no page builder
* Para limpar esse cache de 502 é necessário restartar o php fpm
* Para impedir que isso aconteça, é necessário aumentar os valores do header, ex:
```
# arquivo (/lib/systemd/system/varnish.service)

...

-p http_resp_size=512k \  
-p workspace_backend=256k \
```

2 - Renovação via certbot
* O certbot dá erro ao tentar renovar o ssl, é preciso desligar o varnish antes de atualizar o ssl e liga-lo novamente
```
service varnish stop;
# comando do certbot ...
service varnish start;
service php71-php-fpm reload; 
service nginx reload;
```

3 - inconsistência de cache ou erros 502
* É possível que não tenha configurado corretamente no passo 9.
* Também é possível que algum firewall esteja bloqueando requests do tipo `PURGE` como a sucuri faz por padrão por exemplo. Esse request precisa ser aceito pelo varnish para que ele faça a limpeza do cache e pra isso existe um whitelist no arquivo .vcl
 

## Como testar
Na documentação oficial do Magento tem detalhado como testar, mas é preciso fazer isso no modo developer.
https://devdocs.magento.com/guides/v2.3/config-guide/varnish/use-varnish-cache.html


---

## Instalação (Ubuntu)

A desenvolver, mas é praticamente a mesma coisa, só muda uns arquivos de lugar e os comandos do yum que vira apt kakakaka
