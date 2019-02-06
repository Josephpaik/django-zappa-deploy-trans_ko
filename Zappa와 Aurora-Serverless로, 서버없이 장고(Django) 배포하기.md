# Zappa와 Aurora-Serverless로, 서버없이 장고(Django) 배포하기

Lambda & Aurora-Serverless를 사용하여 django를 서버없이 완벽하게 생성합니다



## 서버없는 장고 배포

Zappa를 사용하여 Django 애플리케이션을 AWS Lambda에 배포하고 AWS Aurora-Serverless를 DB로 사용하는 방법을 살펴 보겠습니다.

AWS Lambda는 완전히 이벤트 중심이며, 컴퓨팅 리소스를 자동으로 관리하는 amazon의 서버리스 컴퓨팅 플랫폼입니다. 응용 프로그램 요청에 따라 필요할 때 자동으로 크기가 조정됩니다.

Zappa는 Python 응용 프로그램을 AWS-Lambda에 배포하는 데 사용되는 Python 프레임 워크입니다. Zappa는 자동으로 모든 설정과 배포를 처리합니다.

그리고 Aurora Serverless는 Amazon AWS의 온디맨드로 자동 확장되는 관계형 데이터베이스 시스템(현재 MySQL과 호환 가능)입니다. 요구사항에 따라 DB를 자동으로 시작하고 종료합니다.



### 환경 설치 및 구성
#### AWS 자격 증명 구성
먼저 AWS를 사용하기 전에 유효한 AWS 계정이 있고 aws 환경 변수 (액세스 키)가 있는지 확인해야합니다.

그런 다음 루트 수준에서 폴더를 만듭니다.

``` shell
 $ mkdir .aws
```

이 폴더에 "credentials"라는 파일을 만들고 aws_access_key_id 및 aws_secret_access_key를 저장합니다.  AWS 액세스 자격 증명을 찾으려면, 먼저

- AWS 콘솔의 IAM 대시 보드로 이동한 후
- "사용자"를 클릭합니다. 그 다음
- "사용자 이름"을 클릭합니다.
- 그 다음 ""보안 자격 증명" 탭으로 이동합니다.
- 그리고 "액세스 키""로 이동합니다.
- 액세스 키 ID(access_key_id)를 메모해둡니다. secret_access_key는 새 사용자를 만들거나 새 액세스 키를 만들 때만 표시되므로 사용자 생성시 access_key_id와 secret_access_key를 기록하거나, 새로 액세스 키를 만들어 두 키를 모두 가지고 있어야합니다 .

``` shell
 ###~/.aws/credentials
  [default]
  aws_access_key_id= XXXXXXXXXXXXXXXXXXXX
  aws_secret_access_key=XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
```



#### Django 앱으로 이동

aws 자격 증명 파일을 설정 한 후 django 프로젝트로 가보겠습니다. 여기서는 Pollsapi (https://github.com/agiliq/building-api-django)를 django 프로젝트로 사용했습니다. 그다음 이 repository에서 pollsapi 앱으로 들어갑니다.

프로젝트에 대한 가상 env를 만들고 아래 명령을 수행합니다.

```
$ pip install -r requirements.txt
```



#### Zappa 설치 및 구성

다음은 zappa 설치

```shell
$ pip install zappa
```

Zappa를 설치 한 후 zappa를 초기화합니다.

```shell
$ zappa init
```

초기화하면서 아래처럼 환경설정을 합니다.

- 환경의 이름 - 디폴트 'dev'

- S3 버켓. 버킷이 없으면 zappa가 생성합니다. Zappa는이 버킷을 사용하여 zappa 패키지를 AWS Lambda로 전송하는 동안 임시로 보관하고 배포가 끝나면 삭제합니다.

  (S3 버킷을 생성하는 것이 더 좋습니다. 나중에 애플리케이션의 정적 파일을 호스팅하는 데 사용합니다)

- 프로젝트 설정 - 'pollsapi.settings' 내용을 가져옴

  

Zappa는 자동으로 Django 설정 파일과 Python 런타임 버전을 찾습니다.







