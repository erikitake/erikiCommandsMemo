# AWSCommands

## EC2操作

### EC2インスタンスの起動、停止

- 起動  
    `aws ec2 start-instances --instance-ids $instanceID`
- インスタンスの状態表示  
    `aws ec2 describe-instance-status --instance-ids $instanceID`
- パブリックDNS名を取得  
    `aws ec2 describe-instances --instance-ids $instanceID --query Reservations[].Instances[].PublicDnsName --output text`
- 停止  
    `aws ec2  stop-instances --instance-ids $instanceID`

## ECR操作
- デフォルトレジストリに対して Docker CLI を認証します。そうすれば、dockerコマンドは、Amazon ECR を使用してイメージをプッシュおよびプルできます。AWS CLI には、認証プロセスをシンプルにする get-login-password コマンドが用意されています。   
    `aws ecr get-login-password --region ap-northeast-1 | docker login --username AWS --password-stdin ${account}.dkr.ecr.ap-northeast-1.amazonaws.com`

- pwapp(rails)  
    `docker build -t password_app .`    #カレントに用意したDockerfileからpassword_appという名前でビルド  
    `docker tag password_app:latest ${account}.dkr.ecr.ap-northeast-1.amazonaws.com/password_app:latest`  #ECR に push できる別名を設定。第1引数に別名を設定したいイメージID、第2引数に別名を指定  
    `docker push ${account}.dkr.ecr.ap-northeast-1.amazonaws.com/password_app:latest`   #リポジトリに対して、イメージを push  

- nginx (pwapp)  
    `docker build -t nginx containers/nginx/ --build-arg webapp_sock='127.0.0.1:3000'`  
    `docker tag nginx:latest ${account}.dkr.ecr.ap-northeast-1.amazonaws.com/password_app:nginx`  
    `docker push ${account}.dkr.ecr.ap-northeast-1.amazonaws.com/password_app:nginx`

### ECSのサービス起動  
```
ecs-cli compose --file ecs/ecs-compose-service.yml --ecs-params ecs/ecs-params.yml --project-name pwapp service up \
--target-group-arn arn:aws:elasticloadbalancing:ap-northeast-1:${account}:targetgroup/EC2Co-Defau-1FP55T6MJEJV0/287a203a05b80f0b \
--container-name nginx --container-port 80
```

## EC2作成
### securityGroupの作成
```
  sgId=`aws ec2 describe-security-groups \
  --group-name ec2-80-443-3000-ssh \
  --query SecurityGroups[].GroupId \
  --output text`
```
### EC2の作成
```
aws ec2 run-instances \
  --image-id $(aws ssm get-parameters --names /aws/service/ami-amazon-linux-latest/amzn2-ami-hvm-x86_64-gp2 --query 'Parameters[0].[Value]' --output text)  \
  --count 1 \
  --instance-type t2.micro \
  --associate-public-ip-address \
  --key-name $keyName \
  --security-group-ids $sgId $sgIdDefault \
  --subnet-id $subnet \
  --region ap-northeast-1 \
  --tag-specifications 'ResourceType=instance,Tags=[{Key=Name,Value=ec2AmazonLinuxForDev}]' \
                       'ResourceType=volume,Tags=[{Key=Name,Value=ec2AmazonLinuxForDev}]' \
  --block-device-mappings '[{"DeviceName":"/dev/xvda","VirtualName":"vol-023286e64e3784bf1"}]' \
  --dry-run
```
- 初回のみEBSのオプションを指定  
```
--block-device-mappings '[{"DeviceName":"/dev/xvda","Ebs":{"DeleteOnTermination":true,"VolumeSize":30,"VolumeType":"gp3","Encrypted":true}}]' \
```

- 2回目以降は既存EBSをアタッチする
```
ec2InstanceId=`aws ec2 describe-instances \
  --filter "Name=tag:Name,Values=ec2AmazonLinuxForDev" \
  --query Reservations[].Instances[].InstanceId \
  --output text`
```
```
aws ec2 terminate-instances --instance-id $ec2InstanceId
```
```
aws ec2 describe-instances \
  --filter "Name=tag:Name,Values=ec2AmazonLinuxForDev" \
  --query Reservations[].Instances[].PublicDnsName \
  --output text`
```

- セキュリティグループの削除
```
  sgId=`aws ec2 describe-security-groups \
  --group-name ec2-80-443-3000-ssh \
  --query SecurityGroups[].GroupId \
  --output text`
  aws ec2 delete-security-group --group-id $sgId
```

- 作成したインスタンスの削除(Nameがec2AmazonLinuxForDev)
```
ec2InstanceId=`aws ec2 describe-instances \
  --filter "Name=tag:Name,Values=ec2AmazonLinuxForDev" \
  --query Reservations[].Instances[].InstanceId \
  --output text`
aws ec2 terminate-instances --instance-id $ec2InstanceId
```

- ALBの作成
```
aws elbv2 create-load-balancer --name pwapp --subnets ${subnets} --security-groups ${sgId}
```
- ターゲットグループの作成
```
aws elbv2 create-target-group --name pwapp --protocol HTTP --port 80 --vpc-id ${vpc}
```

- ターゲットグループにリクエストを転送するデフォルトルールを持つロードバランサーのリスナーを作成
```
aws elbv2 create-listener --load-balancer-arn arn:aws:elasticloadbalancing:ap-northeast-1:${account}:loadbalancer/app/pwapp2/f377fea70f180390 --protocol HTTP --port 80  --default-actions Type=forward,TargetGroupArn=arn:aws:elasticloadbalancing:ap-northeast-1:${account}:targetgroup/pwapp2t/3712a00bf0d2ac48
```
- ELBのルールを追加
```
aws elbv2 create-rule --listener-arn arn:aws:elasticloadbalancing:ap-northeast-1:${account}:listener/app/pwapp2/f377fea70f180390/28504d072023c48e --priority 10 --conditions Field=path-pattern,Values='/pwapp' --actions Type=forward,TargetGroupArn=arn:aws:elasticloadbalancing:ap-northeast-1:${account}:targetgroup/pwapp2t/3712a00bf0d2ac48
```

- ELBの削除
```
aws elbv2 delete-load-balancer --load-balancer-arn arn:aws:elasticloadbalancing:ap-northeast-1:${account}:loadbalancer/app/pwapp2/f377fea70f180390
```
- ターゲットグループの削除
```
aws elbv2 delete-target-group --target-group-arn arn:aws:elasticloadbalancing:ap-northeast-1:${account}:targetgroup/pwapp2t/3712a00bf0d2ac48
```

- Lambdaアップロード
```
aws lambda create-function --function-name checkNR-function --zip-file fileb://lambda_checkNR.zip --handler lambda_checkNR.lambda_handler --runtime python3.8 --role arn:aws:iam::${account}:role/lambda-ex
```
- Lamdba更新
```
export file=lambda_getAndInsertData
cd package/ ; zip -r ../$file .; cd ../
zip -g $file.zip $file.py

aws lambda create-function --function-name $file --zip-file fileb://$file.zip \
  --handler $file.lambda_handler --runtime python3.8 --role arn:aws:iam::${account}:role/lambda-ex

aws lambda update-function-code --function-name $file --zip-file fileb://$file.zip
```

