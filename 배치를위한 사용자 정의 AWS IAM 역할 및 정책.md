#### 배치를 위한 사용자 정의 AWS IAM 역할 및 정책

AWS 자격 증명 파일의 프로필에 해당하는 profile_name 설정을 정의하여 자파 응용 프로그램을 배포하는 데 사용할 로컬 프로필을 지정할 수 있습니다.



#### 실행을 위한 사용자 정의 AWS IAM 역할 및 정책

Zipa가 Lambda를 실행하기 위해 만든 기본 IAM 정책은 매우 관대합니다. CloudWatch, S3, Kinesis, SNS, SQS, DynamoDB 및 Route53 유형의 모든 자원에 대한 모든 작업에 대한 액세스 권한을 부여합니다. lambda : 모든 람다 리소스에 대한 InvokeFunction; 모든 엑스레이 자원을 사용하십시오. 모든 EC2 리소스에 대한 모든 네트워크 인터페이스 작업.  이렇게 하면 대부분의 Lambda가 추가 권한없이 올바르게 작동 할 수 있지만 일반적으로 대부분의 지속적인 통합 파이프 라인 또는 프로덕션 배포에는 허용되는 권한 집합이 아닙니다.  대신 IAM 정책을 수동으로 관리해야 할 수 있습니다.

람다 실행 역할의 정책을 수동으로 정의하려면 manage_roles를 false로 설정하고 Zappa 설정 파일에서 role_name 또는 role_arn을 정의해야합니다.

``` json
{
    "dev": {
        ...
        "manage_roles": false, // Disable Zappa client managing roles.
        "role_name": "MyLambdaRole", // Name of your Zappa execution role. Optional, default: <project_name>-<env>-ZappaExecutionRole.
        "role_arn": "arn:aws:iam::12345:role/app-ZappaLambdaExecutionRole", // ARN of your Zappa execution role. Optional.
        ...
    },
    ...
}
```



