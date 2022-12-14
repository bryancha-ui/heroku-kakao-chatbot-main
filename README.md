# 개요
- 카카오톡 챗봇 만들기를 Python + FLASK를 통해 간단한 튜토리얼을 만들어본다. 

# 사전준비
- OBT 참여승인을 받아야 한다. 

# 기본설정
- 카카오톡 챗봇 버튼 클릭 후, 봇 이름 생성
  + [봇 만들기] - [카카오톡 챗봇]
![](./img/fig01_create_chatbot.png)

- 카카오톡 채널 연결을 진행한다. 
![](./img/fig02_conn_channel.png)

- virtualenv를 활용하여 가상환경을 설정한다. 
```bash
$ virtualenv venv
created virtual environment CPython3.9.7.final.0-64 in 6029ms
  creator CPython3Windows(dest=C:\Users\human\Desktop\heroku-kakao-chatbot\venv, clear=False, no_vcs_ignore=False, global=False)
  seeder FromAppData(download=False, pip=bundle, setuptools=bundle, wheel=bundle, via=copy, app_data_dir=C:\Users\human\AppData\Local\pypa\virtualenv)
    added seed packages: pip==22.0.4, setuptools==62.1.0, wheel==0.37.1
  activators BashActivator,BatchActivator,FishActivator,NushellActivator,PowerShellActivator,PythonActivator
```

# Heroku App 구축
- 간단하게 app 파일을 만들어 Heroku App URL을 확보해보자. 
- app/main.py
```python
from flask import Flask
 
app = Flask(__name__)
 
@app.route('/')
def hello_world():
    return 'Hello, World!'
```

- wsgi.py 생성 
    + app은 폴더를 말하고, main은 main.py를 말한다. 
```python
from app.main import app

if __name__ == "__main__":
    app.run(threaded=True, port=5000)
```

- Procfile 파일 작성
    + 카카오톡 챗봇에서는 포트번호를 입력을 해줘야 한다. 
        - localhost:5000 처럼 명시적으로 입력해주는 것으로 생각하면 된다. 
```
web: gunicorn --bind 0.0.0.0:$PORT wsgi:app
```

- runtime.txt
    + Python 버전을 업로드 한다. 
```
python-3.9.7
```

- Heroku login
    + Heroku 배포 전에 반드시 로그인을 해야 한다. 
```bash
$ heroku login
```

- Heroku Project 생성
    + 이름 다르게 진행! 
    + name_name (x)
    + 로컬 폴더명 == Github Repo == Heroku Project 이름 
```bash
$ heroku create heroku-kakao-chatbot
```

- Heroku 배포
    + Heroku에 배포하기 위해서는 크게 아래 코드만 기억한다. 
```bash
$ git add .
$ git commit -am "your_message"
$ git push origin main # Github Repository에 업데이트
$ git push heroku main # Heroku 코드 배포
```

- 기존 Existing App과 연동하려면 배포 전 아래 코드를 선 실행 후, 배포를 진행한다. 
```bash
$ heroku git:remote -a example-app
```

- 실행하면 아래와 같은 결과물이 나타난다. 
![](/img/fig03_flask_heroku_app.png)

# 스킬 서버 구축 기본편
- 스킬 서버에서 제공하는 2가지 API URI는 다음과 닽다. 

- 단순 텍스트형 응답 스킬
```
POST /api/sayHello
```

- 단순 이미지형 응답 스킬
```
POST /api/showHello
```

## (1) 스킬 서버 예제
- 이제 app.py를 생성한다. 
- hello_kakao_skill 디렉터리를 만들고, app.py 파일을 생성한다. 
    + 코드 `print(body['userRequest']['utterance'])`와 `responseBody` 영역은 추후 설명 할 예정이다. 
