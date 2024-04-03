
# Install Xray RPM
## Prerequsite
### Hardware
- VM with 4Core, 4GB memory is a minimal requirement.
- 200GB for postgres data directory.
- Download Xray package, choose the ["Linux" type](https://releases.jfrog.io/artifactory/jfrog-xray/xray-linux/[RELEASE]/jfrog-xray-[RELEASE]-linux.tar.gz?_gl=1*1hw3bqo*_ga*MTk0Mjk0NDQ3Ni4xNzExNjEzNDUw*_ga_SQ1NR9VTFJ*MTcxMjEwOTI0Ny4xMC4xLjE3MTIxMDk4NTIuMC4wLjExNDk5ODc2MDA.*_fplc*JTJGZVhZSDRQd2l5YjJNYzc1TUtBcTlzMXpobzhVSERMazclMkZmQjYlMkZEVzNSVHZIMkpKJTJGNDZkVWFwUW5hM3ZYNmZEZktvOFF3Y3k5aTJDSFFYUHNwcktsR1hqd3NKME5qNGhrY3lhaEh3bGx5ZUZjbk0lM0Q.).

### Check Artifactory system.yaml 
```
configVersion: 1
shared:
    security:
    node:
        id: "art61"
        ip: "192.168.56.61"
    database:
mc:
    enabled: true
jfconnect:
    enabled: true
```


## Step 1 Installing PostgreSQL 13
```
cd /root/jfrog-xray-3.69.3-rpm/third-party/postgresql
yum localinstall -y postgresql13-libs-13.*-1PGDG.rhel7.x86_64.rpm postgresql13-server-13.*-1PGDG.rhel7.x86_64.rpm postgresql13-13.*-1PGDG.rhel7.x86_64.rpm

```
### Change dir of PG database（Optional）
Notice: The storage size of the database dir should be greater than 200GB.

Create pg storage path：
```
mkdir /data/pg
chown postgres:postgres /data/pg
```

## init database
```shell
sudo su - postgres
/usr/pgsql-13/bin/pg_ctl -D /data/pg/ initdb
```
Notice: the pg log is in /var/lib/pgsql

### Creating a New PostgreSQL Database 
```
sudo su - postgres
/usr/pgsql-13/bin/postgresql-13-setup initdb
exit;
 systemctl start postgresql-13
 systemctl status postgresql-13
 systemctl enable postgresql-13
```
###  Using PostgreSQL Roles and Databases
```shell
sudo su - postgres
psql -c "alter user postgres with password 'password'"
exit;
```
### Change postgressql.conf
```shell
sudo vi /var/lib/pgsql/13/data/postgresql.conf
# line 59
listen_addresses = '*'

data_directory = "/data/pg"

sudo vi /var/lib/pgsql/13/data/pg_hba.conf

# Accept from anywhere (not recommended)
host all all 0.0.0.0/0 md5

```

Restart postgresql
```shell
sudo systemctl restart postgresql-13
```

### Creating a New Database
```shell
sudo su - postgres
createdb xraydb
exit;
```
查看数据库信息：
```
psql -d xraydb -U postgres
xraydb=> \t
```


## Step 2 Install xray
- Follow the [guide](https://jfrog.com/help/r/jfrog-installation-setup-documentation/install-xray-single-node-with-interactive-script-recommended)

```shell
cd /root/jfrog-xray-3.80.9-rpm
./install.sh
```

```shell
Beginning JFrog Xray setup


This script will install Xray and its dependencies.
After installation, logs can be found at /root/jfrog-xray-3.75.12-rpm/install.log
[WARN] Running with 2 CPU Cores. Recommended value: 3 Cores
Running system diagnostics checks, logs can be found at [/root/jfrog-xray-3.75.12-rpm/systemDiagnostics.log]

Installation Directory (Default: /var/opt/jfrog/xray):

The JFrog URL allows Xray to connect to a JFrog Platform Instance.
(You can copy the JFrog URL from Administration > User Management > Settings > Connection details)
JFrog URL [http(s)://artifactory.server.host:8082] (Default: http://192.168.56.101:8082):
[WARN ] Error while initializing File resolver : Config file does not exists : var/etc/system.yaml
Attempt to connect JFrogURL succeeded

The Join Key is the secret key used to establish trust between services in the JFrog Platform.
(You can copy the Join Key from Admin > User Management > Settings > Connection details)
Join Key (****):


For IPv6 address, enclose value within square brackets as follows : [<ipv6_address>]
Please specify the IP address of this machine (Default: 192.168.56.101):

Are you adding an additional node to an existing product cluster? [y/N]: n

The installer can install a PostgreSQL database, or you can connect to an existing compatible PostgreSQL database
(https://service.jfrog.org/installer/System+Requirements#SystemRequirements-RequirementsMatrix)
If you are upgrading from an existing installation, select N if you have externalized PostgreSQL, select Y if not.
Do you want to install PostgreSQL? [Y/n]: n

Provide the database connection details
PostgreSQL url. Example: [postgres://<IP_ADDRESS>:<PORT>/xraydb?sslmode=disable]: postgres://localhost:5432/xraydb?sslmode=disable

Database username (If your existing connection URL already includes the username, leave this empty): postgres

Database password (If your existing connection URL already includes the password, leave this empty) (****):

[WARN ] Error while initializing File resolver : Config file does not exists : var/etc/system.yaml
[TRACE] JDBC to PostgreSQL URL conversion: begin
[TRACE] JDBC to PostgreSQL URL conversion: end

Database connection successful
RabbitMQ dependencies Installation

Installing/Verifying RabbitMQ dependencies (this may take several minutes)...
Installing /root/jfrog-xray-3.75.12-rpm/third-party/rabbitmq/esl-erlang_25.0.3-1.el7.x86_64.rpm ...

Xray Installation

Installing/Verifying Xray (/root/jfrog-xray-3.75.12-rpm/xray/xray.rpm) (this may take several minutes)...

PostgreSQL Installation


Installation complete.
```

Start Xray:
```shell
systemctl start xray
```
### Update vulnerability database:
How the trace the update?
1. Find the update traceId，search "triggering" keyword in Xray console.logs, get the traceId
2. Trace the log of this keyword.
sudo tail -f /var/opt/jfrog/xray/log/console.log |grep 71026de85a4cc027 

## Optional: Create a soft link for data folder in case the default installation folder(/var/opt/jfrog) is less than 5GB.
ln -s /DATA/jfrog /opt/jfrog

## Enable curation
```shell
catalog:
    enabled: true
    uiEnabled: true
    central:
        enabled: true
        url: https://jfrogxraycatalog.jfrog.io/xray
        username: catalog_name
        password: pwd
```

# Install JAS
## Preparation
- Prepare 3 VMs, 1 for k3s master, 2 for k3s node.
- Make sure you have set the Artifactory base url.
- Disable firewalld service for each machine.
- [Optional] If you don't have internet access or cannot access Git Hub, then download the k3s binary and copy it to the xray server node. Refer to this guide: https://jfrog.com/help/r/jfrog-security-documentation/install-jfrog-advanced-security-on-your-self-hosted-environment-without-helm
Then you can remove the download task of k3s
```
cd /root/jfrog-xray-3.83.10-linux/app/bin/k3s-ansible/roles/download/tasks
mv main.yml ~
```
This step will make you skip the download task for ansible playbook.

## Follow the wiki
https://jfrog.com/help/r/jfrog-security-documentation/jas-interactive-script-installation
- After the ansible-playbook was completed, you will get a file k3s_config.yaml, copy it to x-ray "/opt/jfrog/xray/var/etc/"
Change the owner of this file:
```
chown xray:xray k3s_config.yaml
```
Use this config in xray system.yaml. The final system.xml should be look like below:
```
shared:
    database:
        password: aesgcm256
    rabbitMq:
        password: aesgcm256
    jfrogUrl: http://192.168.56.101:8082
    security:
        joinKey: 5b199a.aesgcm256.N18_mT0vg147oonK-RSCjfR4seGrzrkC4tONK_96toaoaOWzMDCXGAZP0aGMUdCnwqLxsjcJ-0i35fi9
    node:
        ip: 192.168.56.102
executionService:
    jobAesKey: key
    kubeconfig:
        path: /opt/jfrog/xray/var/etc/k3s_config.yaml
        namespace: default
        context: default
        enabled: true
rabbitMq:
    ## Enable this to stop rabbitmq along with other services of xray
    ## By default rabbitmq will always be running
    autoStop: true
```

# Install JAS in Airgap environment

## Step1 - Preprae

- guide

        https://jfrog.com/help/r/jfrog-installation-setup-documentation/configure-jas-in-an-air-gapped-non-helm-environment

        follow the official guide to prepare VMs
        
- system

        - The following operating systems are certified

                RHEL 8
                Ubuntu 20
                Ubuntu 22

        - notice
        
                install JAS in the airgap env with Centos 7.8 will report error 
        
## Step2 - config artifactory

1. vi ./system.yaml

        shared:
            ...
        mc:
            enabled: true
        jfconnect:
            enabled: true
            airgap:
                enabled: true
            usage:
                enabled: true

        systemctl restart artifactory

2. generate an admin scoped token

        - result of this step
        
                bearer token

3. generate offline request token

        - notice
        
                no 'system' in the path !
        
        - API

                curl -X GET 'http://x.x.x.x/jfconnect/api/v1/offline/start_register' \
                --header 'Authorization: Bearer <token>' \
                --data ''

        - result of this step
        
                request token

4. post token to JFrog Entitlements Service & get entitlements.json

        - notice
        
                URL is jes.jfrog.io !
                use -v to get response token in header

        - API
                
                curl -v --location 'https://jes.jfrog.io/api/v1/offline_register' \
                --header 'Content-Type: application/json' \
                --data '{
                "offline_request_token": "<token>"
                }' > entitlements.json

        - result of this step
       
                response token
                entitlements.json

5. register

        - notice

                below curl will not work, use postman !

        - API
        
                curl --location 'http://x.x.x.x:8082/jfconnect/api/v1/offline/register' \
                --header 'Authorization: Bearer xx' \
                --form 'offline_response_token="xx"' \
                --form 'entitlements=@"/Users/xx/xx/entitlements.json"'

## Step3 - config xray

        follow the official guide

## Step4 - config k3s node

        follow the official guide

## Step5 - install JAS in airgap env

        follow the official guide

## Step6 - JAS db sync in airgap env

- online sync

        1. open whitelist

              https://jfrog-prod-use1-shared-virginia-main.s3.amazonaws.com

              - if this's not whitelisted, xray's log will report error below
  
                      Caused by: Get "https://jfrog-prod-use1-shared-virginia-main.s3.amazonaws.com/aol-releases
                        ....
                      tls: failed to verify certificate: x509: certificate signed by unknown authority

        2. click online sync



    




        




