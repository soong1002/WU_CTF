## Log Me In
![image](../CSAW%202024/img/19.png)

phần des chẳng có gì gợi ý cả :<

![image](../CSAW%202024/img/20.png)

chall cung cấp 3 feature, khi login sẽ cho 1 cookie `info`
cùng mình view source 

flag nằm ở endpoint `user`
![image](../CSAW%202024/img/21.png)

nó sẽ `decode()` cookie sau đó check `uid == 0`

endpoint `/register`
![image](../CSAW%202024/img/22.png)

qua một loạt if else để check, ban đầu chưa view source mình nghĩ nó sẽ bị sql injection vì thấy có feature register và login, nhưng mà sai lòi =))) 

sau khi register nó sẽ mặc định cho chúng ta `uid = 1`. Soooo, bypass kiểu chóa gì =)) 


endpoint `login` không có gì focus cả
![image](../CSAW%202024/img/23.png)


vấn đề nằm ở check `uid = 0`, nó sử dụng def `decode()` 

![image](../CSAW%202024/img/24.png)

def `encode()` sử dụng XOR với `ENCRYPT_KEY` sau đó chuyển qua dạng `hex` sẽ được cookie, tương tự là `decode()` cũng như vậy

chúng ta cần hiểu về phép XOR trong bài này, mình sẽ phân tích một chút

Loại mã hóa ở chall này mà tác giả muốn đề cập tới là: `OTP = One Time Pad`. OTP sử dụng một khóa ngẫu nhiên có độ dài bằng hoặc lớn hơn độ dài của thông điệp cần mã hóa. Khóa này phải hoàn toàn ngẫu nhiên và không bao giờ được sử dụng lại

Nhưng ở đây khóa đó lại được cố định bởi `os.environ['ENCRYPT_KEY']`, do đó khóa ở đây không được thay đổi -> LỖI

```
để lấy cipher
c=plain_text^key

để đảo ngược lại  
key=c^plain_text

plain_text là tên người dùng đầu vào của bạn và tên hiển thị bạn có thể lấy được khi thử cục bộ
{"username": "test00", "displays": "test00", "uid": 1}

sau đó tạo xor với
cipher txt mà chúng ta tìm thấy trong cookie
```

okey, vậy thì lại thành dễ rồi =)) , viết script exploit thôi

```
import json

def json_to_byte_string(data: dict) -> bytes:
    """
    Chuyển đổi đối tượng Python (từ điển) thành byte string.
    
    :param data: Đối tượng Python (ví dụ: từ điển) để chuyển đổi thành JSON.
    :return: Byte string của chuỗi JSON.
    """
    try:
        # Chuyển đổi đối tượng Python thành chuỗi JSON
        json_string = json.dumps(data)
        
        # Chuyển đổi chuỗi JSON thành byte string
        byte_string = json_string.encode()
        
        return byte_string
    except (TypeError, ValueError) as e:
        print(f"Error: {e}")
        return None

def hex_to_byte_string(hex_string: str) -> bytes:
    """
    Chuyển đổi chuỗi hex thành byte string.
    
    :param hex_string: Chuỗi hex (mã hex) để chuyển đổi thành byte string.
    :return: Byte string tương ứng.
    """
    try:
        # Chuyển đổi chuỗi hex thành byte string
        byte_string = bytes.fromhex(hex_string)
        return byte_string
    except ValueError as e:
        print(f"Error: {e}")
        return None

def xor_byte_strings(b1: bytes, b2: bytes) -> bytes:
    """
    Thực hiện phép toán XOR giữa hai byte string.
    
    :param b1: Byte string đầu tiên.
    :param b2: Byte string thứ hai.
    :return: Byte string kết quả của phép toán XOR.
    """
    # Kiểm tra độ dài của các byte string
    if len(b1) != len(b2):
        raise ValueError("Các byte string phải có độ dài bằng nhau")

    # Thực hiện phép toán XOR giữa từng cặp byte tương ứng
    return bytes([byte1 ^ byte2 for byte1, byte2 in zip(b1, b2)])

# Ví dụ sử dụng
data = {
    'username': 'khiem',
    'displays': 'khiem',
    'uid': 1
}

# Chuyển đổi đối tượng Python thành byte string
json_byte_string = json_to_byte_string(data)
print("Json Byte string:", json_byte_string)

# Chuỗi hex cần chuyển đổi
hex_string = "48674c3731025651282f614a4d541d3a0d5d1f456e4141131e441918352b40515d194f1a2f221d15534754783a19165a7b6e7b14"

# Chuyển đổi chuỗi hex thành byte string
hex_byte_string = hex_to_byte_string(hex_string)
print("Hex Byte string:", hex_byte_string)

# Thực hiện phép toán XOR giữa hai byte string
try:
    xor_result = xor_byte_strings(json_byte_string, hex_byte_string)
    print("Kết quả XOR KEY:", xor_result)
    print("Kết quả XOR (hex):", xor_result.hex())
except ValueError as e:
    print(f"Error: {e}")

```
![image](../CSAW%202024/img/25.png)

và chúng ta lấy được ENCRYPT_KEY, giờ tạo hex với ENCRYPT_KEY nhét vô cookie nào

```
import json
import os

def encode(status: dict) -> str:
    try:
        # Chuyển đổi đối tượng Python thành chuỗi JSON và sau đó thành byte string
        plaintext = json.dumps(status).encode()

        # Lấy ENCRYPT_KEY từ biến môi trường và chuyển đổi thành byte string
        encrypt_key = os.environ['ENCRYPT_KEY'].encode()

        # Đảm bảo rằng ENCRYPT_KEY có cùng độ dài với plaintext bằng cách lặp lại hoặc cắt
        if len(encrypt_key) < len(plaintext):
            # Lặp lại ENCRYPT_KEY nếu nó ngắn hơn plaintext
            encrypt_key = (encrypt_key * ((len(plaintext) // len(encrypt_key)) + 1))[:len(plaintext)]
        elif len(encrypt_key) > len(plaintext):
            # Cắt ENCRYPT_KEY nếu nó dài hơn plaintext
            encrypt_key = encrypt_key[:len(plaintext)]

        # Thực hiện phép toán XOR giữa plaintext và encrypt_key
        out = bytes([i ^ j for i, j in zip(plaintext, encrypt_key)])

        # Trả về kết quả dưới dạng chuỗi hex
        return bytes.hex(out)
    except Exception as s:
        LOG(s)
        return None

def LOG(message):
    print(f"Error: {message}")

# Ví dụ sử dụng
if __name__ == "__main__":
    # Đặt biến môi trường ENCRYPT_KEY
    os.environ['ENCRYPT_KEY'] = '3E9DTp80EJCpmvvRd8rgBacww7itTR3sg9mqGKxxqktZOprxANJi'

    data = {
        'username': 'khiem',
        'displays': 'khiem',
        'uid': 0
    }

    encoded_string = encode(data)
    print("Encoded String:", encoded_string)

```

run python3 lấy cookie hex
![image](../CSAW%202024/img/26.png)

và đây làaaaaa

![image](../CSAW%202024/img/27.png)











