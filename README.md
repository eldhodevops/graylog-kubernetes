Graylog Kuberentes:
------------------

A quick note on my experiment:

Clone the repo:
--------------
`kubectl create -f elasticsearch-deployment.yaml,elasticsearch-service.yaml,graylog-service.yaml,graylog-deployment.yaml,mongo-service.yaml,mongo-deployment.yaml,mongo-pv.yml,mongo-pvc.yml,endpointservice.yml`


Mongodb - Used to store the configuration data of graylog, so I created mountpath with host (Can try with NFS server also)

In this I've used an additional "endpointservice.yml" for exposing the service and provided 

GRAYLOG_WEB_ENDPOINT_URI as http://graylog/api in graylog-deployment.yaml for accessing UI via web.

Also setup an apache reverse proxy with name "graylog" which have DNS binded for it (graylog) with the hosted server

`+++`
`graylog                                    NodePort    10.99.234.120    <none>                             80:32328/TCP,12201:30695/TCP`  
`graylogsrv                                 ClusterIP   None             <none>                             9000/TCP,12201/TCP `              
`+++`

The "graylogsrv" is for interconnecting the internal services (The service name that have to provide in rsyslog conf for collecting log) and  "graylog" I've binded the port 80 with 9000 for external access.

Pushing logs from contianers:
-----------------------------

I've enabled rsyslog in docker containers and pushed log to graylog server.

For collecting log, enabled rsyslog in docker container and provide the configuration.

Enable imfile module by adding this to your rsyslog.conf (In the docker container to collect the log's):


Rsyslog Version <8
------------------

`$ModLoad imfile
$InputFilePollInterval 10
$PrivDropToGroup adm
$WorkDirectory /var/spool/rsyslog`

Then create "Array-graylog.conf" in /etc/rsyslog.d/Array-graylog.conf file and add below line strting with "module".

Rsyslog Version >= 8
--------------------

`module(load="imfile" PollingInterval="10") #needs to be done just once`




Set the files to monitor. Add the following in /etc/rsyslog.d/Array-graylog.conf.

Rsyslog Version <8.
------------------
`Input for FILE1
$InputFileName /<PATH_TO_FILE1>
$InputFileTag <APP_NAME_OF_FILE1>
$InputFileStateFile <UNIQUE_FILE_ID>
$InputFileSeverity info
$InputRunFileMonitor`

Rsyslog Version >= 8
---------------------

For each file to send
---------------------
`input(type="imfile" ruleset="infiles" Tag="<APP_NAME_OF_FILE1>" File="<PATH_TO_FILE1>" StateFile="<UNIQUE_FILE_ID>")
input(type="imfile" ruleset="infiles" Tag="mysql" File="/var/log/mysql/mysql.log" StateFile="bd873967690fab14ffbaa462a34`

Example Array-graylog.conf:

`module(load="imfile" PollingInterval="10") #needs to be done just once
input(type="imfile" ruleset="infiles" Tag="mysql" File="/var/log/mysql/mysql.log" StateFile="bd873967690fab14ffbaa462a34f8ac7" )
#add the graylog server IP
*.* @graylogsrv:514;RSYSLOG_SyslogProtocol23Format
`

Configure log collection:
-------------------------

`Graylog >> Input >> Syslogudp >> Launch new input >> Select the node >> Set the port (which provided in rsysconf in docker container) and save.`

This is a Quick note for basic understanding :)



