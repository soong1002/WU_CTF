---
title: 'Training Problem: Intro to Web'

---

Lâu lắm rồi mình mới quay lại chơi CTF sau 1 thời gian đi thực tập :< Cũng nhớ lắm

Vô thôi, 1 bài CTF của BlueHens2024

![image](https://hackmd.io/_uploads/rkead5yGJx.png)

![image](https://hackmd.io/_uploads/rylyFc1Gkl.png)

truy cập vào thì chỉ có thế này, mình thử fuzz bằng `ffuf` nhưng không có gì đặc biệt lắm ngoài cái thư mục `./git`

thêm cụm từ` version control` là mình thấy bú bú rồi đấy keke

ở bài này mình học đc thêm 1 tool là `GitDumper` để dump được các repo được commit trên git ra

sử dụng command `bash gitdumper.sh <url_targer/.git/> <dir lưu trữ>` là mình nhận được các repo nhưng lại chả có méo gì, có vẻ như nó đã bị xóa

tất nhiên là méo gì làm khó được vì cái gì cũng có lịch sử của nó ( anh mình bảo thế ) =))) 

dùng thêm command `bash bash extractor.sh <path_tới_file_.git_đã_dump_về> <dir lưu trữ>`

![image](https://hackmd.io/_uploads/HkLWqc1Gkx.png)

bú thôi

có file index.html lộ password được encrypt bằng MD5, crack là xong
![image](https://hackmd.io/_uploads/S1BXqckM1l.png)

![image](https://hackmd.io/_uploads/HJPVqcJMkx.png)

FLAG: `udctf{00ph_g1t_b4s3d_l34ks?}`


