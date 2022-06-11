## Installing Docker 

_Run following on all 3 servers (1 master and 2 clients)_  
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -  
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"  
sudo apt-get update  
sudo apt-get install -y docker-ce=18.06.1ce3-0~ubuntu  
sudo apt-mark hold docker-ce  
sudo docker version

---

## Installing kubeadm kubelet kubectl

_Run following on all 3 servers (1 master and 2 clients)_  
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -<br><br>
cat << EOF | sudo tee /etc/apt/sources.list.d/kubernetes.list  
deb https://apt.kubernetes.io/ kubernetes-xenial main  
EOF<br><br>
sudo apt-get update  
sudo apt-get install -y kubelet=1.15.7-00 kubeadm=1.15.7-00 kubectl=1.15.7-00  
sudo apt-mark hold kubelet kubeadm kubectl  

_Verify kubeadm version_    
kubeadm version

---

## Bootstrapping the cluster

_Run following on the master_  
sudo kubeadm init --pod-network-cidr=10.244.0.0/16  

_Once complete, run the folowing_  
mkdir -p $HOME/.kube  
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config  
sudo chown \$(id -u):\$(id -g) $HOME/.kube/config  

_Verify that the cluster is running and kubectl is working_  
kubectl version  

_Earlier kubeadm init command should provide kubeadm join command, use it next on the client nodes_  
sudo kubeadm join 172.31.97.219:6443 --token j8h6hs.l9c2gfy4futpb3h2
--discovery-token-ca-cert-hash sha256:1b92773d0a76c749c4ec92a96a0aa206857d33e42ea4275554557db752b23656

_verify that all the nodes have joined the cluster by running following command on master showing 'NotReady' value_  
kubectl get nodes

---

## Configuring network with Flannel

_Setting sysctl value on all 3 nodes_  
echo "net.bridge.bridge-nf-call-iptables=1" | sudo tee -a /etc/sysctl.conf

_Activating the above command by usind sudo_  
sudo sysctl -p

_Installing Flannel on the master using a YAML File_  
kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml

_Checking nodes status which should now be changed to 'Ready' from 'NotReady'_  
kubectl get nodes

_Checking if Flannel Pods are available_  
kubectl get pods -n kube-system 
