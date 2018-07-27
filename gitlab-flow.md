## GitLab Flow
## Flow사례참조
  - [우아한형제들](http://woowabros.github.io/experience/2017/10/30/baemin-mobile-git-branch-strategy.html)
     + Repo관리 : Upstream Remote Repo (master), Origin Remote Repo (orig), Local Repo
       * Upstream Remote Repo : 메인 Repo
       * Origin Remote Repo : 개발자별 remote Repo
       * Local Repo : Origin Remote Repo에 대한 작업자 로컬 Repo
     + 브런치삭제
       * fork Repo의 작업 브런치가 Merge Request 하여 Merge된 후
       * feature브랜치 가 develop에 Merge된 후
     + Merge
       * Merge Request요청한 주체에서 Merge
       * 코드충돌시 : 뒤에서 Merge하는 사람이 해결, 충돌범위가 큰경우 이전 Merge한 사람과 협업

## Git Flow Utility
  - [소스코드](https://github.com/nvie/gitflow)
     + [SourceTree](https://www.sourcetreeapp.com/)
       | [설명](https://danielkummer.github.io/git-flow-cheatsheet/index.ko_KR.html)
     + [Fork-Source](https://github.com/petervanderdoes/gitflow-avh)
     + [모델]?(http://nvie.com/posts/a-successful-git-branching-model/)
     + install
      | brew install git-flow
     + git flow init
      | git 저장소를 초기화하고 master, develop 브랜치를 생성한다.
     + git flow feature start iss51
      | develop 브랜치로부터 feature/iss51 란 이름의 브랜치를 따오고 체크아웃한다.
     + git flow feature finish iss51
      | feature/iss51 브랜치를 develop 브랜치에 머지하고, 브랜치를 삭제한다.
     + git flow feature track iss70
      | 이미 origin 에 존재하는 feature/iss70 브랜치를 가져오고 체크아웃한다.

## 참조
  - [Git 100% 활용하기: 협업을 위한 브랜치 전략, 팁과 노하우](https://academy.realm.io/kr/posts/360andev-savvas-dalkitsis-using-git-like-a-pro/)
  - [GIT-FLOW-git-flow를-사용해-보자](http://yujuwon.tistory.com/entry/GIT-FLOW-git-flow%EB%A5%BC-%EC%82%AC%EC%9A%A9%ED%95%B4-%EB%B3%B4%EC%9E%90)
  - [Git flow 사용해보기](http://boxfoxs.tistory.com/347)
    + Windows 용 SourceTree 툴이용예시
  - [GitHub 환경에서의 실전 Git 레시피](http://meetup.toast.com/posts/116)
  - [git-flow](https://www.git-tower.com/learn/git/ebook/en/command-line/advanced-topics/git-flow)
