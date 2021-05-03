# MongoDB setup

## What it is
MongoDB is a source-available cross-platform document-oriented database program. Classified as a NoSQL database program, MongoDB uses JSON-like documents with optional schemas.

## Prerequisites 

 - RAM : 8 GB
 - Storage : 100 GB
 - Operating system : Linux/Windows


## What we used : 
- Virtual Machine : 2 vCPU, 4 GB RAM 
- Operating System : Ubuntu 20.4
- Databases created : dev , uat
- Remote database users created : dev, uat, admin
- Database port : 27017 (default)


## Mongo DB on Linux
[MongoDB installation guide](https://docs.mongodb.com/manual/installation/)
 1. Import the public key used by the package management system.


        wget -qO - https://www.mongodb.org/static/pgp/server-4.4.asc | sudo apt-key add -

 2. Create a list file for MongoDB.

        echo "deb [ arch=amd64,arm64 ] https://repo.mongodb.org/apt/ubuntu focal/mongodb-org/4.4 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-4.4.list

 3. Reload local package database.

        sudo apt-get update

 4. Install the MongoDB packages.

        sudo apt-get install -y mongodb-org

 5. Start MongoDB

        sudo systemctl start mongod 


## Create Databases

 - Create 'dev' database

        > use dev
        switched to dev
        > db
        dev
        > db.inventory.insertOne(
            { item: "canvas", qty: 100, tags: ["cotton"], size: { h: 28, w: 35.5, uom: "cm" } }
          )
        > show databases
        admin 0.000GB
        dev 0.000GB
        local  0.000GB

 - Create 'uat' database

        > use uat
        switched to uat
        > db
        uat
        > db.inventory.insertOne(
            { item: "canvas", qty: 100, tags: ["cotton"], size: { h: 28, w: 35.5, uom: "cm" } }
          )
        > show databases
        admin 0.000GB
        dev 0.000GB
        local  0.000GB

## Add remote users to MongoDB

 - Add an admin user


        > use admin;
        > db.createUser({
                user: "admin",
                pwd: "myadminpassword",
                roles: [
                            { role: "userAdminAnyDatabase", db: "admin" },
                            { role: "readWriteAnyDatabase", db: "admin" },
                            { role: "dbAdminAnyDatabase",   db: "admin" }
                        ]
            });

 - Add a dev user

        > db.createUser({
                user: "technox-admin",
                pwd: "tEcHnOx#431",
                roles: [
                            { role: "dbOwner", db: "dev" }
                       ]
            });

 - Add an uat user

        > db.createUser({
                user: "technox-admin",
                pwd: "tEcHnOx#431",
                roles: [
                            { role: "dbOwner", db: "uat" }
                       ]
            });

[Click here](https://docs.mongodb.com/manual/tutorial/install-mongodb-on-windows/) for MongoDB Windows setup.  


  [MongoDB disk and memory requirements](https://learn.fotoware.com/On-Premises/FotoWeb/05_Configuring_sites/Setting_the_MongoDB_instance_that_FotoWeb_uses/MongoDB_disk_and_memory_requirements#:~:text=MongoDB%20requires%20approximately%201GB%20of,performance%2C%20and%20should%20be%20avoided.)

  - Disk
  
    Each asset remains approximately 10 days in MongoDB after deletion. Hence, the approximate formula for calculating the number of assets can be expressed as follows:


        **N = i * (t + 10) + p**

        p: Number of assets that are permanently in the system
        t: Max number of days an asset is kept in the system after ingestion
        i: Number of assets ingested per day (24h)
        N: Total number of assets in the system at any given time.

        
        Each asset requires approximately 10Kb of space. The formula to calculate the disk space required for assets in MongoDB goes like this:

        Disk space (Megabytes) = N * 0,01

- Memory

    MongoDB requires approximately 1GB of RAM per 100.000 assets. If the system has to start swapping memory to disk, this will have a severely negative impact on performance, and should be avoided.



[Performance Best Practices: Hardware and OS Configuration](https://www.mongodb.com/blog/post/performance-best-practices-hardware-and-os-configuration) : 
  - Run on Supported Platforms
  - Ensure Your Working Set Fits in RAM
  - Use Multiple CPU Cores
  - For best performance, you should run one mongod process per host.
  - Configuring the WiredTiger Cache
  - Use Multiple Query Routers
  - Use Interleave Policy on NUMA Architecture
  - Network Compression
  - Use SSDs for IO Intensive Applications
  - Use MongoDB’s Default Compression for Storage and I/O-Intensive Workloads
  

# CI/CD setup 

## What it is

CI/CD or CICD generally refers to the combined practices of continuous integration and either continuous delivery or continuous deployment. CI/CD bridges the gaps between development and operation activities and teams by enforcing automation in building, testing and deployment of applications.


## What is Jenkins

Jenkins is a free and open source automation server. It helps automate the parts of software development related to building, testing, and deploying, facilitating continuous integration and continuous delivery.

## Prerequisites

 - [Jenkins Hardware Recommendations](https://www.jenkins.io/doc/book/scaling/hardware-recommendations/) 
 - 4 GB+ of RAM
 - 50 GB+ of drive space
 - Java Runtime Environment

 ## What we used 

 - Virtual Machine : 2 vCPUs, 4 GB RAM
 - Platform : Jenkins
 - Method : [Pipeline](https://www.jenkins.io/doc/tutorials/#pipeline/) 
 - Users created : admin, dev
 - Plugins : Suggested plugins, Role-based Authorization Strategy


 ## Jenkins Installation (Linux)

  1. Import the GPG keys of the Jenkins repository using the following wget      command:

    wget -q -O - https://pkg.jenkins.io/debian/jenkins.io.key | sudo apt-key add -

  2. Next, add the Jenkins repository to the system with:

    sudo sh -c 'echo deb http://pkg.jenkins.io/debian-stable binary/ > /etc/apt/sources.list.d/jenkins.list'

  3. Once the Jenkins repository is enabled, update the apt package list and install the latest version of Jenkins by typing:

    sudo apt update

  4. Install Jenkins

    sudo apt install jenkins

[Click here](https://www.jenkins.io/doc/book/installing/windows/) for setting up Jenkins on Windows



## Alternatives

|   Platform    |   Service                   |    
|:-------------:| :--------------------------:|
| AWS           | CodePipeline                |  
| GCP           | Cloud Build                 |   
| GitLab        | Gitlab CI/CD                |
| GitHub        | Github Actions              |   


## Source Code Management

- What we used : GiHub


- Other options Available : GiLab, CVS, Mercurial 

## What our pipeline looks like

 - Stage 1 : Clone repositories from Github to our Jenkins server.
 - Stage 2 : Build the Docker image from source code repository
    
    1. Login to ECR
    2. Tag the Docker image with Jenkins $BUILD_NUMBER
    3. Push the image to ECR

 - Stage 3 : Update Image tag with build number.
 - Stage 4 : Deploy Kubernetes cluster.
 - Stage 5 : Cleanup previously built docker images.

            pipeline {

            agent any

            stages {
                stage('Clone Repositories') {
                    steps {
                        dir ("MCI-Admin") {
                            git (
                            url: 'git@github.com:Technoxander-Dev/MCI-Admin.git',
                            credentialsId: 'jenkins-ci-creds',
                            branch: "development"
                            )
                        }
                    }
                }
                stage('Build') {
                    steps {
                        dir('MCI-Admin') {
                            sh '''
                                aws ecr-public get-login-password --region us-east-1 | docker login --username AWS --password-stdin public.ecr.aws/m0t6v6x5
                                docker build -t mci-frontend --build-arg base_url=http://k8s-dev-mcibacke-22d09da984-371631921.us-east-2.elb.amazonaws.com .
                                docker tag mci-frontend:latest public.ecr.aws/m0t6v6x5/mci-frontend:${BUILD_NUMBER}
                                docker push public.ecr.aws/m0t6v6x5/mci-frontend:${BUILD_NUMBER}
                            '''
                        }
                    }
                }
                stage('Update Image') {
                    steps {
                        dir('MCI-Admin') {
                            sh 'sed -i s!BUILD_NUMBER!${BUILD_NUMBER}!g k8s/mci-frontend.yml'
                        }
                    }
                }
                stage('Deploy') {
                    steps {
                        dir('MCI-Admin') {
                            sh 'kubectl apply -f k8s/mci-frontend.yml -n dev'
                            sh 'kubectl apply -f k8s/ingres-frontend.yml -n dev'
                        }
                    }
                }
                stage('Clean up') {
                    steps {
                        sh 'docker rmi public.ecr.aws/m0t6v6x5/mci-frontend:${BUILD_NUMBER}'
                    }
                }
            }
            }

## On premises server requirements 

### Our Recommendation

 - RAM : 4+ GB
 - Storage : 50 GB
 - Operating System : Linux 

### Best Practices

[Jenkins Pipeline Best Practices](https://www.jenkins.io/doc/book/pipeline/pipeline-best-practices/)
   - Making sure to use Groovy code in Pipelines as glue
   - Reducing repetition of similar Pipeline steps
   - Avoiding calls to Jenkins.getInstance
   - Avoiding very large shared libraries

# Docker Registry
## What it is
The Registry is a stateless, highly scalable server side application that stores and lets you distribute Docker images. The Registry is open-source, under the permissive.
## Why use it
You should use the Registry if you want to:
* tightly control where your images are being stored
* tightly control where your images are being stored
* integrate image storage and distribution tightly into your in-house development workflow
## Prerequisites
- RAM: 1GB
- Storage: 500GB
- Operating System: Ubuntu 18.04 LTS or Higher
## What we used
- RAM: 4 GB
- OS: Ubuntu 20.04.2 LTS
- Platform: Elastic Container Registry (ECR)
## Alternatives
Users looking for a zero maintenance, ready-to-go solution are encouraged to head-over to the [Docker Hub](https://hub.docker.com/ "Docker Hub's Homepage"), which provides a free-to-use, hosted Registry, plus additional features (organization accounts, automated builds, and more).
|   Platform    |   Service                   |    
| ------------- |:---------------------:      |  
| AWS           | Elastic Container Registry  |  
| GCP           | Google Container Registry   |   
| Azure         | Azure Container Registry    |
| Git           | Github Container Registry   |  
## Requirements
The Registry is compatible with Docker engine **version 1.6.0 or higher.**
 
​
## Docker Installation 
Before start a registry, you need to Install [ Docker](https://docs.docker.com/engine/install/ubuntu/ "Docker Installation's Homepage") engine on the host. 
### OS requirements 
To install Docker Engine, you need the 64-bit version of one of these Ubuntu versions:
- Ubuntu Hirsute 21.04
- Ubuntu Groovy 20.10
- Ubuntu Focal 20.04 (LTS)
- Ubuntu Bionic 18.04 (LTS)
- Ubuntu Xenial 16.04 (LTS)
​
Docker Engine is supported on x86_64 (or amd64), armhf, and arm64 architectures.
#### 1. Update the apt package index, and install the latest version of Docker Engine and containerd, or go to the next step to install a specific version:
    $ sudo apt-get update
    $ sudo apt-get install docker-ce docker-ce-cli containerd.
#### 2. To install a specific version of Docker Engine, list the available versions in the repo, then select and install:
#### a. List the versions available in your repo:
      $ apt-cache madison docker-ce
    
#### b. Install a specific version using the version string from the second column, for example, 5:18.09.1~3-0~ubuntu-xenial.
      $ sudo apt-get install docker-ce=<VERSION_STRING> docker-ce-cli=<VERSION_STRING> containerd.io
​
#### 3. Verify that Docker Engine is installed correctly by running the hello-world image.
      $ sudo docker run hello-world
​
### Now you are ready to start the Docker registry.
## Commands
#### Start your registry
    docker run -d -p 5000:5000 --name registry registry:2 
#### Pull (or build) some image from the hub
    docker pull ubuntu
#### Tag the image so that it points to your registry
    docker image tag ubuntu localhost:5000/myfirstimage
#### Push it
    docker push localhost:5000/myfirstimage
#### Pull it back
    docker pull localhost:5000/myfirstimage
## Other available Docker Registries
- **Amazon Elastic Container Registry (ECR):** Integrates with AWS Identity and Access Management (IAM) service for authentication. It supports only private repositories and does not provide automated image building.
- **Google Container Registry (GCR):** Authentication is based on Google’s Cloud Storage service permissions. It supports only private repositories and provides automated image builds via integration with Google Cloud Source Repositories, GitHub, and Bitbucket.
- **Azure Container Registry (ACR):** Supports multi-region registries and authenticates with Active Directory. It supports only private repositories and does not provide automated image building.
- **CoreOS Quay:** Supports OAuth and LDAP authentication. It offers both (paid) private and (free) public repositories, automatic security scanning and automated image builds via integration with GitLab, GitHub, and Bitbucket.
- **Private Docker:** Registry supports OAuth, LDAP and Active Directory authentication. It offers both private and public repositories, free up to 3 repositories (private or public).

# K8s setup
 ## What it is :
   - Kubernetes is a tool for orchestrating and managing Docker containers at scale on on-prem server or across hybrid cloud environments. Kubeadm is a tool provided with Kubernetes to help users install a production ready Kubernetes cluster with best practices enforcement.
​
 ## Server types used in deployment of Kubernetes clusters:
  - **Master**: A Kubernetes Master is where control API calls for the pods, replications controllers, services, nodes and other components of a Kubernetes cluster are executed.
  
  - **Node**: A Node is a system that provides the run-time environments for the containers. A set of container pods can span multiple nodes.
​
 ## Requirements :
  - **Memory**: 2 GiB or more of RAM per machine
  - **CPUs**: At least 2 CPUs on the control plane machine.
  - Internet connectivity for pulling containers required (Private registry can also be used)
  - Full network connectivity between machines in the cluster – This is private or public
 ## **What we used :**
  - EKS Cluster
  - K8s Version : 1.19
  - Operating System : Ubuntu 20.04.2 LTS
  - Virtual Machine : 2 vCPUs, 2 GB RAM
​
 ### **Options available in :**
​
|   Platform    |   Service                   |    
| ------------- |:---------------------------:|  
|      GCP      |    Google Kubernetes Engine   | 
|     Azure     |    Azure Kubernetes Service   |
----------------------------------------------------------------------------------------------------
  ## Kubectl Setup
   ## What it is :
- The Kubernetes command-line tool, kubectl, allows you to run commands against Kubernetes clusters. You can use kubectl to deploy applications, inspect and manage cluster resources, and view logs
- The kubectl command line tool lets you control Kubernetes clusters. It allows you to perform every possible Kubernetes operation. The main job of kubectl is to carry out HTTP requests to the Kubernetes API
​
 ## Installing Kubectl
   - #### Update the apt package index and install packages needed to use the Kubernetes apt repository: 
         sudo apt-get update
         sudo apt-get install -y apt-transport-https ca-certificates curl  
   - #### Download the Google Cloud public signing key:
         sudo curl -fsSLo /usr/share/keyrings/kubernetes-archive-keyring.gpg https://packages.cloud.google.com/apt/doc/apt-key.gpg
   
   - #### Add the Kubernetes apt repository:
         echo "deb [signed-by=/usr/share/keyrings/kubernetes-archive-keyring.gpg] https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list
   - #### Update apt package index with the new repository and install kubectl:
         sudo apt-get update
         sudo apt-get install -y kubectl
​
  ## Cluster setup
 - ## Step 1: Install Kubernetes Servers
   - Provision the servers to be used in the deployment of Kubernetes on Ubuntu 20.04.
   - #### Once the servers are ready, update them.
            sudo apt update
            sudo apt -y upgrade 
​
 - ## Step 2: Install kubelet, kubeadm and kubectl
    - #### Once the servers are rebooted, add Kubernetes repository for Ubuntu 20.04 to all the servers.
            sudo apt -y install curl apt-transport-https
            curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
            echo "deb https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list
    - #### Then install required packages.
            sudo apt update
            sudo apt -y install vim git curl wget kubelet kubeadm kubectl
            sudo apt-mark hold kubelet kubeadm kubectl
    - #### Confirm installation by checking the version of kubectl.
            kubectl version --client && kubeadm version
 - ## Step 3: Disable Swap
    - #### The idea of kubernetes is to tightly pack instances to as close to 100% utilized as possible. All deployments should be pinned with CPU/memory limits. So if the scheduler sends a pod to a machine it should never use swap at all. You don't want to swap since it'll slow things down.
            sudo sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab
            sudo swapoff -a
    - #### Configure sysctl.
            sudo modprobe overlay
            sudo modprobe br_netfilter
            
            sudo tee /etc/sysctl.d/kubernetes.conf<<EOF
            net.bridge.bridge-nf-call-ip6tables = 1
            net.bridge.bridge-nf-call-iptables = 1
            net.ipv4.ip_forward = 1
            EOF
            
            sudo sysctl --system
 - ## Step 4: Install Container runtime
      - To run containers in Pods, Kubernetes uses a container runtime. Supported container runtimes are:
        - Docker
        - CRI-O
        - Containerd
      - #### Installing Docker runtime:
            # Add repo and Install packages
            sudo apt update
            sudo apt install -y curl gnupg2 software-properties-common apt-transport-https ca-certificates
            curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
            sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
            sudo apt update
            sudo apt install -y containerd.io docker-ce docker-ce-cli
            
            # Create required directories
            sudo mkdir -p /etc/systemd/system/docker.service.d 
            # Create daemon json config file
            sudo tee /etc/docker/daemon.json <<EOF
            {
              "exec-opts": ["native.cgroupdriver=systemd"],
              "log-driver": "json-file",
              "log-opts": {
               "max-size": "100m"
              },
             "storage-driver": "overlay2"
            }
            EOF
            # Start and enable Services
            sudo systemctl daemon-reload 
            sudo systemctl restart docker
            sudo systemctl enable docker
 - ## Step 5: Initialize master node
      - #### Login to the server to be used as master and make sure that the br_netfilter module is loaded:
             $ lsmod | grep br_netfilter
      - #### Enable kubelet service.
             sudo systemctl enable kubelet
      - #### Pull container images:
             $ sudo kubeadm config images pull
      - #### Set cluster endpoint DNS name or add record to /etc/hosts file.
             $ sudo vim /etc/hosts
      - #### Create cluster:
             sudo kubeadm init \
              --pod-network-cidr=192.168.0.0/16 \
              --control-plane-endpoint=<endpoint>
      - #### Configure kubectl using commands in the output:
            mkdir -p $HOME/.kube
            sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
            sudo chown $(id -u):$(id -g) $HOME/.kube/config
      - #### Check cluster status:
            $ kubectl cluster-info
      - #### Additional Master nodes can be added using the command in installation output:
            kubeadm join k8s-cluster.computingforgeeks.com:6443 --token sr4l2l.2kvot0pfalh5o4ik \
            --discovery-token-ca-cert-hash sha256:c692fb047e15883b575bd6710779dc2c5af8073f7cab460abd181fd3ddb29a18 \
            --control-plane 
    
 - ## Step 6: Add worker nodes
    - #### The join command used to add a worker node to the cluster.
          kubeadm join k8s-cluster.computingforgeeks.com:6443 \
          --token sr4l2l.2kvot0pfalh5o4ik \
          --discovery-token-ca-cert-hash sha256:c692fb047e15883b575bd6710779dc2c5af8073f7cab460abd181fd3ddb29a18
    - #### Run below command on the control-plane to see if the node joined the cluster.
            $ kubectl get nodes   
# Helm Setup
 ## What it is :
   - Helm is a tool for Kubernetes packages called charts. A chart contains pre-configured Kubernetes resources and editable settings for them.
   - Helm installs charts into Kubernetes, creating a new release for each installation. 
​
 ## Prerequisites :
   - A running Kubernetes cluster. As an alternative, you can install a single-node Kubernetes cluster on a local system using Minikube.
   - A kubectl command line tool installed and configured to communicate with the cluster. 
​
 ## Installing Helm :
       curl https://baltocdn.com/helm/signing.asc | sudo apt-key add -
       sudo apt-get install apt-transport-https --yes
       echo "deb https://baltocdn.com/helm/stable/debian/ all main" | sudo tee /etc/apt/sources.list.d/helm-stable-debian.list
       sudo apt-get update
       sudo apt-get install helm
​
 ## Three Big Concepts
  - A **Chart** is a Helm package. It contains all of the resource definitions necessary to run an application, tool, or service inside of a Kubernetes cluster. 
  - A **Repository** is the place where charts can be collected and shared.
  - A **Release** is an instance of a chart running in a Kubernetes cluster. One chart can often be installed many times into the same cluster. And each time it is installed, a new release is created. 
​
 ## Basic Commands :
  ### 1) **'helm search': Finding Charts**
  - Helm comes with a powerful search command. It can be used to search two different types of source:
       - $ helm search hub searches the [Artifact Hub](https://artifacthub.io/), which lists helm charts from dozens of different repositories.
       - $ helm search repo searches the repositories that you have added to your local helm client (with helm repo add). This search is done over local data, and no public network connection is needed.
​
  - #### You can find publicly available charts by running **helm search hub**:
         $ helm search hub wordpress
​
  - #### Using **helm search repo**, you can find the names of the charts in repositories you have already added:
         $ helm repo add brigade https://brigadecore.github.io/charts "brigade" has been added to your repositories
         $ helm search repo brigade
​
  ### 2) **'helm install': Installing a Package**
  - #### To install a new package, use the **helm install** command. At its simplest, it takes two arguments: A release name that you pick, and the name of the chart you want to install.
         $ helm install release-name bitnami/wordpress
  - #### To keep track of a release's state, or to re-read configuration information, you can use **helm status**
         $ helm status release-name
​
  ### 3) **'helm upgrade' and 'helm rollback': Upgrading a Release, and Recovering on Failure**
  - #### When a new version of a chart is released, or when you want to change the configuration of your release, you can use the **helm upgrade** command.
         helm upgrade -f release.yaml release-name bitnami/wordpress
  - ####  If something does not go as planned during a release, it is easy to roll back to a previous release using **helm rollback** [RELEASE] [REVISION].
         $ helm rollback release-name 1
    The above rolls back our release-name to its very first release version. A release version is an incremental revision. Every time an install, upgrade, or rollback happens, the revision number is incremented by 1. The first revision number is always 1. And we can use helm history [RELEASE] to see revision numbers for a certain release.
​
  ### 4) **'helm uninstall': Uninstalling a Release**
  - #### When it is time to uninstall a release from the cluster, use the **helm uninstall** command: 
         $ helm uninstall release-panda
  - #### This will remove the release from the cluster. You can see all of your currently deployed releases with the **helm list** command:
         $ helm list
​
  ### 5) **Creating Your Own Charts**
  - #### Develop your own charts using the **helm create** command
         $ helm create dir-name
  - #### When it's time to package the chart up for distribution, you can run the **helm package** command:
         $ helm package dir-name
  - ### And that chart can now easily be installed by **helm install**:
         $ helm install dir-name ./dir-name-0.1.0.tgz
​
  ### 6) **For more information on these commands, take a look at Helm's built-in help:**
         $ helm help
