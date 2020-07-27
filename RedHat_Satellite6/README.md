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
 [root@satellite ~# satellite-maintain service list
 [root@satellite ~# satellite-maintain health check
