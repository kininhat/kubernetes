# Tìm hiểu chi tiết về  ReplicaSet - Horizontal Pod Autoscaler

1. [Nội dung lab](#contents)

2. [ReplicaSet](#ReplicaSet)

    2.1. [Cách thức hoạt động](#how-work)

    2.2. [Cấu trúc file yaml](#yaml-structure)

    2.3. [Triển khai và 1 số  thao tác khác](#replicaset-do)

3. [Horizontal Pod Autoscaler với ReplicaSet](#hpa-r)

    3.1. [Thực hiện qua cmd (ít dùng)](#hpa-r-cmd)

    3.2. [Thực hiện qua yaml (thường dùng)](#hpa-r-yaml)

## 1. Nội dung lab <a name="contents"></a>

* Ôn lại khái niệm và cmd thông dụng
* Lab về  relicaset và Horizontal Pod Autoscaler

## 2. ReplicaSet <a name="ReplicaSet"></a>

ReplicaSet trong Kubernetes là một cấu trúc quan trọng được sử dụng để quản lý và duy trì số lượng bản sao (replicas) của Pod trong một ứng dụng. Nó đảm bảo rằng một số lượng nhất định của các Pods luôn chạy tại mọi thời điểm.

![replicaset](https://raw.githubusercontent.com/xuanthulabnet/learn-kubernetes/master/imgs/kubernetes052.png)

### 2.1 Cách thức hoạt động <a name="how-work"></a>

Khi định nghĩa một ReplicaSet (định nghĩa trong file .yaml) gồm các trường thông tin, gồm có trường selector để chọn ra các các Pod theo label, từ đó nó biết được các Pod nó cần quản lý (số lượng POD có đủ, tình trạng các POD). Trong nó nó cũng định nghĩa dữ liệu về  Pod trong spec template, để nếu cần tạo Pod mới nó sẽ tạo từ template đó. Khi ReplicaSet tạo, chạy, cập nhật nó sẽ thực hiện tạo / xóa POD với số lượng cần thiết trong khai báo (repilcas).

### 2.2 Cấu trúc file yaml  <a name="yaml-structure"></a>

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: rsapp
spec:
  replicas: 3
  selector:
    matchLabels:
      app: rsapp
  template:
    metadata:
      name: rsapp
      labels:
        app: rsapp
    spec:
      containers:
      - name: app
        image: ichte/swarmtest:node
        resources:
          limits:
            memory: "128Mi"
            cpu: "100m"
        ports:
          - containerPort: 8085
```

* Giải thích các thông số:
  
  * **apiVersion**: apps/v1: Sử dụng phiên bản apps/v1 của Kubernetes API.
  
  * **kind**: ReplicaSet: Loại tài nguyên đang được định nghĩa là ReplicaSet.
  
  * **metadata**: Chứa thông tin như **name** (tên của ReplicaSet = rsapp) và **labels** (nhãn để xác định ReplicaSet = rsapp).

  * **spec**:

    * **replicas: 3** -> Yêu cầu duy trì 3 bản sao của Pod.

    * **selector.matchLabels.app: rsapp**: ReplicaSet sẽ quản lý các Pods có nhãn app: rsapp.

    * **template**: Mô tả cách tạo Pods. Phần này bao gồm:

      * các thiết lập như containers (chứa thông tin về các container trong Pod)

      * name (tên của Pod)

      * và labels (nhãn để  liên kết với ReplicaSet).

### 2.3 Triển khai và 1 số  thao tác khác <a name="replicaset-do"></a>

* Triển khai

```bash
kubectl apply -f 2.rs.yaml
```

* Để lấy các ReplicaSet thực hiện lệnh

* Thông tin chi tiết về ReplicaSet có tên **rsapp**

* Xem trên giao diện

* Từ thông tin chi tiết ReplicaSet biết được các thông tin như:

  * số Replicas đang có trên số lương yêu cầu
  * tình trạng các Pod (pod status)
  * nhãn chọn Pod mà nó quản lý.

* Liệt kê các POD có nhãn "**app=rsapp**"

```bash
kubectl get po -l "app=rsapp"
```

* Rồi kiểm tra chi tiết (desribe) một POD

* Bạn thấy dòng Controlled By: ReplicaSet/rsapp điều này có nghĩa POD được kiểm soát điều khiển bởi ReplicaSet có tên rsapp.

* Nếu bạn xóa POD,ReplicaSet rsapp sẽ thay thế nó bởi một POD mới. Nếu xóa ReplicaSet rsapp thì các POD cũng xóa theo

## 3. Horizontal Pod Autoscaler với ReplicaSet <a name="hpa-r"></a>

* Horizontal Pod Autoscaler là chế độ tự động scale (nhân bản POD) dựa vào mức độ hoạt động của CPU đối với POD, nếu một POD quá tải - nó có thể nhân bản thêm POD khác và ngược lại - số nhân bản dao động trong khoảng min, max cấu hình

### 3.1. Thực hiện qua cmd (ít dùng) <a name="hpa-r-cmd"></a>

* Ví dụ, với ReplicaSet rsapp trên đang thực hiện nhân bản có định 3 POD (replicas), nếu muốn có thể tạo ra một HPA để tự động scale (tăng giảm POD) theo mức độ đang làm việc CPU, có thể dùng lệnh sau:

```bash
kubectl autoscale rs rsapp --max=2 --min=1
#Lệnh trên tạo ra một hpa có tên rsapp, có dùng tham chiếu đến ReplicaSet có tên rsapp để scale các POD với thiết lập min, max các POD
```

* Để liệt kê các hpa gõ lệnh

```bash
kubectl get hpa
```

### 3.2. Thực hiện qua yaml (thường dùng) <a name="hpa-r-yaml"></a>

* Để linh loạt và quy chuẩn, nên tạo ra HPA (HorizontalPodAutoscaler) từ cấu hình file yaml.
* Ví dụ file: [2.hpa.yaml](./pod-lab/2.hpa.yaml)
* Chạy HPA:

```bash
kubectl apply -f 2.hpa.yaml
```

* Tham khảo tại link: <https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale>

>> Mặc dù có thể sử dụng ReplicaSet một cách độc lập, tuy nhiên trong triển khai hiện nay hay dùng Deployment, với Deployment nó sở hữu một ReplicaSet riêng. Bài tiếp theo sẽ nói về  Deployment.

>> Bài được viết khi tự học và tham khảo từ nguồn: <https://xuanthulab.net/replicaset-va-horizontalpodautoscaler-trong-kubernetes.html>.