```python
from flask import Flask, request
app = Flask(__name__)

@app.route('/')
def hello_world():
    return 'Hello, World!'

# 카카오톡 텍스트형 응답
@app.route('/api/sayHello', methods=['POST'])
def sayHello():
    body = request.get_json()
    print(body)
    print(body['userRequest']['utterance'])

    responseBody = {
        "version": "2.0",
        "template": {
            "outputs": [
                {
                    "simpleText": {
                        "text": "안녕 hello I'm Ryan"
                    }
                }
            ]
        }
    }

    return responseBody


# 카카오톡 이미지형 응답
@app.route('/api/showHello', methods=['POST'])
def showHello():
    body = request.get_json()
    print(body)
    print(body['userRequest']['utterance'])

    responseBody = {
        "version": "2.0",
        "template": {
            "outputs": [
                {
                    "simpleImage": {
                        "imageUrl": "https://t1.daumcdn.net/friends/prod/category/M001_friends_ryan2.jpg",
                        "altText": "hello I'm Ryan"
                    }
                }
            ]
        }
    }

    return responseBody



```

## (2) 스킬 서버 등록
- 아래와 같이 스킬 서버 정보를 입력하고 저장한다. 
![](./img/fig04_skill_register.png)

## (3) 시나리오
- 여기가 가장 중요하다. 이제 시나리오를 등록한다. 이 때 중요한 것은 파라미터 설정 탭에서 스킬 선택을 개별적으로 호출 할 수 있다. 
    + Heroku로 배포할 때는 아래와 같이 URL을 입력한다. 
    + URL/와 함께 다음 주소는 api/
![](./img/fig05_skill_call.png)

- 봇 응답에서 스킬 데이터를 선택 후, 저장 버튼을 클릭한다. 
![](./img/fig06_bot_response.png)

## (4) 배포
- 배포 탭을 클릭한 후, 배포를 진행하면 된다. 
![](./img/fig07_deploy.png)

## (5) 테스트
- 봇 테스트를 열고 아래와 같이 테스트를 한다. 
![](./img/fig08_chatbot_test.png)

## (6) 로그 확인
- Heroku 에서는 로그를 쉽게 확인할 수 있다. 
```bash
$ heroku logs 
```

# 로그분석
- 발화 내용 : 발화 내용은 사용자가 입력하는 영역이다. 
- 그럼 응답은 다음 프로세스를 거쳐서 일어난다. 
    + 아래는 전체 응답 로그이다. 
```bash
2022-05-14T01:58:03.160003+00:00 app[web.1]: {'bot': {'id': '627deba39ac8ed7844169488!', 'name': '피자주문봇'}, 'intent': {'id': '627f09619ac8ed784416a1c8', 'name': '단순 이미지 응답', 'extra': {'reason': {'code': 1, 'message': 'OK'}}}, 'action': {'id': '627f094004a7d7314aebc355', 'name': '단순 이미지 응답 스킬', 'params': {}, 'detailParams': {}, 'clientExtra': {}}, 'userRequest': {'block': {'id': '627f09619ac8ed784416a1c8', 'name': '단순 이미지 응답'}, 'user': {'id': 'c7bbb643195214adda0ca02beb4a0401cafecbb7c3dbc9b0268aa017be9df71d1d', 'type': 'botUserKey', 'properties': {'botUserKey': 'c7bbb643195214adda0ca02beb4a0401cafecbb7c3dbc9b0268aa017be9df71d1d', 'bot_user_key': 'c7bbb643195214adda0ca02beb4a0401cafecbb7c3dbc9b0268aa017be9df71d1d'}}, 'utterance': '안녕 라이언\n', 'params': {'surface': 'BuilderBotTest', 'ignoreMe': 'true'}, 'lang': 'kr', 'timezone': 'Asia/Seoul'}, 'contexts': []}
```

- bot은 사용자 발화를 받은 봇의 정보를 담고 있다. 봇 ID와 이름 정보가 있다. 
```bash
{
    'bot': {
        'id': '627deba39ac8ed7844169488!',
        'name': '피자주문봇'
    },
    ...
}
```

- intent는 인식된 사용자의 발화의 블록 정보를 담고 있다.
    + 참조 : https://i.kakao.com/docs/skill-response-format#intent
```bash
'intent': {
    'id': '627f09619ac8ed784416a1c8',
    'name': '단순 이미지 응답',
    'extra': {
        'reason': {
            'code': 1,
            'message': 'OK'
        }
    }
},
```

