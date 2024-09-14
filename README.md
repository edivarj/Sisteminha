# Sisteminha
O sisteminha é um programa de ligações em massa  usando asterisk e chandongle, ele funciona como uma ura reversa com opçoes de discagens,  se o cliente  discar 1, 2,3 etc. ele dispara um sms de acordo com a configuração colocada, rodando em debian 12.


sudo apt update
sudo apt upgrade -y
sudo apt install wget vim net-tools -y
###############

sudo yum groupinstall "Development Tools" -y (para CentOS)
sudo apt install build-essential -y (para Debian)
sudo yum install ncurses-devel libxml2-devel libsqlite3-dev -y
sudo apt install libncurses5-dev libxml2-dev libsqlite3-dev -y


###############

cd /usr/src/
wget http://downloads.asterisk.org/pub/telephony/asterisk/asterisk-18-current.tar.gz
tar -xvf asterisk-18-current.tar.gz
cd asterisk-18*/
./configure
make menuselect
make
sudo make install
sudo make config
sudo ldconfig

#################
sudo systemctl start asterisk
sudo systemctl enable asterisk
##########

cd /usr/src/
git clone https://github.com/jstasiak/asterisk-chan-dongle.git
cd asterisk-chan-dongle
./configure
make
sudo make install
sudo make config


##############


sudo nano /etc/asterisk/dongle.conf



# Instalar dependências
sudo yum install git libusb-devel libudev-devel -y###

# Clonar e compilar o chan_dongle
cd /usr/src/
git clone https://github.com/wdoekes/asterisk-chan-dongle.git
cd asterisk-chan-dongle
./bootstrap
./configure
make
make install

# Configuração
cd /etc/asterisk/
cp dongle.conf.sample dongle.conf

# Editar o arquivo dongle.conf com as configurações do modem
sudo nano dongle.conf



#############
[dongle0]
; Configuração do dongle para chamadas e SMS
audio=/dev/ttyUSB1   ; porta de áudio
data=/dev/ttyUSB2    ; porta de dados
context=from-dongle  ; contexto para chamadas
sms=1                ; habilitar envio de SMS
####

EXTENSION.CONF
sudo nano /etc/asterisk/extensions.conf


[from-dongle]
exten => s,1,Answer()
same => n,Playback(/var/lib/asterisk/sounds/en/custom/audio_inicial)
same => n,WaitExten()

exten => 1,1,Playback(/var/lib/asterisk/sounds/en/custom/audio_opcao1)
same => n,System(echo "SMS enviado para a opção 1" | /usr/local/bin/send_sms.sh ${CALLERID(num)} "Mensagem para opção 1")

exten => 2,1,Playback(/var/lib/asterisk/sounds/en/custom/audio_opcao2)
same => n,System(echo "SMS enviado para a opção 2" | /usr/local/bin/send_sms.sh ${CALLERID(num)} "Mensagem para opção 2")

exten => 0,1,Playback(/var/lib/asterisk/sounds/en/custom/audio_nao_ligar)
same => n,System(echo "${CALLERID(num)}" >> /var/blacklist.txt)
same => n,Hangup()

####
SEND_SMS.SH

sudo nano /var/lib/asterisk/agi-bin/send_sms.sh


#!/bin/bash
NUMBER=$1
MESSAGE=$2
dongle sms dongle0 $NUMBER "$MESSAGE"
__
execute o escript

chmod +x /usr/local/bin/send_sms.sh

#########

sudo nano /var/www/html/get_modems.php

<?php
$output = shell_exec('dongle show devices');
echo json_encode($output);
?>
######

sudo nano /var/www/html/start_campaign.php


<?php
$modemPort = $_POST['modemPort'];
$callAttempts = $_POST['callAttempts'];
$contactsFile = $_FILES['contactsFile']['tmp_name'];
$voiceMessage = $_FILES['voiceMessage']['tmp_name'];

// Lógica para processar os dados e iniciar as chamadas automáticas usando o Asterisk
echo "Campanha iniciada com o modem $modemPort";
?>

###

sudo yum install httpd php php-cli -y


sudo systemctl start httpd
sudo systemctl enable httpd


sudo chown -R asterisk:asterisk /var/www/html
sudo chmod -R 755 /var/www/html
####
tentar acessar o navegador 

http://seu_servidor/get_modems.php

####


se der ruim 

sudo firewall-cmd --permanent --add-port=80/tcp
sudo firewall-cmd --reload
##

sudo systemctl restart asterisk



###
asterisk -rx "dongle show devices"

###########

sudo apt install apache2 php php-mysql php-gd -y (Debian)

####
sudo systemctl start httpd
sudo systemctl enable httpd
##

