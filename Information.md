# TypeScript Study

## 스터디 진행 방식
1. 매주 화요일 오후 8시 디스코드 화상회의로 스터디가 진행됩니다.
2. 2개의 Chapter를 학습하고 정리한 내용을 md 파일로 작성합니다.
3. 스터디 시간에 발표자를 랜덤으로 선정해서 정리내용을 공유합니다.


## Git 사용 가이드

### 📦 초기 셋팅 (최초 1회만 진행)

#### 1. organization 리포지토리를 자기 git 계정에 fork 합니다.

#### 2. fork한 리포지토리를 본인 local에 clone 합니다.
  ```bash
    git clone https://github.com/${username}/typescript-study.git
  ```

#### 3. organization 리포지토리를 원격 리포지토리에 등록합니다.
  ```bash
    git remote add upstream https://github.com/2024-typescript-study/typescript-study.git
  ```

<br>

### ✨ 학습 내용을 git에 업데이트할 때 (매주 진행)

#### 1. organization 리포지토리의 최신 내용을 업데이트 합니다.
  ```bash
    git fetch upstream
    git checkout main
    git merge upstream/main
  ```

#### 2. 브랜치 생성 - 작업내용을 추가할 새로운 브랜치를 생성합니다.
  - 브랜치명 컨벤션: docs/week1-${username}
  ```bash
    git checkout -b docs/week1-kyeongjun
  ```
#### 3. 타입스크립트 정리 내용을 마크다운으로 작성합니다.
  - 파일명 컨벤션: {이름}-chapter{숫자}.md
  ```bash
    week1/chapter1/고경준-chapter1.md
    week1/chapter2/고경준-chapter2.md
  ```
#### 4. 작업 변경사항을 Commit 합니다.
  - 커밋 컨벤션: [docs] week1 {커밋 내용}
  ```bash
    git commit -m '[docs] week1 chapter1 내용 정리'
  ```
#### 5. 본인의 fork된 리포지토리에 Push 합니다.
  - 2번에서 만든 브랜치 이름으로 push
  ```bash
    git push origin docs/week1-kyeongjun
  ```
#### 6. organization 리포지토리의 main 브랜치에 PR을 생성합니다.
  - PR 제목 컨벤션: [Week1] {본인이름} Chapter 1,2 학습정리
  ```bash
    [Week1] 고경준 Chapter 1,2 학습정리
  ```