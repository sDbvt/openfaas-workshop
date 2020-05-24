# Lab 1 - Chuẩn bị cho OpenFaaS

- Bài lab này là điều kiện tiên quyết cho các lab sau
- Lab chỉ thực hiện trên Kubernetes

## Cài đặt Docker

Phải cài đặt được docker trước khi qua các bài lab sau

Tham khảo cách cài ở đây https://github.com/sDbvt/docker-labs/blob/master/install.md

Hoặc sử dụng: https://labs.play-with-docker.com/

## Cài đặt OpenFaaS CLI

curl -sLSf https://cli.openfaas.com | sudo sh

faas-cli help
faas-cli version

## Đăng nhập Docker Hub

Các images sau khi build ra sẽ được tự động đưa lên Docker Hub. Vì thế bạn cần phải

> Tạo một tài khoản ở [Docker Hub](https://hub.docker.com/)

Đăng nhập Hub
```
docker login
```

![alt text](../openfaas-workshop/img/docker-login.png "Docker Login")

Đặt prefix cho Docker Hub, việc này sẽ làm tối ưu thời gian trong việc sửa tên

Chỉnh sửa `~/.bashrc` hoặc `~/.bash_profile` - nếu chưa có thì tạo. Thêm dòng hoặc gõ lệnh vào
```
# Populate with your Docker Hub username
export OPENFAAS_PREFIX="your Docker Hub username"
```

## Tạo một single-node cluster trên Kubernetes

- Cài đặt kubectl

Gõ từng lệnh sau
```
export VER=$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)

curl -LO https://storage.googleapis.com/kubernetes-release/release/$VER/bin/linux/amd64/kubectl

chmod +x kubectl

mv kubectl /usr/local/bin/
```

Kiểm tra
```
kubectl version
```

*Tham khảo: https://kubernetes.io/docs/tasks/tools/install-kubectl/*

- Tạo cluster sử dụng k3d (thuộc k3s)

Dùng lệnh
```
curl -s https://raw.githubusercontent.com/rancher/k3d/master/install.sh | bash
```

![alt text](../openfaas-workshop/img/k3d.png "K3d")

Chạy k3d
```
k3d create
```

![alt text](../openfaas-workshop/img/k3d-create.png "K3d create")

*Tham khảo: https://github.com/rancher/k3d*

## Deploy OpenFaaS

- Cài OpenFaaS qua `arkade`

Cài arkade
```
curl -SLsf https://dl.get-arkade.dev/ | sudo sh
```

Cài ứng dụng OpenFaaS trên arkade
```
arkade install openfaas --load-balancer

#hoặc

arkade install openfaas
```

Kết quả sau khi tạo OpenFaaS trên arkade
```
root@ubuntu:~# arkade install openfaas
Using kubeconfig: /home/tr/.config/k3d/k3s-default/kubeconfig.yaml
Using helm3
Node architecture: "amd64"
Client: "x86_64", "Linux"
2020/05/24 23:57:06 User dir established as: /home/tr/.arkade/
https://get.helm.sh/helm-v3.1.2-linux-amd64.tar.gz
/home/tr/.arkade/bin/helm3/linux-amd64 linux-amd64/
/home/tr/.arkade/bin/helm3/helm linux-amd64/helm
/home/tr/.arkade/bin/helm3/README.md linux-amd64/README.md
/home/tr/.arkade/bin/helm3/LICENSE linux-amd64/LICENSE
2020/05/24 23:58:09 extracted tarball into /home/tr/.arkade/bin/helm3: 3 files, 0 dirs (56.217882924s)
"openfaas" has been added to your repositories
Hang tight while we grab the latest from your chart repositories...
...Successfully got an update from the "openfaas" chart repository
Update Complete. ⎈ Happy Helming!⎈ 
VALUES values.yaml
Command: /home/tr/.arkade/bin/helm3/helm [upgrade --install openfaas openfaas/openfaas --namespace openfaas --values /tmp/charts/openfaas/values.yaml --set clusterRole=false --set openfaasImagePullPolicy=IfNotPresent --set serviceType=NodePort --set gateway.directFunctions=true --set operator.create=false --set faasnetes.imagePullPolicy=Always --set basicAuthPlugin.replicas=1 --set gateway.replicas=1 --set ingressOperator.create=false --set queueWorker.replicas=1 --set queueWorker.maxInflight=1 --set basic_auth=true]
Release "openfaas" does not exist. Installing it now.
NAME: openfaas
LAST DEPLOYED: Sun May 24 23:58:37 2020
NAMESPACE: openfaas
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
To verify that openfaas has started, run:

  kubectl -n openfaas get deployments -l "release=openfaas, app=openfaas"
=======================================================================
= OpenFaaS has been installed.                                        =
=======================================================================

# Get the faas-cli
curl -SLsf https://cli.openfaas.com | sudo sh

# Forward the gateway to your machine
kubectl rollout status -n openfaas deploy/gateway
kubectl port-forward -n openfaas svc/gateway 8080:8080 &

# If basic auth is enabled, you can now log into your gateway:
PASSWORD=$(kubectl get secret -n openfaas basic-auth -o jsonpath="{.data.basic-auth-password}" | base64 --decode; echo)
echo -n $PASSWORD | faas-cli login --username admin --password-stdin

faas-cli store deploy figlet
faas-cli list

# For Raspberry Pi
faas-cli store list \
 --platform armhf

faas-cli store deploy figlet \
 --platform armhf

# Find out more at:
# https://github.com/openfaas/faas

Thanks for using arkade!
root@ubuntu:~# 
```

- Đăng nhập vào OpenFaaS Gateway

Kiểm tra Gateway đã sẵn sàng
```
kubectl rollout status -n openfaas deploy/gateway
```

Chạy OpenFaaS gateway dưới nền
```
kubectl port-forward -n openfaas svc/gateway 8080:8080 &
```

> Ta có gateway URL mặc định là: http://127.0.0.1:8080

Lệnh này dùng để đưa Gateway ra ngoài thông qua địa chỉ LoadBalancer's IP hoặc DNS
```
kubectl get svc -o wide gateway-external -n openfaas
```

Lấy mật khẩu để đăng nhập OpenFaaS UI
```
PASSWORD=$(kubectl get secret -n openfaas basic-auth -o jsonpath="{.data.basic-auth-password}" | base64 --decode; echo)

echo -n $PASSWORD | faas-cli login --username admin --password=$PASSWORD
```

Kiểm tra OpenFaaS có hoạt động hay chưa:
```
faas-cli list
```

- Sửa OpenFaaS URL

Chỉnh sửa `~/.bashrc` hoặc `~/.bash_profile` - nếu chưa có thì tạo. Thêm dòng hoặc gõ lệnh vào
```
export OPENFAAS_URL="http://127.0.0.1:8080" # populate as above
```

Hết. Qua [lab2](lab2.md)