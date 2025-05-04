----Simple example to Deploy an Application and the Kubernetes Dashboard----
=>Note: Kindly view it in raw

Step1: Creation of namespace
kubectl create ns dev
kubectl get ns 

Step2: Set resource quotas for namespace(k8s/labs/workloads/resourcequota.yaml)
kubectl apply -f resourcequota.yaml
kubectl get resourcequota -n dev

Step3: Create user roles and bindings (RBAC)
sudo openssl genrsa -out ram.key 2048
sudo openssl req -new -key ram.key -out ram.csr
sudo openssl x509 -req -in ram.csr -CA /etc/kubernetes/pki/ca.crt -CAkey /etc/kubernetes/pki/ca.key -CAcreateserial -out ram.crt -days 500
kubectl create -f role.yaml
kubectl get roles -n dev

kubectl create -f rolebinding.yaml
kubectl get rolebinding -n dev

kubectl config set-credentials ram --client-certificate=/home/labsuser/dev/ram.crt --client-key=/home/labsuser/dev/ram.key
kubectl config get-contexts
kubectl config set-context ram-context --cluster=kubernetes --namespace=dev --user=ram
kubectl config use-context ram-context
sudo chmod 666 ram.key ram.crt

Step4: Apply namespace restrictions (via NetworkPolicy)
kubectl apply -f deny-from-other-ns.yaml
kubectl get networkpolicy -n dev

Step5: Create PVs and PVCs
kubectl config use-context kubernetes-admin@kubernetes
kubectl apply -f hostpath-pv.yaml
kubectl config use-context ram-context

kubectl apply -f hostpath-pvc.yaml
kubectl get pvc hostpath-pvc

Step6: Deploy MySQL deplyoment
kubectl apply -f mysql-configmap.yaml
kubectl apply -f mysql-secret.yaml
kubectl apply -f mysql.yaml
kubectl apply -f mysql-service.yaml

kubectl get configmap -n dev
kubectl get secrets -n dev
kubectl get deploy -n dev
kubectl get svc -n dev

Test the Vars passed to MySQL pod:
kubectl get pods -n dev
kubectl exec -it <pod-id> -n dev -- env|grep MYSQL_DATABASE

Step7:Create directory in the worker1 as it mounts the host directory /mnt/data into the container via hostPath.
sudo mkdir /mnt/data

Step8: Deploy Wordpress deplyoment
kubectl apply -f wordpress-config.yaml
kubectl apply -f wordpress-secret.yaml
kubectl apply -f wordpress.yaml
kubectl apply -f wordpress-service.yaml

kubectl get configmap -n dev
kubectl get secrets -n dev
kubectl get deploy -n dev
kubectl get svc -n dev

Test the Vars passed to WordPress pod:
kubectl get pods -n dev
kubectl exec -it <pod-id> -- env|grep WORDPRESS_DB_HOST

Step9:Access wordpress from the browser by pasting the Service IP
kubectl get svc wordpress-service -n dev

![image](https://github.com/user-attachments/assets/a2993a9a-ece2-4a39-8898-8a7d00d2a27a)

Step10: Deploy Kubernetes Dashboard
curl https://baltocdn.com/helm/signing.asc | gpg --dearmor | sudo tee /usr/share/keyrings/helm.gpg > /dev/null
sudo apt-get install apt-transport-https --yes
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/helm.gpg] https://baltocdn.com/helm/stable/debian/ all main" | sudo tee /etc/apt/sources.list.d/helm-stable-debian.list
sudo apt-get update
sudo apt-get install helm

helm repo add kubernetes-dashboard https://kubernetes.github.io/dashboard/
helm upgrade --install kubernetes-dashboard kubernetes-dashboard/kubernetes-dashboard --create-namespace --namespace kubernetes-dashboard
helm status kubernetes-dashboard --namespace kubernetes-dashboard	

kubectl apply -f k8s-dashboard.yaml
kubectl -n kubernetes-dashboard create token admin-user
kubectl -n kubernetes-dashboard port-forward svc/kubernetes-dashboard-kong-proxy 8444:443
https://localhost:8444
![image](https://github.com/user-attachments/assets/27169a4a-d72f-4bd8-a866-6ae51d9056a5)

