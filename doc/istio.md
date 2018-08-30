minikube start --memory=8192 --cpus=4 --kubernetes-version=v1.9.4 \
    --extra-config=controller-manager.cluster-signing-cert-file="/var/lib/localkube/certs/ca.crt" \
    --extra-config=controller-manager.cluster-signing-key-file="/var/lib/localkube/certs/ca.key" \
    --extra-config=apiserver.admission-control="NamespaceLifecycle,LimitRanger,ServiceAccount,PersistentVolumeLabel,DefaultStorageClass,DefaultTolerationSeconds,MutatingAdmissionWebhook,ValidatingAdmissionWebhook,ResourceQuota" \
    --vm-driver=`virtualbox`

$ minikube start --memory=8192 --cpus=4 --kubernetes-version=v1.10.0 \
    --extra-config=controller-manager.cluster-signing-cert-file="/var/lib/localkube/certs/ca.crt" \
    --extra-config=controller-manager.cluster-signing-key-file="/var/lib/localkube/certs/ca.key" \
    --vm-driver=`virtualbox`

--vm-driver string               VM driver is one of: [virtualbox vmwarefusion kvm xhyve hyperv hyperkit kvm2 none] (default "virtualbox")

#helm 安装
$ wget https://storage.googleapis.com/kubernetes-helm/helm-v2.7.2-darwin-amd64.tar.gz
#不能获得服务 
lindeMacBook-Pro:istio-1.0.1 lin$ kubectl get svc istio-ingressgateway -n istio-system
No resources found.
Error from server (NotFound): services "istio-ingressgateway" not found
