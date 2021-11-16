---
layout: post
title: Introduce Github pages and Jekyll
categories: Other
---

# Github pages와 Jekyll

### Github pages와 Jekyll이 무엇인지 알아보자.



## Github pages

---

Github page는 github에서 제공하는 정적 웹 호스팅 서비스로, 자신의 repository에 HTML, CSS, JavaScript 파일을 push하는 방식으로 간단하게 웹 사이트를 생성할 수 있습니다. Jekyll 등의 정적 웹사이트 생성기를 통해 markdown을 사용해서도 생성할 수 있습니다.

정적 웹 호스팅 서비스이기 때문에 항상 일정한 화면만 제공해줍니다.





## Github pages의 장단점

---

#### 장점

- Jekyll을 사용하는 경우 다양한 테마가 있어서 커스터마이징이 자유로움
- markdown 지원
- repository에 push만 하면 글이 작성됨
- 정적인 페이지기 때문에 속도가 빠름

#### 단점

- 초기 설정에 많은 시간이 소요됨
- 커스터마이징이 자유로운 만큼 모든 것을 직접 만들어야 함
- git을 사용할 줄 알아야 함





## Jekyll

---

Github pages에서는 정적인 페이지만 표시할 수 있기 때문에 정적 웹사이트 생성기와 연동해 사용하기도 합니다. 정적 웹사이트 생성기에는 Jekyll, Hexo, Hugo 등이 있는데, 여기에서는 Jekyll에 대해 다루겠습니다.

Jekyll은 정적 웹사이트 생성기 중 하나로, Ruby로 개발되었습니다. 대부분 웹 서비스들은 사용자의 요청을 해석해 서버 측에서 데이터를 가공한 후 웹 페이지를 생성해 내보내게 됩니다. 하지만 Jekyll은  마크업 언어로 작성된 게시글을 빌드해 미리 웹 페이지를 만들어 두고 사용자의 요청이 들어오면 미리 만들어 둔 HTML 화면을 그대로 응답하는 방식으로 작동합니다. 따라서, Jekyll은 모든 화면에 대해 미리 빌드를 진행해 둡니다.





## Jekyll 구조

---

[Jekyll 공식 페이지](https://jekyllrb.com/docs/structure/)에 의하면 Jekyll 프로젝트를 생성할 때 아래와 같은 파일 구조로 생성된다고 합니다. 테마를 선택한 경우 다를 수 있습니다.

```
.
├── _config.yml
├── _data
│   └── members.yml
├── _drafts
│   ├── begin-with-the-crazy-ideas.md
│   └── on-simplicity-in-technology.md
├── _includes
│   ├── footer.html
│   └── header.html
├── _layouts
│   ├── default.html
│   └── post.html
├── _posts
│   ├── 2007-10-29-why-every-programmer-should-play-nethack.md
│   └── 2009-04-26-barcamp-boston-4-roundup.md
├── _sass
│   ├── _base.scss
│   └── _layout.scss
├── _site
├── .jekyll-cache
│   └── Jekyll
│       └── Cache
│           └── [...]
├── .jekyll-metadata
└── index.html # can also be an 'index.md' with valid front matter
```

### _config.yml

Jekyll로 생성될 정적 페이지의 환경설정 정보들이 담겨있습니다.  프로젝트 빌드할 때 이 정보를 읽고 페이지들을 생성합니다.

### _data

사용자 정의 데이터를 .yml, .yaml, .json, .csv, .tsv 형식으로 저장합니다.

### _posts

블로그 포스트를 저장하는 곳입니다. 파일명을 YEAR-MONTH-DAY-title 형식으로 저장합니다.

파일의 머릿글에 YAML Front Matter 섹션이 있으면 Jekyll 엔진에 의해서 변환됩니다. 다음과 같은 형식으로 작성합니다. 아래에 이 게시글의 머릿글을 예시로 작성했습니다.

```
---
layout: post
title: introduce Github pages and Jekyll
categories: Other
---
```

### _drafts

아직 게시하지 않은 포스트를 보관하는 곳입니다. 파일명을 포스트와 달리 title만 적어둡니다.

### _includes

재사용을 위한 파일들을 저장하는 공간입니다.

### _layouts

레이아웃과 관련된 템플릿을 저장하는 곳입니다.

### _sass

sass 파일을 저장하는 곳입니다. main.scss에 import할 수 있으며, 하나의 CSS파일로 가공됩니다.

### _site

Jekyll 프로젝트를 빌드할 때 생성된 정적 사이트가 저장되는 곳입니다.

### .jekyll-metadata

Jekyll이 변경된 내용만을 빌드하기 위해 관리하는 파일입니다.

### index.html

블로그의 첫 화면입니다. Jekyll은 프로젝트 폴더에서 index.html을 찾아 첫 화면으로 렌더링합니다.









