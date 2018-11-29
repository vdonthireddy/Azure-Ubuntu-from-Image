# Create variables that can be used later in the script
VM_NAME=vjd-ubuntu-master-2-vm
RESOURCE_GROUP=${VM_NAME}-rg
USER_NAME=vmadminuser
PASSWORD=P@ssw0rd@Don#*
PUBLIC_IP=${VM_NAME}-pip
VM_IMAGE_NAME=${VM_NAME}-image

# Login to Azure
az login

# Create a resource group
az group create -l westus2 -n $RESOURCE_GROUP

# Create a VM using Ubuntu Linux Distribution image (any image can be used)
az vm create --resource-group $RESOURCE_GROUP --name $VM_NAME --public-ip-address-dns-name $PUBLIC_IP --authentication-type password --admin-username $USER_NAME --admin-password $PASSWORD --image UbuntuLTS --nsg-rule SSH

# Open the required ports (i.e. 80, 8080, 3389, 21 etc.)
az vm open-port -g $RESOURCE_GROUP -n $VM_NAME --port 8080

# Once the VM is created, log in to VM using Terminal (You will be asked to enter the password)
ssh <user name>@<public ip address>

# E.g.
ssh vmadminuser@13.66.201.147

# Install the docker using the following steps (not sure if all these steps are required, but, if you run all these commands, it will work for sure)
sudo apt-get update
sudo apt-get install apt-transport-https ca-certificates curl software-properties-common
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
sudo apt-get update
sudo apt-get install docker-ce

# Test the docker installation
sudo docker --version
sudo docker run hello-world

# Pull the following docker image to test the inbound web traffic on port 8080
sudo docker run -p:8080:8080 vdonthireddy/hello-vj:2.0
# Once the Tomcat is up and running, open your browser and type http://<public ip address>:8080/ to see "Hello World!"

# Sys-Prep the VM to get it ready to create the VM image (WARNING: After this command, the VM can no longer be used)
sudo waagent -deprovision+user

# Exit out of the ssh of VM and follow these steps to create the VM Image
az vm deallocate --resource-group $RESOURCE_GROUP --name $VM_NAME
az vm generalize --resource-group $RESOURCE_GROUP --name $VM_NAME
az image create --resource-group $RESOURCE_GROUP --name $VM_IMAGE_NAME --source $VM_NAME

# Once the image is created, you can use the following set of variables to create a new VM from the image just created.
VM_NAME=vjd-ubuntu-master-3-vm
RESOURCE_GROUP=${VM_NAME}-rg
USER_NAME=vmadminuser
PASSWORD=P@ssw0rd@Don#*
PUBLIC_IP=${VM_NAME}-pip
VM_IMAGE_NAME=${VM_NAME}-image

# Get the IMAGE_PATH by running the following command:
az image list --resource-group <image resource group name> --query [0].id -o json

# Create a variable with the Image path (Resource ID)
IMAGE_PATH=/subscriptions/d6cf69c4-7b7a-40dd-96e0-ead300adde79/resourceGroups/vjd-ubuntu-master-2-vm-rg/providers/Microsoft.Compute/images/vjd-ubuntu-master-2-vm-image

# Create the VM from the existing image
az vm create --resource-group $RESOURCE_GROUP --name $VM_NAME --public-ip-address-dns-name $PUBLIC_IP --authentication-type password --admin-username $USER_NAME --admin-password $PASSWORD --image $IMAGE_PATH --nsg-rule SSH

# Open the port 8080 on the new VM
az vm open-port -g $RESOURCE_GROUP -n $VM_NAME --port 8080

# SSH into the new VM
ssh <user name>@<public ip address>

# E.g.
ssh vmadminuser@13.66.201.147

# Test the docker image (This VM should have all the docker engine, and images available already)
sudo docker run -p:8080:8080 vdonthireddy/hello-vj:2.0
# Once the Tomcat is up and running, open your browser and type http://<public ip address>:8080/ to see "Hello World!"
