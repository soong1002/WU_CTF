![image](../img/1.1.png)

start chall lên không thấy gì đặc biệt, mình view source
![image](../img/1.2.png)

đoạn này chứa mảng giá trị ip được cho phép, truyền vào trong `$allowed_ip`, giá trị ip được lấy qua header `X-Forwared-For`
![image](../img/1.3.png)

đoạn code này check xem ip có trong `$allowed_ip` không, nếu có thì nhả ra flag -> bypass check ip khá đơn giản
![image](../img/1.4.png)

expoit lấy flag
![image](../img/1.5.png)

FLAG: `CACI{1_lik3_g1raff3s_4_l0t}`