Zappa 배포에 필요한 최소 정책 요구 사항에 대한 지속적인 토론은 [여기](https://github.com/Miserlou/Zappa/issues/244)에서 찾을 수 있습니다. 이러한 인 타이틀먼트를 관리하는 더 강력한 솔루션이 곧 구현 될 것입니다.



``` shell
(venv) jspaik@ZenBook3-jspaik:~/building-api-django/pollsapi$ zappa deploy dev
Calling deploy for stage dev..
Creating pollsapi-dev-ZappaLambdaExecutionRole IAM Role..
Error: Failed to manage IAM roles!
You may lack the necessary AWS permissions to automatically manage a Zappa execution role.
To fix this, see here: https://github.com/Miserlou/Zappa#custom-aws-iam-roles-and-policies-for-deployment

(venv) jspaik@ZenBook3-jspaik:~/building-api-django/pollsapi$ zappa deploy dev
Calling deploy for stage dev..
Creating pollsapi-dev-ZappaLambdaExecutionRole IAM Role..
Creating zappa-permissions policy on pollsapi-dev-ZappaLambdaExecutionRole IAM Role.

```

deployuser에 IAMFullAccess권한 추가함

그 결과는... "CREATE_FAILED"

![](D:\Dev\Zappa_Aurora-serverless\img\이미지 20190206_001_cloudformation_eror.png)

APIGateway Post권한이 없음.

![](D:\Dev\Zappa_Aurora-serverless\img\이미지 20190206_003.png)



현재 deployuser의 현재 권한은 AmazonAPIGatewayInvokeFullAccess, 이 권한으로는 POST 불가하므로,

[AmazonAPIGatewayAdministrator](https://console.aws.amazon.com/iam/home?region=ap-northeast-2#/policies/arn%3Aaws%3Aiam%3A%3Aaws%3Apolicy%2FAmazonAPIGatewayAdministrator) 권한으로 대체한다. 아래 그림 참고!



![](D:\Dev\Zappa_Aurora-serverless\img\이미지 20190206_001.png)



에러난 배포버전을 아래 명령으로 거두어 들인다(undeploy)

``` shell
$zappa undeploy dev
```



새로운 권한 설정으로 다시 배포(deploy)한다. 



``` shell
(venv) jspaik@ZenBook3-jspaik:~/building-api-django/pollsapi$ zappa deploy dev
Calling deploy for stage dev..

```

(조금 많이 기다려야 한다...)

```shell
Downloading and installing dependencies..
 - markupsafe==1.1.0: Using locally cached manylinux wheel
 - sqlite==python36: Using precompiled lambda package
Packaging project as zip.
Uploading pollsapi-dev-1549419081.zip (16.9MiB)..
100%|████████████████████████████████████████████████████████████████| 17.7M/17.7M [00:30<00:00, 564KB/s]
Scheduling..
Scheduled pollsapi-dev-zappa-keep-warm-handler.keep_warm_callback with expression rate(4 minutes)!
Uploading pollsapi-dev-template-1549421234.json (1.6KiB)..
100%|███████████████████████████████████████████████████████████████| 1.63K/1.63K [00:00<00:00, 37.1KB/s]
Waiting for stack pollsapi-dev to create (this can take a bit)..
 75%|███████████████████████████████████████████████████▊                 | 3/4 [00:09<00:04,  4.87s/res]
Deploying API Gateway..
Deployment complete!: https://8sczsdgiy1.execute-api.ap-northeast-2.amazonaws.com/dev
(venv) jspaik@ZenBook3-jspaik:~/building-api-django/pollsapi$
```



배포가 완료되었다



![](D:\Dev\Zappa_Aurora-serverless\img\이미지 20190206_004.png)



웹브라우저를 열어 아래 배포 주소로 접속한다.

https://8sczsdgiy1.execute-api.ap-northeast-2.amazonaws.com/dev

접속하면 아래와 같이 보안 오류 메시지가 나타난다. 

![](D:\Dev\Zappa_Aurora-serverless\img\이미지 20190206_005.png)

메시지는 ALLOWED_HOSTS에 배포서버 주소를 추가가 필요하다는 내용이다.

그래서,  `pollsapi/settings.py` 의 ALLOWED_HOSTS 항목을 아래처럼 수정한다.

```python
ALLOWED_HOSTS = ['127.0.0.1], "8sczsdgiy1.execute-api.ap-northeast-2.amazonaws.com"]
```



pollsapi/settings.py 파일이 있는 경로는 deploy한 위치가 아니므로 deploy update하는 위치가 틀리지 않도록 주의한다.

settings.py 파일이 있는 곳에서 deploy update하면 아래와 같은 에러 메시지가 나타난다.

(참고: ll 명령은 ls -al 의 alias이다.)

``` shell
(venv) jspaik@ZenBook3-jspaik:~/building-api-django/pollsapi$
(venv) jspaik@ZenBook3-jspaik:~/building-api-django/pollsapi$ ll
total 28
drwxrwxrwx 1 jspaik jspaik 4096 Feb  6 11:47 ./
drwxrwxrwx 1 jspaik jspaik 4096 Feb  6 07:36 ../
-rw-rw-rw- 1 jspaik jspaik   29 Feb  6 07:36 .gitignore
-rw-rw-rw- 1 jspaik jspaik 5820 Feb  6 07:36 docs.raml
-rw-rw-rw- 1 jspaik jspaik 6245 Feb  6 07:36 docs.swagger.json
-rwxrwxrwx 1 jspaik jspaik  540 Feb  6 07:36 manage.py*
drwxrwxrwx 1 jspaik jspaik 4096 Feb  6 07:36 polls/
drwxrwxrwx 1 jspaik jspaik 4096 Feb  6 07:36 pollsapi/
-rw-rw-rw- 1 jspaik jspaik 1628 Feb  6 10:27 pollsapi-dev-template-1549416430.json
-rw-rw-rw- 1 jspaik jspaik 1628 Feb  6 10:42 pollsapi-dev-template-1549417358.json
-rw-rw-rw- 1 jspaik jspaik 1628 Feb  6 10:49 pollsapi-dev-template-1549417763.json
-rw-rw-rw- 1 jspaik jspaik   83 Feb  6 07:39 requirements.txt
drwxrwxrwx 1 jspaik jspaik 4096 Feb  6 07:46 venv/
-rw-rw-rw- 1 jspaik jspaik  250 Feb  6 08:05 zappa_settings.json
(venv) jspaik@ZenBook3-jspaik:~/building-api-django/pollsapi$ cd pollsapi/
(venv) jspaik@ZenBook3-jspaik:~/building-api-django/pollsapi/pollsapi$ ll
total 8
drwxrwxrwx 1 jspaik jspaik 4096 Feb  6 07:36 ./
drwxrwxrwx 1 jspaik jspaik 4096 Feb  6 11:47 ../
-rw-rw-rw- 1 jspaik jspaik    0 Feb  6 07:36 __init__.py
-rw-rw-rw- 1 jspaik jspaik 3478 Feb  6 07:36 settings.py
-rw-rw-rw- 1 jspaik jspaik  797 Feb  6 07:36 urls.py
-rw-rw-rw- 1 jspaik jspaik  393 Feb  6 07:36 wsgi.py
(venv) jspaik@ZenBook3-jspaik:~/building-api-django/pollsapi/pollsapi$ vi settings.py
(venv) jspaik@ZenBook3-jspaik:~/building-api-django/pollsapi/pollsapi$ zappa update dev
Oh no! An error occurred! :(

==============

Traceback (most recent call last):
  File "/home/jspaik/building-api-django/pollsapi/venv/lib/python3.6/site-packages/zappa/cli.py", line 2712, in handle
    sys.exit(cli.handle())
  File "/home/jspaik/building-api-django/pollsapi/venv/lib/python3.6/site-packages/zappa/cli.py", line 488, in handle
    self.load_settings_file(self.vargs.get('settings_file'))
  File "/home/jspaik/building-api-django/pollsapi/venv/lib/python3.6/site-packages/zappa/cli.py", line 2129, in load_settings_file
    settings_file = self.get_json_or_yaml_settings()
  File "/home/jspaik/building-api-django/pollsapi/venv/lib/python3.6/site-packages/zappa/cli.py", line 2109, in get_json_or_yaml_settings
    raise ClickException("Please configure a zappa_settings file or call `zappa init`.")
click.exceptions.ClickException: Please configure a zappa_settings file or call `zappa init`.

==============

Need help? Found a bug? Let us know! :D
File bug reports on GitHub here: https://github.com/Miserlou/Zappa
And join our Slack channel here: https://slack.zappa.io
Love!,
 ~ Team Zappa!
(venv) jspaik@ZenBook3-jspaik:~/building-api-django/pollsapi/pollsapi$ cd ..
(venv) jspaik@ZenBook3-jspaik:~/building-api-django/pollsapi$ ll
total 28
drwxrwxrwx 1 jspaik jspaik 4096 Feb  6 11:47 ./
drwxrwxrwx 1 jspaik jspaik 4096 Feb  6 07:36 ../
-rw-rw-rw- 1 jspaik jspaik   29 Feb  6 07:36 .gitignore
-rw-rw-rw- 1 jspaik jspaik 5820 Feb  6 07:36 docs.raml
-rw-rw-rw- 1 jspaik jspaik 6245 Feb  6 07:36 docs.swagger.json
-rwxrwxrwx 1 jspaik jspaik  540 Feb  6 07:36 manage.py*
drwxrwxrwx 1 jspaik jspaik 4096 Feb  6 07:36 polls/
drwxrwxrwx 1 jspaik jspaik 4096 Feb  6 12:52 pollsapi/
-rw-rw-rw- 1 jspaik jspaik 1628 Feb  6 10:27 pollsapi-dev-template-1549416430.json
-rw-rw-rw- 1 jspaik jspaik 1628 Feb  6 10:42 pollsapi-dev-template-1549417358.json
-rw-rw-rw- 1 jspaik jspaik 1628 Feb  6 10:49 pollsapi-dev-template-1549417763.json
-rw-rw-rw- 1 jspaik jspaik   83 Feb  6 07:39 requirements.txt
drwxrwxrwx 1 jspaik jspaik 4096 Feb  6 07:46 venv/
-rw-rw-rw- 1 jspaik jspaik  250 Feb  6 08:05 zappa_settings.json
(venv) jspaik@ZenBook3-jspaik:~/building-api-django/pollsapi$ zappa update dev
Calling update for stage dev..

(잠시 기다린다...)

Downloading and installing dependencies..
 - markupsafe==1.1.0: Using locally cached manylinux wheel
 - sqlite==python36: Using precompiled lambda package
Packaging project as zip.
Uploading pollsapi-dev-1549425202.zip (16.9MiB)..
100%|████████████████████████████████████████████████████████████████| 17.7M/17.7M [00:30<00:00, 572KB/s]
Updating Lambda function code..
Updating Lambda function configuration..
Uploading pollsapi-dev-template-1549425427.json (1.6KiB)..
100%|███████████████████████████████████████████████████████████████| 1.63K/1.63K [00:00<00:00, 39.1KB/s]
Deploying API Gateway..
Scheduling..
Unscheduled pollsapi-dev-zappa-keep-warm-handler.keep_warm_callback.
Scheduled pollsapi-dev-zappa-keep-warm-handler.keep_warm_callback with expression rate(4 minutes)!
Your updated Zappa deployment is live!: https://8sczsdgiy1.execute-api.ap-northeast-2.amazonaws.com/dev
(venv) jspaik@ZenBook3-jspaik:~/building-api-django/pollsapi$

```



업데이트 완료!



![](D:\Dev\Zappa_Aurora-serverless\img\이미지 20190206_006.png)

정적 파일을 사용할 수 없다 !!



### 정적 파일 서비스하기

정적 파일을 제공하기 위해 S3 버킷 (이전에 생성 한 버킷)을 사용합니다.

우리는 브라우저가 다른 URL에서 리소스 / 파일을 가져올 수있게하는 S3 버킷에 대해 [CORS](Cross-Origin Resource Sharing)를 활성화해야합니다. S3 버킷 속성으로 이동한 다음 권한으로 이동하여 CORS 구성을 클릭하고 아래 행을 붙여 넣습니다.

``` XML
<CORSConfiguration>
        <CORSRule>
            <AllowedOrigin>*</AllowedOrigin>
            <AllowedMethod>GET</AllowedMethod>
            <MaxAgeSeconds>3000</MaxAgeSeconds>
            <AllowedHeader>Authorization</AllowedHeader>
        </CORSRule>
</CORSConfiguration>
```



S3를 위한 Django 구성

``` shell
(venv) jspaik@ZenBook3-jspaik:~/building-api-django/pollsapi$ pip install django-s3-storage
Collecting django-s3-storage
  Downloading https://files.pythonhosted.org/packages/68/6e/7150252d92028b88b5f73a3fb727f6370f799ffdca3462a0757e1a5ce414/django-s3-storage-0.12.4.tar.gz
(... 중간 생략 ...)
>botocore<1.13.0,>=1.12.87->boto3<2,>=1.4.4->django-s3-storage) (1.12.0)
Building wheels for collected packages: django-s3-storage
  Building wheel for django-s3-storage (setup.py) ... done
  Stored in directory: /home/jspaik/.cache/pip/wheels/56/e5/82/1f4eb367d114e896f86ac04b903e0de7e9439cc1d7ffcadbff
Successfully built django-s3-storage
Installing collected packages: django-s3-storage
Successfully installed django-s3-storage-0.12.4
(venv) jspaik@ZenBook3-jspaik:~/building-api-django/pollsapi$
```



그리고  `requirements.txt` 파일에 아래 내용을 추가한다.

```
...
django-s3-storage==0.12.4
...
```

이제 *settings.py* 파일의  `INSTALLED_APPS` 에  *‘django_s3_storage’* 를 추가한다.

```
INSTALLED_APPS = (
          ...,
          'django_s3_storage',
     )
```

그리고나서 아래 내용을 *settings.py* 파일 맨 끝에 추가한다.

```python
S3_BUCKET = "zappa-staticfiles1234"

STATICFILES_STORAGE = "django_s3_storage.storage.StaticS3Storage"

AWS_S3_BUCKET_NAME_STATIC = S3_BUCKET

STATIC_URL = "https://%s.s3.amazonaws.com/" % S3_BUCKET
```



### 정적파일들을 클라우드로 밀어올린다

아래 명령으로 밀어올릴수있다.

```python
$ python manage.py collectstatic --noinput
```

그리고 나서 아래처럼 업데이트한다.

```
$ zappa update dev
```

zappa를 업데이트하고 나서 페이지를 다시 로드하여 체크한다