---
title: "[AWS] Marketplace"
last_modified_at: 2022-09-20T12:35:00+09:00
categories:
    - AWS
tags:
    - AWS
    - AWS Marketplace
toc: true
toc_sticky: true
toc_label: "목차"
---

AWS: AWS Marketplace는 무엇이고 어떻게 등록하는지 기록 해보자. 지금은 정신이 없는 관계로 대충 막 쓰고 나중에 정신이 돌아온다면 정돈하여 쓰도록 하겠다.
{: .notice--info}

지정된 Region 에서만 tax 등록이 가능
- 일본

NHN 테코러스 통해 상품 등록.

우리는 상품 관리가 가능 하도록 우리 별도의 계정에 ROLE 만 전달 받는다.

NHN 테코러스
: NHN 엔터프라이즈의 일본 법인


AWS 마켓 플레이스 관련 API 사용을 위해 `access key` 발급 필요.(`secret-access-key`는 `access key`와 다르다. 별도로 기록하자.)

구독 후 우리 서버로 토큰(`x-amzn-marketplace-token`) 전달 됨.

```
x-amzn-marketplace-token=MEsdCCtvXJKpRzzEqpYnD0D5Ug5UHnHxKULmo90TJE82xbLS9DXUpsomvGe97y3vUrUotBdIMGeC53CP2o32OX00G9cZSUynRA54od6hBAIs4A%2BgOxAPCU1jRZl9U8lecqpPvytXOlmvaq5gbj88OgCkU7Srykx9NpgMxTQWEEW3wJLkxzgyLw%3D%3D
```

- 유저 정보 확인
Java SDK 사용 `ResolveCustomer` 호출


(Java SDK 를 통해) 구독시 전달 된 토큰으로 부터 유저 정보를 획득하기 위해, ResolveCustomer 를 호출하는데, assume-role을 통해 iam role에 부여된 권한을 사용하여 접속 할 client 객체 생성, 이후에 ResolveCustomer 를 호출해야 함

assume-role 획득 예시

- https://docs.aws.amazon.com/ko_kr/AmazonS3/latest/userguide/AuthUsingTempSessionToken.html

ResolveCustomer API

- https://docs.aws.amazon.com/ko_kr/marketplace/latest/userguide/saas-code-examples.html

호출 예시

```
access-key-id: AKIAVM7AC7IQH7K2JTUA
secret-access-key: ...
role-arn: arn:aws:iam::434519024304:role/NHNMarketPlaceAccessRole
```

AWS SQS
: Amazon Simple Queue Service, AWS 의 메시지 대기열 서비스

마켓플레이에서 구독/구독취소 등에 대한 정보를 전달 받기 위해 사용

---

* **<span style="color:#0052cc">가이드 문서</span>**
    * sass 연동 가이드 :[https://github.com/jinseo-jang/aws-marketplace-saas](https://github.com/jinseo-jang/aws-marketplace-saas)
        * [SaaS Subscription PDF](https://awsmp-loadforms.s3.amazonaws.com/AWS+Marketplace+-+SaaS+Integration+Guide.pdf)(11page)
        * ![image](https://user-images.githubusercontent.com/53864640/192408059-0fda69c2-19a4-4055-83cf-98d2a4e16d2d.png)
        * ![image](https://user-images.githubusercontent.com/53864640/192408081-464869ab-0320-4994-86d0-fe126a3939ef.png)
            * Token 은 임시발급이며 이를 ResolveCustomer API 호출에 사용하여 customerID 를 반환 받는다.
            * customerID 가 영구적인 고유식별값으로 우리 서비스와 연동 할 때 사용한다.
        * ![image](https://user-images.githubusercontent.com/53864640/192408116-8cc06ef0-17ab-40e6-a951-16d59c667c22.png)
            * SQS 를 통해 구독 성공/실패/취소 등에 대한 event 를 감지하여 우리쪽에서 권한을 처리하는 것으로 보임.
            * `subscribe-success` 알림 받기 전에는 사용자에게 권한을 부여하지 않도록 주의한다. (`subscribe-success`알림 받기 전에는 어차피 미터링 되지 않음)
# 용어

* AWS STS : AWS Security Token Service ([Git example](https://github.com/awsdocs/aws-doc-sdk-examples/tree/main/gov2/sts))
* AMP : AWS Marketplace
* EULA : [End User License Agreement(최종 사용자 라이선스 계약)](https://d1.awsstatic.com/legal/AWS_ELEMENTAL_EULA/AWS%20ELEMENTAL%20END%20USER%20LICENSE%20AGREEMENT%20KOREAN%202020-10-23.pdf)
* ARN : Amazon Resource Number
* AMMP : AWS Marketplace Management Portal
* AWS SQS : [AWS Simple Queue Service](https://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSDeveloperGuide/welcome.html)
