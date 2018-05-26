# PoC CICD with Ansible Tower
Deploy 3 tier app in a cloud using Tower as CICD

## Launch Laboratory
Run the Opentlc lab

## Create ssh.cfg for your deployment
We are going to use the bastion as a jumphost to the internal hosts

```yaml
Host bastion                     
  Hostname bastion.7a71.example.opentlc.com                        
  IdentityFile /home/amarirom/.ssh/id_rsa                          
  ForwardAgent yes               
  User amarirom-redhat.com       
  ControlMaster auto             
  ControlPath /tmp/%h-%r         
  ControlPersist 5               
  StrictHostKeyChecking no       

Host *.7a71.internal             
  User ec2-user                  
  IdentityFile ./7a71key.pem                                       
  ProxyCommand ssh bastion.7a71.example.opentlc.com -W %h:%p                                                                           
  StrictHostKeyChecking no
```

## Create the ansible.cfg

Configure ansible to uset the ssh.cfg 
```yaml
[defaults]
inventory=./inventory
timeout=180                                                         

[ssh_connection]                                                    
ssh_args="-F ./ssh.cfg -o ControlMaster=auto -o ControlPersist=180s"
host_key_checking=False
```

## Build the inventory discovering what groups are in the 3app lab

Figure out what groups are:

```yaml
[root@bastion ~]# ansible all -m debug -a "var=groups"                                                  
app1.7a71.internal | SUCCESS => {                                                                       
    "groups": {                                                                                         
        "3tierapp": [                                                                                   
            "app1.7a71.internal",                                                                       
            "app2.7a71.internal",                                                                       
            "appdb1.7a71.internal",                                                                     
            "support1.7a71.internal",                                                                   
            "frontend1.7a71.internal"                                                                   
        ],                                                                                              
        "all": [                                                                                        
            "app1.7a71.internal",                                                                       
            "app2.7a71.internal",                                                                       
            "frontend1.7a71.internal",                                                                  
            "appdb1.7a71.internal",                                                                     
            "support1.7a71.internal"                                                                    
        ],                                                                                              
        "appdbs": [                                                                                     
            "appdb1.7a71.internal"                  
        ],                                          
        "apps": [                                   
            "app1.7a71.internal",                   
            "app2.7a71.internal"                    
        ],                                          
        "frontends": [                              
            "frontend1.7a71.internal"               
        ],                                          
        "support": [                                
            "support1.7a71.internal"                
        ],                                          
        "ungrouped": []                             
    }                                               
} 
```

The inventory looks like this:

```yaml
[all:vars]                
#GUID=7a71                

[internal:vars]           
ansible_ssh_user=ec2-user 
ansible_become=yes        
timeout=60                

[bastions]                
bastion                   

[frontends]               
frontend ansible_host=frontend1."{{ GUID }}".internal                                                   

[appdbs]                  
appdb1 ansible_host=appdb1."{{ GUID }}".internal    

[apps]                    
app1 ansible_host=app1."{{ GUID }}".internal        
app2 ansible_host=app2."{{ GUID }}".internal        

[support]                 
support1 ansible_host=support1."{{ GUID }}".internal                                                    

[internal:children]       
frontends                 
appdbs                    
apps  
```
