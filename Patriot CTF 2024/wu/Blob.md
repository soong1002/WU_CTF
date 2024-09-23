![image](../img/6.1.png)

chall cung cấp source code vài dòng, ngang đánh đố =))

![image](../img/6.2.png)

start chall lên và thấy ko có gì, view source có cái này
![image](../img/6.3.png)

nó sử dụng lib `ejs` để tự động render các template. Đến đây mình nghĩ ngay tới SSTI

Research về ejs vulnerability mình tìm được cái [này](https://github.com/mde/ejs/issues/735)

tiến hành exploit thôi

![image](../img/6.4.png)

lấy flag
![image](../img/6.5.png)

FLAG: `CACI{bl0b_s4y_pl3453l00k0utf0rpr0707yp3p0llut10n}`



