# Tìm hiểu chi tiết về  Pod,Node,kubectl qua cmd và nhắc lại khái niệm

## 1. Nội dung lab <a name="contents"></a>

* Ôn lại khái niệm và cmd thông dụng
* Lab về pod chứa 1 container và pod chứa nhiều container
* Truy cập pod ở trình duyệt thông qua proxy

## 2. Khái niệm và cmd

### 2.1. kubectl

### 2.1.1. Syntax

```bash
kubectl [command] [TYPE] [NAME] [flags]
```

* where **command**, **TYPE**, **NAME**, and **flags** are:

  * **command**: Specifies the operation that you want to perform on one or more resources, _for example create, get, describe, delete._

  * **TYPE**: Specifies the resource type. Resource types are case-insensitive and you can specify the singular, plural, or abbreviated forms. Normaly: pods, namespace, node, pvc, ... For example, the following commands produce the same output:

  ```bash
    kubectl get pod pod1
    kubectl get pods pod1
    kubectl get po pod1
   ```
  
  * **NAME**: Specifies the name of the resource. Names are case-sensitive. If the name is omitted, details for all resources are displayed, for _example kubectl get pods_

    * When performing an operation on multiple resources, you can specify each resource by type and name or specify one or more files:

    * To specify resources by type and name:

      * To group resources if they are all the same type: TYPE1 name1 name2 name<#>.
      * Example: kubectl get pod example-pod1 example-pod2

      * To specify multiple resource types individually: TYPE1/name1 TYPE1/name2 TYPE2/name3 TYPE<#>/name<#>.
      * Example: kubectl get pod/example-pod1 replicationcontroller/example-rc1

    * To specify resources with one or more files: -f file1 -f file2 -f file<#>

      * Use YAML rather than JSON since YAML tends to be more user-friendly, especially for configuration files.
      * Example: kubectl get -f ./pod.yaml

  * **flags**: Specifies optional flags. For example, you can use the -s or --server flags to specify the address and port of the Kubernetes API server.

> Caution: Flags that you specify from the command line override default values and any corresponding environment variables.
> Tham khảo tại link: <https://kubernetes.io/docs/reference/kubectl/>

### 2.2 Node

### 2.2.1. Khái niệm

* Trong Kubernetes Node là đơn vị nhỏ nhất xét về phần cứng. Nó là một máy vật lý hay máy ảo (VPS) trong cụm máy (cluster).

### 2.2. Thực hành

* Để xem các node tron cluster

```bash
kubectl get nodes
#hoặc xem chi tiết hơn với cmd:
kubectl get nodes -o wide
```

