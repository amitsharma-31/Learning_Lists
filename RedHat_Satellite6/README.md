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

[root@satellite6 ~]# hammer subscription list --organization OpenTLC
---|------|------|------|----------|---------|---------|----------|----------|---------
ID | UUID | NAME | TYPE | CONTRACT | ACCOUNT | SUPPORT | END DATE | QUANTITY | CONSUMED
---|------|------|------|----------|---------|---------|----------|----------|---------

[root@satellite6 ~]# hammer subscription upload --file Sat6_Class_manifest.zip --organization "OpenTLC" <br>
warning: Overriding "Content-Type" header "multipart/form-data" with "multipart/form-data; boundary=----RubyFormBoundaryIep843W1bLNARYqJ" due to payload <br>
[...................................................................................................................................................................] [100%] <br>
[root@satellite6 ~]# hammer subscription list --organization OpenTLC <br>
![image](https://user-images.githubusercontent.com/42198424/88528113-d2f22380-d01b-11ea-80ba-8ed8576f2aec.png)
