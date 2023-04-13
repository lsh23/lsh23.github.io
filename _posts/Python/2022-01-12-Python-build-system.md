---
title: Python Build System
date: 2023-01-12 23:00:00 +0900
categories: [ Python ]
tags: [ Python ]
comments: true
---

Python Project를 build하는 방식에 대한 히스토리와 방법 소개

## 개요
python project를 build하는 방법에 대해서 찾아보면 여러가지 방법이 나온다.  
distutils, setuptools, poetry를 이용한 방법을 찾을 수 있다.
간단히 요약해보면 setup.py를 사용하는 방법은 legacy가 되었고, 현재는 distutils를 사용하는 방법은 legacy이며 python3.12 이후로는 deprecated 예정이다.  
권장하는 방법은 pyproject.toml과 을 이용하 방법이 권장된다.  
간단한 히스토리와 각각의 방법에 대해 설명하는글을 작성하려고 한다.

## setup.py(distutils)

파이썬 최초의 빌드 패키지,
단순히 코드를 패키징하고 빌드만 할수있다.
의존성 관리도 할 수 없는 아주 단순한 도구이다. 

```python
from distutils.core import setup
setup(
    name='mypackage',
    version='0.0.1',
    py_modules=['mypackage'], 
)
```
{: file='setup.py'}

```shell
python setup.py sdist
python setup.py bdist
```

## setup.py(setuptools)

위 `distutils`의 확장판  
distutils의 setup.py 포맷을 그대로 따르면서 의존성을 관리할 수 있게 됐고,    
easy_install이라는 명령어로 PyPI에 업로드된 패키지를 설치하는 기능까지 제공한다.
> 추후에 pip가 나오면서 `easy_install`을 통한 PyPI 패키지 설치는 잘 사용하지 않게 되었다.
{: .prompt-info }

```python
from setuptools import setup

setup(
    name='mypackage',
    version='0.0.1',
    install_requires=[
        'requests',
        'importlib-metadata; python_version == "3.8"',
    ],
)
```
{: file='setup.py'}
```shell
python setup.py sdist
python setup.py bdist_wheel
```


## pyproject.toml
위에서 본 setup.py을 사용하는 방법은 패키지의 메타데이터를 저장하는 setup.py가 setuptools가에 의존적이고 종속적이라는 문제가 있다.  
따라서 특정 빌드시스템에 종속적이지 않는 방식이 필요했고 이에 따라 pyproject.toml이 등장하게 됐다.

pyproject.toml에 어떤 빌드시스템을 사용할지 지정할 수 있다. setuptools도 사용할 수 있고, poetry[^poetry], flit등 다른 빌드시스템을 사용한다고 선언하기만 하면 된다.




```python
[build-system]
requires = ["setuptools"]
build-backend = "setuptools.build_meta"

[project]
name = "mypackage"
version = "0.0.1"
dependencies = [
    "requests",
    'importlib-metadata; python_version<"3.8"',
]
```
{: file='pyproject.toml'}

> 위의 예시는 build system을 setuptools로 선언한 예시다.
{: .prompt-info }


```shell
python -m build
```

## 정리
패키지의 메타데이터를 저장하는 파일이 `setup.py(distutils) -> setup.py(setuptools) -> pyproject.toml` 순으로 발전해왔고 

setup.py의 경우 setuptools에 종속적이였으나, pyproject.toml로 오면서 특정 빌드시스템에 종속적인 문제가 해결됐음을 알 수있다.  

오픈소스프로젝트를 보면 아직도 setup.py를 사용하는 프로젝트도 많이보이고 pyproject.toml을 사용하는 프로젝트도 많이 보인다.  

위의 히스토리를 간단하게 이해만 하고 있어도 파이썬 패키징에 대해 혼동하지 않고 방향을 잡아갈 수 있다고 생각한다. 위에 나온 방법들로 간단한 프로젝트를 패키징해보면 이해하는데 도움이 될 것이다. 또 최근 poetry라고 하는 툴은 프로젝트 셋업부터 PyPI 업로드까지 전부지원하므로 한번 사용해보는 것을 추천한다.

## reference
* <https://ryanking13.github.io/2021/07/11/python-packaging.html/>
* <https://tech.buzzvil.com/blog/setup.py-%EB%A9%88%EC%B6%B0//>
* <https://setuptools.pypa.io/en/latest/userguide/quickstart.html#setup-py/>
* <https://blog.gyus.me/2020/python-packaging-history/>

[^poetry]: https://python-poetry.org/