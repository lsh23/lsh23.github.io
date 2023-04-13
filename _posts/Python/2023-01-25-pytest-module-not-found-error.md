---
title: pytest module not found error
date: 2023-01-25 19:07:00 +0900
categories: [ Python ]
tags: [ pytest ]
comments: true
---

pytest 실행시 발생하는 ModuleNotFoundError에 대한 글

## 개요

FastAPI test code를 작성하고 pytest로 확인하는 과정에서 ModuleNotFoundError가 발생했다.

project structure 아래와 같다
```console
├── src
│   ├── exceptions.py
│   ├── main.py
│   ├── models.py
│   └── pagination.py
├── tests
│   └── main
│       └── api_test.py
```

api_test.py의 내용은 아래와 같다.

```python
from fastapi.testclient import TestClient
from src.main import app
client = TestClient(app)

def test_read_main():
    response = client.get("/")
    assert response.status_code == 200
    assert response.json() == {"message": "Hello World"}
```
{: file='api_test.py'}


## pytest 수행 결과
프로젝트 경로(src,tests폴더를 자식으로 갖는 경로)에서 pytest를 수행하면 
```console
$ ls
src tests
```
아래와 같이 src를 import할 수 없다고 error 메시지가 나온다.
```console
$ pytest
...
==================================================== ERRORS ====================================================
___________________________________ ERROR collecting tests/main/api_test.py ____________________________________
ImportError while importing test module '/Users/solver/PycharmProjects/fastApiProjectTemplate/tests/main/api_test.py'.
Hint: make sure your test modules/packages have valid Python names.
Traceback:
/Library/Developer/CommandLineTools/Library/Frameworks/Python3.framework/Versions/3.9/lib/python3.9/importlib/__init__.py:127: in import_module
    return _bootstrap._gcd_import(name[level:], package, level)
tests/main/api_test.py:3: in <module>
    from src.main import app
E   ModuleNotFoundError: No module named 'src'
```

## 원인은 sys.path
관련해서 글을 찾아본결과 sys.path에 프로젝트 경로를 넣어주면 된다라는 방향의 글과,  
같은 의미로 PYTHONPATH 환경 변수에 프로젝트 경로를 넣어줘서 sys.path에 프로젝트 경로가 추가 되도록 하라는 내용이 많이 나왔다.

pytest 수행시 sys.path 값을 살펴보기위해 아래와 같은 임시 테스트 파일을 만들고  stdout을 출력하기위해 rP 옵션을 주고 실행해봤다.

```python
import sys

def test_tmp():
    print(sys.path)
```
{: file='tmp_test.py'}

```console
$ pytest -rP ./tests/main/tmp_test.py
...
==================================================== PASSES ====================================================
___________________________________________________ test_tmp ___________________________________________________
--------------------------------------------- Captured stdout call ---------------------------------------------
['/Users/solver/PycharmProjects/fastApiProjectTemplate/tests/main', '/Users/solver/PycharmProjects/fastApiProjectTemplate/venv/bin', '/Library/Developer/CommandLineTools/Library/Frameworks/Python3.framework/Versions/3.9/lib/python39.zip', '/Library/Developer/CommandLineTools/Library/Frameworks/Python3.framework/Versions/3.9/lib/python3.9', '/Library/Developer/CommandLineTools/Library/Frameworks/Python3.framework/Versions/3.9/lib/python3.9/lib-dynload', '/Users/solver/PycharmProjects/fastApiProjectTemplate/venv/lib/python3.9/site-packages']
```

프로젝트 경로가 `/Users/solver/PycharmProjects/fastApiProjectTemplate/`{: .filepath} 인데 해당 경로는 sys.path 값에 존재하지 않았다.

***python의 import는 sys.path 리스트에 있는 경로를 탐색하여 불러올 파일을 찾기 때문에 프로젝트 경로가 sys.path에 존재하지 않아 ModuleNotFoundError가 발생했던 것이다.***

## python -m pytest 과 pytest의 차이

```console
$ python -m pytest -rP ./tests/main/tmp_test.py
...
==================================================== PASSES ====================================================
___________________________________________________ test_tmp ___________________________________________________
--------------------------------------------- Captured stdout call ---------------------------------------------
['/Users/solver/PycharmProjects/fastApiProjectTemplate/tests/main', '/Users/solver/PycharmProjects/fastApiProjectTemplate', '/Library/Developer/CommandLineTools/Library/Frameworks/Python3.framework/Versions/3.9/lib/python39.zip', '/Library/Developer/CommandLineTools/Library/Frameworks/Python3.framework/Versions/3.9/lib/python3.9', '/Library/Developer/CommandLineTools/Library/Frameworks/Python3.framework/Versions/3.9/lib/python3.9/lib-dynload', '/Users/solver/PycharmProjects/fastApiProjectTemplate/venv/lib/python3.9/site-packages']
```

프로젝트 경로인 `/Users/solver/PycharmProjects/fastApiProjectTemplate/`{: .filepath}가 sys.path 목록에 존재한다.

`python -m pytest`의 sys.path에는 `/Users/solver/PycharmProjects/fastApiProjectTemplate/`{: .filepath}

`pytest`의 sys.path에는 `/Users/solver/PycharmProjects/fastApiProjectTemplate/venv/bin`{: .filepath}

sys.path 공식 docs[^docs]를 확인해보니 
***python -m module command line: prepend the current working directory.*** 라는 설명이 있었다.

그래서 python -m pytest로 실행하면 현재 경로를 sys.path에 추가해주는 것이 였고,
pytest로 pytest를 수행했을때 뒤에 /venv/bin이 붙는 이유는 아래보이는 것처럼 pytest binary 폴더의 위치를 넣어주기 때문이다.
```console
$ which pytest
/Users/solver/PycharmProjects/fastApiProjectTemplate/venv/bin/pytest
```

## Refercence
* <https://betterprogramming.pub/fastapi-best-practices-1f0deeba4fce/>
* <https://stackoverflow.com/questions/54895002/modulenotfounderror-with-pytest/>
* <https://stackoverflow.com/questions/14405063/how-can-i-see-normal-print-output-created-during-pytest-run/>

[^docs]: <https://docs.python.org/3/library/sys.html#sys.path/>
