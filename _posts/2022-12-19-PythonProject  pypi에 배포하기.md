---
title: PythonProject PyPI에 배포하기
date: 2022-12-21 21:35:00 +0900
categories: [ Python ]
tags: [ Python, PyPI ]
comments: true
---

PyPI(The Python Packages Index)에 내가 개발한 python project 배포하기

## 개요
python으로 개발을 하는 누구나 pip를 통해서 유용한 package를 설치하고 import 해서 개발해본 경험이 있을 것이다. 해당 글에서는 무심코 사용해 왔던 pip에 대해서 간단히 설명하고, 또한 직접 개발한 module을 다른 개발자가 사용할 수 있도록 배포하는 방법에 대해서 알아본다. 

## PyPI와 PIP
### PyPI(Python Package Index)  
PyPI는 python package 저장소이다.  
pip install 명령어를 통해서 설치하는 pacakge를 저장하고 있는 저장소이다.  
다른 언어 진영의 예시를 들어보면, javascript는 npm registry, java는 mvn repository등과 같다고 생각하면 된다.
### PIP(Package Installer for Python)
PIP는 python pacakge 괸리 도구라고 이해하면 된다.
PIP를 통해서 PyPI에 저장된 pacakge를 로컬 환경에 설치할 수 있고, 삭제할 수 있으며, 설치된 pacakge를 조회한다거나 하는 등의 기능을 제공한다.


## 배포 - 소스 구성
`src` 폴더 하위에 프로젝트 코드가 위치하고,  
`pyproject.toml`파일은 프로젝트의 metadata를 가지고 있는 파일이다.  
pyproject.toml의 자세한 내용은 공식 문서를 참고하자.[^pyproject_toml]
> setup.cfg, setup.py를 이용하는 방법은 legacy 방식이며, python 3.12 부터 지원하지 않을 예정이다.
{: .prompt-info }

```
pypi-deploy/
├── LICENSE
├── pyproject.toml
├── README.md
├── src
│   ├── lsh23_calculator
│       ├── calculator.py
│       └── __init__.py
├── tests
│   ├── __init__.py
│   └── test_calculator.py
```

```toml
[project]
name = "lsh23-calculator"
version = "0.0.1"
description = "A deploy test package"
authors = [
  { name="lsh23", email="sehyeong.dev@gmail.com" },
]
readme = "README.md"
requires-python = ">=3.7"
license = {file = "LICENSE"}
classifiers = [
    "Programming Language :: Python :: 3",
    "License :: OSI Approved :: MIT License",
    "Operating System :: OS Independent",
]

[project.urls]
"Homepage" = "https://github.com/lsh23/lsh23-calculator"
"Bug Tracker" = "https://github.com/lsh23/lsh23-calculator/issues"

[build-system]
requires = ["setuptools>=61.0"]
build-backend = "setuptools.build_meta"
```
{: file='pyproject.toml'}




## 배포 - build
python build 모듈 실행
```console
$ python -m build
* Creating venv isolated environment...
* Installing packages in isolated environment... (setuptools>=61.0)
...
...
Successfully built lsh23-calculator-0.0.1.tar.gz and lsh23_calculator-0.0.1-py3-none-any.whl
```

빌드가 성공되었다면 프로젝트 루트 경로의 dist 폴더 아래에  
tar.gz(source distribution)과 whl(built distribution) 두가지 형태로 빌드된 파일이 생성된다.
> tar.gz(sdist)는 소스파일을 통째로 압축한 파일이고, whl(bdist)은 바로 실행가능한 형태의 파일이다. 
> 만약 PyPI 저장소에 sdist 파일만 존재하면 pip install시 sdist을 다운로드하고 압축을 풀고 build를 해서 whl 파일을 생성 후 설치를 진행하고, 저장소에 bdist 파일이 존재하면 build 과정 없이 바로 설치된다.
{: .prompt-info }
## 배포 - pypi 계정 준비
PyPI로 업로드를 위해 계정을 등록한다. https://pypi.org/account/register/

## 배포 - 업로드
PyPI로 업로드를 도와주는 유틸리티 twine을 설치하고 업로드 한다.
```console
$ python3 -m pip install twine
```
```console
$ python3 -m twine upload dist/*
Uploading distributions to https://upload.pypi.org/legacy/
Enter your username: lsh23
Enter your password: 
Uploading lsh23_calculator-0.0.1-py3-none-any.whl
100% ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 64.4/64.4 kB • 00:00 • 121.9 MB/s
Uploading lsh23-calculator-0.0.1.tar.gz
100% ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 75.8/75.8 kB • 00:00 • 115.2 MB/s

View at:
https://pypi.org/project/lsh23-calculator/0.0.1/

```

## reference
* <https://github.com/pypa/sampleproject/>
* <https://packaging.python.org/en/latest/tutorials/packaging-projects/>
* <https://towardsdatascience.com/setuptools-python-571e7d5500f2/>
* <https://towardsdatascience.com/requirements-vs-setuptools-python-ae3ee66e28af/>
* <https://setuptools.pypa.io/en/latest/userguide/development_mode.html/>
* <https://packaging.python.org/en/latest/discussions/src-layout-vs-flat-layout//>
* <https://realpython.com/python-wheels//>
[^pyproject_toml]: <https://pip.pypa.io/en/stable/reference/build-system/pyproject-toml/>
