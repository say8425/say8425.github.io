---
layout: post
title: AWS EC2 Tags를 ENV로 사용하기
categories: AWS
---

EC2 서버를 굴리다보면 필연적으로 Environment Value(a.k.a ENV)를 설정할 일이 생긴다. 전통적으로 터미널에서 굴리는 방법도 있지만, AWS에서 굴리는 방법은 없나? 필자가 찾아본 방법으로는 3가지가 있었다.

## AWS ElasticBeanstalk

우선 첫번째로 AWS ElasticBeanstalk을 사용하는 방법이 있다. application을 생성할 때부터 설정할 수 있으며, 생성 이후에도 언제든지  [Environment properties 메뉴](https://docs.aws.amazon.com/ko_kr/elasticbeanstalk/latest/dg/environments-cfg-softwaresettings.html#environments-cfg-softwaresettings-console)에서 설정 할 수 있다. **개꿀** 주의할 점은 EB 특성상 매 설정마다, 서버 reboot을 요구한다. 이 떄문에 ~~시말서와~~ downtime이 없게 Rolling updates를 설정해주자. 하지만 EC2 서버에 ENV를 어떻게 쑤셔넣을까 고민할 시점이 왔다면, ElasticBeanstalk은 이미 고려대상에서 없었을 것이다.

## S3

두번재 방법으로 S3를 활용하는 방법이 있다. S3의 적당한 버킷에, 적당한 디렉토리에, 적당한 dotenv를 작성해넣고, EC2에서 가져와 적당히 export 해주면 된다. 당연하지만 귀찮다고 public으로 만들어 털리지 말고, IAM User를 만들어 접근하자. 이 방법의 가장 큰 장점은 이미 이런 작업을 수행해줄 라이브러리가 존재한다는 것이다. [Go](https://github.com/sachaos/s3env)나 [Ruby](https://github.com/the40san/dotenv-s3)등으로 작성된 것이 있으며, 그 외 언어로 작성된 것도 찾아보면 있다.

## EC2 Tags

마지막 방법으로 필자가 시도해보고 이번 포스트에서 소개하고자 하는 방법인, EC2 Tag를 사용하는 방법이 있다.
S3를 사용하면 관리 포인트가 증가하고 수정하기 번거로운 반면, EC2 Tag를 수정하면 되므로 편리하다.

### IAM User 생성하기

우선 Tag 접근용 IAM User 를 만들어야 한다. 다만 Tag만 접근 가능한 policy가 AWS에는 별도로 없다. 그러므로 따로 policy를 만들어야 한다.

IAM에서 policy 생성하기 들어가서 아래와 같은 json을 입력해주자.

```json
# Policy name: AmazonEC2TagReadOnlyAccess
# Policy desc: Provides read-only access to Amazon EC2 Tags.

{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Action": "ec2:DescribeTags",
            "Resource": "*",
            "Effect": "Allow"
        }
    ]
}
```

그리고 해당 policy를 갖고 있는 IAM User를 생성하고, key를 다운받자.

### 필요한 패키지 설치 - awscli jq ec2-metadata

IAM User를 생성했으면 EC2에 접속하자. 접속해서 [aws-cli](https://github.com/aws/aws-cli), [jq](https://github.com/stedolan/jq) 그리고 [ec2-metadata](https://aws.amazon.com/ko/code/ec2-instance-metadata-query-tool/)패키지를 설치하자. aws-cli는 방금 생성한 IAM User를 사용하고, tag를 가져오기 위해 필요하다. jq는 가져온 json type의 tag를 사용하기 위해 필요하다. 이 때문에, 굳이 [jq를 사용하지 않고, python을 사용한 용자](https://gist.github.com/marcellodesales/a890b8ca240403187269#gistcomment-2358609)들도 보인다. ec2-metadata는 ec2의 관련 정보를 담고 있는 metadata를 가져오기 위해 필요하다. 이 metadata 중 현재 ec2의 instance-id도 있다. 이것을 활용해, tag를 가져오면 된다.

우선 패키지를 설치하자. debian 계열 OS 기준 아래 스크립트을 입력해 설치할 수 있다. ~그 외는 알아서 잘 설치하자~

```shell
sudo apt-get --assume-yes awscli jq
wget http://s3.amazonaws.com/ec2metadata/ec2-metadata
chmod u+x ec2-metadata
```

ec2-metadata에 권한 주는 것을 잊지말자.

### AWS IAM 설정

설치가 다 끝나면, 아까 만든 IAM User를 사용하는 aws configure를 만들자. 그냥 `aws configure` 명령어를 쳐서 IAM User access key를 기본으로 사용하게 만드는 방법도 있지만, 필자는 관리 차원에서 `ec2-tags` 라는 별도 프로필을 만들어서 사용했다.

```shell
aws configure --profile ec2-tags
# access key 입력
# secreet key 입력
```

### Tags 가져와서 ENV에 export 하기

#### Tags 가져오기

이제 본격적인 스크립트를 작성하자. 우선 tag를 가져오자. tag를 가져오려먼 awscli의 [describe-tags](https://docs.aws.amazon.com/cli/latest/reference/ec2/describe-tags.html) 명령어를 사용하면 된다. 단 그냥 모든 인스턴스로부터 가져오므로 filter를 걸어줘야 한다. 직접 region과 instance-id를 입력해서 필터를 걸어줄 수도 있지만, ec2-metadata 를 사용해서, 현재 접속한 인스턴스의 id와 region을 받아와서 쓸 수 있다. 그렇게 작성된 코드가 아래와 같다.

```shell
get_instance_tags () {
    EC2_AVAIL_ZONE=$(ec2metadata --availability-zone)
    EC2_REGION="`echo \"$EC2_AVAIL_ZONE\" | sed -e 's:\([0-9][0-9]*\)[a-z]*\$:\\1:'`"
    INSTANCE_ID=$(ec2metadata --instance-id)

    aws ec2 describe-tags --region $EC2_REGION --filters "Name=resource-id,Values=$INSTANCE_ID" --profile ec2-tags
}
```

사실 ec2-metadata로는 region을 알 수 없다. 하지만 가용영역은 알 수 있으므로, 가져와서 뒤에 붙은 zone 명을 regex로 제거해서 써먹는거다. 가져온 태그는 아래과 같은 꼴이다.

```json
{
    "Tags": [
        {
            "Key": "RAILS_ENV",
            "ResourceId": "i-038669d150cbdd5b7",
            "ResourceType": "instance",
            "Value": "production"
        },
        {
            "Key": "PG_HOST",
            "ResourceId": "i-038669d150cbdd5b7",
            "ResourceType": "instance",
            "Value": "localhost"
        },
        {
            "Key": "PG_USER",
            "ResourceId": "i-038669d150cbdd5b7",
            "ResourceType": "instance",
            "Value": "foofoo"
        }
    ]
}
```

#### 가져온 Tags를 export 하기

가져온 tag 정보는 json 타입이다. 이걸 bash가 알아먹을리 없으므로 적절히 가공해서 export 해주자. 본 글은 [jq](https://stedolan.github.io/jq/)을 사용하는 스크립트를 소개하지만, 굳이 이걸 쓸 필요없고 자신 있는 언어를 사용해도 좋다.

```shell
tags_to_env () {
    tags=$1

    for key in $(echo $tags | jq -r ".[][].Key"); do
        value=$(echo $tags | jq -r ".[][] | select(.Key==\"$key\") | .Value")
        key=$(echo $key | tr '-' '_' | tr '[:lower:]' '[:upper:]')
        export $key="$value"
    done
}

instance_tags=$(get_instance_tags)
tags_to_env "$instance_tags"
```

## 결론

결론부터 말하자면 aws-cli만 설치되어있으면 tag를 가져올 수 있다. 다만 여기에 access key와 자신의 instance-id 정도만 추가로 필요하다. 사실 `ec2metadata`도 설치할 필요가 없고, `169.254.169.254`와 통신해서 metadata를 알 수 있다. 예를 들어 가용영역을 알고싶다면 아래와 같이 쓸 수 있다.

```shell
# ec2-metadata를 사용한 예
ec2metadata --availability-zone

# 169.254.169.254를 사용한 예
curl --silent http://169.254.169.254/latest/meta-data/placement/availability-zone
```

그리고 [ec2 tag 자체에도 제한](https://docs.aws.amazon.com/ko_kr/AWSEC2/latest/UserGuide/Using_Tags.html#tag-restrictions)이 있다. 특히 ec2 tag는 등록할 수 있는 개수가 ~설마 누가 그렇게씩이나 많이 등록할리 없는~ 50개로 한정되어있다. 또 한가지 더 주의할 점은 `aws`키는 예약어이다.

---

* Related Links
  * [gist에 올라온 스크립트](https://gist.github.com/marcellodesales/a890b8ca240403187269#gistcomment-2358609)
  * [AWS EC2 Meta Data 얻기](https://docs.aws.amazon.com/ko_kr/AWSEC2/latest/UserGuide/ec2-instance-metadata.html#instancedata-data-retrieval)
  * [AWS EC2 Meta Dada Query Tool](https://aws.amazon.com/ko/code/ec2-instance-metadata-query-tool/)
  * [AWS CLI dscribe tag 문서](https://docs.aws.amazon.com/cli/latest/reference/ec2/describe-tags.html)
