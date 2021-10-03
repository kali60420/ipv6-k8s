# ipv6-k8s

setting up magento on k8s ipv6, arm64 (rpi 4)

1. install k8s for ipv6: https://github.com/zhlsunshine/k8s-ipv6
2. install nfs-subdir-external-provisioner: https://github.com/kubernetes-sigs/nfs-subdir-external-provisioner
   add default storageclass as nfs-client (requires NFS Server / Storage solution)
3. install metallb with ipv6 addresses
4. install nginx-ingress
5. install magento (create image on arm64, run make step-3)
