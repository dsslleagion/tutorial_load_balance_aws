<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">

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
<code>#!/bin/bash
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh
</code>
</pre>

<ol start="3">
    <li>Abra as portas necessárias na máquina master no site do AWS, a porta 2377 e também 4500 com o protocolo TCP e a origem 0.0.0.0/0.</li>
</ol>

<ol start="4">
    <li>Faça um IP elástico para cada máquina, que permanecerá mesmo depois que a máquina reiniciar.</li>
</ol>

<ol start="5">
    <li>Nomeie a máquina master da seguinte forma:</li>
</ol>

<pre>
<code>hostnamectl set-hostname master</code>  
</pre>

<ol start="6">
    <li>Inicialize o Docker Swarm na máquina master:</li>
</ol>

<pre>
<code>docker swarm init</code>
</pre>

<h3>Máquinas do Balanceamento</h3>

<ol start="7">
    <li>Nomeie as máquinas para facilitar a visualização:</li>
</ol>

<pre>
<code># Para a primeira máquina do balanceamento
hostnamectl set-hostname node1

# Para a segunda máquina do balanceamento
hostnamectl set-hostname node2
</code>
</pre>

<ol start="8">
    <li>Nas máquinas do balanceamento, execute o comando gerado pelo <code>docker swarm init</code> da máquina master para ingressar no swarm:</li>
</ol>

<pre>
<code>docker swarm join --token SWMTKN-1-4htvk6294tbj63qniy4st5wwgws1b5q75qga69uk4fxr2vwzd4-131ssj1pwtg6gn0i62ptdakr3 172.31.39.216:2377</code>
</pre>

<ol start="9">
    <li>Utilize o seguinte comando na máquina master para verificar se todas as máquinas foram conectadas corretamente:</li>
</ol>

<pre>
<code>docker node ls</code>
</pre>

<ol start="10">
    <li>Este comando exibe o token de gerenciamento após a inicialização do Swarm, caso deseje adicionar mais máquinas:</li>
</ol>

<pre>
<code>docker swarm join-token manager</code>
</pre>

<h2>Passo 2: Configurar o Balanceamento de Carga com Nginx</h2>

<h3>Máquina Master</h3>

<ol>
    <li>Crie um arquivo <code>nginx.conf</code> no diretório pessoal da máquina master com o seguinte conteúdo:</li>
</ol>

<p>O primeiro IP deve ser o da máquina master.</p>

<pre>
<code>
http {
    upstream all {
        server 44.213.55.202:80; 
        server 3.234.200.81:80;
        server 44.217.22.118:80;
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
    <li>Crie um arquivo no diretório pessoal chamado <code>Dockerfile</code> com o seguinte conteúdo:</li>
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

<p><code>--restart always </code> para que o contêiner seja executado quando a máquina iniciar.</p>

<pre>
<code>
docker run --name proxy -d -p 4500:4500 proxy --restart always
</code>
</pre>

<h2>Passo 3: Configurar um Volume para o Site</h2>

<ol>
    <li>Crie um volume Docker:</li>
</ol>

<pre>
<code>docker volume create app</code>
</pre>

<ol start="2">
    <li>Crie um serviço para o site usando o volume criado:</li>
</ol>

<pre>
<code>docker service create --name site --replicas 10 -d -p 80:80 --mount type=volume,src=app,dst=/usr/local/apache2/htdocs httpd</code>
</pre>

<ol start="3">
    <li>Com o caminho que exibe onde está localizado o arquivo <code>index.html</code>, altere o HTML e coloque "Master".</li>
</ol>

<ol start="4">
    <li>Agora, nas máquinas do balanceamento, pegue o mesmo caminho e altere o arquivo <code>index.html</code> para facilitar a visualização da troca. No HTML, coloque "node 1" na primeira máquina e "node 2" na segunda.</li>
</ol>

<ol start="5">
    <li>Para visualizar, basta abrir um navegador e inserir o IP da máquina master seguido por <code>:4500</code>, que foi a porta definida.</li>
</ol>

</body>
</html>
