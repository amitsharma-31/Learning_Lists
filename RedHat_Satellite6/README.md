# RedHat Satellite 6 Deployment and relevant Notes

## Special/Important things to consider before deployment

Satellite will run with Integrated database or External Database? <br>
In case of external who will manage or configure/tune the External DB ####This is good if DBA's available to manage MongoDB & Postgress mgmt & availability <br>
HW Considerations <br>
OS Considerations <br>
Firewall Considerations <br>
Time Sync/Chrony service should be configured and running on satellite with trusted NTP source <br>
Hostname considerations (better to keep it only 3 level domain. eg xyz.companydomain.com or satellite.companydomain.com <br>
Number of nodes to manage as this will introduce to plan for seperate capsule server (Better choice to split into multiple capsule rather external DB) <br>
Would you be using satellite to run containers infra?? If yes, then need container registry internal or external access via satellite## IMPORTANT  <br> 
DNS localhost resolution should properly work #ping -c1 localhost and #ping -c1 $(hostname -f) <br>

## Hardware Requirements
The Red Hat Satellite 6 host must meet the following minimum hardware specifications: <br>
	• x86_64 architecture.<br>
	• Minimum of four 2.0 GHz CPU cores.<br>
	• Minimum of 20 GB of physical memory, and a minimum of 4 GB of swap space.<br>

## SW/OS Requirements
only install the @Base package group

## Firewall Considerations
    [root@satellite ~]# firewall-cmd --permanent \
    --add-port="53/udp" --add-port="53/tcp" \
    --add-port="67/udp" --add-port="69/udp" \
    --add-port="80/tcp"  --add-port="443/tcp" \
    --add-port="5000/tcp" --add-port="5647/tcp" \
    --add-port="8000/tcp" --add-port="8140/tcp" \
    --add-port="9090/tcp"
    success
    [root@satellite ~]# firewall-cmd --reload
    success

    [root@satellite ~]# firewall-cmd --permanent --add-service=RH-Satellite-6
    success
    [root@satellite ~]# firewall-cmd --reload
    success
    
## Satellite Health or Services status check
 [root@satellite ~# satellite-maintain service list ###To verify the status of satellite services
 [root@satellite ~# satellite-maintain health check ###various health checks in the Satellite Server installation

## Before you create your manifest, consider the following:
	• Include the Satellite Server subscription in the manifest if planning a disconnected Red Hat Satellite installation. <br>
	• Include subscriptions for all Red Hat Satellite Capsule Servers. <br>
	• Include subscriptions for all Red Hat products to manage with Red Hat Satellite. <br>
	• Plan in advance the manifest renewals based on their expiration dates, and use the future-dated subscriptions to ease this process. <br>
Modify and update the manifest for any organization to include additional infrastructure subscriptions. Do not delete the active manifests in your Satellite installation from the Red Hat Customer Portal, because doing so will unregister all your hosts. <br>

## Sample Reference Architecture for huge setup
![alt text](https://user-images.githubusercontent.com/42198424/88525836-c3251000-d018-11ea-96b2-a3e46dfd491c.png)

## Troubleshoot issues 
[satellite]# sosreport
[satellite]# foreman-debug

## Logs files and clues
| Log path | What to look for  |
| :---:   | :-: | 
| /var/log/messages | "ERROR" |
| /var/log/katello-installer/* | "ERROR" |
| /var/log/tomcat6/catalina.out | Java tracebacks |
| /var/log/foreman/db_migrate.log | |
| /var/log/foreman/*startup.log | |
| /usr/share/foreman/tmp/pids/dynflow_executor.output | Useful if the foreman-tasks service fails to start |

## To reset password of satellite in case you forgot it
foreman-rake permissions:reset   # This will randomly set any password

## Satellite repository download policy can be set as default (on_demand, immediate, background)

## Searchquery in remote execution :- 
name = satclient.example.com
## HAMMER CLI Setup
--------------------------------------------------------------------------------------------------------------------------------
[root@satellite6 ~]# cat >> /root/.bashrc <<EOF 
> export ORG='TEST_ORG'
> export LOCATION='NCR' 
> export ORGLABEL='TEST_ORG' 
> EOF 
[root@satellite6 ~]# source /root/.bashrc

[root@satellite6 ~]# cat >> ~/.hammer/cli.modules.d/foreman.yml <<EOF 
>   :host: 'https://satellite6.example.com'    #####Check the allignment##
> EOF 
[root@satellite6 ~]# cat ~/.hammer/cli.modules.d/foreman.yml

hammer -u <USER> -p <PASSWORD> subscription upload --file manifest.zip --organization "Default_Organization"

[root@satellite6 ~]# hammer subscription list --organization TEST_ORG
`---|------|------|------|----------|---------|---------|----------|----------|---------
ID | UUID | NAME | TYPE | CONTRACT | ACCOUNT | SUPPORT | END DATE | QUANTITY | CONSUMED
---|------|------|------|----------|---------|---------|----------|----------|---------`

[root@satellite6 ~]# hammer subscription upload --file Sat6_Class_manifest.zip --organization "TEST_ORG" <br>
warning: Overriding "Content-Type" header "multipart/form-data" with "multipart/form-data; boundary=----RubyFormBoundaryIep843W1bLNARYqJ" due to payload <br>
[...................................................................................................................................................................] [100%] <br>
[root@satellite6 ~]# hammer subscription list --organization TEST_ORG <br>
![image](https://user-images.githubusercontent.com/42198424/88528113-d2f22380-d01b-11ea-80ba-8ed8576f2aec.png)

[root@satellite6 ~]# hammer task list   ##(Equal to satellite dashboard monitor -> task)
[root@satellite6 ~]# hammer task info --id d2907880-bb5b-4854-8aad-a89dfb16ebef   ##(Equal to satellite dashboard monitor -> task and selecting any task for detail) <br>
ID:          d2907880-bb5b-4854-8aad-a89dfb16ebef <br>
Action:      Create organization {"text"=>"organization 'OpenTLC'", "link"=>"/organizations/1/edit"} <br>
State:       stopped <br>
Result:      success <br>
Started at:  2020/07/22 13:43:01 <br>
Ended at:    2020/07/22 13:43:27 <br>
Owner:       foreman_api_admin <br>
Task errors: <br>

[root@satellite6 ~]# hammer subscription refresh-manifest --organization TEST_ORG
[root@satellite6 ~]# hammer repository list    #initiate repo list
[root@satellite6 ~]# hammer repository synchronize --id 1 --id 2 --id 3 --organization TEST_ORG  #initiate repo sync

[root@satellite6 ~]# hammer --no-headers --output yaml repository list
---
- Id: 3
  Name: Red Hat Enterprise Linux 7 Server Kickstart x86_64 7.7
  Product: Red Hat Enterprise Linux Server
  Content Type: yum
  URL: https://cdn.redhat.com/content/dist/rhel/server/7/7.7/x86_64/kickstart
- Id: 1
  Name: Red Hat Enterprise Linux 7 Server RPMs x86_64 7Server
  Product: Red Hat Enterprise Linux Server
  Content Type: yum
  URL: https://cdn.redhat.com/content/dist/rhel/server/7/7Server/x86_64/os
- Id: 2
  Name: Red Hat Satellite Tools 6.5 for RHEL 7 Server RPMs x86_64
  Product: Red Hat Enterprise Linux Server
  Content Type: yum
  URL: https://cdn.redhat.com/content/dist/rhel/server/7/7Server/x86_64/sat-tools/6.5/os

[root@satellite6 ~]#  for i in `hammer --no-headers --output yaml repository list |grep Id |cut -d: -f2`; do

## Client installation
[root@satclient ~]# wget http://satellite6.example.com/pub/katello-ca-consumer-latest.noarch.rpm 
[root@satclient ~]# yum -y install katello-ca-consumer-latest.noarch.rpm

[root@satclient ~]# grep baseurl /etc/rhsm/rhsm.conf
baseurl= https://satellite6.example.com/pulp/repos

[root@satclient ~]# subscription-manager register --org=TEST_ORG --environment=Library
subscription-manager list --available
subscription-manager attach --pool 40288094737a092601737a44f75535dc
yum clean all
subscription-manager refresh
subscription-manager list --consumed
subscription-manager repos
subscription-manager repos --enable rhel-7-server-satellite-tools-6.5-rpms
yum search katello-agent
yum -y install katello-agent
katello-package-upload  ###In case katello agent status in the host->content hosts is not showing green for katello agent

## Satellite Deployment Command
satellite-installer --foreman-initial-organization TEST_ORG <br> --foreman-initial-location NCR <br> --scenario satellite <br> --foreman-proxy-tftp true <br> --foreman-proxy-dhcp true <br> --foreman-proxy-dhcp-gateway "192.168.0.2" <br> --foreman-proxy-dhcp-interface "eth0" <br> --foreman-proxy-dhcp-range "192.168.0.200 192.168.0.250" <br> --foreman-proxy-dns true <br> --foreman-proxy-dns-forwarders "192.168.0.10" <br> --foreman-proxy-dns-interface "eth0" <br> --foreman-proxy-dns-reverse 0.168.192.in-addr.arpa <br> --foreman-proxy-dns-server "127.0.0.1" <br> --foreman-proxy-dns-zone "example.com" <br> --foreman-admin-password 'redhat'

## Satellite uninstall steps
![image](https://user-images.githubusercontent.com/42198424/88530295-cb804980-d01e-11ea-913f-5c6c73836fca.png)

## Common Issue or handy References
https://access.redhat.com/solutions/2755731
foreman-rake foreman_tasks:cleanup TASK_SEARCH='label ~ *' AFTER='30d' VERBOSE=true NOOP=true
https://access.redhat.com/solutions/3997131
 satellite-installer --foreman-puppetrun=true (this will allow option puppet run from satellite https://access.redhat.com/solutions/2162121 puppet run deprecated)

To sign puppet certificates in satellite for new hosts
https://access.redhat.com/solutions/1228973  complete puppet use
`Capsules -> <select capsulename> -> Puppet CA -> Certificates`

## Puppet agent certificate issue 
To fix this, remove the certificate from both the master and the agent and then start a puppet run, which will automatically regenerate a certificate.
On the master:
  puppet cert clean satclient.example.com
On the agent:
  1a. On most platforms: find /opt/puppetlabs/puppet/cache/ssl -name satclient.example.com.pem -delete
  1b. On Windows: del "\opt\puppetlabs\puppet\cache\ssl\certs\satclient.example.com.pem" /f
  2. puppet agent -t
Than sign again in satellite portal

## How to check satellite version running
[root@satellite6 ~]# rpm -q satellite --queryformat "%{VERSION}"
6.5.1[root@satellite6 ~]#

## Admin credentials can be find in  cat /root/.hammer/cli.modules.d/foreman.yml

## Satellite server backup & restore :- https://access.redhat.com/solutions/1595963
`[root@satellite6 ~]# satellite-maintain backup offline /var/backup_directory
Starting backup: 2020-07-25 18:09:51 +0200
Running preparation steps required to run the next scenarios
================================================================================
Make sure Foreman DB is up:
/ Checking connection to the Foreman DB                               [OK]
--------------------------------------------------------------------------------``


Running Backup
================================================================================
Confirm turning off services is allowed:
WARNING: This script will stop your services.

Do you want to proceed?, [y(yes), q(quit)] y
                                                                      [OK]
--------------------------------------------------------------------------------
Prepare backup Directory:
Creating backup folder /var/backup_directory/satellite-backup-2020-07-25-18-09-51
                                                                      [OK]
--------------------------------------------------------------------------------
Check if the directory exists and is writable:                        [OK]
--------------------------------------------------------------------------------
Generate metadata:
| Saving metadata to metadata.yml                                     [OK]
--------------------------------------------------------------------------------
Backup config files:
| Collecting config files to backup                                   [OK]
--------------------------------------------------------------------------------
Turn on maintenance mode:                                             [OK]
--------------------------------------------------------------------------------
Stop applicable services:
Stopping the following service(s):

rh-mongodb34-mongod, postgresql, qdrouterd, qpidd, squid, pulp_celerybeat, pulp_resource_manager, pulp_streamer, pulp_workers, smart_proxy_dynflow_core, tomcat, dynflowd, httpd, puppetserver, foreman-proxy
- All services stopped                                                [OK]
--------------------------------------------------------------------------------
Backup Pulp data:
`| Collecting Pulp data                                                [OK]
`--------------------------------------------------------------------------------
Backup mongo offline:
`\ Collecting Mongo data                                               [OK]
`--------------------------------------------------------------------------------
Backup Candlepin DB offline:
`\ Collecting data from /var/lib/pgsql/data/                           [OK]
`--------------------------------------------------------------------------------
Backup Foreman DB offline:
`Already done                                                          [OK]
`--------------------------------------------------------------------------------
Start applicable services:
Starting the following service(s):

rh-mongodb34-mongod, postgresql, qdrouterd, qpidd, squid, pulp_celerybeat, pulp_resource_manager, pulp_streamer, pulp_workers, smart_proxy_dynflow_core, tomcat, dynflowd, httpd, puppetserver, foreman-proxy
- All services started                                                [OK]
`--------------------------------------------------------------------------------
Turn off maintenance mode:                                            [OK]
--------------------------------------------------------------------------------
Compress backup data to save space:
\ Compressing backup of Postgress DB
- Compressing backup of Mongo DB                                      [OK]
--------------------------------------------------------------------------------

Done with backup: 2020-07-25 18:20:15 +0200
`**** BACKUP Complete, contents can be found in: /var/backup_directory/satellite-backup-2020-07-25-18-09-51 ****`

## Satellite restoration from backup :- I simulated  the running and configured setup by running "katello-remove"

We need to ensure time sync, selinux configured properly as it was earlier, hostname & host file is setup as previous, install satellite and puppetserver start puppetserver service
yum install satellite; yum install puppetserver; systemctl start puppetserver  (same version of packages)

`[root@satellite6 ~]# satellite-maintain restore /var/backup_directory/satellite-backup-2020-07-25-18-09-51
Running Restore backup
================================================================================
Check if command is run as root user:                                 [OK]
--------------------------------------------------------------------------------
Validate backup has appropriate files:                                [OK]
--------------------------------------------------------------------------------
Confirm dropping databases and running restore:

WARNING: This script will drop and restore your database.
Your existing installation will be replaced with the backup database.
Once this operation is complete there is no going back.
Do you want to proceed?, [y(yes), q(quit)] y
                                                                      [OK]
--------------------------------------------------------------------------------
Validate hostname is the same as backup:
                             [OK]
--------------------------------------------------------------------------------
Setting file security:
| Restoring SELinux context                                           [OK]
--------------------------------------------------------------------------------
Restore configs from backup:
\ Restoring configs                                                   [OK]
--------------------------------------------------------------------------------
Ensure restored MongoDB storage engine matches the current DB:        [OK]
--------------------------------------------------------------------------------
Run installer reset:
\ Installer reset
\ Installer reset                                                     [OK]
--------------------------------------------------------------------------------
Stop applicable services:
Stopping the following service(s):

rh-mongodb34-mongod, postgresql, qdrouterd, qpidd, squid, pulp_celerybeat, pulp_resource_manager, pulp_streamer, pulp_workers, smart_proxy_dynflow_core, tomcat, dynflowd, httpd, puppetserver, foreman-proxy
/ All services stopped                                                [OK]
--------------------------------------------------------------------------------
Extract any existing tar files in backup:
/ Extracting pgsql data                                               [OK]
--------------------------------------------------------------------------------
Migrate pulp db:
| Migrating pulp database                                             [OK]
--------------------------------------------------------------------------------
Start applicable services:
Starting the following service(s):

rh-mongodb34-mongod, postgresql, qdrouterd, qpidd, squid, pulp_celerybeat, pulp_resource_manager, pulp_streamer, pulp_workers, smart_proxy_dynflow_core, tomcat, dynflowd, httpd, puppetserver, foreman-proxy
/ All services started                                                [OK]
--------------------------------------------------------------------------------
Run daemon reload:                                                    [OK]
--------------------------------------------------------------------------------
[root@satellite6 ~]# satellite-maintain health check
Running ForemanMaintain::Scenario::FilteredScenario
================================================================================
Check for verifying syntax for ISP DHCP configurations:               [OK]
--------------------------------------------------------------------------------
Check whether all services are running:                               [OK]
--------------------------------------------------------------------------------
Check whether all services are running using hammer ping:             [OK]
--------------------------------------------------------------------------------
Check for paused tasks:                                               [OK]
--------------------------------------------------------------------------------

[root@satellite6 ~]#`
