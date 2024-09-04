## PickleBall

1 chall mức độ warn-up, cùng mình xem nó có gì nhé
![image](../KMA%20CTF%202024%20lần%201/img/1.png)

có feature register và login nhưng không có back-end nên không xử lí được gì =)), mình sẽ tập trung vào font-end
![image](../KMA%20CTF%202024%20lần%201/img/2.png)

sau một hồi recon ctrl+U mình không phát hiện được gì trong đó, nghĩ tới các path ẩn trên server, một file mình luôn check theo bản năng là `robots.txt`

![image](../KMA%20CTF%202024%20lần%201/img/3.png)

vậy là có part 1 của flag, tương tự mình tìm tiếp part 2,3,..
![image](../KMA%20CTF%202024%20lần%201/img/4.png)
![image](../KMA%20CTF%202024%20lần%201/img/30.png)

FLAG: `KMACTF{p1Ckleb4ll_WitH-uU_piCklepal_5a6b89113abb}`

## malicip

chall này cung cấp source code
![image](../KMA%20CTF%202024%20lần%201/img/5.png)

dễ dàng để mình nhận ra nó dính lỗi sql injection ở 2 endpoint `/list-ip` và `/check-ip`
![image](../KMA%20CTF%202024%20lần%201/img/6.png)

cả 2 endpoint này đều có cái cần bypass, ở list-ip cần bypass với limit và offset nhập vào phải là số nguyên(`int`), còn ở endpoint `/check-ip` chúng ta cần bypass func `ip_address`

NOTE: func `ip_address` sẽ check xem para ip mình nhập vào có đúng định dạng của 1 ipv4 hay ipv6 hay không. Nếu đúng nó trả về true còn không thì ném ra 1 lỗi `valueError
`

bài này lúc mình tham gia thi ban đầu mình đã focus vào endpoint `/check-ip` nhưng sau khá nhiều thời gian ngồi mày mò search google các kiểu kết quả là không ăn thua, mình chuyển sang endpoint `list-ip` với hi vọng tận dụng được limit và offset, và tất nhiên rồi vẫn không ăn thua =))), đến khi mình tạo ticket mới được author hint cho nên focus vào endpoint `check-ip`. oke

![image](../KMA%20CTF%202024%20lần%201/img/7.png)
phân tích một chút về end-point này

para `ip` hoàn toàn có thể control được bởi hacker và được truyền trực tiếp không qua sanitize nào vào trong câu query (database ở đây sử dụng là MySQL). Khi ip thoả mãn func `ip_address` nó sẽ return về json ip và message

mình sẽ thử nhập ipv4 là `127.0.0.1` cho ae xem
![image](../KMA%20CTF%202024%20lần%201/img/8.png)

vì trong db không có ip nào là 127.0.0.1 nên không có json trả về thôi, biết nó dính sql injection, đơn giản nhất là inject dấu nháy đơn `'` vào và xem nó response gì

![image](../KMA%20CTF%202024%20lần%201/img/9.png)
ngay lập tức nó báo ip nhập vào không thoả mãn func `ip_address`

![image](../KMA%20CTF%202024%20lần%201/img/10.png)
mình sẽ xem xem IPv4Address và IPv6Address

IPv4
![image](../KMA%20CTF%202024%20lần%201/img/11.png)
address được truyền vào kiểm tra dạng `int`,`bytes`,`string` để check từng loại, vì `string` là khả quan nhất nên đi vào `_ip_int_from_string()`

![image](../KMA%20CTF%202024%20lần%201/img/12.png)
dựa vào `split` để bỏ dấu chấm rồi chuyển qua dạng `int` và tiếp tục xử lí qua` _parse_octet`, không thấy nghi vấn gì nên mình chuyển qua IPv6

IPv6

Tương tự IPv4
![image](../KMA%20CTF%202024%20lần%201/img/13.png)

trước khi đưa vào `_ip_int_from_string()` nó được qua `_split_scope_ip` để phân tách dấu %
![image](../KMA%20CTF%202024%20lần%201/img/14.png)

ví dụ với `2001:db8::%eth0` thì `addr = 2001:db8::` và `scope_id = eth0`

![image](../KMA%20CTF%202024%20lần%201/img/15.png)
và đến đây mình nhận thấy nó chỉ check `add_str` chứ không check `scope_id` phía sau


đây là payload mình sử dụng test

`/check-ip?ip=fe80::1ff:fe23:4567:890a%xxx%27union%20select%201,2%20--%20a`

![image](../KMA%20CTF%202024%20lần%201/img/16.png)

giờ thì lấy bảng và cột rồi lấy flag thôi

`/check-ip?ip=fe80::1ff:fe23:4567:890a%khiem%27union%20select%20null,table_name%20from%20information_schema.tables%20--%20a`

![image](../KMA%20CTF%202024%20lần%201/img/17.png)
![image](../KMA%20CTF%202024%20lần%201/img/18.png)

![image](../KMA%20CTF%202024%20lần%201/img/19.png)


FLAG: `KMACTF{actually__this_flag-is_not_so_malicious_but_the_ipv6_is}`