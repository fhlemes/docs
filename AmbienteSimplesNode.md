## Preparando Ambiente NodeJS Simples para Deploy

Lembrando que, estou logado como root no servidor, por este motivo não utilizei o sudo.


### Dependências

- OS Ubuntu 16.x ou Superior
- Nginx
- Nvm
- Pm2
- Editor Nano (pico)
- Certbot (SSL)
  
<br />

#### Atualizando ambiente

```apt update```

```apt upgrade -y ```

<br />

#### Instalando o editor de textos "nano"

```apt install nano -y ```

<br />

#### Instalando o servidor Web Proxy Nginx

```apt install nginx -y```

```service nginx start```

<br />

#### Instalando o gerenciador de versões do Node o "nvm"

```curl -o- https://raw.githubusercontent.com/creationix/nvm/v0.33.11/install.sh | bash```

```source ~/.bashrc```

```nvm -v```

<br />

#### Instalando uma versão do Node com o NVM

```nvm install v10.13.0```

```npm -v```

```node -v```

<br />

#### Instalando o PM2

```npm i -g pm2```

```pm2 list```

<br />

#### Instalando o Certbot (Let’s Encrypt)

```apt-get install software-properties-common```

```add-apt-repository ppa:certbot/certbot```

```apt-get update```

```apt-get install python-certbot-nginx```

<br />

### Fazendo o deploy de uma aplicação simples

*Abaixo é apenas um exemplo*


--- Acesse o diretório de sua conta e faça o clone da sua aplicação, no meu caso eu uso o git


```git clone https://github.com/fhlemes/docs.git```


--- Após realizar o clone do app, acesse o diretório que foi clonado e rode o comando abaixo


```npm install```


--- Instalado as depêndencias do projeto, iremos utilizar o pm2 para gerenciar o deploy da aplicação, vamos rodar comando


```pm2 --name app start index.js```


--- Agora vamos a parte legal, que é do Nginx. Navegue até o diretório ***/nginx/sites-enabled/*** e crie um arquivo que irá
ter as configurações de sua aplicação


```cd /etc/nginx/sites-enabled/``` [aperte enter]


```touch meuapp```


```nano meuapp```


--- Com o arquivo ***meuapp** aberto, vou utilizar o seguinte código abaixo. (_Lembrando que eu ajustei e reduzi o código, em outro estará diferente_)

```
server {

  server_name api.exemplo.com.br;

  root /var/www/html/exemplo.com.br/gitclonado;

  location / {
    proxy_pass http://localhost:3000;
    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection 'upgrade';
    proxy_set_header Host $host;
    proxy_cache_bypass $http_upgrade;
  }

}
```


--- Feito os ajustes no arquivo de configuração, aperte a tecla ***CTRL + O*** para salvar e ***CTRL + W*** para sair.


--- Agora vamos validar se o arquivo criado não possui erros, para isso, rode o comando


```nginx -t```


[ _nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful_ ]

--- Estando tudo certo e não apresentado erros, rode o comando para o Nginx salvar as configurações e reinicar


```nginx -s reload```


--- Agora vamos instalar o certificado SSL, para isso o rode o comando abaixo


```certbot -d api.exemplo.com.br```


[Irá ser solicitado algumas informações, como email, pais, e se você deseja realizar o direcionamento automático para https ]


--- Instalado o certificado SSL, o seu arquivo de configuração do app, estará da seguinte forma


```
server {

  server_name api.exemplo.com.br;

  root /var/www/html/exemplo.com.br/gitclonado;

  location / {
    proxy_pass http://localhost:3000;
    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection 'upgrade';
    proxy_set_header Host $host;
    proxy_cache_bypass $http_upgrade;
  }

  listen 443 ssl; # managed by Certbot
  ssl_certificate /etc/letsencrypt/live/api.exemplo.com.br/fullchain.pem; # managed by Certbot
  ssl_certificate_key /etc/letsencrypt/live/api.exemplo.com.br/privkey.pem; # managed by Certbot
  include /etc/letsencrypt/options-ssl-nginx.conf; # managed by Certbot
  ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem; # managed by Certbot

}

server {

  if ($host = www.api.exemplo.com.br) {
      return 301 https://$host$request_uri;
  } # managed by Certbot

  if ($host = api.exemplo.com.br) {
      return 301 https://$host$request_uri;
  } # managed by Certbot

  server_name www.api.exemplo.com.br api.exemplo.com.br;
  listen 80;
  return 404; # managed by Certbot

}

```

<br />

#### Pronto agora temos o deploy de uma api em node com certificado SSL habilitado :D
