Stage 1:  
    Provision a ec2 machine of the required instance type  
Stage 2:  
    Install Kops Cluster  

    ```console
    curl -Lo kops https://github.com/kubernetes/kops/releases/download/$(curl -s https://api.github.com/repos/kubernetes/kops/releases/latest | grep tag_name | cut -d '"' -f 4)/kops-linux-amd64
    chmod +x kops
    sudo mv kops /usr/local/bin/kops
    ```
        
Stage 3:  
    Install kubectl  