## Neonify
![image](../img/8.1.png)
![image](../img/8.2.png)
mình thử nhập payload test xss `<h1>khiem</h1>` thì:
![image](../img/8.3.png)

view source:
![image](../img/8.4.png)

mình cứ nghĩ là nó dính xss nhưng có vẻ không phải =)), khả năng cao là SSTI

mình thấy trong biểu thức regex chỉ có cờ i để không phân biệt chữ hoa, thường. nhưng lại không có cờ m (multi line)

mình thử inject `%0a` tương ứng với newline xem sao. Nhớ encode

payload: `khiem%0akhiem` thành công, xác nhận lỗi SSTI với regex thiếu cờ `m`

![image](../img/8.5.png)

dựa luôn vào index.erb mình thử truyền payload: `khiem%0a<%= 1+1 %>` sau khi encode thành `khiem%0a<%25%3d1*2%25>`
![image](../img/8.6.png)
list ls
![image](../img/8.7.png)
đọc flag
![image](../img/8.8.png)

