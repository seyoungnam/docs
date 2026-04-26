# CICD Lab Environment

![cicd-flow](../../assets/img/eks/cicd/cicd-flow.webp)

- 워크숍 제공 실습 환경 설명
    - GitOps
        - Terraform 소스 코드를 “Gitea”를 통해 관리 - [Gitea](https://about.gitea.com/)
        - Helm 차트로 컨테이너 이미지를 관리하며, 컨테이너 이미지는 ECR에 배포
        - Amazon EKS에서 https://github.com/flux-iac/tofu-controller 에 의해 Flux로 “Watch”
        - [Argo Workflow](https://argoproj.github.io/workflows/)로 워크플로우 확인 (실습 3)
    - SaaS에서 실행하는 마이크로서비스
        - 앱
            - Producer
            - Consumer
        - MicroService Git 저장소를 Gitea로 관리
        - [Gitea Actions](https://docs.gitea.com/usage/actions/overview)에 의해 GitOps를 통해 배포/관리가 이루어지는 ECR에 업로드된 컨테이너 이미지를 실행
- 실습 환경 직접 구성하기: https://github.com/ianychoi/eks-saas-gitops 에 나와 있는 [구현 가이드](https://aws-solutions-library-samples.github.io/compute/building-saas-applications-on-amazon-eks-using-gitops.html#deploy-the-guidance)를 통해 워크숍 환경 대신 구성 가능
    - 비즈니스 이메일로 신청
    - 레퍼런스 아키텍처 전체를 배포하지 않음

- (참고) 개인 AWS 계정으로 직접 배포 시 : VSCode 서버 인스턴스 배포 (24분 소요) - [Link](https://github.com/ianychoi/eks-saas-gitops/tree/README-korean/helpers)
    - AWS 계정의 AWS CloudFormation 콘솔로 이동합니다
    - "스택 생성"을 클릭하고 "새 리소스 사용(표준)"을 선택합니다
    - "템플릿 파일 업로드"를 선택하고 이 리포지토리의 `helpers/vs-code-ec2.yaml` 파일을 업로드합니다
    - "다음"을 클릭하고 스택 이름을 입력합니다 (예: "**eks-saas-gitops-vscode**")
    - 필요한 매개변수를 구성하고 "다음"을 클릭합니다
    - **참고**: 기본 허용 IP는 0.0.0.0/0(모든 IP 주소)으로 설정되어 있습니다. 프로덕션 배포의 경우 보안 강화를 위해 특정 IP 범위(예. 집 공인 IP)로 제한하는 것을 고려하세요.
    - 구성을 검토하고 "스택 생성"을 클릭합니다
    - CloudFormation 스택 배포가 완료될 때까지 기다립니다 (약 30분 소요)
    - Terraform 인프라는 VSCode 서버 인스턴스 설정의 일부로 자동 배포됩니다
    - VSCode 인스턴스에는 모든 필수 도구(AWS CLI, Terraform, Git, kubectl, Helm, Flux CLI)가 사전 설치되어 있습니다
    - `helpers/vs-code-ec2.yaml` 파일 내용
        
        ```bash
        AWSTemplateFormatVersion: 2010-09-09
        
        Description: This stack creates an EC2 instance with VS Code server environment for the Solution Guidace on Building SaaS applications on Amazon EKS using GitOps"
        
        Parameters:
          EnvironmentName:
            Description: An environment name that is prefixed to resource names
            Type: String
            Default: "eks-saas-gitops"
          InstanceType:
            Description: EC2 instance type
            Type: String
            Default: t3.large
            AllowedValues:
              - t3.medium
              - t3.large
              - t3.xlarge
            ConstraintDescription: Must be a valid EC2 instance type
          AllowedIP:
            Description: Allowed IP address for connecting to the VSCode server and Gitea (CIDR)
            AllowedPattern: ^([0-9]{1,3}\.){3}[0-9]{1,3}/([0-9]|[1-2][0-9]|3[0-2])$
            ConstraintDescription: Must be a valid IP CIDR range of the form x.x.x.x/x
            Type: String
            Default: 0.0.0.0/0
          LatestAmiId:
            Type: "AWS::SSM::Parameter::Value<AWS::EC2::Image::Id>"
            Default: "/aws/service/ami-amazon-linux-latest/al2023-ami-kernel-default-x86_64"
        
        Resources:
          ################## PERMISSIONS AND ROLES #################
          EC2Role:
            Type: AWS::IAM::Role
            Properties:
              RoleName: eks-saas-gitops-admin
              Tags:
                - Key: Environment
                  Value: !Sub ${EnvironmentName}
              AssumeRolePolicyDocument:
                Version: "2012-10-17"
                Statement:
                  - Effect: Allow
                    Principal:
                      Service:
                        - ec2.amazonaws.com
                        - ssm.amazonaws.com
                        - eks.amazonaws.com
                        - codebuild.amazonaws.com
                    Action:
                      - sts:AssumeRole
              ManagedPolicyArns:
                - arn:aws:iam::aws:policy/AdministratorAccess
              Path: "/"
        
          ################## ARTIFACTS BUCKET ###############
          OutputBucket:
            Type: AWS::S3::Bucket
            DeletionPolicy: Retain
            Properties:
              VersioningConfiguration:
                Status: Enabled
              AccessControl: Private
              BucketEncryption:
                ServerSideEncryptionConfiguration:
                  - ServerSideEncryptionByDefault:
                      SSEAlgorithm: AES256
              PublicAccessBlockConfiguration:
                BlockPublicAcls: true
                BlockPublicPolicy: true
                IgnorePublicAcls: true
                RestrictPublicBuckets: true
        
          ################## VPC etc ######################
          VPC:
            Type: AWS::EC2::VPC
            Properties:
              CidrBlock: "10.0.0.0/16"
              EnableDnsHostnames: true
              EnableDnsSupport: true
              Tags:
                - Key: Name
                  Value: eks-saas-gitops-vscode-vpc # This is the tag Terraform will look for
                - Key: Environment
                  Value: !Sub ${EnvironmentName}
        
          PublicSubnet:
            Type: AWS::EC2::Subnet
            Properties:
              VpcId: !Ref VPC
              CidrBlock: "10.0.1.0/24"
              AvailabilityZone: !Select [0, !GetAZs ""]
              MapPublicIpOnLaunch: true
              Tags:
                - Key: Name
                  Value: !Sub ${EnvironmentName}-vscode-subnet
                - Key: Environment
                  Value: !Sub ${EnvironmentName}
        
          # Internet Gateway
          InternetGateway:
            Type: AWS::EC2::InternetGateway
            Properties:
              Tags:
                - Key: Name
                  Value: !Sub ${EnvironmentName}-vscode-igw
        
          AttachGateway:
            Type: AWS::EC2::VPCGatewayAttachment
            Properties:
              VpcId: !Ref VPC
              InternetGatewayId: !Ref InternetGateway
        
          # Route Table
          PublicRouteTable:
            Type: AWS::EC2::RouteTable
            Properties:
              VpcId: !Ref VPC
              Tags:
                - Key: Name
                  Value: !Sub ${EnvironmentName}-vscode-rt
        
          PublicRoute:
            Type: AWS::EC2::Route
            DependsOn: AttachGateway
            Properties:
              RouteTableId: !Ref PublicRouteTable
              DestinationCidrBlock: 0.0.0.0/0
              GatewayId: !Ref InternetGateway
        
          PublicSubnetRouteTableAssociation:
            Type: AWS::EC2::SubnetRouteTableAssociation
            Properties:
              SubnetId: !Ref PublicSubnet
              RouteTableId: !Ref PublicRouteTable
        
          ################## SSM Bootstrap for VS Code ##################
          SSMDocument:
            Type: AWS::SSM::Document
            Properties:
              Tags:
                - Key: Environment
                  Value: !Sub ${EnvironmentName}
              DocumentType: Command
              Content:
                schemaVersion: "2.2"
                description: Bootstrap VS Code Instance
                parameters:
                  allowedIp:
                    type: String
                    description: Allowed IP address
                    default: ""
                mainSteps:
                  - action: aws:runShellScript
                    name: VSCodebootstrap
                    inputs:
                      runCommand:
                        - "#!/bin/bash"
                        - "mkdir -p /home/ec2-user/environment"
                        - "chown -R ec2-user:ec2-user /home/ec2-user/environment"
                        - "curl -fsSL https://code-server.dev/install.sh | sudo -u ec2-user sh"
                        - "export CODER_PASSWORD=$(openssl rand -base64 12)"
                        - "mkdir -p /home/ec2-user/.config/code-server/"
                        - "echo 'bind-addr: 0.0.0.0:8080' > /home/ec2-user/.config/code-server/config.yaml"
                        - "echo 'auth: password' >> /home/ec2-user/.config/code-server/config.yaml"
                        - "echo password: $CODER_PASSWORD >> /home/ec2-user/.config/code-server/config.yaml"
                        - "chown -R ec2-user:ec2-user /home/ec2-user/.config/"
                        - 'aws ssm put-parameter --name ''coder-password'' --type ''String'' --value "$CODER_PASSWORD" --overwrite'
                        - "yum update -y"
                        - "yum install -y docker && systemctl start docker && systemctl enable docker"
                        - "yum install -y vim git jq bash-completion moreutils gettext yum-utils perl-Digest-SHA"
                        - "yum install -y git-lfs"
                        - "yum install -y tree"
                        - curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
                        - "chmod +x kubectl && mv kubectl /usr/local/bin/"
                        - "/usr/local/bin/kubectl completion bash > /etc/bash_completion.d/kubectl"
                        - "curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash"
                        - "/usr/local/bin/helm completion bash > /etc/bash_completion.d/helm"
                        - curl --silent --location "https://github.com/fluxcd/flux2/releases/download/v2.7.5/flux_2.7.5_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp
                        - "mv /tmp/flux /usr/local/bin"
                        - "/usr/local/bin/flux completion bash > /etc/bash_completion.d/flux"
                        - "wget https://github.com/mikefarah/yq/releases/latest/download/yq_linux_amd64 -O /usr/bin/yq && sudo chmod +x /usr/bin/yq"
                        - "yum-config-manager --add-repo https://rpm.releases.hashicorp.com/AmazonLinux/hashicorp.repo"
                        - "yum -y install terraform"
                        - export TOKEN=$(curl -X PUT "http://169.254.169.254/latest/api/token" -H "X-aws-ec2-metadata-token-ttl-seconds:60")
                        - export AWS_REGION=$(curl -H "X-aws-ec2-metadata-token:${TOKEN}" -s http://169.254.169.254/latest/meta-data/placement/availability-zone | sed 's/\(.*\)[a-z]/\1/') && echo "export AWS_REGION=${AWS_REGION}" >> /home/ec2-user/.bashrc
                        - export ALLOWED_IP="{{allowedIp}}"
                        - "git clone https://github.com/aws-samples/eks-saas-gitops.git /home/ec2-user/environment/eks-saas-gitops; echo 'solution=true' > /home/ec2-user/environment/eks-saas-gitops/terraform/workshop/terraform.tfvars"
                        - "chown -R ec2-user:ec2-user /home/ec2-user/environment"
                        - "sudo -u ec2-user nohup /usr/bin/code-server --port 8080 --host 0.0.0.0 > /dev/null 2>&1 &"
                        - "export WAIT_HANDLE_URL=$(aws ssm get-parameter --name '/eks-saas-gitops/waitcondition-url' --query 'Parameter.Value' --output text --region $AWS_REGION)"
                        - "cd /home/ec2-user/environment/eks-saas-gitops/terraform"
                        - "chmod +x install.sh"
                        - 'sudo -u ec2-user ./install.sh ${AWS_REGION} "{{allowedIp}}" > /home/ec2-user/environment/terraform-install.log 2>&1'
                        - 'if [ $? -eq 0 ]; then'
                        - '  curl -X PUT -H ''Content-Type: application/json'' --data-binary ''{"Status" : "SUCCESS", "Reason" : "Environment Completed", "UniqueId" : "123456", "Data" : "Complete"}'' "$WAIT_HANDLE_URL"'
                        - 'else'
                        - '  curl -X PUT -H ''Content-Type: application/json'' --data-binary ''{"Status" : "FAILURE", "Reason" : "Terraform installation failed", "UniqueId" : "123456", "Data" : "Failed"}'' "$WAIT_HANDLE_URL"'
                        - 'fi'
        
          SSMBootstrapAssociation:
            Type: AWS::SSM::Association
            Properties:
              Name: !Ref SSMDocument
              Parameters:
                allowedIp: [!Ref AllowedIP]
              OutputLocation:
                S3Location:
                  OutputS3BucketName: !Ref OutputBucket
                  OutputS3KeyPrefix: bootstrapoutput
              Targets:
                - Key: tag:SSMBootstrapSaaSGitOps
                  Values:
                    - Active
        
          ################## WAIT CONDITION ##################
          WaitHandle:
            Type: AWS::CloudFormation::WaitConditionHandle
        
          WaitCondition:
            Type: AWS::CloudFormation::WaitCondition
            DependsOn: SSMBootstrapAssociation
            Properties:
              Handle: !Ref WaitHandle
              Timeout: "2000"
        
          WaitConditionUrlParameter:
            Type: "AWS::SSM::Parameter"
            Properties:
              Name: !Sub /${EnvironmentName}/waitcondition-url
              Type: "String"
              Value: !Ref WaitHandle
        
          ################## Instance Profile ##################
          EC2InstanceProfile:
            Type: AWS::IAM::InstanceProfile
            Properties:
              Path: "/"
              Roles:
                - Ref: EC2Role
        
          EC2Instance:
            Type: AWS::EC2::Instance
            Properties:
              ImageId: !Ref LatestAmiId
              InstanceType: !Ref InstanceType
              SubnetId: !Ref PublicSubnet
              IamInstanceProfile: !Ref EC2InstanceProfile
              SecurityGroupIds:
                - !Ref EC2SecurityGroup
              Tags:
                - Key: Name
                  Value: !Sub ${EnvironmentName}-Instance
                - Key: SSMBootstrapSaaSGitOps
                  Value: Active
                - Key: Environment
                  Value: !Sub ${EnvironmentName}
        
          EC2SecurityGroup:
            Type: AWS::EC2::SecurityGroup
            Properties:
              VpcId: !Ref VPC
              GroupName: EC2SecurityGroup
              GroupDescription: Allow SSH and Code-Server access
              SecurityGroupIngress:
                - IpProtocol: tcp
                  FromPort: 8080
                  ToPort: 8080
                  CidrIp: !Ref AllowedIP
                  Description: Allow HTTP traffic from provided prefix
              SecurityGroupEgress:
                - IpProtocol: "-1"
                  CidrIp: 0.0.0.0/0
              Tags:
                - Key: Name
                  Value: eks-saas-gitops-vscode-sg
                - Key: Environment
                  Value: !Sub ${EnvironmentName}
        
        Outputs:
          VsCodeIdeUrl:
            Description: The URL to access VS Code IDE
            Value: !Sub "http://${EC2Instance.PublicDnsName}:8080/?folder=/home/ec2-user/environment"
          VsCodePassword:
            Description: The VS Code IDE password
            Value: !Sub "https://${AWS::Region}.console.aws.amazon.com/systems-manager/parameters/coder-password"
        ```
        
        - 배포 결과 기본 확인 명령어
            
            ```bash
            # SSM 관리 대상 인스턴스 목록 조회
            **aws ssm describe-instance-information \
              --query "InstanceInformationList[*].{InstanceId:InstanceId, Status:PingStatus, OS:PlatformName}" \
              --output text** # table
            
            # session-manager-plugin 을 통한 인스턴스 접속 후 기본 정보 확인
            export MYINSTANCE=*i-02df087b2f05eed0d*
            
            **aws ssm start-session --target $**MYINSTANCE
            ------------------------------------------
            # 관리자 전환
            sudo su -
            
            # 자격 증명 확인
            aws configure list
            **aws sts get-caller-identity**
            
            # 작업 디렉터리 이동
            **cd /home/ec2-user/**
            tree -a -L 3
            tree -a
            
            # code-server 설정 파일 확인 : 아래 암호는 System Manager 파라미터 스토어에 code-password 값
            **cat .config/code-server/config.yaml** 
            *bind-addr: 0.0.0.0:8080
            auth: password
            password: **ZrO3yG6dNdsIDAOs***
            
            # code-server 실행 정보 확인
            ps -ef |grep code-server
            ss -tnlp | grep 8080
            
            # terraform 버전 확인
            **terraform version**
            *Terraform v1.14.8*
            
            # terraform 을 통한 기본 실습 환경 배포 스크립트 파일 확인
            **cat environment/eks-saas-gitops/terraform/install.sh**
            *...
            # terraform 실행
            echo "Initializing Terraform..."
            **terraform init**
            
            echo "Planning Terraform deployment (infrastructure only) in region: ${AWS_REGION}..."
            export TF_VAR_aws_region="${AWS_REGION}"
            export TF_VAR_allowed_ip="${ALLOWED_IP}"
            **terraform plan**  -target=module.vpc \
                            -target=module.ebs_csi_irsa_role \
                            ...
            
            echo "Applying Terraform configuration (infrastructure only)..."
            **terraform apply** -target=module.vpc \
                            -target=module.ebs_csi_irsa_role \
                            ...*
            
            # terraform 을 통한 기본 실습 환경 배포 스크립트 실행 확인
            **ps -ef | grep install**
            *root       27978    1817  0 04:46 ?        00:00:00 sudo -u ec2-user ./install.sh ap-northeast-2 ...*
            
            # 관련 로그 확인
            more /home/ec2-user/environment/terraform-install.log
            **tail -f /home/ec2-user/environment/terraform-install.log**
            
            # (옵션) 설치 완료 후 테라폼 정보 확인
            cd /home/ec2-user/eks-saas-gitops/terraform/workshop
            tree .terraform
            terraform output
            
            cd /home/ec2-user/
            
            # 빠져나오기
            exit
            ------------------------------------------
            
            ```
            
    - VS Code for the Web 접속
        
        ![vscode-for-web](../../assets/img/eks/cicd/vscode-for-web.webp)
        
        ![VS Code 에서 Terminal 창 실행 후 whoami, kubectl cluster-info 확인](attachment:b1afa128-e7e5-47fd-8eff-e0d5a7882625:image.png)
        
        VS Code 에서 Terminal 창 실행 후 whoami, kubectl cluster-info 확인
        
    - (실습 완료 후 ) 개인 AWS 계정으로 직접 배포 후 삭제 시 : [https://github.com/ianychoi/eks-saas-gitops?tab=readme-ov-file#삭제-스크립트-실행](https://github.com/ianychoi/eks-saas-gitops?tab=readme-ov-file#%EC%82%AD%EC%A0%9C-%EC%8A%A4%ED%81%AC%EB%A6%BD%ED%8A%B8-%EC%8B%A4%ED%96%89)
        - 삭제 스크립트 실행 : **`cd /home/ec2-user/eks-saas-gitops/terraform** && **sh destroy.sh ap-northeast-2**`
        - (eks 삭제 후 vpc 삭제 되는 시점에서) **aws cloudformation** 삭제 `eks-saas-gitops-vscode`
        - 잔여 리소스 있을 경우 직접 수동 제거 : ELB 대상 그룹, SQS, DynamoDB Table, 파라미터 스토어, VPC, S3, IAM Role
    
    - 실습 환경 관련 참고 링크
        - https://catalog.workshops.aws/eks-saas-gitops/en-US
        - https://github.com/aws-solutions-library-samples/eks-saas-gitops
        - https://github.com/ianychoi/eks-saas-gitops
        - https://docs.aws.amazon.com/prescriptive-guidance/latest/eks-gitops-tools/introduction.html