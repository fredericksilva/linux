# Linux
Dicas úteis para quem como desenvolvedor e utiliza Linux. Estes exemplos foram utilizados e testados no Ubuntu, porém, nada impede que os mesmos funcionem em outras distribuições, porém, cada uma pode ter uma peculiaridade.

## LAMP
Instalando os pacotes necessários para o [LAMP](http://pt.wikipedia.org/wiki/LAMP):
```sh
aptitude install php5 apache2 mysql-server libapache2-mod-php5`
```

Ativando [mod_rewrite](http://cirofeitosa.com.br/post/urls-mod-rewrite/) do Apache.
```sh
a2enmod rewrite
```

Reiniciando o Apache para ativar as novas configurações:
```sh
invoke-rc.d apache2 restart
```

### XDebug
Primeiramente, devemos instalar a extensão:
```sh
aptitude install php-pear php5-xdebug
```

Devemos ter o caminho da extensão xdebug para inserir no arquivo de configuração do php.
```sh
updatedb && locate xdebug.so
```

Abra o arquivo de configuração **(/etc/php5/apache2/php.ini)** e acrescentar a linha:
```sh
zend_extension="/usr/lib/php5/20090626/xdebug.so"
```


Ainda no arquivo **php.ini**, devemos mudar a flag html_errors de "Off" para "On":
```sh
html_errors = On
```

Para validar as alterações feitas devemos reiniciar o Apache:
```sh
invoke-rc.d apache2 restart
```

## PHPUnit
Instalando o PHPUnit:
```sh
sudo apt-get install phpunit
sudo pear upgrade pear
sudo pear channel-discover pear.phpunit.de
sudo pear channel-discover components.ez.no
sudo pear channel-discover pear.symfony-project.com
sudo pear install --alldeps phpunit/PHPUnit
```

Para testar se tudo foi instalado corretamente, basta digitar o comando:

```sh
phpunit --version
```

## Git
Instalando o pacote referente:
```sh
aptitude install git
```

### Adicionando diretórios vazios no git
```sh
for i in $(find . -type d -regex ``./[^.].*'' -empty); do touch $i"/.gitignore"; done;
```

## Configurar interface de rede manualmente
O arquivo de configuração em distribuições baseadas no Debian é o */etc/network/interfaces*.

### Configurando interface de rede wireless via DHCP
<pre><code>auto wlan0 
iface wlan0 inet dhcp
	wpa-ssid [nome da rede]
	wpa-psk  [password da rede]</code></pre>

### Configurando interface de rede Ethernet com IP estático
<pre><code>auto eth0
iface eth0 inet static
	address 10.1.60.XXX
	netmask 255.255.0.0
	gateway 10.1.230.2
	network 10.1.0.0
	dns-nameservers 172.16.2.11 172.16.2.12</code></pre>

###### Testando as configurações
Devemos reiniciar o serviço de rede através do comando:
```sh
invoke-rc.d networking restart
```

Depois verificamos através do comando `ifconfig` se todo o procedimento foi feito de maneira correta.

    
## Compilar um novo kernel no Ubuntu
O diretório padrão para o armazenamento dos arquivos do kernel segundo o [FHS](http://pt.wikipedia.org/wiki/Filesystem_Hierarchy_Standard) é o */usr/src*.

Dentro deste diretório, basta efetuar o download da última versão disponível no site oficial [http://kernel.org/](http://kernel.org/).
```sh
wget http://www.kernel.org/pub/linux/kernel/v3.0/linux-3.4.6.tar.bz2
```

Para podermos utilizar o [menuconfig](http://en.wikipedia.org/wiki/Menuconfig), precisamos instalar sua biblioteca, a [Ncurses](http://en.wikipedia.org/wiki/Ncurses): 
```sh
aptitude install libncurses5-dev
```

Descompactamos o arquivo baixado utilizando o comando.
```sh
tar -xvjf linux-3.4.6
```

Devemos criar um link simbólico do diretorio criado linux-3.4.6 em /usr/src/ com o nome de linux onde, a estrutura ficará da seguinte maneira: 
* */usr/src/linux*

Isso se deve ao fato de que, novos programas ao serem instalados, consultam neste diretório para descobrir a versão atual do kernel.
```sh
ln -s linux-3.4.6 linux
```

### Arquivo de configuração.
Para compilarmos o kernel, devemos ter um arquivo que contém todas as configurações de hardware, para isso, podemos reaproveitar a configuração atual ou criar um e personalizá-lo.

##### Utilizando arquivo de configuração do kernel atual.
Copiando o arquivo de configuração do kernel atual para o que será compilado, assim, não é necessário com o make config ficar fazendo as milhares de configurações, para isso basta executar o seguinte comando:
```sh
cp /boot/config-$(uname -r) /usr/src/linux/.config
```

Após copiado o arquivo, basta executar o comando abaixo onde, ele usará as configurações do kernel atual da maquina.
```sh
make localmodconfig
```

##### Utilizar uma nova configuração personalizada
Caso você desejar personalizar as configurações para uma nova compilação, o mais indicado é o [menuconfig](http://en.wikipedia.org/wiki/Menuconfig), para isto, basta executar o comando:
```sh
make menuconfig
```

### Criando a imagem do kernel
Agora que temos o arquivo de configuração completo, podemos começar a compilação. O comando **make** quando informado o parâmentro **-j** recebe um número inteiro que indica o número de núcleos/processadores que serão utilizados para esta tarefa, deste modo, a compilação será mais rápida. Em meu caso, informo como parâmetro **-j 8 ** devido meu processador possuir 8 núcleos de procesamento.
```sh
make -j 8 bzImage
```

Este procedimento pode demorar um pouco dependendo de seu hardware.

### Gerar a imagem com o novo kernel em um pacote .deb
Para gerar um pacote e posteriormente instalarmos, basta executar o seguinte comando:
```sh
make-kpkg --initrd kernel_image
```

### Instalação
Tendo gerado o pacote **.deb**, este estará em */usr/src/*, podemos efetuar a instalação através do comando:
```sh
dpkg -i linux-image-[tab]
```

Após a conclusão de todos os procedimentos, basta reiniciar seu computador e conferir a nova versão através do comando:
```sh
uname -r
```

### Remover uma versão de kernel
Primeiramente, devemos listar as versões de kernel que temos instalados em nosso sistema.
```sh
dpkg --get-selections | grep linux-image
```

Identificada a versão que deseja remover, basta executar o comando
```sh
apt-get remove --purge linux-image-xxx
```
ou
```sh
aptitude purge linux-image-xxx
```

Caso este procedimento não tenha as imagens existentes dentro do diretório **/boot**, faz-se necessária a remoção manual destes arquivos.


## Atualizar sistemas de arquivos ext3 para ext4
A atualização do tipo do sistema de arquivos faz-se necessária, para isso, com as as partições *desmontadas*, faça o seguinte comando:
```sh
tune2fs -O extents,uninit_bg,dir_index /dev/sdaX
```

Após a conversão, devemos efetuar uma checagem de disco para reparar quaisquer eventuais problemas.
```sh
fsck -pf /dev/sdaX
```