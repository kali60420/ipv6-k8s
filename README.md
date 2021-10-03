/etc/hosts

# ipv6-k8s

2001:db8:1234:5678::1   raspberrypi
2a00:1098:0008:0182:0000:0000:0000:0001        rpi-8-2
2a00:1098:0008:0182:0000:0000:0000:0002        rpi-8-2-gateway
10.74.8.9        rpi-8-2
10.74.8.10        rpi-8-2-gateway


127.0.0.1    localhost raspberrypi
::1        localhost ip6-localhost ip6-loopback raspberrypi
ff02::1        ip6-allnodes
ff02::2        ip6-allrouters

2606:4700::6810:1423 registry.yarnpkg.com
2606:4700::6810:1423 registry.npmjs.org
2606:4700::6810:ab63 yarnpkg.com

/etc/network/interfaces.d/eth0

auto eth0.1
iface eth0.1 inet6 static
        address 2001:db8:1234:5678::1
        netmask 64
        up ip -6 route add to 2001:db8:1234:5678::2 via 2001:db8:1234:5678::3 dev eth0.1

iface eth0.1 inet6 static
        address 2001:db8:1234:5678::4
        netmask 64
        
        

setting up magento on k8s ipv6, arm64 (rpi 4)

1. install k8s for ipv6: https://github.com/zhlsunshine/k8s-ipv6

kubeadm init --config=./01-k8s-ipv6/init.yaml
kubectl apply -f ./01-k8s-ipv6/calico-ipv6.yaml

2. install nfs-subdir-external-provisioner: https://github.com/kubernetes-sigs/nfs-subdir-external-provisioner
   add default storageclass as nfs-client (requires NFS Server / Storage solution)
  
helm repo add nfs-subdir-external-provisioner https://kubernetes-sigs.github.io/nfs-subdir-external-provisioner/

helm install nfs-subdir-external-provisioner nfs-subdir-external-provisioner/nfs-subdir-external-provisioner \
    --set nfs.server=2a00:1098:0008:0183:0000:0000:0000:0001 \
    --set nfs.path=/mnt/nfsdir \
    --set 'nfs.mountOptions={nolock}'
   
3. install metallb with ipv6 addresses

kubectl get configmap kube-proxy -n kube-system -o yaml | \
sed -e "s/strictARP: false/strictARP: true/" | \
kubectl diff -f - -n kube-system

kubectl get configmap kube-proxy -n kube-system -o yaml | \
sed -e "s/strictARP: false/strictARP: true/" | \
kubectl apply -f - -n kube-system

kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.10.2/manifests/namespace.yaml

kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.10.2/manifests/metallb.yaml

kubectl apply -f ./03-storageclass/namespace.yaml

4. install nginx-ingress

kubectl create namespace ingress-nginx

helm install syntak-ingress ingress-nginx/ingress-nginx -n ingress-nginx --values ./04-ingress/ingress-nginx.yaml

5. install magento (create image on arm64, run make step-3)

cd ./05-magento && make step-3
