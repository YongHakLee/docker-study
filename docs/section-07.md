# Continuous Integration and Deployment with AWS

## Github Setup

[GitHub Repository](https://github.com/YongHakLee/docker-react)

### How Github Actions Works

GitHub Actions revolves around four main concepts: triggers (when to run), jobs (what to do), steps (how to do it), and actions (reusable units of code).

### Folder Structure

```tree
project-root/
├── .github/
│   └── workflows/
│       └── main.yml
└── (rest of your project files)
```

The `.github/workflows` directory in your repository root is where you place your workflow files.

## AWS Configuration Cheat Sheet

### Docker Compose config Update

1. Rename the development docker-compose.yml file to **docker-compose-dev.yml**.

```yml
# docker-compose-dev.yml
version: "3"
services:
  web:
    build:
      context: .
      dockerfile: Dockerfile.dev
    ports:
      - "3000:3000"
    volumes:
      - /app/node_modules
      - .:/app
  tests:
    stdin_open: true
    build:
      context: .
      dockerfile: Dockerfile.dev
    volumes:
      - /app/node_modules
      - .:/app
    command: ["npm", "run", "test"]
```

### Create EC2 IAM Instance Profile

1. Go to AWS Management Console

2. Search for **IAM** and click the IAM Service.

3. Click **Roles** under **Access** **Management** in the left sidebar.

4. Click the **Create** **role** button.

5. Select **AWS Service** under **Trusted entity type**. Then select **EC2** under **common use cases**.

6. Search for **AWSElasticBeanstalk** and select the **AWSElasticBeanstalkWebTier**, **AWSElasticBeanstalkWorkerTier** and **AWSElasticBeanstalkMulticontainerDocker** policies. Click the **Next** button.

7. Give the role the name of **aws-elasticbeanstalk-ec2-role**

8. Click the **Create role** button.

### Create Elastic Beanstalk Environment

1. Go to AWS Management Console

2. Search for **Elastic** **Beanstalk** and click the Elastic Beanstalk service.

3. If you've never used Elastic Beanstalk before you will see a splash page. Click the **Create** **Application** button. If you have created Elastic Beanstalk environments and applications before, you will be taken directly to the Elastic Beanstalk dashboard. In this case, click the **Create** **environment** button. There is now a flow of 6 steps that you will be taken through.

4. You will need to provide an Application name, which will auto-populate an Environment Name.

5. Scroll down to find the Platform section. You will need to select the Platform of **Docker**. This will auto-select several default options. Change the Platform branch to **Docker running on 64bit Amazon Linux 2**. The new 2023 branch currently has issues with single-container deployments.

6. Scroll down to the Presets section and make sure that **free tier eligible** has been selected:

7. Click the **Next** button to move to Step #2.

8. You will be taken to a Service Access configuration form.

Select **Create and use new service role** and name it **aws-elasticbeanstalk-service-role**. You will then need to set the **EC2 instance profile** to the **aws-elasticbeanstalk-ec2-role** created earlier (this will likely be auto-populated for you).

10. Click the **Skip to Review** button as Steps 3-6 are not applicable.

11. Click the **Submit** button and wait for your new Elastic Beanstalk application and environment to be created and launch.

12. Click the link below the checkmark under Domain. This should open the application in your browser and display a Congratulations message.

### Update Object Ownership of S# Bucket

1. Go to AWS Management Console

2. Search for **S3** and click the S3 service.

3. Find and click the elasticbeanstalk bucket that was automatically created with your environment.

4. Click **Permissions** menu tab

5. Find **Object Ownership** and click Edit

6. Change from **ACLs disabled** to **ACLs enabled**. Change **Bucket owner Preferred** to **Object Writer**. Check the box acknowledging the warning.

7. Click **Save changes**.

### Add AWS configuration details to .github/workflows/deploy.yaml file's deploy script

```yaml
# .github/workflows/deploy.yaml

# GitHub Actions 워크플로우의 이름 정의
name: Deploy Frontend

# 워크플로우 실행을 유발하는 이벤트 설정
on:
  # 코드가 푸시되었을 때 실행
  push:
    # 감지할 대상 브랜치 목록
    branches:
      # main 브랜치에 푸시가 발생할 때만 트리거
      - main

# 실행할 작업(Job) 목록 정의
jobs:
  # 'build'라는 식별자를 가진 작업 정의
  build:
    # 최신 Ubuntu 가상 머신 환경에서 실행
    runs-on: ubuntu-latest

    # 순차적으로 실행될 단계(Step) 정의
    steps:
      # 저장소 소스 코드를 러너 환경으로 체크아웃
      - uses: actions/checkout@v4

      # GitHub Secrets를 사용해 Docker Hub 로그인
      - run: >-
          docker login
          -u ${{ secrets.DOCKER_USERNAME }}
          -p ${{ secrets.DOCKER_PASSWORD }}

      # 개발용 Dockerfile.dev 기반으로 이미지 빌드
      - run: >-
          docker build
          -t feint225/react-test
          -f Dockerfile.dev .

      # 컨테이너 내에서 React 단위 테스트 실행
      # CI=true 설정으로 테스트 후 프로세스 즉시 종료
      - run: >-
          docker run
          -e CI=true
          feint225/react-test npm test

      # EB 배포용 압축 파일 생성 단계 지정
      - name: Generate deployment package
        # .git 디렉터리를 제외하고 전체 파일을 deploy.zip으로 압축
        run: zip -r deploy.zip . -x '*.git*'

      # AWS Elastic Beanstalk 자동 배포 단계
      - name: Deploy to EB
        # EB 배포를 지원하는 오픈소스 커스텀 액션 사용
        uses: einaregilsson/beanstalk-deploy@v22
        # 액션 실행에 필요한 파라미터 전달
        with:
          # AWS IAM 사용자의 Access Key (Secrets 참조)
          aws_access_key: ${{ secrets.AWS_ACCESS_KEY }}
          # AWS IAM 사용자의 Secret Key (Secrets 참조)
          aws_secret_key: ${{ secrets.AWS_SECRET_KEY }}
          # AWS EB에 생성된 애플리케이션 이름
          application_name: frontend
          # AWS EB에 생성된 환경(Environment) 이름
          environment_name: Frontend-env
          # 배포 패키지(zip)를 저장할 AWS S3 버킷명
          existing_bucket_name: >-
            elasticbeanstalk-ap-southeast-2-136589555745
          # 대상 Elastic Beanstalk 환경의 AWS 리전
          region: ap-southeast-2
          # 현재 배포 커밋의 고유 SHA 해시값을 버전 라벨로 사용
          version_label: ${{ github.sha }}
          # 업로드할 실제 배포 압축 파일 경로 지정
          deployment_package: deploy.zip
```

### Create an IAM User

1. Search for the "IAM Security, Identity & Compliance Service"

2. Click "Create Individual IAM Users" and click "Manage Users"

3. Click "Add User"

4. Enter any name you’d like in the "User Name" field.

eg: docker-react-travis-ci

5. Click "Next"

6. Click "Attach Policies Directly"

7. Search for "beanstalk"

8. Tick the box next to "AdministratorAccess-AWSElasticBeanstalk"

9. Click "Next"

10. Click "Create user"

11. Select the IAM user that was just created from the list of users

12. Click "Security Credentials"

13. Scroll down to find "Access Keys"

14. Click "Create access key"

15. Select "Command Line Interface (CLI)"

16. Scroll down and tick the "I understand..." check box and click "Next"

Copy and/or download the Access Key ID and Secret Access Key to use in the Github Secrets and Variables Setup.

### Github Secrets and Variables Setup

1. Repository -> Settings -> Secrets and variables -> Actions

2. Add New Repository Secret

### Deploying App

1. Make a small change to your src/App.js file in the greeting text.

2. commit and push

3. Go to Github Repository and check the status of your build.

4. The status should eventually return with a green checkmark.

5. Go to your AWS Elastic Beanstalk application

6. It should say "Elastic Beanstalk is updating your environment"

7. It should eventually show a green checkmark under "Health". You will now be able to access your application at the external URL provided under the environment name.
