# Deployment

1. [Nội dung lab](#contents)

2. [Phân biệt Deployment và Replicase](#deployment-relicaset)

    2.1. [Sử Dụng Deployment Khi](#d-use)

    2.2. [Sử Dụng ReplicaSet Khi](#r-use)

    2.3. [Tính khả dụng](#available)

    2.4. [Cấu trúc file yaml](#yaml-structure)

3. [Triển khai và 1 số  thao tác khác](#deployment-do)

    3.1. [Cập nhật Deployment](#deployment-update)

    3.2. [Rollback Deployment](#deployment-rollback)

    3.3. [Scale Deployment](#deployment-scale)

## 1. Nội dung lab <a name="contents"></a>

* Lab về  deloyment

## 2. Phân biệt Deployment và Replicase <a name="deployment-relicaset"></a>

### 2.1. Sử Dụng Deployment Khi <a name="d-use"></a>

* Quản Lý Phiên Bản và Cập Nhật Liên Tục (Rolling Updates): Deployment hỗ trợ cập nhật liên tục và quản lý phiên bản. Nếu bạn cần triển khai các bản cập nhật mới cho ứng dụng mà không gây gián đoạn dịch vụ, Deployment là lựa chọn tốt nhất.

* Quản Lý Lịch Sử và Quay Lùi (Rollback): Deployment cho phép bạn quản lý lịch sử các cập nhật và cung cấp khả năng quay lùi về một trạng thái trước đó nếu cần.

* Tự Động Quản Lý ReplicaSet: Deployment tự động quản lý việc tạo và cập nhật các ReplicaSet. Điều này đơn giản hóa quy trình quản lý bản sao của Pods.

* Đơn Giản Hóa Quy Trình Cấu Hình: Khi bạn cần một cách đơn giản để quản lý các Pods và cập nhật, Deployment cung cấp một cấu hình đơn giản và mạnh mẽ.

### 2.2. Sử Dụng ReplicaSet Khi <a name="r-use"></a>

* Duy Trì Số Lượng Bản Sao Cố Định: Nếu bạn chỉ cần duy trì một số lượng cố định của Pods và không quan tâm đến cập nhật hoặc quản lý phiên bản, ReplicaSet có thể đáp ứng nhu cầu này.

* Quản Lý Cấp Thấp Hơn: Khi bạn cần kiểm soát chi tiết hơn trên từng Pod hoặc khi bạn làm việc với các mẫu Pods phức tạp và cụ thể không cần cập nhật, ReplicaSet là sự lựa chọn phù hợp.

* Trường Hợp Sử Dụng Cụ Thể và Thử Nghiệm: Trong một số trường hợp cụ thể, như khi triển khai một dịch vụ không thay đổi thường xuyên hoặc khi đang thực hiện thử nghiệm và phát triển, sử dụng ReplicaSet có thể là lựa chọn hợp lý.

### 2.3. Tính khả dụng <a name="available"></a>

Trong hầu hết các trường hợp, Deployment là lựa chọn tốt nhất cho quản lý ứng dụng trong Kubernetes, vì nó cung cấp các tính năng quản lý cấp cao hơn như cập nhật liên tục và quản lý phiên bản. ReplicaSet thường được sử dụng thông qua Deployment và ít khi được sử dụng trực tiếp, trừ khi bạn có những yêu cầu cụ thể liên quan đến quản lý Pod mà không cần cập nhật liên tục.

### 2.4. Cấu trúc file yaml <a name="yaml-structure"></a>

* Ví dụ tạo file: **1.myapp-deploy.yaml** có nội dung sau

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  # tên của deployment
  name: deployapp
spec:
  # số POD tạo ra
  replicas: 3

  # thiết lập các POD do deploy quản lý, là POD có nhãn  "app=deployapp"
  selector:
    matchLabels:
      app: deployapp

  # Định nghĩa mẫu POD, khi cần Deploy sử dụng mẫu này để tạo Pod
  template:
    metadata:
      name: podapp
      labels:
        app: deployapp
    spec:
      containers:
      - name: node
        image: ichte/swarmtest:node
        resources:
          limits:
            memory: "128Mi"
            cpu: "100m"
        ports:
          - containerPort: 8085
```

* Trong ví dụ này:

  * **apiVersion** đặt là apps/v1.

  * **kind** - loại xác định là Deployment.

  * **metadata** - nhận dạng: chứa thông tin nhận dạng cho 1 đối tượng như cần được đặt tả

  * **spec** - đặc tả: khai báo chi tiết cho các deployment/container/...

  * **replicas**: 3 (Deployment sẽ duy trì 3 bản sao của Pod).

  * **selector** sử dụng matchLabels để xác định Pods nào thuộc về  Deployment.

## 3. Triển khai và 1 số  thao tác khác <a name="deployment-do"></a>

* Dùng cmd sau để triển khai/tạo deployment từ file yaml.

```bash
kubectl apply -f 1.myapp-deploy.yaml
```

* Get thông tin deployments được tạo

```bash
kubectl get deployments -o wide
#hoặc
kubectl get deploy -o wide
```

![deployapp-get](https://i.imgur.com/J3YkfJn.png)

* Kiểm tra số  lượng replicas (các pod được tạo ra từ các deployment)

```bash
kubectl get pods -o wide
```

 >> Các pod từ deployment tạo ra sẽ được đánh lables như trong file **1.myapp-deploy.yaml** định nghĩa

![deployapp-pod](https://i.imgur.com/7SPyroY.png)

* Về  giao diện ở dashboard mình không đề  cặp tới vì chỉ thể hiện được thông tin và không thao tác được gì khác

![deployapp-gui]https://i.imgur.com/liqgtmD.png

### 3.1. Cập nhật Deployment <a name="deployment-update"></a>

### 3.2. Rollback Deployment <a name="deployment-rollback"></a>

### 3.3. Scale Deployment <a name="deployment-scale"></a>
