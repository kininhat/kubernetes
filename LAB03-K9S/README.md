# LAB03-K9S

* K9S là công cụ CLI để quản lý, tương tác với K8S (Kubernetes) với giao diện dòng lệnh trên Terminal, tuy nhiên vẫn rất trực quan.

* Download k9s tại link: <https://github.com/derailed/k9s/releases>

> Hiện tại bài viết đang sử dụng bản: Release v0.30.4

```bash
wget https://github.com/derailed/k9s/releases/download/v0.30.4/k9s_linux_amd64.deb
dpkg -i k9s_linux_amd64.deb
```

* Sau khi cài đặt xong sử dụng cmd: **k9s**

```bash
k9s
```

* Sử dụng K9S

  * Sau khi cài đặt k9s gõ lệnh k9s vào terminal nó xuất hiện giao diện như trên, bạn chỉ cần đảm bảo nếu kubectl được cấu hình kết nối đến Cluster chính xác thì k9s cũng dùng chính cấu hình đó để kết nối để Kubernetes.  Giao diện cmd trực quan sẽ hiển thị như ảnh đính kèm: <https://i.imgur.com/GQt7SEx.png>

  * Mặc định nó đang liệt kê các Pod chạy trên Cluster, nếu muốn liệt kê các loại tài nguyên khác hãy nhấn shift + : rồi gõ loại tài nguyên muốn hiện thị như: pod, node, srv, ev ...

Muốn biết các tên này gõ lệnh:

```bash
kubectl api-resources
```

* Tùy loại tài nguyên đang liệt kê mà có những lệnh tương ứng theo các dòng bạn chọn, những lệnh này thể hiện bởi các phím tắt gợi ý ngay trên đầu giao diện. Ví dụ khi bạn bấm các phím mũi tên chọn vào một POD đang chạy, nó có các phím gợi ý sẵn như khi nhấn phím:

  * ctrl+d xóa POD
  * d tương ứng lệnh describe của POD
  * l xem log
  * 0 xem trên tất cả các namespace
  * 1 xem cho namespace mặc định
...

* Nói chung trong quá trình làm việc, các lệnh nó luôn gợi ý trên màn hình để hướng dẫn thực hiện, những lệnh chung nhất như

* ? hiện thị bảng lệnh

* ctrl+c thoat k9s
