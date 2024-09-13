![image](../img/2.1.png)


1 bài về weak signing key, đọc des mình nhận ra ngay là có thể brute force được key

cho ae nào chưa biết thì có 1 tool sp crack password ( trong th này là key) tên `John the Ripper`

ban đầu ta cứ login bằng tk được cung cấp `wiener:peter` để lấy JWT nhé

sau khi lấy được JWT lưu nó vào 1 file có tên `my_jwt.txt` và dùng john thôi

![image](../img/2.2.png)

thu được weak key: `secret1`
dùng key sign lại với username = `administrator`

thành công bypass authen

soong1002neverdie =)))
