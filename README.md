# SIEM FROM SCRATCH

This project creates a "drop in" ELK SIEM component for use in a infosec redteam lab. It will install the ELK stack, register a trial, create TLS certificates, setup users, setup beat index templates etc etc. (see ACTIVITIES). 

To create a complete lab the only thing required should be to install beats agents on boxes and point them at the SIEM.

## Prerequisites

This project is designed to be run is a UNIX-like environment such as Linux, BSD, OS X, or at least cygwin. It requires the following tools be installed:

* wget
* openssl
* vagrant
* Internet connection

## Quickstart

Change to the toplevel directory of the project in a shell and run the following command 

    vagrant up

Install the root certificate at siem/certs/myca/myCA.crt

Navigate to https://172.28.128.20/ in a browser and login with user "elastic" and password "Password1" (no quotes with either username or password) 

It is recommended to quickstart the project at least once to create the SIEM root cert and download all the resources. If you then copy this "siem/" folder into your labs in future you will not need to re-install any certificates or re-download any resources.

## Lab Use

Copy the "siem/" folder into the vagrant directory of your lab. Edit your VagrantFile to include the SIEM_IP and SIEM_CN at the top:

    SIEM_IP   = "172.28.128.20"
    SIEM_CN   = "siem.lab"

Change the IP and hostname to whatever is desired for your lab.

Include the following lines within the "Vagrant.configure("2") do |config|" section to provision the SIEM virtual machine:


    config.vm.define "siem" do |cfg|
        cfg.vm.box = "ubuntu/xenial64"
        cfg.vm.hostname = "siem"
        cfg.vm.network :private_network, ip: SIEM_IP
        cfg.vm.provision "shell", path: "siem/scripts/debian-install-siem-one-line.sh", args: [SIEM_CN, SIEM_IP]
        cfg.vm.provider "virtualbox" do |vb, override|
          vb.customize ["modifyvm", :id, "--memory", 4096]
          vb.customize ["modifyvm", :id, "--cpus", 2]
          vb.customize ["modifyvm", :id, "--clipboard-mode", "bidirectional"]
        end
    end

You can base copy this from the example VagrantFile in the project directory. A more granular example VagrantFile is provided at "VagrantFile.verbose" if you need more control over the provisioning process.

### Root Certificate

The first time "vagrant up" is run, a root cert will be created in "siem/certs/myca/myCA.crt". You'll need to install this locally on whatever machine you are using to access the Kibana SIEM dashboard. 

### SIEM Dashboard

The dashboard will become available on https://SIEMIP:5601/

The username and password is elastic / Password1

## Beats Agents

Beats are the datashippers for the siem. The SIEM is configured to accept logstash input from beats agents on SIEMIP:5044, no encryption is configured. More information about beats can be found at https://www.elastic.co/beats/

## Configuration

The SIEM is setup to work out of the box. In general the only thing you should need to change are the SIEM_CN and SIEM_IP in the VagrantFile -- however further customisation is possible under the conf/ directory. 

* SIEM_IP: set this to the IP address of the SIEM
* SIEM_CN: set this to the FQDN of the SIEM


If you want to use your own root certificate, you will need edit conf/siem/config.sh and set ROOTCERT, as well as create and sign .key, .p12 and .crt files and set the appropiate variables. 

* ./conf/siem/config.sh is a sourced shell file with major options. 
    * ELKVERSION is the version of the ELK stack to install
    * ROOTCERT is the vagrant inside the project of the root certificate to use (autogenerated on first run)
    * ELASTICSEARCH_* are the certificate .key, .p12 and .crt files to use for the kibana interface and elasticsearch api (autogenerated on first run)
    * SETUPLOCALBEATS is set to true if beats agents are to be run on the SIEM itself. This is enabled by default so the interface will be populated with data for testing, however you'll probably want to set it to false in a lab environment.
    * INSTALL_DASHBOARDS controls the installation of regular kibana dashboards (useless for SIEM, disabled by default)
* ./conf/elasticsearch/jvm.options will allow you configure elasticsearch memory 
* ./conf/auditbeat/\*, ./conf/filebeat/\*, ./conf/packetbeat/\* etc are the config files for the local beat agent setup

## Activities

The VagrantFile will perform the following actions:

* create a root certificate and a create a signed sub certificate 
* download all required files (such as elasticsearch .deb packages etc) into ./resources
* install Java 
* install root certificate 
* install elasticsearch, kibana and logstash
* configure elasticsearch to use the created certs
* register elasticsearch for platinum trial
* create elasticsearch users
* configure kibana with created certs and users
* configure logstash with created certs and users
* install index templates and kibana dashboards for auditbeat, filebeat, packetbeat and winlogbeat
* create SIEM signal index
* install default detection rules
* enable default detection rules
* set SIEM dashboard as kibana homepage

## Project Layout

* siem/ -- main project folder, drop this into your lab
    * conf/ -- conf files, see above
    * resources/ -- files such as .deb packages will be downloaded to this directory
    * certs/
    	* myca/ -- generated root certificate
    	* siem/ -- generated siem certificates
    * helpers/
    	* make-ca.sh -- generate a root certificate
    	* create-lab-cert.sh -- generate a signed certificate 
    	* get-resources.sh -- download needed files such as .deb packages
    	* make-clean.sh -- delete resources and generated certs
        * update-powershell-conf.sh -- generate powershell config.ps1 variable include file from siem/conf/siem/config.sh
    * scripts/
    	* debian-check-siem-certs.sh -- check siem certs for sanity
    	* debian-check-siem-resources.sh -- check for resources and download if required
    	* debian-install-java11.sh -- install java
    	* debian-install-root-cert.sh -- install a root certificate
    	* debian-install-siem.sh -- install and configure the majority of components (see ACTIVITIES)
    	* debian-upgrade.sh -- update package lists from repositories and upgrade


## Caveats

I am not a SIEM expert and I have never worked on the blueteam in any capacity -- in fact I am actually a complete beginner when it comes to incident response. This project was born purely out of a desire to see what sort of signals my actions on the redteam would create, and to "scratch an itch" of having a SIEM easily available to place in my labs.

Without a doubt, this project will contain many glaring omissions and errors that are entirely obvious to anyone skilled in the art. For this I apologise in advance, however any feedback, suggestions, pull requests or raised issues are gratefully accepted.

## Gotchas

### Statically Bundled Winlogbeat Index Template

Unlike other beat shippers, currently there is no way of generating the winlogbeat index template dynamically from within linux.  This means it has been bundled with the project statically. If you change ELKVERSION you will need to regenerate this file by running the following commands within powershell:

    $VERSION=7.9.0
    .\winlogbeat.exe export template --es.version $VERSION | Out-File -Encoding UTF8 "/vagrant/conf/winlogbeat/winlogbeat-$VERSION.template.json"

replacing the 7.9.0 on the $VERSION= line with your ELK version.

You should copy the generated template file to ./conf/winlogbeat/