![k-get-node](https://i.imgur.com/HtxQ8QH.png)

* Để lấy thông tin chi tiết về một Node nào đó như tài nguyên bộ nhớ, CPU, các container ... sử dụng lệnh:

```bash
kubectl describe node tên-node
#Example
kubectl describe node k-worker1.nhatkini.online
```

![k-d-n-nn1](https://i.imgur.com/NmDFhfT.png)

![k-d-n-nn2](https://i.imgur.com/galKnwp.png)

### 2.3. Nhãn của Node

* Labels trong Kubernetes đóng vai trò quan trọng trong việc tổ chức và quản lý các tài nguyên. Labels là các cặp khóa-giá trị mà bạn gán cho các đối tượng như Pods, Services, và Replication Controllers. Dưới đây là một số ứng dụng chính của labels:

* Nhóm và Lọc Tài Nguyên:

  * Labels cho phép bạn nhóm các tài nguyên dựa trên các đặc điểm chung, giúp dễ dàng quản lý và lọc chúng. Ví dụ, bạn có thể gán labels cho các pods dựa trên môi trường (ví dụ: env=production, env=staging), dự án, hoặc bất kỳ tiêu chí nào khác phù hợp với nhu cầu của bạn.

* Chọn và Liên Kết Tài Nguyên:

  * Labels được sử dụng để xác định các tài nguyên cần được liên kết với nhau. Ví dụ, một Service trong Kubernetes sử dụng labels để xác định các Pods mà nó cần phải chuyển tiếp lưu lượng mạng.

* Triển Khai và Quản Lý Tự Động:

  * Các công cụ triển khai và quản lý tự động như Replication Controllers, Deployments, và StatefulSets sử dụng labels để xác định các Pods cần quản lý. Điều này giúp tự động hóa việc triển khai và quản lý tài nguyên.

* Chính Sách Bảo Mật và Kiểm Soát Truy Cập:

  * Labels có thể được sử dụng để áp dụng chính sách bảo mật và kiểm soát truy cập. Ví dụ, bạn có thể cấu hình Network Policies để chỉ cho phép lưu lượng mạng giữa các Pods có labels cụ thể.

* Giám Sát và Ghi Nhật Ký:

  * Trong giám sát và ghi nhật ký, labels giúp bạn phân loại và lọc dữ liệu dễ dàng hơn. Ví dụ, bạn có thể lọc nhật ký hoặc chỉ số hiệu suất dựa trên labels của Pods hoặc Nodes.

* Sắp Xếp và Lập Lịch:

  * Labels có thể được sử dụng để hỗ trợ việc lập lịch Pods trên các Nodes cụ thể trong cluster. Ví dụ, bạn có thể sử dụng labels để chỉ định Pods nên chạy trên Nodes có tài nguyên phần cứng cụ thể.

* Labels trong Kubernetes cung cấp một cách linh hoạt và mạnh mẽ để quản lý và tổ chức các tài nguyên, giúp các quản trị viên và nhà phát triển dễ dàng quản lý các tài nguyên phức tạp trong môi trường Kubernetes.

* Thiết lập nhãn

  ```bash
  kubectl label node worker1.xtl tennhan=giatrinhan
  ```

* Lấy các tài nguyên có nhãn nào đó

  ```bash
  kubectl get node -l "tennhan=giatrinhan"
  ```

* Xóa nhãn

  ```bash
  kubectl label node worker1.xtl tennhan-
  ```

> Tham khảo từ link: <https://xuanthulab.net/tim-hieu-ve-pod-va-node-trong-kubernetes.html>

### 2.3. Pod

### 2.3.1. Khái niệm

* Kubernetes không chạy các container một cách trực tiếp, thay vào đó nó bọc một hoặc vài container vào với nhau trong một cấu trúc gọi là POD. Các container cùng một pod thì chia sẻ với nhau tài nguyên và mạng cục bộ của pod.

![pod](https://i.imgur.com/ITE4n19.png)

* Pod là thành phần đơn vị (nhỏ nhất) để Kubernetes thực hiện việc nhân bản (replication), có nghĩa là khi cần thiết thì Kubernetes có thể cấu hình để triển khai nhân bản ra nhiều pod có chức năng giống nhau để tránh quá tải, thậm chí nó vẫn tạo ra nhiều bản copy của pod khi không quá tải nhằm phòng lỗi (ví dụ node bị die).

* Pod có thể có nhiều container mà pod là đơn vị để scale (có nghĩa là tất cả các container trong pod cũng scale theo) nên nếu có thể thì cấu hình ứng dụng sao cho một Pod có ít container nhất càng tốt.

  * Cách sử dụng hiệu quả và thông dụng là dùng loại Pod trong nó chỉ chạy một container.
  
  * Pod loại chạy nhiều container trong đó thường là đóng gọi một ứng dụng xây dựng với sự phối hợp chặt chẽ từ nhiều container trong một khu vực cách ly, chúng chia sẻ tài nguyên ổ đĩa, mạng cho nhau.

#### 3.1.1 Mạng / Volume với POD

* Mỗi POD được gán một địa chỉ IP, các container trong Pod chia sẻ cùng địa chỉ IP này. Các container trong cùng một Pod có thể liên lạc với nhau qua localhost (Giống PHP truy cập đến MySQL khi hai thành phần này cài đặt trong một máy).

* Một Pod có thể có nhiều ổ đĩa được chia sẻ để các container có thể truy cập đọc/ghi dữ liệu.

### 3.2 Lab

### 3.2.1. Truy cập pop thông qua proxy

* Truy cập pod được tạo thông qua service hoặc proxy

* Ở bài viết này sẽ truy cập qua proxy bằng cách chạy cmd sau:
  * **kubectl proxy** (#Nếu đang lab ở local thì dùng cmd này)
  * Hoặc
  * **kubectl proxy --address="0.0.0.0" --accept-hosts='^*$'** (#Nếu đang lab ở thiết bị có ip thật public được thì dùng cmd này, bài viết dùng cmd này)

* Link truy cập dạng: <http://domain:8001/api/v1/namespaces/tên_namespamce/pods/tên_pod:port_listsen/proxy/>

#### 3.2.2 Tạo pod 1 container

* Có thể tạo ra Pod một cách trực tiếp (thực tế ít dùng) hoặc qua triển khai Deployment để Controller thực hiện, khi Pod được tạo ra nó được lên kế hoạch chạy trong một Node nào đó của cụm máy, nó tồn tại cho đến khi tiến trình chạy kết thúc hoặc bị xóa, bị lỗi Node ...

* Controller: Khi tạo các Pod không trực tiếp, tức thông qua cấu hình Deployment thì có một Controller thực hiện việc tạo quản lý các Pod, thực hiện cách này nó cung cấp khả năng cập nhật giám sát, phục hồi ...

### 3.2.2.1 Tạo pod mới

Có nhiều cách để  tạo pod mới, thường thấy là tạo qua cmd với file yaml

#### a. Tạo pod qua cmd với file yaml

* Ví dụ tạo file: **1-swarmtest-node.yaml**

```yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    app: app1
    ungdung: ungdung1
  name: ungdungnode
spec:
  containers:
  - name: c1
    image: ichte/swarmtest:node
    resources:
      limits:
        memory: "150M"
        cpu: "100m"
    ports:
      - containerPort: 8085
  ```

* File trên khai báo một Pod tại namspace default (mặc định nếu không truyền namespace khác vào), đặt tên là ungdungnode, gán nhãn app: app1, ungdung: ungdung1 trong Pod chạy một Container từ image ichte/swarmtest:php, cổng của Container 8085

* Sử dụng cmd sau để tạo pod: **kubectl apply -f 1-swarmtest-node.yaml**

* Xem chi tiết quá trình tạo bằng cách sử dụng cmd từ kubectl (**kubectl get pods -o wide**) hoặc k9s

#### a. Tạo pod qua giao diện

Sau khi cài đặt dashboard nhiều thao tác có thể  thực hiện trực tiếp tại giao diện thay vì ssh vào node: master để thực hiện.
![pod-create-1](https://i.imgur.com/JMF9Bb7.png)

* Tạo bằng cách dán trực tiếp file yaml vào ở chức năng **Create from input**
![pod-create-2-upload-1](https://i.imgur.com/ta78ASe.png)

![pod-create-2-upload-2] https://i.imgur.com/ttaIDZM.png

![pod-create-2-upload-3](https://i.imgur.com/3NBt9Lm.png)

* Tạo bằng cách upload file yaml ở chức năng **Create from file**

![pod-create-2-upload-4](https://i.imgur.com/tqNbuHy.png)

* Tạo bằng cách điền form ở chức năng **Create from from**

Phần này mình sẽ giới thiệu ở phần sau

#### 3.2.3 Tạo và thao tác với pod nhiều container

* Tạo pod từ file: [4-nginx-swamtest.yaml](./pod-lab-more-containers/4-nginx-swamtest.yaml) với cmd: **kubectl apply -f 4-nginx-swamtest.yaml**

* Trong đó container n1, chạy từ image nginx:1.17.6, container s1 từ image ichte/swarmtest:node

* 1 số thao tác với pod nhiều container:

  * Kiểm tra chi tiết pod:
    * Bằng cmd: **kubectl get pods nginx-swarmtest -o yaml**
    * Hoặc bằng trình duyệt, điều kiện là đã chạy kube proxy trước đó, link truy cập dạng: <http://domain:8001/api/v1/namespaces/tên_namespamce/pods/tên_pod>
    * Hoặc bằng dashboard

  * Thực hiện cmd, cho container chỉ định: **kubectl exec tên_pod -c tên_container -- cmd**
    * Ví dụ, thực hiện cmd nginx -v ở container n1, container s1 không chạy nginx nên sẽ báo lỗi

    ![pod-exec-container-define](https://i.imgur.com/uDGqkFa.png)
