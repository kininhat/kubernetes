# LAB02-Dashboard

1. [Nội dung lab](#contents)

2. [Cài đặt dashboard](#install-dashboard)

    a. [Điều chỉnh config dashboard và tạo user](#dashboard-cf-u)

    b. [Run cmd deploy from config file](#dashboard-deploy)

## 1. Nội dung lab <a name="contents"></a>

* Cài đặt web-ui-dashboard cho kubernetes
* Tạo user & password & token để  đăng nhập vào link quản trị của dashboard
* Public link dashboard để client kết nối và thực hiện quản trị
* Cài đặt SSL cho link dashboard: <https://k-master.nhatkini.online:31000>

* Nhắc lại mô hình đang lab hiện tại

Hostname & Vai trò | Thông tin | IP
:---------:|:----------:|:---------:
 k-master.nhatkini.online | HĐH Ubuntu 20.04, Docker CE, Kubernetes | 222.255.214.25
 k-worker1.nhatkini.online | HĐH Ubuntu 20.04, Docker CE, Kubernetes | 103.90.224.251
 k-worker2.nhatkini.online | HĐH Ubuntu 20.04, Docker CE, Kubernetes | 103.90.225.65

## 2. Cài đặt dashboard <a name="install-dashboard"></a>

* Chỉ áp dụng trên master
* Hiện tại bản stable kiến nghị trên <https://kubernetes.io> kiến nghị là bản dashboard-v.2.7.0.yaml, nên bài lab sẽ sử  dụng phiên bản này
* Link chi tiết: <https://raw.githubusercontent.com/kubernetes/dashboard/v2.7.0/aio/deploy/recommended.yaml>

* Tạo thư mục chứa file dashboard và ssl key

```bash
mkdir -p /etc/kubernetes/dashboard/certs
```

### a. Điều chỉnh config dashboard và tạo user <a name="dashboard-cf-u"></a>

* Tải file dashboard

```bash
wget -O /etc/kubernetes/dashboard/dashboard-v2.7.0.yaml https://raw.githubusercontent.com/kubernetes/dashboard/v2.7.0/aio/deploy/recommended.yaml
```

* Trong file được tải về  tìm và điểu chỉnh block **kind: Service** với **name: kubernetes-dashboard** đổi kiểu sang NodePort để truy cập https vào port được định nghĩa thay vì sử dụng mặc định

```bash
---

kind: Service
apiVersion: v1
metadata:
  labels:
    k8s-app: kubernetes-dashboard
  name: kubernetes-dashboard
  namespace: kubernetes-dashboard
spec:
  # Đổi kiểu sang NodePort
  type: NodePort
  ports:
    - port: 443
      targetPort: 8443
      # Chọn cổng truy cập qua Node là 3100
      nodePort: 31000
  selector:
    k8s-app: kubernetes-dashboard

---
```

* Xóa Secret có tên kubernetes-dashboard-certs

```bash
#---

#apiVersion: v1
#kind: Secret
#metadata:
#  labels:
#    k8s-app: kubernetes-dashboard
#  name: kubernetes-dashboard-certs
#  namespace: kubernetes-dashboard
#type: Opaque
```

* Thêm SSL khi truy cập website dashboard trong block: **kind: Deployment** với **name: kubernetes-dashboard** phần **args** thay đổi comment **auto-generate-certificates** và bổ  sung  tls-cert-file and tls-key-file arguments.

> **File ssl trong bộ lab được lấy từ certbot với định dạng: cert.pem and privkey.pem và cần đổi tên lại thành tls.crt and tls.key**
> **Upload them to certs where our kubectl tool is installed. certs directory is: /etc/kubernetes/dashboard/certs**

```bash
---

kind: Deployment
apiVersion: apps/v1
metadata:
  labels:
    k8s-app: kubernetes-dashboard
  name: kubernetes-dashboard
  namespace: kubernetes-dashboard
spec:
  replicas: 1
  revisionHistoryLimit: 10
  selector:
    matchLabels:
      k8s-app: kubernetes-dashboard
  template:
    metadata:
      labels:
        k8s-app: kubernetes-dashboard
    spec:
      securityContext:
        seccompProfile:
          type: RuntimeDefault
      containers:
        - name: kubernetes-dashboard
          image: kubernetesui/dashboard:v2.7.0
          imagePullPolicy: Always
          ports:
            - containerPort: 8443
              protocol: TCP
          args:
            #- --auto-generate-certificates
            - --namespace=kubernetes-dashboard
            - --tls-cert-file=/tls.crt
            - --tls-key-file=/tls.key
            # Uncomment the following line to manually specify Kubernetes API server Host
            # If not specified, Dashboard will attempt to auto discover the API server and connect
            # to it. Uncomment only if the default does not work.
            # - --apiserver-host=http://my-address:port
          volumeMounts:
            - name: kubernetes-dashboard-certs
              mountPath: /certs
              # Create on-disk volume to store exec logs
            - mountPath: /tmp
              name: tmp-volume
          livenessProbe:
            httpGet:
              scheme: HTTPS
              path: /
              port: 8443
            initialDelaySeconds: 30
            timeoutSeconds: 30
          securityContext:
            allowPrivilegeEscalation: false
            readOnlyRootFilesystem: true
            runAsUser: 1001
            runAsGroup: 2001
      volumes:
        - name: kubernetes-dashboard-certs
          secret:
            secretName: kubernetes-dashboard-certs
        - name: tmp-volume
          emptyDir: {}
      serviceAccountName: kubernetes-dashboard
      nodeSelector:
        "kubernetes.io/os": linux
      # Comment the following tolerations if Dashboard must not be deployed on master
      tolerations:
        - key: node-role.kubernetes.io/master
          effect: NoSchedule

---
```

* Tạo user: **admin-user** để  login qua token và quản trị dashboard thông qua file: [admin-user.yaml](./dashboard/admin-user.yaml)

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: admin-user
  namespace: kubernetes-dashboard

---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: admin-user
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
- kind: ServiceAccount
  name: admin-user
  namespace: kubernetes-dashboard
```

### b. Run cmd deploy from config file <a name="dashboard-deploy"></a>

_**Truy cập vào đường dẫn: /etc/kubernetes/dashboard để  chạy các cmd**_

* Deploy dashboard

```bash
kubectl apply -f dashboard-v2.7.0.yaml
```

![deploy-d](https://i.imgur.com/45gJ28n.png)

* Create and view/describe secret

```bash
kubectl create secret generic kubernetes-dashboard-certs --from-file=certs -n kubernetes-dashboard
kubectl describe secret -n kubernetes-dashboard kubernetes-dashboard-certs
```

![create-view-describe-secret](https://i.imgur.com/QXtAVSJ.png)

* Deploy admin-user

```bash
kubectl apply -f admin-user.yaml
```

![deloy-admin](https://i.imgur.com/MP7kFeP.png)

* Get admin-user token

```bash
kubectl -n kubernetes-dashboard create token admin-user
```

![gen-admin-token](https://i.imgur.com/9CfPh7E.png)

> Create user and get user token at link: <https://github.com/kubernetes/dashboard/blob/master/docs/user/access-control/creating-sample-user.md>
> Mỗi lẫn chạy cmd này thì token sẽ được tạo mới

* Login thông qua link: <https://k-master.nhatkini.online:31000>, ảnh như đính kèm: <https://i.imgur.com/UR12XWl.png>

* 1 số  cmd để  kiểm tra:
  * Kiểm tra các pods đang hoạt của các pods trong namespace: kubernetes-dashboard
  
  ```bash
    kubectl get pods -n kubernetes-dashboard
  ```

  > Nếu các pods trong kubernetes-dashboard này nếu trong 1phút chưa tạo xong khả năng lỗi rất cao

  * Kiểm tra các pods đang hoạt của các pods trong tất cả namespace

  ```bash
    kubectl get pods --all-namespaces
  ```
