# Phân tích lỗi cấu hình Spring MVC

## 1. Lỗi trong MyWebAppInitializer.java

### Nguyên nhân

Cấu hình:

```java
return new String[] { "/api/*" };
```

DispatcherServlet chỉ lắng nghe các request bắt đầu bằng `/api/`

### Hệ quả

* URL `/welcome` không thuộc `/api/*`
* Request không đi vào DispatcherServlet
* Spring MVC không xử lý
* Tomcat trả về 404 Not Found

### DispatcherServlet đang nghe:

* `/api/users`
* `/api/orders`
* `/api/welcome`

Không nghe:

* `/welcome`

---

## 2. Lỗi trong WebConfig.java

### Cấu hình sai

```java
@ComponentScan(basePackages = "com.demo.service")
```

### Nguyên nhân

* Spring chỉ scan package `com.demo.service`
* Trong khi Controller nằm ở `com.demo.controller`

### Hệ quả

* Không phát hiện @Controller
* Không đăng ký bean Controller
* Không có mapping URL → Controller

### Kết luận

Spring đang tìm Controller ở `com.demo.service` nhưng thực tế nằm ở `com.demo.controller` nên không tìm thấy

---

## 3. Tổng hợp tình huống

### Nếu chỉ sửa lỗi 1 (Servlet Mapping)

Sửa:

```java
return new String[] { "/" };
```

### Điều xảy ra

* Request `/welcome` vào được DispatcherServlet
* Nhưng Spring không có Controller tương ứng

### Kết quả

Vẫn 404 Not Found

### Lý do

* DispatcherServlet nhận request
* Nhưng HandlerMapping không tìm thấy Controller

---

## Kết luận chung

| Lỗi                 | Ảnh hưởng                        |
| ------------------- | -------------------------------- |
| Servlet Mapping sai | Request không vào Spring         |
| ComponentScan sai   | Spring không tìm thấy Controller |

Cả hai lỗi đều gây 404 nhưng ở hai tầng khác nhau:

* Lỗi 1: tầng Servlet (Tomcat)
* Lỗi 2: tầng Spring MVC

---

## Tóm gọn

* `/api/*` chặn `/welcome`
* Scan sai package nên không có Controller
* Chỉ sửa 1 lỗi thì ứng dụng vẫn không chạy
