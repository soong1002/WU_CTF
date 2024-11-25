---
title: Fuzzybytes

---

![image](../img/1.png)


dùng thử thì mình thấy có 1 chức năng upload file thôi nhé ae, và phải là file đuôi extension `.tar.gz`

cùng mình view source

![image](../img/2.png)

dòng 33 sử dụng hàm nguy hiểm là `exec`, mình nghĩ tới cmd to RCE 

xem qua file python xử lý check melicious code
![image](../img/3.png)

có thể thấy sau khi nó extracts file `.tar` ra nó sẽ check content của file, sau đó remove đi luôn mà không lưu

nhưng để ý, tại dòng 17 dev sử dụng hàm nguy hiểm là `extractall`, mình research thì phát hiện nó là func dính lỗi, dễ bị tấn công bởi kĩ thuật `path traversal`

thêm việc sử dụng nhiều file zip khiến mình có suy nghĩ tới lỗ hổng `Zip Slip`

tài liệu thêm cho ae đọc tham khảo
![image](../img/4.png)

oke tiến hành check thôi, mình sẽ tạo script python

![image](../img/5.png)

![image](../img/6.png)

upload file tar này lên localhost

![image](../img/7.png)


và đã upload thành công
![image](../img/8.png)

theo đúng lí thuyết thì sẽ có 1 file `test.php` nằm trong `/var/www/html/` trên server, mình thử xem nào

![image](../img/9.png)

đúng là có, giờ truy cập vào và RCE thôi =))) 

![image](../img/10.png)
 thành công exec lệnh bằng hàm system()

flag nằm ở /root/flag.txt
![image](../img/11.png)

nhưng
![image](../img/12.png)
 có vẻ như chúng ta không có quyền đọc file này
 
 ![image](../img/13.png)
chuẩn cmn luôn, phải là root mới chạy đc lệnh

![image](../img/14.png)
còn về lí do tại sao chạy được `test.php` thì do nó chạy với người dùng `www-data` chứ không phải `root` nhé mng

hmmmm, còn cách nào để lấy được flag nhỉ? 

![image](../img/15.png)

nếu chúng ta chạy được code php thì sẽ ra sao nếu chúng t zip mẹ nó file `/root/flag.txt` vào xong ném nó ra `/var/www/html` ??? hay vl

exploit lại nào
![image](../img/16.png)


![image](../img/17.png)

giờ truy cập vào flag.tar để tải về nàoo

![image](../img/18.png)

í sời í sời, zip slip takes you everywhere

lấy flag trên máy server thôi nào

![image](../img/19.png)

quẩy lên ae






