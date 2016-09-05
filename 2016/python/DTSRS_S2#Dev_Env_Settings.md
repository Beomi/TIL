#2016.09.05(Mon)

## 장고 투스쿱 읽기모임 월요일 2회차
####1장: 코딩컨벤션
1. '암시적' import VS '명시적' import가 헷갈리는 것에 대해.
    ```
    #corn/views.py
    from corn.models import ClassName // 암시적import
    from .models import ClassName // 명시적import
    ```
    가장 우선적으로 알아야 하는 사실은 위 아래 모두 상대 import라는 것이다.
    명시적 / 암시적이라는 단어의 뜻이 경로를 명시적/암시적으로 알려준다는 것이 아니다.
    위의 import의 경우에는 corn이라는 모듈이 어디에서 왔는지 명시적이지 않다.
    즉, 외부 모듈인지, 앱이름인지 한눈에 식별이 불가능하다는 것이다.
    '상대'import라는 것을 '암시적'으로 혹은 '명시적'으로 드러낸다는 뜻이다.

2. PEP8 vs Flake8 vs etc..?
    파이썬에서 가장 잘 통용화되는 PEP8 -> 너무 짧다!
    코딩컨벤션은 한 프로젝트 내에서 통일

####2장: 장고 개발환경
1. 같은 DB를 이용할 것.
    Production레벨과 Dev(Local)레벨에서의 사용 DB가같아야함.
    기본적으로 장고ORM이 잡아주지만, 잡아주지 못하는 부분도있음.
    DB단에서 문제가 생기는 경우에는 해결이 어려움.
2. pip + VirtualEnv
3. Git
    Git등을 통한 버전관리는 필수!
    그러나, 세팅 파일 업로드 / 특히, API키 등은 업로드금지!
4. Docker?

####3장: 장고프로젝트 형태
1. django-admin startproject
2. cookiecutter-django
    투스쿱 책과는 다르게 프로젝트 디렉토리가 구성되어있음. 주의!
    쿠키커터를 이용해 프로젝트 생성 후 기존 프로젝트에 작성 내용만 복사 붙여넣기.

####4장: 장고 앱디자인
1. 앱이름 정하기
    모델에서 가장 핵심적인 모델Class의 복수형
    단, blog등은 예외(고유적인 역할)

####5장: Settings / requirements 설정 / 환경변수
1. Production Level에 따른 다른 settings파일
    Dev / Production 등의 서버에 따라 다른 세팅파일 이용하기.
    settings.py를 없애고 settings/local.py settings/production.py으로 교체
    단, wsgi.py와 manage.py파일에서 settings의 위치에 .local .production을 적어줘야한다.
    (하지않으면 순환참조 오류가 난다)
2. 환경변수를 통한 인자 전달
    git등의 버전관리에 올리면 안되는 보안상의 정보(DB연결정보 / API Key 등) 등은
    환경변수를 통해 인자로 전달해줘야 한다.
    json파일로 로딩하거나 ini 파일 등으로도 전달이 가능함.
    AWS에서 인스턴스 실행시 자동으로 init Script를 통해 환경변수 설정.
