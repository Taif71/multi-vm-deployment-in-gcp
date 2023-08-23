[![Phase](https://img.shields.io/badge/version-1.0-green?style=flat-square&logo=#&logoColor=white)](#)


# multi-vm-deployment-in-gcp
The repo shows how to deploy a multi-vm full-stack application in Google Cloud Platform (GCP 🚀) under the same VPC (Virtual Private ☁️). We establish nginx which will act as both a load balancer and a reverse proxy. Only one VM should have public IP and other three VMs will not contain any public IP. For Egress communication they should use cloud NAT coupled with a cloud Router. The following diagram (Figure 1) shows that architecture.

<br/>
<figure><img src="./assets/multi-vm-deployment.png" alt="multi-vm-deoployment"/>
<figcaption align = "center">Fig.1 - A schematic representation of the demo</figcaption></figure>
<br/>


### Step 1 | Create a VPC
Go to VPC network and create a VPC with the following configurations:<br/>
Name: deployment-vpc<br/>
region: us-east1 <br/>
IP stack type: IPv4 (single stack)</br>
IPv4 range: 172.168.0.0/24 </br>
Firewalls: Allow all firewall rules </br>

Now create the VPC.
</br></br>


### Step 2 | Create 4 Virtual Machines

Remember that, the VM responsible for load-balancing shall have both public and private IPs, whereas, the other three VMs will not have any public IP. They will communicate either through the load balancer or for egress communication they will use Cloud NAT and Cloud Router. But let's create the VMs first so that we get to know the private IPs.

i) For load-balancer VM configuration:
</br>
VM name: nginx-vm
region: us-east1
Access Scope: allow default access
Firewall: Allow Http, Https
Advanced > network > new network interfaces: 
network - deployment-vpc </br>
Subnetwork - deployment-subnet
Primary Internal address: Ephemeral
Externam IPv4 Address: Ephemeral

Create the VM.

ii) other three VM's configurations are quite similar. So I will rush through them:
For frontend, </br>
name: frontend-vm
firewall: allow all firewalls
network: deployment-vpc
subnet: deployment-subnet
private IP: Ephemeral
public IP: None

Create VM.

Similarly create VMs for backend1 and backend 2.

In the end we will have VMs with the following IPs (In our case):

nginx-vm: 172.168.0.2(private ip) and 104.196.59.74 (public ip) </br> 
frontend-vm: 172.168.0.3 </br>
backend1-vm: 172.168.0.4 </br>
backend2-vm: 172.168.0.5 </br>

### Step 3 | Create Cloud Router and Cloud Nat

### Step 4 | Setup the load balancer configuration

we will now first setup reverse proxy and make it bahave like a load balancer. We have to setup nginx first. SSH into the nginx-vm. </br>
Run the following commands step by step:

$ sudo apt install update - y 

$ sudo apt install nginx -y 

$ sudo systemctl status nginx 
/* Should show it's running*/


Now go to your browser and search http://104.196.59.74 and it should show nginx Welcome page.

Now, let's setup nginx configurations.
in the terminal inside the nginx-vm do:
</br> cd /etc/nginx
</br> vim nginx.conf

or, you could connect you vm to your local machine VM and edit the nginx.conf file like I did.


<details>
<summary>Configuring <code>nginx</code></summary><br/>

<img src="./assets/load-balancer/nginx_conf_file_directory.png" alt=""/>
</br>
<img src="./assets/load-balancer/nginx.conf file.png" alt=""/>

</details>



### Step 5 | Setting up the frontend VM

SSH into the frontend-VM:

$ sudo su

// install nodejs

$ curl -fsSL https://deb.nodesource.com/setup_20.x

$ sudo apt-get install nodejs -y

$ node -v

$ sudo corepack enable // so that we can install yarn

$ yarn -v 

$ yarn create vite

cd vite-project (default project name)

$ yarn

$ yarn build 

$ yarn preview --host --port 80


now from browser use the load balancer public IP and we should see the front-end vite boilerplate running!

<details>
<summary>run <code>frontend</code></summary><br/>

<img src="./assets/vite boilerplate.png" alt=""/>
</br>
</details>



### Step 6 | Setting up the backend servers

We have already configured the load balancer. However now we have to set up the apps in both servers so that the load balancer can route its requests evenly to these servers.

SSH into backend server 1:

sudo apt update -y </br>
sudo su </br>
curl -fsSL https://deb.nodesource.com/setup_19.x  </br>
apt-get install nodejs -y </br>
node -v </br>

mkdir backend1 </br>
cd backend1 </br>
sudo corepack enable </br>
npm init -y </br>
yarn add express </br>


// now vim into index.js
<details>
<summary>express <code>backend1 app</code></summary><br/>

<img src="./assets/backends/express-be1.png" alt=""/>
</br>
</details>

Now run the server code: </br>
sudo node index.js
<details>
<summary> <code>server running</code></summary><br/>

<img src="./assets/backends/node_1.png" alt=""/>
</br>
</details>

</br></br>
Finally, if we follow the same steps for backend 2  VM it will run in its own server.


</br>
Now, if we go to the public ip of load balancer then it will take us to the frontend.
However, If we keep hitting 104.196.59.74/api, we will see that the requests are being handled by the load balancer and being routed to one of the 2 backend services