- action은 실행되는 스킬 정보를 담고 있다. 
    + 일반 파라미터를 추가하면 사용자 발화에서 추출한 개체 정보(엔티티)를 params, detailParams, clientExtra 필드에 포함시킬 수 있다. 
    + 참조 : https://i.kakao.com/docs/skill-response-format#action
```bash
'action': {
    'id': '627f094004a7d7314aebc355',
    'name': '단순 이미지 응답 스킬',
    'params': {},
    'detailParams': {},
    'clientExtra': {}
},
```

- userRequests는 사용자 정보를 담고 있는 필드이다. 스킬 페이로드에서 가장 중요한 정보들을 담고 있다. 
    + 참조 : https://i.kakao.com/docs/skill-response-format#userrequest
    + 'block'은 사용자 발화가 인식된 정보를 나타냄. 
    + 'user'는 챗봇 사용자 정보를 나타냄. 
    + 'utterance'는 봇 시스템에 전달된 사용자 발화
    + 즉, 여기에서 필요한 건 `utterance` 필드 데이터만 추출해 챗봇 API 서버에 전달하는 것이다. 
```bash
'userRequest': {
    'block': {
        'id': '627f09619ac8ed784416a1c8',
        'name': '단순 이미지 응답'
    },
    'user': {
        'id': 'c7bbb643195214adda0ca02beb4a0401cafecbb7c3dbc9b0268aa017be9df71d1d',
        'type': 'botUserKey',
        'properties': {
            'botUserKey': 'c7bbb643195214adda0ca02beb4a0401cafecbb7c3dbc9b0268aa017be9df71d1d',
            'bot_user_key': 'c7bbb643195214adda0ca02beb4a0401cafecbb7c3dbc9b0268aa017be9df71d1d'
        }
    },
    'utterance': '안녕 라이언\n',
    'params': {
        'surface': 'BuilderBotTest',
        'ignoreMe': 'true'
    },
    'lang': 'kr',
    'timezone': 'Asia/Seoul'
}, 'contexts': []
```

# 스킬 응답 JSON 
- 스킬 응답의 구조의 주요 내용은 링크를 참조한다. 
    + URL : https://i.kakao.com/docs/skill-response-format#skillpayload
- 버전 정보가 없다면 구 스킬(v1) 응답으로 간주하기 때문에 반드시 version 필드를 포함해야 한다. 
- template은 스킬 응답 출력 형태 정보를 담고 있다. 이를 SkillTemplate이라 부른다. 
```python
    responseBody = {
        "version": "2.0",
        "template": {
            "outputs": [
                {
                    "simpleImage": {
                        "imageUrl": "https://t1.daumcdn.net/friends/prod/category/M001_friends_ryan2.jpg",
                        "altText": "hello I'm Ryan"
                    }
                }
            ]
        }
    }
```

