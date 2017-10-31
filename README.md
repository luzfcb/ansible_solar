# Script Ansible para Deploy do Solar


Este script [Ansible](http://docs.ansible.com/) , irá baixar, instalar, configurar e deixar executando,
todas as dependencias necessárias para obter o SOLAR funcionando.


Ele é composto basicamente por 3 arquivos:

* `solar.yml`: playbook ansible para instalar o SOLAR em determinada maquina.
  Esse playbook irá instalar e/ou configurar os seguintes software:

  * configurar os repositorios do debian para apontar para http://mirror.unesp.br/linux/debian/
  * [motd](https://wiki.debian.org/motd): Mostra informações sobre espaço em disco, memoria e outras informações uteis, ao fazer login via ssh.
  * ntp: Instará e configurará o NTP, de modo a ele consultar os servidores do projeto [ntp.br](https://ntp.br/guia-linux-avancado.php).
  * [memcached](https://memcached.org/): Utilizado pelo SOLAR para fazer cache de algumas consultas em nivel de aplicação, e cache de paginas HTML.
  * [redis](https://redis.io/): Utilizada pelo SOLAR como broker para execução de tarefas assincronas em background via [Celery](http://docs.celeryproject.org/en/3.1/).
  * [wkhtmltopdf](https://wkhtmltopdf.org/): Utilizado pelo SOLAR para gerar os PDF's apartir de paginas HTML.
  * [uWSGI](https://uwsgi-docs.readthedocs.io/en/latest/): Servidor de Aplicação que implementa a especificação [WSGI](https://wsgi.readthedocs.io/en/latest/what.html), utilizado pelo SOLAR.
  * [nginx](http://nginx.org/): Servidor WEB utilizado para servir arquivos estáticos e repassar a requisição para o servidor de aplicação.
  * [supervisor](http://supervisord.org/): Gerenciador de processos, utilizado iniciar o uWSGI e controlar as instancias da aplicação.
  * [pyenv](https://github.com/pyenv/pyenv): Utilizado para baixar, compilar e instalar o interpretador Python e gerenciar multiplas instalações do interpretador Python.
  * SOLAR: Clonar o projeto, baixar/instalar/configurar as dependencias do sistema operacional, baixar/instalar/configurar as dependencias Python, criar as pastas e arquivos de log, coletar os arquivos estáticos a ser disponibilizados pelos nginx, executar as migrações de schema no base de dados.


* `postgresql.yml`: playbook ansible para instalar o Postgresql 9.6 em determinada maquina, e criar um banco de dados vazio para posterior utilização do SOLAR - AINDA NÃO ESTÁ 100% funcional


* `parametros.yml`: Contem as variaveis obrigatórias, que devem ser preenchidas, e que serão utilizadas ao executar `solar.yml` e/ou `postgresql.yml`



Requisitos para utilização deste script:


Você deve executar esse script em uma maquina Linux qualquer (de preferencia Ubuntu). Não deve ser executado na mesma maquina onde ficará o servidor do Solar.:

#### No Ubuntu 16.04:




```
sudo apt-get update
sudo apt-get install software-properties-common --yes
sudo apt-add-repository ppa:ansible/ansible --yes
sudo apt-get update
sudo apt-get install ansible sshpass python-pip git --yes
sudo -H pip install jmespath
```


```
git clone https://gitlab.defensoria.to.gov.br/defensoria/solar_ansible.git
cd solar_ansible
```



Feito isto, você vai precisar instalar duas maquinas com [Debian 9.2 64bits ou superior](https://cdimage.debian.org/debian-cd/current/amd64/iso-dvd/debian-9.2.1-amd64-DVD-1.iso):


* Maquina 1 será onde ficará a aplicação.

* Maquina 2 será onde ficará o banco de dados. (opcional, você pode usar uma base de dados com Postgresql 9.5 ou superior)



### Instalação Maquina 1 ( requisito para utilização do `solar.yml` ):

Na maquina 1, deverá ser configurada 1 usuario comum linux, e o usuario root deve ter senha configurada.
A maquina 1 deverá ter acesso a internet.


### Instalação Maquina 2 (base de dados)  ( requisito para utilização do `postgresql.yml` ):

Os mesmos requisitos são necessários para a maquina 2.
> a automatização de instalação dessa opção ainda não está 100% funcional.
  Por favor, faça a instalação manual do postgresql 9.5 ou superior.
  Lembre-se de liberar criar manualmente a base de dados `db_solar`, de configurar um usuario e senha com acesso total a essa base de dados
  e de liberar no `pg_hba.conf` o endereço ip da maquina 1 com autenticação utilizando o metodo `md5`.




###

Após já ter esses dados, modifique o arquivo `parametros.yml` e preencha os valores solicitados


Após preencher os dados, execute:

```
ansible-playbook solar.yml -v
```

Aguarde a conclusão da instalação.




