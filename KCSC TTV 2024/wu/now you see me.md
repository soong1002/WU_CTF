## now you see me

![image](../img/1.1.png)

mình dùng thử xem có gì trong chall này trước

nhập vào bất kì thứ gì
![image](../img/1.2.png)

nó redirect tới `/register`
![image](../img/1.3.png)

tuy nhiên không reflect lại tên mình vừa input, nhưng ctr+U thì có 
![image](../img/1.4.png)

mình nhận được hint flag nằm ở /

dùng burp suite để nhập input và quan sát dễ hơn
Mình có nghĩ tới XSS và SSTI

tuy nhiên thử payload đơn giản này thẻ h1 không được render nên mình chuyển qua exploit SSTI
![image](../img/1.5.png)

quan sát respone trả về mình biết server là `Werkzeug/3.0.4 Python/3.12.6`
![image](../img/1.6.png)

và python có một số template như : `Jinja2, Django, Mako, Chameleon,... `. Mình tiến hành check lần lượt, và may mắn mình confirm template sử dụng là Jinja2

![image](../img/1.7.png)

việc cần làm là research payload, sau một lúc research mình tìm được payload ưng ý tại [hacktrick](https://book.hacktricks.xyz/pentesting-web/ssti-server-side-template-injection/jinja2-ssti)

![image](../img/1.8.png)

thay đổi path để lấy flag
![image](../img/1.9.png)

FLAG: `KCSC{flagrandomngaunhienlagicungdu0c}`