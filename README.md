<!DOCTYPE html>
<html>
<head>
   
</head>
<body>

<h1>Configuração do Docker Swarm e Balanceamento de Carga</h1>

<h2>Passo 1: Configurar o Docker Swarm</h2>

<h3>Máquina Master</h3>

<ol>
    <li>Crie 3 máquinas virtuais Ubuntu.</li>
    <li>Instale o Docker nas máquinas com o seguinte comando:</li>
</ol>

<pre>
<code>
#!/bin/bash
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh
</code>
</pre>

<ol start="3">
    <li>Abra as portas necessárias na máquina master:(AWS apenas configure no site)</li>
</ol>

<pre>
<code>
sudo ufw allow 2377/tcp
sudo ufw allow 4500/tcp
</code>
</pre>

<ol start="4">
    <li>Nomeie as máquinas da seguinte forma:</li>
</ol>

<pre>
<code>
hostnamectl set-hostname master # Para a máquina master
hostnamectl set-hostname node1  # Para a primeira máquina do balanceamento
hostnamectl set-hostname node2  # Para a segunda máquina do balanceamento
</code>
</pre>

<ol start="5">
    <li>Inicialize o Docker Swarm na máquina master:</li>
</ol>

<pre>
<code>
docker swarm init
</code>
</pre>

<h3>Máquinas do Balanceamento</h3>

<ol start="6">
    <li>Nas máquinas do balanceamento, execute o comando gerado pelo <code>docker swarm init</code> da máquina master
        para ingressar no swarm:</li>
</ol>

<pre>
<code>
docker swarm join --token SWMTKN-1-0x3gimqxscvxspokupwhutw8wspjujsud3hjd5s4wudmg1er0s-boaulopoluofrimd7mknovx87
172.31.34.172:2377
</code>
</pre>

<p> tenha noção que os ips variam porem a porta é recomendado ter um padrão</p>

<h2>Passo 2: Configurar o Balanceamento de Carga com Nginx</h2>

<h3>Máquina Master</h3>

<ol>
    <li>Crie um arquivo <code>nginx.conf</code> na máquina master com o seguinte conteúdo:</li>
</ol>

<pre>
<code>
http {
    upstream all {
        server 54.82.216.122:80; // IPs das máquinas do balanceamento
        server 54.234.46.213:80;
        server 35.175.214.92:80;
    }

    server {
        listen 4500;
        location / {
            proxy_pass http://all/;
        }
    }
}

events {}
</code>
</pre>

<ol start="2">
    <li>Crie um arquivo <code>Dockerfile</code>:</li>
</ol>

<pre>
<code>
FROM nginx
COPY nginx.conf /etc/nginx/nginx.conf
</code>
</pre>

<ol start="3">
    <li>Crie uma imagem Docker com o arquivo <code>Dockerfile</code>:</li>
</ol>

<pre>
<code>
docker build . -t proxy
</code>
</pre>

<ol start="4">
    <li>Execute o serviço de balanceamento de carga:</li>
</ol>

<pre>
<code>
docker run --name proxy -d -p 4500:4500 proxy
</code>
</pre>

<h2>Passo 3: Configurar um Volume para o Site</h2>

<ol>
    <li>Crie um volume Docker:</li>
</ol>

<pre>
<code>
docker volume create app
</code>
</pre>

<ol start="2">
    <li>Crie um serviço para o site usando o volume criado:</li>
</ol>

<pre>
<code>
docker service create --name site --replicas 10 -d -p 80:80 --mount type=volume,src=app,dst=/usr/local/apache2/htdocs httpd
</code>
</pre>

<ol start="2">
    <li>Comando que exibe aonde está localizado o arquivo index.html</li>
</ol>
<pre>
<code>
docker volume inspect app
</code>
</pre>

<h2>Passo 4: Nas Máquinas Subsequentes</h2>

<p>Execute o comando abaixo em cada máquina subsequente para ingressar no Docker Swarm:</p>

<pre>
<code>
docker swarm join --token SWMTKN-1-0x3gimqxscvxspokupwhutw8wspjujsud3hjd5s4wudmg1er0s-boaulopoluofrimd7mknovx87
172.31.34.172:2377
</code>
</pre>

<p>Agora o Docker Swarm está configurado com balanceamento de carga usando Nginx e um volume para o site.</p>

</body>
</html>
