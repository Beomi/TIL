<!-- $theme: gaia -->
<!-- $width=1024 -->
<!-- $height=768 -->

# Gnuboard to Django
# Series #01
### (그누보드와 장고가 한 유저DB를 사용하기)
#### 2016. 09. 28.
#### 이준범(jun@beomi.net)
---
# 사건의 발단
### (이라 쓰고 삽질의 시작이라 읽는다)
---
## 구상
1. 현재 웹 서비스 하나가 ==그누보드==로 돌아가고있다.
2. 새로운 서비스를 만들고 싶다! 근데 PHP는 싫어
3. 그리고 싸이클럽에서 갈아엎은지도 얼마 되지 않았는데..
4. 또 새롭게 가입하게 만들면 ==유저의 피로도==는?

--> Soultions

1. 그냥 다 갈아엎자.
2. 포기하면.. 편해
3. Django가 ==기존 DB를 사용==할 수 있진 않을까? --> 선택!
---
# Django가 기존 DB를 <br> 사용하게 하자
---
## 어떻게 해야하나(난관들)
1. 기존 DB구조에 맞게 장고가 동작하게 하려면?
   수백개의 DB 스키마를 직접 짤수는 도저히 없다.
2. 제일 큰 문제: 유저 로그인
   장고에서 기본으로 제공해주는 암호화 방식이 아니다!
   (그누보드는 MySQL의 password()함수를 이용한다)
3. 사용자가 눈치채지 못하고 기존 서비스와
   새로운 서비스가 연속적으로 이어져야 한다.
---
## Solutions
1. 어떻게 장고가 기존 DB를 사용하게 할까?
  - 장고에는 자동으로 DB(default로 설정된)의 <br>스키마를 가져오게 할 수 있다!
    ```sh
    $ python manage.py inspectdb > legacydb.py
    ```
  - inspectdb를 사용할 경우 자동으로 <br>장고에서 사용할 수 있는 모델단을 만들어 준다. <br>단, Relation와 위치관계는 수동으로 정렬해줘야 한다.
  - inspectdb는 기본적으로 default로 설정된 DB를 체크
    따라서 검사할 DB를 미리 settings.py에 등록해두자.
---
## Solutions
2. 어떻게 장고가 그누보드의 User 모델을 사용하게 할까?
  - 필요한 것
    - 1번에서 만든 DB Model
      - 이 안에 많은 추가 설정이 필요하다.
    - Custom User매니저(BaseUserManage를 상속받은)
    - Custom AuthBackend
