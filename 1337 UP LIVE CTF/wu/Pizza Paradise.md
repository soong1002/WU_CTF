---
title: Pizza Paradise

---

1 bài web đơn giản

![image](../img/6.png)

fuzz 1 hồi sẽ ra file `robots.txt` nhé mng

![image](../img/7.png)

![image](../img/8.png)

truy cập vào thì yêu cầu có tài khoản, mình view file js có nhé ae, md5 thì crack phát một

![image](../img/9.png)

và login vô chỉ có chức năng download

![image](../img/10.png)

bắt bằng burp suite

mình thử down file `/etc/passwd` thì báo path cook =))), 
![image](../img/11.png)
chắc dev cho nó mapping với `/assets/inmages` rồi, oke `Path traversal`

![image](../img/12.png)
bypass thành công, mình đọc luôn file php trong ảnh

![image](../img/13.png)

FLAG: `INTIGRITI{70p_53cr37_m15510n_c0mpl373}`




