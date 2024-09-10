![image](../CSAW%202024/img/28.png)

chall này lúc mình viết wu thì nó down mất rồi, ib cho host họ còn kh rep :))) 

source code là 1 file app.py, mình sẽ để bên dưới

```
from flask import Flask, render_template, request, session, jsonify, send_file
from hashlib import sha256
from os import path as path

app = Flask(__name__)
app.secret_key = 'safe'

leaderboard_path = 'leaderboard.txt'
safetytime = 'csawctf{i_look_different_in_prod}'

@app.route('/')
def index() -> str:
    cookie = request.cookies.get('session')
    
    if cookie:
        token = cookie.encode('utf-8')
        tokenHash = sha256(token).hexdigest()
        
        if tokenHash == '25971dadcb50db2303d6a68de14ae4f2d7eb8449ef9b3818bd3fafd052735f3b':
            try:
                with open(leaderboard_path, 'r') as file:
                    lbdata = file.read()
            
            except FileNotFoundError:
                lbdata = 'Leaderboard file not found'
            
            except Exception as e:
                lbdata = f'Error: {str(e)}'
                
            return '<br>'.join(lbdata.split('\n'))
    
    open('logs.txt', mode='w').close()
    return render_template("index.html")


@app.route('/report')
def report() -> str:
    return render_template("report.html")


@app.route('/clear_logs', methods=['POST'])
def clear_logs() -> Flask.response_class:
    try:
        open('logs.txt', 'w').close()
        
        return jsonify(status='success')
    
    except Exception as e:
        return jsonify(status='error', reason=str(e))

    
@app.route('/submit_logs', methods=['POST'])
def submit_logs() -> Flask.response_class:
    try:
        logs = request.json
        
        with open('logs.txt', 'a') as logFile:
            for log in logs:
                logFile.write(f"{log['player']} pressed {log['key']}\n")
        
        return jsonify(status='success')
    
    except Exception as e:
        return jsonify(status='error', reason=str(e))


@app.route('/get_logs', methods=['GET'])
def get_logs() -> Flask.response_class:
    try:
        if path.exists('logs.txt'):
            return send_file('logs.txt', as_attachment=False)
        else:
            return jsonify(status='error', reason='Log file not found'), 404
    
    except Exception as e:
        return jsonify(status='error', reason=str(e))


@app.route('/get_moves', methods=['POST'])
def eval_moves() -> Flask.response_class:
    try:
        data = request.json
        reported_player = data['playerName']
        moves = ''
        if path.exists('logs.txt'):
            with open('logs.txt', 'r') as file:
                lines = file.readlines()
                
                for line in lines:
                    if line.strip():
                        player, key = line.split(' pressed ')
                        if player.strip() == reported_player:
                            moves += key.strip()
        
        return jsonify(status='success', result=moves)
    
    except Exception as e:
        return jsonify(status='error', reason=str(e))


@app.route('/get_eval', methods=['POST'])
def get_eval() -> Flask.response_class:
    try:
        data = request.json
        expr = data['expr']
        
        return jsonify(status='success', result=deep_eval(expr))
    
    except Exception as e:
        return jsonify(status='error', reason=str(e))


def deep_eval(expr:str) -> str:
    try:
        nexpr = eval(expr)
    except Exception as e:
        return expr
    
    return deep_eval(nexpr)


if __name__ == '__main__':
    app.run(host='0.0.0.0')
```

do mình đã exploit rồi nên wu này chỉ tập trung vào việc tìm flag (vì là server down nên mình hơi lười phân tích :<)
thêm ảnh cap màn cho ae dễ nhìn nha

app có một loạt `route`, nhưng chỉ có route này dính lỗi nè

![image](../CSAW%202024/img/29.png)

ở `def deep_eval()` có sử dụng `evel()` là func nguy hiểm có thể exec script

![image](../CSAW%202024/img/30.png)

do đó nếu chúng ta vào route `/get_eval` với methods POST rồi cung cấp json `expr = open("leaderboard.txt", "r").read())` chúng ta sẽ nhận được bảng xếp hạng, trong đó có Flag, bài này dễ mà ae đừng nghi ngờ tui lừa ae =)))) 

