---
layout: post
title:  "Github Page를 활용해 블로그 만들기"
author: 윤상준
date: 2018-04-06
categories: etc
---

Github는 소스코드 형상관리 도구인 Git을 사용하는 프로젝트를 지원하는 웹 호스팅 서비스입니다.<br />
Private repository를 만들지 않는 이상 무료로 제공되는 서비스입니다.<br />
Github Page는 Github에 업로드한 소스코드를 웹 호스팅 해주는 기능입니다. 추가적인 비용을 들여 웹 호스팅 서비스를 신청하지 않아도 자신만의 블로그, 웹 사이트를 서비스 할 수 있습니다.<br />
<br />
이 문서에서는 Github Page를 활용해 블로그 만드는 방법을 설명합니다.

# Jekyll 설치
아래 명령어를 실행해 jekyll 모듈을 설치합니다. Ruby가 설치되어 있어야 합니다.
```
gem install jekyll
```

# Github repository 생성
블로그 페이지로 사용할 github repository를 생성합니다.<br />

Github Site > Add Repository > Owner와 Repository Name을 입력 후 생성<br />
생성한 repository를 clone 받고 README 파일을 push 합니다.

```
git clone https://github.com/yunsangjun/blog.git
cd blog
echo "# blog" >> README.md
git init
git add README.md
git commit -m "first commit"
git remote add origin https://github.com/yunsangjun/blog.git
git push -u origin master
```

# Jekyll template 생성
아래 명령어를 실행해 블로그 템플릿을 만듭니다.
```
# jekyll new sample-blog
.
├── 404.html
├── CNAME
├── Gemfile
├── Gemfile.lock
├── README.md
├── _config.yml
├── _posts
├── _site
├── about.md
├── index.md

```

# 블로그 환경 설정
`_config.yml` 파일을 열고 아래와 같이 편집합니다.

- Github Page 기본 도메인 사용 시

  ```
  baseurl: "/blog" # the subpath of your site, e.g. /blog
  url: "" # the base hostname & protocol for your site, e.g. http://example.com
  ```

- 사용자 도메인 사용 시

  ```
  baseurl: "" # the subpath of your site, e.g. /blog
  url: "http://blog.example.com" # the base hostname & protocol for your site, e.g. http://example.com
  ```

# 블로그 페이지 작성
`_posts` 디렉토리 하위에 포맷(`yyyy-mm-dd-title.markdown`)을 맞춰 블로그 페이지를 작성합니다.<br />
date는 UTC 기준입니다. KST 기준으로 변경하려면 +0900을 추가합니다.

```
# vi _post/2018-04-06-how-to-create-blog.markdown
---
layout: post
title:  "Example"
author: 저자
date: yyyy-mm-dd hh:mm:ss +0900
categories: etc
---
...

```

# 블로그 로컬 환경에서 실행
아래 명령어를 실행해 로컬 환경에서 블로그 페이지를 확인할 수 있습니다.

- Github Page 기본 도메인 사용 시

  ```
  jekyll serve --watch

  http://127.0.0.1:4000/blog/
  ```

- 사용자 도메인 사용 시

  ```
  jekyll serve --watch

  http://127.0.0.1:4000/
  ```

# Github repository에 commit & push
아래 명령어를 실행해 변경 내용을 Github repository에 반영합니다.

```
git add --all
git commit -m "init"
git push
```

# Github Page 활성화
아래와 같이 Github Page 기능을 활성화 합니다.<br />

```
Github repository home > Settings > Github Pages > Source > master branch > Save
```

설정을 완료하면 아래와 같이 접속가능한 Github Page 주소가 출력됩니다.<br />
이 주소의 GITHUB_REPOSITORY_NAME는 블로그 페이지 용도로 생성한 repository 이름이고, GITHUB_OWNER는 repository의 소유자 이름입니다.

```
## https://GITHUB_OWNER.github.io/GITHUB_REPOSITORY_NAME/
Your site is published at https://yunsangjun.github.io/blog/
```

# 사용자 도메인 설정(옵션)
사용자 도메인을 사용할 경우 Github repository와 사용자 도메인 관리페이지에서 추가 설정이 필요합니다.

- Github repository

  ```
  Github repository home > Settings > Github Pages > Source > Custom domain > blog.example.com
  ```

- 사용자 도메인 관리페이지
도메인 제공 업체마다 설정 방법은 다를 수 있습니다. (이 가이드는 GoDaddy 기준)<br />
도메인 관리페이지에서 아래와 같이 CNAME을 설정합니다.<br />
CNAME 뒤에 첫번째 값은 위에서 설정한 사용자 도메인 주소, 두번째 값은 부여받은 Github Page 주소입니다.<br />
DNS 정보가 업데이트 완료되면 사용자 도메인으로 접속해 블로그 페이지를 확인 할 수 있습니다.(DNS 정보 업데이트 소요 시간은 도메인 제공 업체 별로 상이합니다.)

  ```
  ## CNAME CUSTOM_DOMAIN GITHUB_PAGE_ADDRESS
  CNAME blog.example.com yunsangjun.github.io
  ```