---
# 만들어봅시다
---
#### InspectDB로 만든<br>models.py와 G5Member [[source]](https://gist.github.com/Beomi/e4774fa948207acd24fb13c2c84d8120)
```py
# 공간의 제약으로 일부 모델을 생략했습니다. 원본은 source를 참고해주세요
class G5Member(AbstractBaseUser):
    mb_no = models.AutoField(primary_key=True)
    email = models.CharField(max_length=255, db_column='mb_email', null=True)
    mb_level = models.IntegerField()
    mb_id = models.CharField(unique=True, max_length=20)
    password = models.CharField(max_length=255, db_column='mb_password')
    last_login = models.DateTimeField(db_column='mb_today_login', null=True)
    username = models.CharField(max_length=255, db_column='mb_nick', null=True)
    
    USERNAME_FIELD = 'mb_id' #unique=True여야 합니다
    REQUIRED_FIELDS = ['mb_level','email','username']
```
---
```
    def get_username(self):
        return self.mb_id
    def has_perm(self, perm, obj=None):
        return self.is_superuser()
    def has_module_perms(self, app_label):
        return self.is_superuser()
    def get_short_name(self):
        return self.username
    def is_superuser(self):
        if self.mb_level == 10: # 그누보드는 Level10이 관리자
            return True
        else:
            return False
    def is_staff(self):
        if self.mb_level == 10: # 그누보드는 Level10이 관리자
            return True
        else:
            return False
```
---
```
    class UserManager(BaseUserManager):
        use_in_migrations = True
        def _create_user(self, username, email, password, **extra_fields):
            """
            Creates and saves a User with the given username, email and password.
            """
            if not username:
                raise ValueError('The given username must be set')
            email = self.normalize_email(email)
            username = self.model.normalize_username(username)
            user = self.model(email=email, username=username)
            user.set_password(password)
            user.save(using=self._db)
            return user

        def create_user(self, username, email=None, password=None, **extra_fields):
            extra_fields.setdefault('is_staff', False)
            extra_fields.setdefault('is_superuser', False)
            return self._create_user(username, email, password, **extra_fields)
```
---
```
        def create_superuser(self, username, email, password, **extra_fields):
            extra_fields.setdefault('is_staff', True)
            extra_fields.setdefault('is_superuser', True)

            if extra_fields.get('is_staff') is not True:
                raise ValueError('Superuser must have is_staff=True.')
            if extra_fields.get('is_superuser') is not True:
                raise ValueError('Superuser must have is_superuser=True.')

            return self._create_user(username, email, password, **extra_fields)
            
    objects = UserManager()

    class Meta:
        managed = False
        db_table = 'g5_member'
```
---
## Point
- 최대한 그누보드 DB이름을 사용
- Django AUTH_USER_MODEL에서 원하는 User Attribute를 모두 추가해두기
  - USERNAME_FIELD는 장고가 Unique한 username으로 관리하는 DB 컬럼을 지정
  - password필드의 이름은 바꾸지 못한다.
    따라서 db_column='컬럼명'으로
    기존 DB의 암호 컬럼에 연결해야한다.
  - 위 코드에 쓰여진 method/attribute는
    장고 BaseUserManager가 필요로 하는 최소.
---
#### Custom Auth 만들기[[source]](https://gist.github.com/Be…/c72a72382bc6667b727ef24397e54c7c)
```py
from django.utils.crypto import constant_time_compare
import hashlib
from .models import G5Member as User
class MyUserAuthBackend(object):
    def check_legacy_password(self, db_password, supplied_password):
        return constant_time_compare('*'+ hashlib.sha1(hashlib.sha1(supplied_password.encode('utf-8')).digest()).hexdigest().upper(), db_password)
    def authenticate(self, username=None, password=None):
        try:
            user = User.objects.get(mb_id=username)
            if '$' not in user.password:
                if self.check_legacy_password(user.password, password):
                    return user
                else:
                    return None
            else:
                if user.check_password(password):
                    return user
        except User.DoesNotExist:
            return None
```
---
```
    def get_user(self, user_id):
        try:
            return User.objects.get(pk=user_id)
        except User.DoesNotExist:
            return None
```
- 이와 같이 작성해야한다.
- 이 파일의 MyUserAuthBackend를 장고 프로젝트의 settings파일에 등록해주어야 한다.
---
#### CustomAuth 등록하기[[source]](https://gist.github.com/Beomi/ec88c2788334cff35a8d836495505800)
```py
# ...somesettings..

INSTALLED_APPS = [
    # ...
    'auth_app',
    # ...
]

AUTHENTICATION_BACKENDS = ['auth_app.auth.MyUserAuthBackend']

AUTH_USER_MODEL = (
    'auth_app.G5Member'
)

# othersettings...
```
---
# 정리
---
### 정리
- 이와같이 유저모델을 정의해준다면 장고에서 그누보드에서 사용하는 유저를 함께 사용할 수 있다.
- 그누보드를 유지하고 그누보드쪽에서 회원 생성을 관리하고 뷰와 다른 로직들을 장고에서 사용할 수 있다.
- Happy Coding with Django!
---
# Source 정리

## 장고 앱중 하나에서 [models.py](https://gist.github.com/Be…/e4774fa948207acd24fb13c2c84d8120)
## 위 models.py와 함께하는 [auth.py](https://gist.github.com/Be…/c72a72382bc6667b727ef24397e54c7c)
## 장고 프로젝트 [settings.py](https://gist.github.com/Be…/ec88c2788334cff35a8d836495505800)