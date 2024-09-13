## jscalc

![image](../img/3.1.png)

![image](../img/3.2.png)

có vẻ như nó là 1 tool calc dùng `eval()` để đánh giá và exec

![image](../img/3.3.png)

View source code:
![image](../img/3.4.png)

 
nó có 2 router get và post xử lí yếu cầu tới path tương ứng

mình chú ý tới router.post, ở đây nó khởi tạo 1 biến `formula` lấy data từ req.body, đây là biến mà mình có thể kiểm soát được
Nếu có giá trị nó sẽ gọi method `Calculator` với value là `formula` truyền vào để tính toán rồi return ra result

![image](../img/3.5.png)

dùng eval() để đánh giá và exec, all right. Theo kinh nghiệm, mình sẽ dùng eval() để đọc file trên server


`keyword search: how to read directory in nodeJS`

![image](../img/3.6.png)

payload của mình như sau:
```
require('fs').readdirSync('/').toString()
```
![image](../img/3.7.png)

giờ đọc flag.txt với `readFileSync()`

![image](../img/3.8.png)