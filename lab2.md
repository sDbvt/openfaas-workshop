# Lab 2 - Kiểm tra

- Cách deploy và invoke function
  - Trên Store
  - Trên CLI
- Trước khi vào bài, hãy tạo thư mục mới: `mkdir -p OFlab2 && cd OFlab2`

## Sử dụng UI Portal

Trước hết, phải set cho $OPENFAAS_URL. Kiểm tra

```
echo $OPENFAAS_URL
```

Nếu chưa có gì. Thì set lại

```
export OPENFAAS_URL="http://127.0.0.1:8080"
```

Cùng deploy một vài function có sẵn trên OpenFaaS

```
faas-cli deploy -f https://raw.githubusercontent.com/openfaas/faas/master/stack.yml
```

Kiểm tra thử một function, function Markdown

Input
```
## The **OpenFaaS** _workshop_
```

Sau khi ấn `Invoke`, cho ra Output
```
<h2>The <strong>OpenFaaS</strong> <em>workshop</em></h2>
```

> Chức năng của function này sẽ đưa đoạn trên từ format markdown sang HTML

Ảnh minh họa

![alt text](/img/OpenFaaS-Portal.png "OpenFaaS Portal")

Tìm hiểu sâu hơn về các thông số có trong Portal
- Status: thể hiện fucntion có sẵn sàng để chạy không, nếu có sẽ là Ready
- Replicas: số lượng bản sao của function đang chạy trên cluster
- Image: image và version, image này đã được chia sẻ trên Docker Hub
- Invocation count: số lần gọi đến function, tự động cập nhật sau mỗi 5s

## Sử dụng Store

Ở phần trên, ta deploy function qua lệnh của faas-cli. Lần này ta deploy function trực tiếp trên Store của UI

- Click *Deploy New Function*
- Click *From Store*
- Click *Figlet* hoặc chọn *figlet* trong search box và click *Deploy*

Ảnh minh họa

![alt text](/img/deploy-Store-OpenFaaS.png)

## Một vài lệnh về faas-cli

Như bạn đã thực hiện, OPENFAAS_URL mặc định sẽ là http://127.0.0.1:8080/

Bạn cũng có thể thay đổi chúng thành http://openfaas.endpoint.com:8080, chỉ cần thêm flag

```
faas deploy --gateway
```

- Liệt kê các function đã deploy

Dùng lệnh
```
faas-cli list
```

Bạn sẽ thấy các function đã deploy, số lần gọi và bản sao của chúng. Giờ ta thêm flag

```
faas-cli list --verbose

# hoặc

faas-cli list -v
```

Sẽ hiển thị thêm các Docker images.

## Invoke function qua CLI

Chọn 1 function có trong danh sách `faas-cli list`, ví dụ `markdown` 

```
faas-cli invoke markdown
```

Nhập Input vào, ấn Ctrl + D khi đã nhập xong.

Hoặc bạn có thể dùng lệnh pipeline

```
echo Hi | faas-cli invoke markdown

uname -a | faas-cli invoke markdown

cat lab2.md | faas-cli invoke markdown
```

## Bảng giám sát (Monitoring dashboard)

OpenFaaS theo dõi các số liệu thông qua Prometheus. Để có giao diện dễ dàng theo dõi, phần này sẽ sử dụng mã nguồn mở miến phí Grafana

Deploy OpenFaaS Grafana

Tạo pod grafana
```
kubectl -n openfaas run --image=stefanprodan/faas-grafana:4.6.3 --port=3000 grafana
```

Deploy images vừa tạo
```
kubectl -n openfaas create deploy --image=stefanprodan/faas-grafana:4.6.3 grafana
```

Expose cho ra cổng 3000, với type là NodePort
```    
kubectl -n openfaas expose deployment grafana --type=NodePort --port=3000 --name=grafana
```

Cho grafana chạy dưới nền.
```
kubectl port-forward deployment/grafana 3000:3000 -n openfaas &
```

Giờ bạn có thể truy cập Grafana qua http://127.0.0.1:3000

![alt text](/img/grafana-login.png)

Đăng nhập Grafana bằng tài khoản mặc định: `admin/admin`

![alt text](/img/grafana.png)

## Tổng kết

**Muốn deploy và invoke trên**
- Store: vào trang UI Portal
- CLI: deploy qua `YAML file` và `faas-cli invoke`

**Theo dõi, giám sát thời gian thực: sử dụng Grafana**

Hết. Qua [lab3](lab3.md)