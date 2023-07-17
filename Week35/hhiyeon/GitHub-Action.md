### 📌GitHub Actions 실습


#### CI/CD 란?
- CI : Continuous Integration
  - 지속적인 통합
  - 코드, 빌드, 테스트 부분을 자동화해서 
  - 커밋할 때마다 빌드와 자동 테스트로 동작을 확인해서 문제가 생기지 않도록 보장한다.
  - 개발에 더 많은 시간을 투자 가능
- CD : Continuous Delivery / Deployment
  - 지속적인 제공, 배포
  - 배포 자동화 과정
  - CI 프로세스를 통과하면 마지막에 배포하는 과정

#### GitHub Actions
- Workflow
- Job
- Step
- Action


#### yaml 파일 예시


```yaml
name: 'Workflows name'

on: push // on은 script 가 언제 돌아가야하는지에 대한 부분 -> push 가 들어갈 때마다 실행된다.

jobs: // 스크립트가 해야하는 행동
  first-job: // job의 이름이고, 정해진 규칙은 없다.
    name: 'First Job' // name에 지정한 것은 각각 안에서 여러가지 나뉘는데 각각 job에 대한 이름 설정 안하면 위에 셋팅한 이름으로 default

    runs-on: ubuntu-latest // 어느 가상 환경에 올릴 것인지 보통 3가지 - windows mac linux

    steps:
      - name: Say Hello World 1
        shell: bash
        run: |
          echo "Hello World from step 1"

      - name: Say Hello World 2
        shell: pwsh
        run: |
          echo "Hello World from step 2"

```


- GitHub Actions 관련 문서 링크


https://docs.github.com/en/actions/using-workflows/events-that-trigger-workflows


---
#### GitHub Action 요구사항 해결 방안
- OS 종류와 Node 버전이 다양한 경우
  
-> 매트릭스 빌드 사용

- 빌드할 때마다 테스트 실행이 불편한 경우

-> 템플릿 변경으로 빌드 테스트 분리

- 다른 job에서 빌드 아티팩트 접근이 안되는 경우

-> Built-in 스토리지 이용


---
#### 매트릭스 빌드
- 각 경우의 수에 맞게 빌드 아티팩트를 만들어서 실행


```yaml
...
  strategy:
    matrix:
      os: [ubuntu-latest, windows-2016]
      node-version: [12.x, 14.x]
...
```


- 우분투 12, 14버전
- 윈도우 12, 14버전 실행

---
#### 빌드, 테스트 분리
- build를 build와 test로 분리


```yaml
jobs:
  build:
    - run: npm ci
    - run: npm run build --if-present
    - run: npm test
```


```yaml
jobs:
  build:
  ...
  test:
  ...
```

---
#### job 병렬 실행 -> 종속적으로 실행 시키기


```yaml
jobs:
  build:
    ...
  test:
    needs: build // 이 부분이 실행되기 위해서는 build job이 필요
    ...
```


---
#### GitHub Actions 요구 사항 해결 방안
- main 브랜치 에러 위험

-> 브랜치 정책 설정

- Pull Request 는 언제 merge?

-> PR 리뷰 강제화

- 늘어나는 Issue 와 PR 관리의 어려움

-> Custom action 으로 라벨링

---
#### Branch 정책 설정 방법
- Repo 의 Settings
- Branches 메뉴에서 
- Branch protection rules 에 있는 Add rule


<img width="682" alt="스크린샷 2023-07-16 오후 2 38 31" src="https://github.com/hhiyeon/hhiyeon/assets/52193680/54f6a757-176e-415a-ba43-3d9d880b833f">


<img width="666" alt="스크린샷 2023-07-16 오후 2 39 15" src="https://github.com/hhiyeon/hhiyeon/assets/52193680/7df34170-9b3a-44f6-ad3c-0f66652fa5c0">


- Require pull request reviews before merging
  - 브랜치를 다른 브랜치와 merge 하려고할 때, PR 요청을 통해서 리뷰되어야만 가능


- Require status checks to pass before merging
  - merge 전에 status check 를 통과해야 한다.

---
#### 실습
- Comment 에 "/close" 커맨드 입력 시 이슈 닫는 워크플로우 작성

```yaml
name: 'Close Issue Condition Workflows'

on:
  issue_comment: 
    types: [ created ]

jobs: 
  close-issue:
    name: 'Close Issue Job' 
    if: contains(github.event.comment.body, '/close')
    runs-on: ubuntu-latest

    steps:
    - name: Closed Issue
      uses: actions/github-script@v6
      with:
        script: |
          github.rest.issues.update({
            issue_number: context.issue.number,
            owner: context.repo.owner,
            repo: context.repo.repo,
            state: 'closed'
          })
```