- outputs는 출력 그룹을 나타내며, 출력 그룹은 1~3개까지 출력 요소를 포함시킬 수 있다. 
- 출력요소는 크게 7가지이다. (2022년 5월 기준)
    + simpleText : 간단 텍스트 (https://i.kakao.com/docs/skill-response-format#simpletext)
    + simpleImage : 간단 이미지 (https://i.kakao.com/docs/skill-response-format#simpleimage)
    + basicCard : 기본 카드 (https://i.kakao.com/docs/skill-response-format#basiccard)
    + commerceCard : 커머스 카드 (https://i.kakao.com/docs/skill-response-format#commercecard)
    + listCard : 리스트 카드 (https://i.kakao.com/docs/skill-response-format#listcard)
    + ItemCard : 아이템 말풍성 카드 (https://i.kakao.com/docs/skill-response-format#itemcard)
    + Carousel : 케로셀 카드 (https://i.kakao.com/docs/skill-response-format#carousel)

- 그 외에도 다양한 기능들은 도움말 및 예제코드를 확인하도록 한다. 

# 사칙 연산 챗봇 구현
- 사칙 연산을 구현하는 챗봇을 만들어 본다. 
![](/img/fig09_calculator_sample.png)

## (1) 엔티티 구성
- 엔티티(Entity)란 봇이 이해할 수 있는 용어를 체계적으로 정리한 데이터 사전을 말한다. 
- 엔티티 구조는 크게 엔티티명, 대표 엔트리, 동의어 총 세 가지 요소로 구성된다. 
![](./img/fig10_entity_structure.png)
- 엔티티의 자세한 설명은 다음 링크에서 확인한다. 
    + URL : https://i.kakao.com/docs/key-concepts-entity

- 사칙연산을 예로 들면 아래와 같이 지정할 수 있다. 
    + division, subtraction 등은 대표 엔트리를 말한다. 
    + 동의어에 해당하는 값이 나오면 추후에 확인할 로그 값에는 대표 엔트리만 별도로 데이터를 수집할 수 있다. 
![](./img/fig11_entity_example.png)

## (2) 시나리오 구성
- 엔티티를 구성했다면, 이제 시나리오를 구성한다. 
- 패턴 발화를 할 때 원하는 파라미터를 별도로 지정할 수 있다. 
![](./img/fig12_scenario.png)

- 패턴 발화를 보면 빨간색 네모 박스를 확인할 수 있는 데, 이 부분에서 파라미터를 설정할 수 있다. 
    + 더하기, +에서의 파라미터는 엔티티 그룹을 설정하게 된다. 
![](./img/fig13_parameter_settings.png)

- 실제 파라미터 설정된 값은 아래와 같다. 
![](./img/fig14_parameter_check.png)

## (3) 파이썬 코드 
- 이제 다음 코드를 작성하여 기존 main.py에 추가한다. 
- 코드 작성시, 한가지 유의 사항은 json.loads() 함수를 반드시 써줘야 한다. Python Dictionary를 String으로 변경하는 것은 str() 함수를 사용하면 쉽게 변경이 가능하지만, 문자열 dictionary를 Dictionary로 바로 변경은 되지 않는다. 따라서, json.loads()를 사용한다. 
    + 참조 : https://blog.metafor.kr/224

```python
import json

# 메인 로직!! 
def cals(opt_operator, number01, number02):
    if opt_operator == "addition":
        return number01 + number02
    elif opt_operator == "subtraction": 
        return number01 - number02
    elif opt_operator == "multiplication":
        return number01 * number02
    elif opt_operator == "division":
        return number01 / number02
.
.
.


# 카카오톡 Calculator 계산기 응답
@app.route('/api/calCulator', methods=['POST'])
def calCulator():
    body = request.get_json()
    print(body)
    params_df = body['action']['params']
    print(type(params_df))
    opt_operator = params_df['operators']
    number01 = json.loads(params_df['sys_number01'])['amount']
    number02 = json.loads(params_df['sys_number02'])['amount']

    print(opt_operator, type(opt_operator), number01, type(number01))

    answer_text = str(cals(opt_operator, number01, number02))

    responseBody = {
        "version": "2.0",
        "template": {
            "outputs": [
                {
                    "simpleText": {
                        "text": answer_text
                    }
                }
            ]
        }
    }

    return responseBody
```

## (4) 배포 및 로그 확인
- 코드 작성이 끝났다면 배포를 시작한다. 
- 로그 확인을 통해서 자주 반복적으로 배포를 해야 하기 때문에 `deploy.sh` 파일을 작성했다. 
```bash
git add .
git commit -m "updated"
git push origin main
git push heroku main
```
- shell 스크립트 실행 위해서는 먼저 파일 권한을 변경 후, 실행한다. 
```bash
$ chmod -R 777 deploy.sh
$ ./deploy.sh
```

- 간단하게 테스트를 진행한 후, 로그를 확인하기 위해서는 다음과 같이 실행한다. 
```bash
$ heroku logs
```
- 실행 결과는 아래와 같다. 여기 로그값을 확인하면서 코드 에러를 해결해간다. 
![](./img/fig15_1_logs.png)
![](./img/fig15_2_logs.png)

# 최종 파일 구조 확인
- 가상환경 관리 폴더인 `venv`을 제외하면 전체 파일 구조는 아래와 같다. 
```bash
(venv) $ tree -L 3
.
├── Procfile
├── README.md
├── app
│   └── main.py
├── deploy.sh
├── requirements.txt
├── runtime.txt
└── wsgi.py
```