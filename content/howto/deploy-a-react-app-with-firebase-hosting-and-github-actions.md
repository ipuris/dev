---
title: "Deploy A React App With Firebase Hosting and Github Actions"
date: 2020-01-19T23:44:05+09:00
tags:
- Firebase
- Firebase Hosting
- GitHub
- GitHub Actions
- React
- Continuous Integration
- Continuous Delivery
---

이 포스트에서는 React를 이용해 만든 웹앱을 Firebase Hosting를 이용해 배포하고, 이 과정을 GitHub Actions를 이용해 자동화하는 방법을 설명합니다.

전체 과정을 간단히 요약하면 아래와 같습니다.

1. Firebase Hosting 프로젝트 설정
2. React 프로젝트 생성
3. Firebase Hosting 프로젝트에 React 프로젝트 통합
4. GitHub Actions 설정
5. 테스트

저는 React 앱 설정에 비해 Firebase Hosting 설정이 익숙하지 않아서 Firebase Hosting 프로젝트에 [create-react-app](https://create-react-app.dev)으로 생성한 React 앱을 포함시키는 방법을 사용했습니다.

하지만 React는 개발 프레임워크가 아닌 자바스크립트 라이브러리이기 때문에 훨씬 유연한 방식으로 적용할 수 있습니다. 이미 React에 익숙하신 분들은 [React 프로젝트 생성](#create-a-react-project) 및 [Firebase Hosting 프로젝트에 React 프로젝트 통합](#integrate-the-two-projects) 과정을 좀 더 쉽고 효과적인 방법으로 대체하실 수 있으실 것 같습니다.

이 포스트에는 Firebase 서비스 가입과 프로젝트 생성, Firebase Hosting 커스텀 도메인 설정, git 및 GitHub 사용 방법 등 기본적인 내용에 대한 설명은 포함하지 않았습니다.



## Configure Hosting for Firebase Project

Firebase Console에 들어가서 Firebase Hosting 서비스를 설정합니다.
아직 Hosting 서비스를 설정하지 않았다면 아래와 같은 화면이 나옵니다.

![Firebase Hosting Console](/images/howto/firebase-hosting-console.png)

시작하기 버튼을 누르면 아래와 같은 화면이 나옵니다.

![Configure Firebase Hosting](/images/howto/firebase-hosting-configure.png)

안내하는 과정을 그대로 따라가면 됩니다.
간단히 요약하면 아래와 같습니다.

```sh
npm install -g firebase-tools # Firebase CLI 설치
firebase login # 구글 로그인
firebase init # 프로젝트 설정
firebase deploy # 웹 앱 배포
```

그러면 최종적으로 Firebase Hosting 프로젝트가 생성됩니다.
여기서는 `firebase-hosting` 폴더에 프로젝트가 생성되었다고 가정하겠습니다.
아래와 같은 파일들이 자동으로 생성됩니다.

```sh
firebase-hosting/
- .gitignore
- .firebaserc
- firebase.json
- public/
  - index.html
  - 404.html
```

왠지 `.firebaserc` 파일과 `firebase.json` 파일이 중요해보입니다.
`public` 폴더 안에는 `index.html` 파일도 있네요.
Apache 등의 웹 서버를 다뤄보신 경험이 있으시다면 이 폴더 구조만 보고도 어떤 식으로 작동하는지 감이 오실 것 같습니다.

이제 Firebase Hosting 프로젝트가 하나 만들어졌네요.
저는 이 단계에서 이 저장소를 GitHub에 생성, 푸시해두었습니다.

```sh
git remote add origin https://github.com/사용자아이디/저장소이름.git
git push -u origin master
```



## Create A React Project

이제 React 프로젝트를 생성합니다.
저는 `create-react-app` 명령어를 이용해 TypeScript 템플릿을 포함한 프로젝트를 생성하겠습니다.

```sh
npx create-react-app react-app --template typescript
```

이제 `react-app` 폴더에 React 프로젝트가 생성되었습니다.

이 포스팅의 시작부분에서 말씀드린 것처럼 React 사용에 익숙하신 분들은 굳이 이 방식을 따르지 않으셔도 상관없습니다.
이미 React 사용에 익숙하시다면 위에서 생성한 `firebase-hosting` 프로젝트에 바로 React 프로젝트를 설정하는 것이 가장 쉽고 빠를 것 같습니다.



## Integrate The Two Projects

이제 위에서 생성한 두 프로젝트를 합쳐야 합니다.

### Put Files Into One Folder

먼저 파일들을 한 폴더로 합쳐봅시다.
저는 Firebase Hosting 프로젝트 설정에 익숙치 않기 때문에, Firebase Hosting 프로젝트 폴더 `firebase-hosting`는 그대로 둔 채로 React 프로젝트 폴더 `react-app`의 파일들을 `firebase-hosting` 폴더로 옮길 것입니다.

저는 간편하게 아래의 명령어로 프로젝트 폴더 전체를 옮기고 싶습니다.

```sh
mv react-app/* firebase-hosing/
```

바로 위의 명령을 실행해도 되지만, 저는 좀 더 깔끔하게 프로젝트를 옮기기 위해 아래와 같은 작업을 먼저 진행했습니다.

- `react-app/.git` 폴더 삭제
- `react-app/node_modules` 폴더 삭제

### Change Default Public Folder

웹 방문자가 Firebase Hosting을 이용해 서비스를 제공 중인 웹사이트에 접속하면, Firebase Hosting은 `public` 폴더의 내용을 보여주게 됩니다.
위에서 Firebase Hosting을 처음 설정하고 나면 자동으로 `public` 폴더가 생성되고, 그 안에 `index.html` 파일과 `404.html` 파일이 생성되었던 것이 기억나실 겁니다.
이 파일들을 이용해 사용자에게 웹페이지를 표시해주는 것 같습니다.

하지만 `create-react-app`으로 생성한 React 프로젝트를 빌드하면 `build` 폴더에 최종적으로 생성된 파일들이 위치하게 됩니다.
따라서 우리는 Firebase Hosting이 `public` 폴더가 아닌 `build` 폴더의 내용을 웹 방문자에게 표시하도록 설정을 수정해줘야 합니다.
이는 `firebase.json` 파일에서 `"public"` 항목을 통해 바꿀 수 있습니다.
`firebase.json` 파일을 아래와 같이 수정해줍시다.

```json
{
  "hosting": {
    "public": "build",
    "ignore": [
      "firebase.json",
      "**/.*",
      "**/node_modules/**"
    ]
  }
}
```

`build` 폴더의 내용은 git을 이용해 버전관리가 될 필요가 없기 때문에 `.gitignore`에도 추가해주는게 좋을 것 같습니다.

```sh
# 아래의 라인을 기존 .gitignore 파일에 추가

# production
/build
```

이제 Firebase Hosting 프로젝트와 React 프로젝트가 성공적으로 합쳐졌습니다.
React가 정상적으로 작동하는지 확인해봅시다.

```sh
npm install
npm start
```

빌드도 문제없이 진행되어 `build` 폴더에 파일들이 생성되는지 확인해봅시다.

```sh
npm run build
```



## GitHub Actions 설정

위의 단계까지 진행할 경우 아래 명령어로 바로 Firebase Hosting 서비스에 배포를 진행할 수도 있습니다.

```sh
firebase deploy
```

하지만 프로덕션 레벨의 서비스일 경우 아무런 테스트나 확인이 없이 진행되는 이런 방식의 배포는 상당히 불안합니다.
프로덕션 레벨의 서비스가 아닌 개인적인 토이 프로젝트의 경우에도 git을 이용한 버전 관리 작업과 배포 과정이 분리되어 있어 매번 배포를 신경쓰기 귀찮기도 하고, 실수할 여지가 있어보입니다.

GitHub Actions를 이용해 `master` 브랜치에 푸시가 되면 자동으로 React 빌드를 진행하고, 빌드가 성공할 경우 자동으로 Firebase Hosting에 배포를 진행하도록 만들어보겠습니다.

아래와 같은 두 과정을 거쳐야 합니다.

- `main.yml` 작성
- `FIREBASE_TOKEN` 설정

### Add main.yml

이제 GitHub Actions에서 자동화할 작업의 명세인 워크플로우 파일을 작성해봅시다.
이 워크플로우 파일은 YAML 파일 형태로 작성하게 됩니다.

웹브라우저를 이용해 GitHub 서비스에서 현재 작업 중인 프로젝트의 저장소 페이지에 들어가면 Actions 메뉴를 확인하실 수 있습니다.
눌러봅시다.

![GitHub Actions menu](/images/howto/github-actions-menu.png)

GitHub Actions에는 이미 다양한 종류의 공식 action (workflow) 들이 존재하고, 이외에도 다른 사람들이 만들어놓은 많은 종류의 action들이 존재합니다.
그 중에서 마음에 드는 것을 골라 그대로 사용해도 상관없습니다.
하지만 저는 간단한 작업이니만큼 공부도 할 겸 직접 action을 작성해보았습니다.
(GitHub Actions 작동 방식과 문법에 대한 내용은 공식 문서를 포함해 이를 자세히 설명한 글이 많기 때문에 생략하도록 하겠습니다.)

오른쪽 위에 있는 'Set up a workflow yourself' 버튼을 누릅니다.
다음은 제가 사용한 워크플로우 파일(`main.yml`)입니다.
이대로 설정하시고 커밋해줍시다.
그러면 `.github/workflows/main.yml` 파일이 생성됩니다.

```yaml
name: Firebase Hosting Deploy

on:
  push:
    branches:
      - master

jobs:
  build:

    runs-on: ubuntu-latest

    steps:
    - uses: actions/checkout@master
    - name: Build React
      run: |
        npm install
        npm run build
    - name: Deploy to Firebase Hosting
      run: |
        sudo npm install -g firebase-tools
        firebase deploy --token ${{ secrets.FIREBASE_TOKEN }}
```

그런데 마지막에 `${{ secrets.FIREBASE_TOKEN }}` 은 무엇일까요?


### Set FIREBASE_TOKEN

Git과 같은 버전 관리 시스템에는 Access Token이나 API 키와 같은 비밀 정보(credentials)는 포함시키지 않아야 합니다.
Node.js 프로젝트에서는 이를 해결하기 위해 쉘 환경변수를 사용하는 방법, `dotenv` 등을 이용해 별도의 설정 파일을 사용하는 방법 등을 일반적으로 많이 활용합니다.

그러면 GitHub Actions에서 credential이 필요할 때는 어떻게 사용할 수 있을까요?
이를 위해 GitHub에서는 creadential을 별도로 저장해두고, 이를 `${{ secrets.이름 }}`의 형태로 불러와서 사용할 수 있도록 하고 있습니다.

여기서도 Firebase CLI에서 `firebase deploy` 명령을 내리기 위해서는 내가 이 Firebase 프로젝트에 배포 권한이 있다는 것을 증명하기 위한 인증/인가 정보가 필요합니다.
Firebase는 이를 Access Token을 이용해서 증명할 수 있도록 하고 있습니다.
먼저 이 Access Token을 얻어봅시다.  

혹시 Firebase CLI가 설치되어 있지 않다면 먼저 아래의 명령어로 설치합니다.

```sh
npm install -g firebase-tools
```

이제 아래 명령어를 입력하면 FIREBASE_TOKEN 값을 얻을 수 있습니다.

```sh
firebase login:ci
```

터미널에 출력되는 값을 복사해 둡니다.

이제 GitHub 웹사이트에 접속해서 저장소 페이지의 Settings 메뉴에 가면 Secret 항목을 보실 수 있습니다.

![GitHub Actions](/images/howto/github-actions-menu.png)

이곳에서 'Add a new secret'을 선택한 후 Name에는 `FIREBASE_TOKEN`을, Value에는 아까 복사해두었던 값을 붙여넣어 줍니다.

이제 git 저장소에는 credential 정보를 포함시키지 않은 상태에서도 우리는 `FIREBASE_TOKEN` 이라는 이름으로 해당 정보에 접근할 수 있게 되었습니다.



## Test

다 완료되었습니다!

이제 우리가 로컬 환경에서 React 앱을 작성한 후 `master` 브랜치로 푸시를 하면 GitHub Actions가 자동으로 그 파일들을 빌드해서 Firebase Hosting에 배포해줄 겁니다.
한 번 테스트를 해볼까요?

저는 `creat-react-app` 으로 생성된 기본 페이지에서 이상하게 빙빙 돌고 있는 React 로고 이미지를 없애고, 문구를 바꿔 봤습니다.
첫 문구는 역시 `Hello, world!`죠.

이제 커밋을 하고, `master` 브랜치로 푸시를 해봅시다.
아, 그 전에 먼저 GitHub 사이트의 저장소 페이지에 접속해 Actions 메뉴에서 우리가 만들었던 `Firebase Hosting Deploy` 워크플로우 화면을 띄워둡시다.

![GitHub Actions](/images/howto/firebase-hosting-react-github-actions-proceeding.png)

무언가 막 진행이 되더니...

![GitHub Actions](/images/howto/firebase-hosting-react-github-actions-complete.png)

Complete job 이 떴습니다!
이제 웹사이트를 가보면...

![Hello, world!](/images/howto/firebase-hosting-react-github-actions-hello-world.png)

헬로, 월드!



# Reference

- https://dev.to/gautemeekolsen/getting-started-with-github-actions-ci-cd-firebase-deploy-5g87
