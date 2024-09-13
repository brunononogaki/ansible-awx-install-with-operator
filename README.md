# Ansible AWX Installation with AWX Operator and Rancher
This is a step-by-step guide for dummies to install AWX Operator in a Fresh Installed Linux CentOS 9, providing external access to the Web Interface with the Linux IP Address.


## 1. Preparing the environment

### 1.1 Run yum update
```
sudo yum update
````

### 1.2 Install git
```
sudo yum install git
```

### 1.3 Install Rancher
+ Download and run install script ([Full Documentation](https://docs.k3s.io/quick-start))
```
sudo curl -sfL https://get.k3s.io | sh -
```

+ Change k3s.yaml file permissions. <br>
*Note: Change admin:admin to your user and group. You can check it with the commands whoami and groups.*

```
sudo chown admin:admin /etc/rancher/k3s/k3s.yaml
```

+ You can check if kubectl was successfully installed by issuing:
```
kubectl version
```

### 1.4 Install Kustomize
+ Download Kustomize binaries ([Full Documentation](https://kubectl.docs.kubernetes.io/installation/kustomize/binaries/))
```
curl -s "https://raw.githubusercontent.com/kubernetes-sigs/kustomize/master/hack/install_kustomize.sh"  | bash
```

+ Move the binary to PATH, so you can run kustomize command:
```
sudo mv kustomize /usr/local/bin/
```


## 2. Clone this repository
```
git clone https://github.com/brunononogaki/ansible-awx-install-with-operator.git
````

## 3. Install AWX Operator (Step 1)

### 3.1 Step 1
+ From the project folder, run the command below:
```
kustomize build ./awx/step1/ | kubectl apply -f -
```

+ Check if the pod was created in the namespace **awx**:
```
kubectl get pods -n awx
```

+ Wait about 1 minute until it is Running:
```
[admin@ansible ~]$ kubectl get pods -n awx
NAME                                               READY   STATUS    RESTARTS   AGE
awx-operator-controller-manager-666ddcf9c5-gq4wx   2/2     Running   0          66s
````

### 3.2 Step 2
+ From the project folder, run the command below:
```
kustomize build ./awx/step2/ | kubectl apply -f -
```

+ Check if more pods were created:
```
[admin@ansible ~]$ kubectl get pods -n awx
NAME                                               READY   STATUS              RESTARTS   AGE
awx-operator-controller-manager-666ddcf9c5-gq4wx   2/2     Running             0          3m26s
awx-postgres-15-0                                  1/1     Running             0          55s
awx-task-75d7448f5f-7dqpq                          0/4     Init:0/2            0          6s
awx-web-76dcd6466f-tbhh7                           0/3     ContainerCreating   0          7s 
````

+ Wait approximately 10 minutes for the pods to start. You can follow the logs with this command:
```
kubectl logs -f deployments/awx-operator-controller-manager -c awx-manager -n awx
```

+ When you see PLAY RECAP, it is done. Make sure unreachable and failed are 0:
```
PLAY RECAP *********************************************************************
localhost                  : ok=88   changed=0    unreachable=0    failed=0    skipped=85   rescued=0    ignored=1   
```

+ Check if the pods are running:
```
[admin@ansible ~]$ kubectl get pods -n awx
NAME                                               READY   STATUS      RESTARTS        AGE
awx-migration-24.6.1-8kztz                         0/1     Completed   0               8m3s
awx-operator-controller-manager-666ddcf9c5-gq4wx   2/2     Running     0               15m
awx-postgres-15-0                                  1/1     Running     0               10m
awx-task-6654cd5f64-mdq2k                          4/4     Running     0               9m28s
awx-web-7555d5f654-ltxb9                           3/3     Running     0               9m29s
```

## 4. Get the AWX admin password:
+ Run the following command to obtain the admin password for AWX interface
```
kubectl get secret awx-admin-password -o jsonpath="{.data.password}" -n awx | base64 --decode ; echo
```

+ The output will be a string. Copy this password.


## 5. Access AWX
+ You can access AWX using the port specified in the file awx.yaml (inside the folder step2)
```
http://<Linux_IP_Address>:30080/#/login
```
+ Use admin and the password generated in step 4.

## 6. Build a custom Execution Environment
If you want to build a custom execution environment, follow this other [guide](https://github.com/brunononogaki/ansible-awx-install-with-operator).