![image](../img/15.1.png)

một bài khá tốt để luyện về blind OOB

dựa vào des thì mình thử change methods qua `OPTIONS` để xem có method nào được accept

![image](../img/15.2.png)

có `HEAD, GET, OPTIONS`

nếu mình thử với `GET` thì
![image](../img/15.3.png)
 nó sẽ trả về cmd
 
 còn khi dùng `HEAD`
 ![image](../img/15.4.png)

nó return 200, chứng tỏ nó có accept, nhưng do flag không được nhảy vào hàm `print()` nên nó không in ra flag

tiến hành code python exploit, ae phải biết về `bin/bash, sh`

![image](../img/15.5.png)

powww

![image](../img/15.6.png)
