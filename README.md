# SIEM FROM SCRATCH

This Vagrant project creates a functioning elastic.co ELK SIEM stack for use in a infosec lab or similar. It will install the ELK stack, register a trial, create TLS certificates, setup users, setup beat index templates etc etc. (see ACTIVITIES). 

To create a complete lab the only thing required should be to install beats agents on boxes and point them at the SIEM.

## PREREQUISITES

This project is designed to be run is a UNIX-like environment such as Linux, BSD, OS X, or at least cygwin. It requires the following tools be installed:

* wget
* openssl
* vagrant

## QUICKSTART

Change to the toplevel directory of the project in a shell and run the following command 

    vagrant up

Install the root certificate at certs/myca/myCA.crt

Navigate to https://172.28.128.20/ in a browser and login with user "elastic" and password "Password1" (no quotes with either username or password) 

## LAB USE

### ROOT CERT

The first time "vagrant up" is run, a root cert will be created in "certs/myca/myCA.crt". You'll need to install this locally on whatever machine you are using to access the Kibana SIEM dashboard. 

### SIEM DASHBOARD

The dashboard will become available on https://SIEMIP:5601/

The username and password is elastic / Password1

## BEATS AGENTS 

Beats are the datashippers for the siem. The SIEM is configured to accept logstash input from beats agents on SIEMIP:5044, no encryption is configured. More information about beats can be found at https://www.elastic.co/beats/

## CONFIGURATION

The SIEM is setup to work out of the box. In general the only thing you should need to change are the SIEM_CN and SIEM_IP in the VagrantFile -- however further customisation is possible under the conf/ directory. 

* SIEM_IP: set this to the IP address of the SIEM
* SIEM_CN: set this to the FQDN of the SIEM


If you want to use your own root certificate, you will need edit conf/siem/config.sh and set ROOTCERT, as well as create and sign .key, .p12 and .crt files and set the appropiate variables. 

* ./conf/siem/config.sh is a sourced shell file with major options. 
    * ELKVERSION is the version of the ELK stack to install
    * ROOTCERT is the vagrant inside the project of the root certificate to use (autogenerated on first run)
    * ELASTICSEARCH_* are the certificate .key, .p12 and .crt files to use for the kibana interface and elasticsearch api (autogenerated on first run)
    * SETUPLOCALBEATS is set to true if beats agents are to be run on the SIEM itself. This is enabled by default so the interface will be populated with data for testing, however you'll probably want to set it to false in a lab environment.
    * INSTALL_DASHBOARDS controls the installation of regular kibana dashboards (useless for SIEM). Disabled by default.
* ./conf/elasticsearch/jvm.options will allow you configure elasticsearch memory 
* ./conf/auditbeat/\*, ./conf/filebeat/\*, ./conf/packetbeat/\* etc are the config files for the local beat agent setup

## ACTIVITIES

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

## LAYOUT

* ./conf/ -- conf files, see above
* ./resources/ -- files such as .deb packages will be downloaded to this directory
* ./certs/
	* ./certs/myca/ -- generated root certificate
	* ./certs/siem/ -- generated siem certificates
* ./helpers/
	* ./helpers/make-ca.sh -- generate a root certificate
	* ./helpers/create-lab-cert.sh -- generate a signed certificate 
	* ./helpers/get-resources.sh -- download needed files such as .deb packages
	* ./helpers/make-clean.sh -- delete resources and generated certs
* ./scripts/
	* ./scripts/debian-check-siem-certs.sh -- check siem certs for sanity
	* ./scripts/debian-check-siem-resources.sh -- check for resources and download if required
	* ./scripts/debian-install-java11.sh -- install java
	* ./scripts/debian-install-root-cert.sh -- install a root certificate
	* ./scripts/debian-install-siem.sh -- install and configure the majority of components (see ACTIVITIES)
	* ./scripts/debian-upgrade.sh -- update package lists from repositories and upgrade


## GOTCHAS

### STATICALLY BUNDLED WINLOGBEAT INDEX TEMPLATE

Currently there is no way of generating the winlogbeat index template from within linux so it is bundled statically. If you change ELKVERSION you will need to regenerate this file by running the following commands within powershell:

    $VERSION=7.9.0
    .\winlogbeat.exe export template --es.version $VERSION | Out-File -Encoding UTF8 "/vagrant/conf/winlogbeat/winlogbeat-$VERSION.template.json"

replacing the 7.9.0 on the $VERSION= line with your ELK version.

You should the generated template file copy this file to ./conf/winlogbeat/










