# Install

## 사전 환경 준비
* Kubernetes Cluster >= v1.17
* 최소 200Gi 이상 사용 가능한 Kubernetes Storage Class가 있어야 한다.
## Tacoplay를 사용하여 설치
* Prepare Tacoplay  
        git clone https://github.com/openinfradev/tacoplay.git

        cd tacoplay && ./fetch_sub-projects.sh

        sudo pip install -r requirements.txt --upgrade --ignore-installed

        sudo pip install --upgrade pip

* Ansible Inventory 작성  
tacoplay/inventory/dev/hosts.ini 파일:
        
        master-node access_ip=127.0.0.1 ansible_connection=local 

        [admin-node]
        master-node    
tacoplay/inventory/dev/extra-vars.yml 파일:  

        taco_storageclass_name: "" # storage class name    

* Install Decapod  

        // Run commands inside tacoplay directory
        ansible-playbook -b -i inventory/test/hosts.ini \
        -e @inventory/test/extra-vars.yml site.yml \
        --tags decapod --skip-tags ceph,k8s