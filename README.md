# ceph-ansible-learnings

* Make sure that hostname matches the monitor interface name. I am not too sure whether this is mandatory, but my issue of mons cluster and mgr cluster not restarting got solved after that. Therefore, *hostnamectl set-hostname* was executed

* I also removed hyphens from the host names.

* I repeatedly got stuck at task [ceph-mgr : get keys from monitors], finally I hacked it. The change I made was in the file **roles/ceph-mgr/tasks/common.yml**.  
  Modified Task **"name: create and copy keyrings"** by replacing delegation. original line is commented and the change is now active.   
   **delegate_to: "{{ groups[mon_group_name][0] if running_mon is undefined else running_mon }}"**
   *#delegate_to: "{{ groups[mon_group_name][0] }}" -- above change by vbg*

* Another change done by me was to restrict monitor to version 2. Change in groups/all.yml :  
  **mon_host_v1:
  enabled: False**

* Did a bluestore backend deployment and accordingly changed osds.yml and all.yml in the group_vars folder

* Final change was to change the docker image tag in group_vars/all.yml. Instead of the tag *latest* for the docker image, the tag was changed to *latest-nautilus* 

* Finally, due to many issues, such as timeout while pulling image and other errors, re-installation became necessary. The necessary steps to clean partial installation in ceph servers due to ceph-ansible errors are mentioned below :  
   1. Check for systemctl services for ceph, if any
    - **$ systemctl list-targets | grep ceph**
   2. If found, stop and disable those services
    - **$ systemctl stop ceph-mon@<hostname>.service**  
    - **$ systemctl disable ceph-mon@<hostname>.service**   
    - **$ systemctl stop ceph-mgr@<hostname>.service**  
    - **$ systemctl disable ceph-mgr@<hostname>.service**   
    - **$ rm /etc/systemd/system/ceph&ast;**  
    - **$ systemctl daemon-reload**  
    - **$ systemctl reset-failed**  
   3.  Clean any ceph data
    - **$ rm -rf /etc/ceph** 
    - **$ rm -rf /var/log/ceph** 
    - **$ rm -rf /var/lib/ceph** 
    - **$ rm -rf /var/run/ceph** 

* **NOTE** - The above paths and commands are for containerised deployment on CentOS7 for nautilus
