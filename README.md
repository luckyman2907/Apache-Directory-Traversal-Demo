# Tìm hiểu và thực nghiệm về lỗ hổng Apache Directory Traversal 
- [Tìm hiểu và thực nghiệm về lỗ hổng Apache Directory Traversal](#tìm-hiểu-và-thực-nghiệm-về-lỗ-hổng-apache-directory-traversal)
  - [1. Directory Traversal là gì?](#1-directory-traversal-là-gì)
    - [1.1. Khái niệm](#11-khái-niệm)
    - [1.2. Phân loại](#12-phân-loại)
    - [1.3. Directory Traversal hoạt động như thế nào?](#13-directory-traversal-hoạt-động-như-thế-nào)
    - [1.4. Sự khác biệt giữa Local File Inclusion và Directory Traversal là gì?](#14-sự-khác-biệt-giữa-local-file-inclusion-và-directory-traversal-là-gì)
  - [2. TRIỂN KHAI THỰC NGHIỆM LỖ HỔNG APACHE DIRECTORY TRAVERSAL](#2-triển-khai-thực-nghiệm-lỗ-hổng-apache-directory-traversal)
    - [2.1. Web demo path traversal](#21-web-demo-path-traversal)
    - [2.2. Web-demo-path-traversal-lfi](#22-web-demo-path-traversal-lfi)

## 1. Directory Traversal là gì?

### 1.1. Khái niệm

**Directory Traversal** là một lỗ hổng web cho phép kẻ tấn công đọc các file không mong muốn trên server. Nó dẫn đến việc bị lộ thông tin nhạy cảm của ứng dụng như thông tin đăng nhập , một số file hoặc thư mục của hệ điều hành. Trong một số trường hợp cũng có thể ghi vào các files trên server, cho phép kẻ tấn công có thể thay đổi dữ liệu hay thậm chí là chiếm quyền điều khiển server.

Máy chủ web cung cấp hai cấp độ cơ chế bảo mật chính:
- **Danh sách kiểm soát truy cập**: được sử dụng trong quá trình cấp phép. Đây là danh sách mà người quản trị máy chủ web sử dụng để chỉ ra người dùng hoặc nhóm nào có thể truy cập, sửa đổi hoặc thực thi các tệp cụ thể trên máy chủ, cũng như các quyền truy cập khác.
- **Thư mục gốc** là một thư mục cụ thể trên hệ thống tệp máy chủ mà người dùng bị giới hạn. Người dùng không thể truy cập bất kỳ thứ gì ở trên thư mục gốc này. Kẻ tấn công thường dùng ký tự `../` vào đường dẫn, kẻ tấn công có thể cố gắng thoát khỏi thư mục hiện tại và truy cập các thư mục cha.

Một số tệp hệ thống có thể bị tấn công:

| Tên tệp       | Nội dung                                    |
| ------------- | ------------------------------------------- |
| /etc/passwd   | Chứa thông tin về tài khoản của người dùng  |
| /etc/group    | Chứa các nhóm của người dùng                |
| /etc/profile  | Chứa các biến môi trường cho người dùng     |
| /etc/issue    | Chứa thông báo hiển thị trước khi đăng nhập |
| /proc/version | Chứa phiên bản Linux đang được sử dụng      |
| /proc/cpuinfo | Chứa thông tin bộ xử lý                     |

### 1.2. Phân loại

Lỗ hổng **Path Traversal** có thể được phân loại thành hai loại chính:
- **Path Traversal dựa trên đường dẫn tương đối**: Kẻ tấn công sử dụng các ký tự đặc biệt như `../` để điều hướng lên thư mục cha hoặc qua các thư mục khác, vượt qua giới hạn ban đầu. Ví dụ: `../../../etc/passwd`.
- **Path Traversal dựa trên đường dẫn tuyệt đối**: Kẻ tấn công cung cấp đường dẫn tuyệt đối hoặc một phần của đường dẫn tuyệt đối, bao gồm cả thư mục gốc, để truy cập vào các tệp tin và thư mục ngoài phạm vi. Ví dụ: `/etc/passwd` hoặc `/var/www/html/../../etc/passwd`.

### 1.3. Directory Traversal hoạt động như thế nào?

Các cuộc tấn công truyền qua thư mục thao túng các biến tham chiếu đường dẫn tệp trong các ứng dụng web. Kẻ tấn công sửa đổi các biến đường dẫn để di chuyển lên trên trong cấu trúc thư mục hoặc đi qua các thư mục khác nhau. Điều này thường được thực hiện bằng cách sử dụng các chuỗi cụ thể như `.. /` hoặc `.. \` trong các hệ thống Unix và Windows, tương ứng.

Đây là một lỗ hổng rất nguy hiểm vì nó có thể gây ảnh hưởng đến hệ thống. Ở mức độ đơn giản, hacker có thể đọc được các file trong thư mục web hay thậm chí là các file nhạy cảm trong hệ thống.

Với 1 số cách khai thác và lỗ hổng ở mức độ chuyên sâu hơn, hacker có thể ghi được file vào hệ thống từ đó chèn thêm mã độc. Tệ nhất là có thể dẫn đến **RCE**.

Kẻ tấn công có thể sử dụng các chuỗi trong URL hoặc trường nhập liệu trong nỗ lực lừa máy chủ trả về tệp từ bên ngoài thư mục gốc của tài liệu.

> Ví dụ:

![](https://github.com/luckyman2907/Apache-Directory-Traversal-Demo/blob/main/images/23-06-52.png?raw=true)

Yêu cầu một URL như ví dụ trên sẽ cố gắng để máy chủ trả về một tệp hệ thống nhạy cảm.

Các cuộc tấn công bắt nguồn từ việc truyền tải thư mục có thể gây tổn hại nếu chúng được sử dụng để hiển thị các tệp hệ thống hoặc tải xuống thông tin nhạy cảm. Nó có thể cho phép kẻ tấn công xem, chỉnh sửa hoặc thực thi các tệp tùy ý trên hệ thống tệp của máy chủ, dẫn đến khả năng xâm phạm máy chủ.

### 1.4. Sự khác biệt giữa Local File Inclusion và Directory Traversal là gì?

Local File Inclusion (LFI) và Directory Traversal đều là các lỗ hổng bảo mật được khai thác để truy cập trái phép vào các tệp trên máy chủ, nhưng chúng khác nhau về hoạt động và tác động tiềm ẩn.

Local File Inclusion (LFI) cho phép kẻ tấn công bao gồm một tệp, thường khai thác tập lệnh trên máy chủ và thực thi nó từ một thư mục cục bộ trong máy chủ. Lỗ hổng này xảy ra khi một ứng dụng web sử dụng đầu vào người dùng không vệ sinh để xây dựng đường dẫn tệp để thực thi. LFI cho phép thực thi các tập lệnh, điều này có khả năng dẫn đến sự thỏa hiệp toàn bộ hệ thống.

Các cuộc tấn công Directory Traversal, hoặc path traversal, nhằm mục đích truy cập các tệp trong một thư mục mà kẻ tấn công không nên có quyền truy cập bằng cách thao tác các biến tham chiếu đường dẫn tệp. Kẻ tấn công không nhất thiết phải thực thi một tệp, vì mục tiêu chính là đọc các tệp nhạy cảm để thu thập dữ liệu. Loại tấn công này thường được sử dụng để thu thập thông tin để thông báo cho các cuộc tấn công trong tương lai.

Trong khi cả hai lỗ hổng bảo mật khai thác các cơ chế bao gồm tệp của các ứng dụng web, LFI cho phép thực thi tập lệnh, trong khi truyền thư mục thường được sử dụng để truy cập dữ liệu trái phép.

## 2. TRIỂN KHAI THỰC NGHIỆM LỖ HỔNG APACHE DIRECTORY TRAVERSAL

### 2.1. Web demo path traversal

Lợi dụng chức năng upload album ảnh để đọc nội dung tập tin bí mật của thư mục gốc.

Cấu trúc thư mục

![](https://github.com/luckyman2907/Apache-Directory-Traversal-Demo/blob/main/images/23-18-30.png?raw=true)

- Thư mục configs: chứa các config của Apache web server. Trong đó có 2 file *000-default.conf* và *apache2.conf*.

1. **000-default.conf**
   
   - Đây là tệp cấu hình của Virtual Host mặc định được Apache tạo ra khi cài đặt.
   - Đường dẫn thường là `/etc/apache2/sites-available/000-default.conf`.
   - Virtual Host trong tệp này xác định cách Apache xử lý các yêu cầu HTTP khi chưa có cấu hình cụ thể nào khác được áp dụng.

   ![](https://github.com/luckyman2907/Apache-Directory-Traversal-Demo/blob/main/images/23-24-49.png?raw=true)

2. **apache2.conf**
   
   - Đây là tệp cấu hình chính của Apache, nơi xác định các thiết lập chung áp dụng cho toàn bộ máy chủ.
   - Đường dẫn thường là `/etc/apache2/apache2.conf`.
   - Tệp này bao gồm các thiết lập cơ bản, như:
     - Các thư mục mặc định.
     - Cách xử lý module.
     - Thiết lập quyền và hạn chế cho toàn bộ hệ thống
   
   - Nội dung của file apache2.conf có thể bao gồm các thông tin:
     - Cấu hình chung cho server (Timeout, KeepAlive, User, Group).
     - Cấu hình quyền truy cập cho thư mục.
     - Cấu hình log và các định dạng log.
     - Cấu hình module được nạp.
     - Cấu hình bảo mật cho thư mục nhạy cảm.
     - Cấu hình MIME Types.
     - Cấu hình Virtual Host và các tệp liên quan (ports.conf, sites-enabled).
     - Bảo mật bổ sung cho các thư mục tải lên hoặc ngăn thực thi mã độc hại.

    ![](https://github.com/luckyman2907/Apache-Directory-Traversal-Demo/blob/main/images/23-29-54.png?raw=true)

    - Thư mục src: chứa mã nguồn của trang web
    - Dockerfile: là một tệp văn bản được sử dụng để định nghĩa cách xây dựng một 

    ![](https://github.com/luckyman2907/Apache-Directory-Traversal-Demo/blob/main/images/23-31-01.png?raw=true)

Hệ thống giả lập được build lên như sau:

![](https://github.com/luckyman2907/Apache-Directory-Traversal-Demo/blob/main/images/23-31-38.png?raw=true)

- Ánh xạ port 80 trong container sang port 8081 của maý chính
- Truy cập `localhost:8081` hoặc `127.0.0.1:8080` , trong khi ứng dụng bên trong container đang chạy trên port 80.
  
![](https://github.com/luckyman2907/Apache-Directory-Traversal-Demo/blob/main/images/23-32-39.png?raw=true)

- Đây là một website cho phép upload album ảnh và xem ảnh từ các album này
- Xem ảnh từ album free có sẵn
  
  ![](https://github.com/luckyman2907/Apache-Directory-Traversal-Demo/blob/main/images/23-34-04.png?raw=true)

- Ngoài ra , đối với người sử dụng trang web, có thể tự tạo album của riêng mình, upload lên server, lưu trữ và xem các ảnh trong album của mình
  
  ![](https://github.com/luckyman2907/Apache-Directory-Traversal-Demo/blob/main/images/23-34-40.png?raw=true)

  ![](https://github.com/luckyman2907/Apache-Directory-Traversal-Demo/blob/main/images/23-35-04.png?raw=true)

- Xem ảnh vừa upload lên album
  
  ![](https://github.com/luckyman2907/Apache-Directory-Traversal-Demo/blob/main/images/23-36-09.png?raw=true)

- Đầu tiên, ta kiểm tra thử xem liệu chúng ta có thể upload và thực thi file php hay không bằng cách tạo một file tên *exploit.php* với nội dung là `<?php phpinfo(); ?>` và upload lên website.
  - Upload thành công nhưng file *exploit.php* không được thực thi mà hiển thị dưới dạng text
  
  ![](https://github.com/luckyman2907/Apache-Directory-Traversal-Demo/blob/main/images/23-38-21.png?raw=true)

  ![](https://github.com/luckyman2907/Apache-Directory-Traversal-Demo/blob/main/images/23-38-46.png?raw=true).

  - Nguyên nhân là do trong folder `configs` đã cấu hình trong file *apache2.conf* mặc định không xử lí cho tất cả các file nằm trong đường dẫn `/var/www/html/upload/`. Một số ngoại lệ như các file `.jpg`, `.png` được hiển thị dạng ảnh, các file `.html`, `.txt`, `.php` hiển thị dạng text

  ![](https://github.com/luckyman2907/Apache-Directory-Traversal-Demo/blob/main/images/23-40-46.png?raw=true)

  - Lúc này ta đặt ra giả thuyết rằng liệu có thể upload vào thư mục khác có khả năng thực thi code php hay không? Cụ thể là **DocumentRoot**, nơi thực thi được file `index.php`, điều này cũng được thấy rõ trong Dockerfile

  ![](https://github.com/luckyman2907/Apache-Directory-Traversal-Demo/blob/main/images/23-41-44.png?raw=true)

  - Phân tích source code để hiểu rõ hơn về cách hoạt động của website
  
  ![](https://github.com/luckyman2907/Apache-Directory-Traversal-Demo/blob/main/images/23-42-12.png?raw=true)

  - Đoạn code thực hiện nhiệm vụ tạo ra một album mới mà người dùng tạo ra, được gán vào `$_SESSION['dir']`, kiểm tra xem biến phiên `$_SESSION['dir']` đã tồn tại chưa. Nếu chưa, đoạn mã bên trong dấu ***{}*** sẽ được thực hiện. Trong trường hợp này, nó tạo ra một tên thư mục duy nhất bằng cách kết hợp `/var/www/html/upload/` với một chuỗi hex ngẫu nhiên dài 16 byte. Cụ thể, *random_bytes(16)* tạo ra một chuỗi ngẫu nhiên gồm 16 byte và *bin2hex()* sẽ chuyển nó thành chuỗi hex. Minh chứng có thể thấy khi ta tạo một album mới
  
  ![](https://github.com/luckyman2907/Apache-Directory-Traversal-Demo/blob/main/images/23-44-33.png?raw=true)

  - `$dir` có giá trị `/var/www/html/upload/. bin2hex(random_bytes(16))` và ta không thể kiểm soát được giá trị này
  - Tiếp theo là đoạn code tạo album
  
  ![](https://github.com/luckyman2907/Apache-Directory-Traversal-Demo/blob/main/images/23-46-06.png?raw=true)

  - `$album` có giá trị `$dir . "/" . strtolower($_POST['album'])` --> ta có thể kiểm soát giá trị này thông qua *unstrusted data* `$_POST['album']`
  - Đoạn code thực hiện việc lưu file

  ![](https://github.com/luckyman2907/Apache-Directory-Traversal-Demo/blob/main/images/23-49-04.png?raw=true)

  - Unsafe method ở đây là hàm *`move_uploaded_file($files["tmp_name"][$i], $newFile)`* sẽ upload file từ `$files["tmp_name"][$i]` vào đường dẫn `$newFile` trên server mà ta có thể kiểm soát được biến $album nên có thể điều hướng file upload vào **DocumentRoot**
  - Vì vậy *unstrusted data* `$_POST['album']` sẽ mang giá trị: **`../..`**
  
  ![](https://github.com/luckyman2907/Apache-Directory-Traversal-Demo/blob/main/images/23-52-23.png?raw=true)

  ![](https://github.com/luckyman2907/Apache-Directory-Traversal-Demo/blob/main/images/23-52-36.png?raw=true)

  > Thao túng nội dung file *phpinfo()* thành công

- Bây giờ mục tiêu là đọc được nội dung file *secret* được cấu hình trong Dockerfile
- Payload simple reverse shell như sau:
  
![](https://github.com/luckyman2907/Apache-Directory-Traversal-Demo/blob/main/images/23-54-27.png?raw=true)

- Upload lên server và thao túng param *cmd*
  
![](https://github.com/luckyman2907/Apache-Directory-Traversal-Demo/blob/main/images/23-56-15.png?raw=true)

![](https://github.com/luckyman2907/Apache-Directory-Traversal-Demo/blob/main/images/23-56-26.png?raw=true)

![](https://github.com/luckyman2907/Apache-Directory-Traversal-Demo/blob/main/images/23-56-57.png?raw=true)

> Đọc nội dung tập tin bí mật thành công

**Giải pháp khắc phục:**

1. Hạn chế quyền truy cập vào thư mục upload
   
   - Cấu hình Apache để ngăn việc thực thi mã trong thư mục upload
   
   ![](https://github.com/luckyman2907/Apache-Directory-Traversal-Demo/blob/main/images/23-59-21.png?raw=true)

   - **SetHandler None**: Ngăn Apache xử lý tệp với bất kỳ module nào (như PHP). Đảm bảo các file tải lên chỉ được lưu dưới dạng dữ liệu không thể thực thi.

2. Thiết lập quyền truy cập phù hợp

   - Đảm bảo rằng thư mục upload không có quyền thực thi:
  
  ![](https://github.com/luckyman2907/Apache-Directory-Traversal-Demo/blob/main/images/00-00-53.png?raw=true)

### 2.2. Web-demo-path-traversal-lfi

Tấn công *Directory Traversal* và *Local File Inclusion (LFI)* qua Apache bằng cách lợi dụng tính năng ghi log là một kỹ thuật phổ biến trong các cuộc tấn công an ninh mạng. Dưới đây là các bước thực hiện demo và hiểu rõ cách thức hoạt động.

- Kẻ tấn công lợi dụng các tham số đầu vào không được kiểm tra kỹ để truy cập các file bên ngoài phạm vi dự kiến `(e.g., ../../../etc/passwd)`.
- LFI xảy ra khi ứng dụng web cho phép tải nội dung từ file nội bộ thông qua các tham số đầu vào. Kẻ tấn công có thể kết hợp với *Directory Traversal* để đọc file hoặc thực thi mã độc được tiêm vào.
- Apache ghi các yêu cầu HTTP vào file log `(e.g., /var/log/apache2/access.log)`.
- Kẻ tấn công có thể chèn mã độc vào file log thông qua **User-Agent** hoặc **URL**.
- Sau đó, LFI được dùng để `include` nội dung của file *log*, dẫn đến thực thi mã độc.

**Hình ảnh cấu trúc sourcode**

![](https://github.com/luckyman2907/Apache-Directory-Traversal-Demo/blob/main/images/00-07-08.png?raw=true)

- **configs** : chứa các cấu hình server
- **src**: chứa mã nguồn của chương trình, bên trong folder này bao gồm:
    - **Folder static**: chứa các file ảnh và âm thanh
    - **Folder views**: chứa các file html dùng để hiển thị giao diện cho người dùng
    - **File index.php**: là file xử lý chính của ứng dụng

- **scripts**: Chứa tệp script *clearlog.sh* với nội dung: `echo > /var/log/apache2/access.log` 
  >Tệp này được dùng để xóa nội dung file log của Apache (access.log)
- **Dockerfile**: File này chứa cấu hình để dựng môi trường Docker, mô phỏng máy chủ cho thử nghiệm.
- Deploy ứng dụng bằng docker

![](https://github.com/luckyman2907/Apache-Directory-Traversal-Demo/blob/main/images/00-12-00.png?raw=true)

- Trang web cung cấp 1 ứng dụng chơi game khi nhấn *button* **Start Game**, để có thể chơi được game nhấn phím `Space`

![](https://github.com/luckyman2907/Apache-Directory-Traversal-Demo/blob/main/images/00-12-53.png?raw=true)

- Nếu bị va chạm vào các thanh chướng ngại vật màu xanh ván game sẽ kết thúc hiển thị lại số điểm người chơi giành được và điều hướng người chơi sang game 2

![](https://github.com/luckyman2907/Apache-Directory-Traversal-Demo/blob/main/images/00-13-32.png?raw=true)

- Khi chuyển đến game 2 thì không chơi được game số 2 này và nhận được thông báo game sắp ra mắt
  
![](https://github.com/luckyman2907/Apache-Directory-Traversal-Demo/blob/main/images/00-14-00.png?raw=true)

- Bây giờ phân tích source để hiểu luồng hoạt động của ứng dụng (file `index.php`):

![](https://github.com/luckyman2907/Apache-Directory-Traversal-Demo/blob/main/images/00-14-36.png?raw=true)

- Một số thông tin rút ra được từ file trên:
  - Có sự xuất hiện của *untrusted data* là biến `$_GET['game']`
  - Mặc định khi truy cập, trang web sẽ hiển thị *view* từ file **fatty-bird-1.html**
  - Nếu `$_GET` gửi lên có kèm theo giá trị của tham số *game*, thì giá trị này sẽ được gán vào biến `$game`
  - Ở gần cuối file biến `$game` sau đó được cộng chuỗi với một đường dẫn và đi vào hàm `include`
- Hàm `include` trong PHP là một hàm cho phép copy hết tất cả nội dung của file khác vào file hiện tại, rồi thực thi
  > Tóm lại, file `index.php` làm nhiệm vụ chính là hiển thị giao diện dựa vào giá trị của tham số ***GET game***

- Ý tưởng khai thác: khai thác vào hàm `include`
  - Ta có thể tác động vào `$_GET['game']` vì đây là dữ liệu được gửi từ *client*
  - Thử thay đổi giá trị của game thành một file khác cũng trong thư mục *views*
  
  ![](https://github.com/luckyman2907/Apache-Directory-Traversal-Demo/blob/main/images/00-22-47.png?raw=true)

  ![](https://github.com/luckyman2907/Apache-Directory-Traversal-Demo/blob/main/images/00-23-05.png?raw=true)

  >Ta thấy trang web *render* đúng nội dung của từng file mình vừa `include`
- Vậy còn những file khác trên server thì sao? Liệu truyền bất kì đường dẫn file nào vào `include` cũng đọc được? Chú ý `$game` đã bị *prefix* bởi `./views/`, ta có thể `include` một file khác không nằm trong thư mục *views* được không? Liệu có thể sử dụng `../` để **Directory Traversal** thoát ra khỏi thư mục này?
- Thử với file text `/etc/passwd`
  
  ![](https://github.com/luckyman2907/Apache-Directory-Traversal-Demo/blob/main/images/00-25-35.png?raw=true)

  >Tấn công *directory traversal* thành công và đọc được file bất kỳ trên hệ thống
- Tiếp theo như **header** của ứng dụng còn muốn chúng ta tấn công **RCE** vào ứng dụng, còn một điểm nữa là *log* của apache, liệu nó có phải là một điểm yếu để khai thác **RCE** không?
- Như đã nói ở trên, `include` sau khi copy nội dung file, nó sẽ thực thi luôn **nếu có code PHP** trong đó, nếu lợi dụng hàm `include` này và kiểm soát nội dung trong file được `include` vào hoàn toàn chúng ta có thể kiểm soát được hệ thống
  > Ta có thể nghĩ đến việc upload một file PHP và `include `file này rơi vào trong nội dung của file đó
- Tuy nhiên ứng dụng không cho phép upload gì lên *server*. Vậy có cách nào không cần upload file nhưng vẫn đưa được code PHP của mình vào nội dung một file nào đó trên *server*, sau đó chỉ cần `include` file này?
- Để làm được như vậy, ta có thể nghĩ đến cách làm cho *untrusted data* rơi vào trong nội dung của một file nào đó có sẵn trên *server*
- Ví dụ một trong các tính năng mà có thể sẽ ghi dữ liệu của user vào nội dung file đó là tính năng **log**. Cụ thể đối với *httpd Apache*, mặc định các request sẽ được ghi log lại ở đường dẫn `/var/log/apache2/access.log`
- Thông thường khi cài đặt apache người ta sẽ cấu hình 2 file là **access log** và **error log** để theo dõi các request gửi lên web server và điều tra khi có sự cố trong lúc xử lý request. Để xem cấu hình này, ta vào file *000-default.conf*

  ![](https://github.com/luckyman2907/Apache-Directory-Traversal-Demo/blob/main/images/00-34-59.png?raw=true)

- Folder lưu trữ file **access.log** là `${APACHE_LOG_DIR}`, mặc định nếu không can thiệp folder đó sẽ là `/var/log/apache2` hay `/var/log/apache2/access.log`
- Các dòng log được lưu trữ trong **access.log** sẽ trông như sau:

  ![](https://github.com/luckyman2907/Apache-Directory-Traversal-Demo/blob/main/images/00-36-43.png?raw=true)

- Cấu trúc của một dòng *log*:

  ![](https://github.com/luckyman2907/Apache-Directory-Traversal-Demo/blob/main/images/00-37-32.png?raw=true)

1. **IP Client**
   - 172.17.0.1: Địa chỉ IP của client gửi yêu cầu tới server. 
2. **Thời gian yêu cầu**
   - *[22/Nov/2024:12:13:45 +0000]*: Thời gian mà yêu cầu được gửi đến server, kèm theo múi giờ (UTC+0). 
3. **Yêu cầu HTTP**
   - **GET / HTTP/1.1**: Phương thức HTTP là GET, yêu cầu tài nguyên là trang chủ (/), sử dụng giao thức **HTTP/1.1**.
4. **Mã trạng thái HTTP**
   - **302**: Mã trạng thái phản hồi từ server là **302**, tức là chuyển hướng tạm thời (*Found*). 
5. **Kích thước phản hồi**
   - 1489: Kích thước của phản hồi từ server, bao gồm cả headers và body, là 1489 bytes. 
6. **Referrer**
   - **"-"** : Không có thông tin *referrer*, có thể yêu cầu được gửi trực tiếp từ *client* hoặc không có referrer.
7. **User-Agent**
   - **"Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/117.0.5938.63 Safari/537.36"**: Thông tin về trình duyệt và hệ điều hành của *client*. Trình duyệt là **Chrome** trên hệ điều hành Windows 10 (64-bit).
   > Đây là một yêu cầu bình thường từ một *client* với mã trạng thái 302, cho thấy có thể có một chuyển hướng xảy ra từ *server*. 
- Ta có thể thấy các trường thông tin như ***yêu cầu HTTP*** , ***Referer*** , ***User-agent*** đều là 
các trường *header request* mà có thể quản lý và can thiệp được. Vậy nếu ta đổi 1 trong 3 trường thành đoạn code `<?php phpinfo(); ?>` thì sao?
- **Directiory Traversal** đọc file *access.log*
  
  ![](https://github.com/luckyman2907/Apache-Directory-Traversal-Demo/blob/main/images/00-52-13.png?raw=true)

- Sửa đổi 1 trong 3 trường có thể là các *unstruted data*
  
  ![](https://github.com/luckyman2907/Apache-Directory-Traversal-Demo/blob/main/images/00-53-00.png?raw=true)

  > Đọc nội dung file *phpinfo()* thành công 
- Tiến hành khai thác tương tự như [web demo path traversal](#21-web-demo-path-traversal) với payload `{<?php system($_GET['cmd']); ?>}`
  
  ![](https://github.com/luckyman2907/Apache-Directory-Traversal-Demo/blob/main/images/01-01-37.png?raw=true)

- Cuối cùng hiển thị và đọc nội dung file bí mật ở thư mục gốc 
  
  ![](https://github.com/luckyman2907/Apache-Directory-Traversal-Demo/blob/main/images/01-02-14.png?raw=true)

  ![](https://github.com/luckyman2907/Apache-Directory-Traversal-Demo/blob/main/images/01-02-41.png?raw=true)

  > RCE thành công.

