---
title: "(PT-BR) OffSec - FunboxEasyEnum"
classes: wide
header:
  teaser: /assets/images/FunboxEasyEnum/capa.png
  overlay_image: /assets/images/FunboxEasyEnum/capa.png
  overlay_filter: 0.5
ribbon: Red
excerpt: "Write-Up da máquina FunboxEasyEnum da OffSec."
description: "Write-Up da máquina FunboxEasyEnum da OffSec."
categories:
  - Write-Up
tags:
  - Unix-Easy
toc: false
toc_sticky: true
toc_label: "On This Blog"
toc_icon: "biohazard"
---

# Introdução
A exploração da máquina trata de um File Upload.

# Enumeração

Para começar o processo de enumeração foi utilizado o Ping para testar a conectividade com o host informado, além de descobrir o TTL da máquina.

``` bash
ping -c 5 192.168.223.132
```
![](/assets/images/FunboxEasyEnum/2.png){: .align-center}

Nesse caso a máquina já estava ativa e tinha TTL=61, possivelmente um sistema Linux.

Iniciando o processo de enumeração foi utilizado o Nmap para descobrir quais portas estavam abertas no host.

``` bash
nmap -v -Pn -n -sT 192.168.223.132 -p- --min-rate 5000
```
![](/assets/images/FunboxEasyEnum/1.png){: .align-center}

Foram identificadas as portas 22 e 80.

### Porta 22
```bash
nc -vn 192.168.223.132 22
```
![](/assets/images/FunboxEasyEnum/3.png){: .align-center}

``` bash
ssh root@192.168.223.132
```
![](/assets/images/FunboxEasyEnum/4.png){: .align-center}

### Porta 80
```bash
whatweb -a 3 192.168.223.132
```
![](/assets/images/FunboxEasyEnum/5.png){: .align-center}

Abrindo o endereço 192.168.223.132, encontrada tela default do Apache.

![](/assets/images/FunboxEasyEnum/6.png){: .align-center}

O código fonte da página também foi checado e não apresentou nada relevante.

### Fuzzing

Continuando com a enumeração, foi feito Fuzzing para descobrir possíveis diretórios acessíveis.

``` bash
gobuster dir -u http://192.168.223.132 -w /usr/share/wordlists/seclists/Discovery/Web-Content/common.txt -x php -n -e -t 50
```
![](/assets/images/FunboxEasyEnum/8.png){: .align-center}

Entrando nos diretórios encontrados.

- /robots.txt
![](/assets/images/FunboxEasyEnum/10.png){: .align-center}
- /phpmyadmin
![](/assets/images/FunboxEasyEnum/11.png){: .align-center}
- /mini.php
![](/assets/images/FunboxEasyEnum/12.png){: .align-center}

## Reverse Shell

O diretório mini.php apresentou um sistema de upload de arquivos. Dessa forma, foi configurado um "shell.php" para uma possível reverse shell.

![](/assets/images/FunboxEasyEnum/14.png){: .align-center}

![](/assets/images/FunboxEasyEnum/15.png){: .align-center}

Preparada a conexão com nc.

![](/assets/images/FunboxEasyEnum/16.png){: .align-center}

Executando o arquivo.

![](/assets/images/FunboxEasyEnum/17.png){: .align-center}

Obtido acesso ao servidor interno.

Primeiramente, upgrade na shell com Python.
``` python
python3 -c "__import__('pty').spawn('/bin/bash')"
```
![](/assets/images/FunboxEasyEnum/18.png){: .align-center}

Navegando nas pastas do servidor não foi possível encontrar a flag nos diretórios de usuários.

Porém, procurando dentro da pasta do web server foi possível capturar a primeira flag.
![](/assets/images/FunboxEasyEnum/20.png){: .align-center}

## Enumerando o sistema

``` bash
cat /etc/passwd
```
![](/assets/images/FunboxEasyEnum/19.png){: .align-center}

Foi possível observar que o usuário Oracle tinha um hash possível.

Utilizado o hash-identifier para descobrir o tipo de hash.
![](/assets/images/FunboxEasyEnum/21.png){: .align-center}

Usando o john para quebrar o hash.
![](/assets/images/FunboxEasyEnum/22.png){: .align-center}

Logando no usuário oracle.
![](/assets/images/FunboxEasyEnum/23.png){: .align-center}

Não foi possível listar as permissões de sudo.
![](/assets/images/FunboxEasyEnum/24.png){: .align-center}

## PrivEsc

Dentro da pasta do usuário oracle também não foi possível encontrar nada relevante.

Tentando autenticar com credenciais default nos usuários presentes, foi encontrado o goat:goat.
```
su goat
Password: goat
```
![](/assets/images/FunboxEasyEnum/26.png){: .align-center}

Listando as permissões de sudo foi possível encontrar o /usr/bin/mysql.

``` bash
sudo -l 
```
![](/assets/images/FunboxEasyEnum/27.png){: .align-center}

Pesquisando no GTFOBins.
![](/assets/images/FunboxEasyEnum/28.png){: .align-center}

Executando o comando.
![](/assets/images/FunboxEasyEnum/29.png){: .align-center}

Procurando a última flag.
![](/assets/images/FunboxEasyEnum/30.png){: .align-center}


