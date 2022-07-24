---

#lng_pair: "id_About_MyBlog"
title: "Github Blog 02 Chirpy Theme Setting "

# post specific
author: kae
# publish date
date: 2022-07-23 12:24:06 +0900
category: [Github,Blog]
tags: [github, blog, writeup]

# disable comments on this page
#comments_disable: true

# thumbnail image for post
# img: ":Flutter.jpeg"

# data path
img_path: /assets/img/2022-07-23-Github-Blog-Setting-01/

# if pinned Posts
# pin: true

# lcb: "{"

---

이 포스트는 [jekyll Chirpy Theme](https://chirpy.cotes.page)의 설정을 이것저것 기록해 놓은 내용이다.

Chirpy 테마를 사용하고 있어서 Chirpy 테마 세팅이라고는 했는데, 사실 테마에 한정되어 있기 보다는 jekyll과 Markdown 사용팁에 가깝다.

기본적으로 제작자가 제공해준 정보를 보고 그냥 내가 설정한 내용만 뽑아 정리해 놓은 내용이니, 자세한 건 이하의 제작자 공식 튜토리얼 포스트를 참고하도록 하자.

- [Customize the Favicon](https://chirpy.cotes.page/posts/customize-the-favicon/)
- [Writing a New Post](https://chirpy.cotes.page/posts/write-a-new-post/)

타이틀에 (영문)형식으로 링크를 걸어놓은 항목은 원문 포스트의 관련 부분을 맵핑해 놓은 것이니 이해가 안되면 그쪽으로 들어가 원문을 보면 된다. 

***

# 본인 정보로 수정

## _config.yml [(Configuration)](https://chirpy.cotes.page/posts/getting-started/#configuration)
> 로컬 서버로 수정 확인을 하는 경우, `_config.yml` 수정 후에는 반드시 로컬 서버를 재기동 해줘야 수정이 반영된다.
{: .prompt-warning }

공식페이지를 보면 `_config.yml`의
- url
- avatar
- timezone
- lang

이 4항목의 수정을 하라고 하는데, 이건 최소한이고 실제로는 이것저것 많이 수정해 줘야한다.  
그냥 본인의 정보를 넣는 곳이다 싶은 키는 전부 수정해주면 된다. 수정하는거다 싶은 목록을 테이블로 작성했보았다. 되도록 `_config.yml` 에 작성되어있는 순서로 적었다.

|key|value format|value example|specific|
|---|---------|------|-----------------|
|lang|언어 코드 알파벳 두글자|en|`_data/locales`폴더에 있는 파일 이름의 앞 2글자라고 생각하면 쉽다. <br> `en`이 기본 값인데, 한국어를 쓰고싶으면 `kr`로 변경.|
|timezone|TZ database name 형식|Asia/Seoul|한국에서 이용중인 경우는 `Asia/Seoul`, 그 외 지역은 [timezone database](https://en.wikipedia.org/wiki/List_of_tz_database_time_zones)를 참고.|
|title|*|Kae's Dev|본인의 블로그 타이틀|
|tagline|*|honyarara|블로그 타이틀 밑에 출력되는 간단한 소개글|
|description|*|Developer, Study, Github, Programming| 검색 엔진에 걸릴 키워드들을 ,로 구분해 입력|
|url|github blog url|https://kaesknzxvf.github.io|이 블로그의 url, github blog 니까 당연히 `https://repository명.github.io` 형식|
|github.username|*|kaesknzxvf|github 계정 입력|
|twitte.username|*|kzaxevsf52|twitter 계정 입력|
|social.name|*|kae|사용할 닉네임 입력|
|social.email|email 주소|kaesknzxvf@gmail.com|사용할 이메일 주소 입력|
|social.links|- SNS urls|- https://twitter.com/kzaxevsf52 <br>- https://github.com/kaesknzxvf |본인의 SNS 링크들 입력, 문자열 선두에 - 를 포함해 개행하여 구분|
|google_site_verification|sitemap content code|생략|google sitemap을 작성할 때 제공해주는 컨텐츠 코드를 복사해 입력, sitemap 등록 전에는 생략|
|google_analytics.id|G-xxxxxxxxxx|생략|google analytics 등록 코드를 입력, sitemap과 마찬가지로 등록 전에는 생략|
|theme_mode|light 또는 dark|dark|블로그의 테마 색상 선택|
|img_cdn|CND url|https://cdn.com|[CND(Content Delivery Network)](https://ko.wikipedia.org/wiki/콘텐츠_전송_네트워크) 주소가 있으면 넣어주면 된다. 난 없어서 생략|
|avatar|이미지 경로|/assets/img/main_img.png|프로필 이미지의 경로를 입력|
|toc|bool|true| Table of Contents, 게시물 오른쪽에 목차를 표시할지 여부|
|paginate|숫자|10|한 페이지에 몇 개의 포스트를 출력할지 설정|


## ABOUT 페이지
친절하게 팁이 써져있다. `_tabs/about.md` 파일을 마크다운으로 작성해 주면 된다.  
자기소개를 적으면 되는데, 뭐.. 적을 만한게 없어서 방치중

<!-- 그외에 CATEGORIES, TAGS, ARCHIVES 페이지도 각각 `_tabs/categories.md`, `_tabs/tags.md`, `_tabs/ahrchives.md` 에서 커스텀 가능한 듯 한데, 템플릿이 깔끔하게 잘되어 있어서 아직 수정할 생각은 없다. -->

***

# UI 커스텀
보이는 거 관련해서 수정한 설정

## Footer 출력 정보 수정

Footer는 블로그 맨 하단의 저자, 소유권/저작권 등을 나타내는 정보인데 chirpy 테마에서는 이하와 같이 표시되며, `_include/footer.html` 에서 수정할 수 있다.

![footer_chirpy](footer_chirpy.png)

{% raw %}
```html
<!-- The Footer -->

<footer class="row pl-3 pr-3">
  <div class="col-12 d-flex justify-content-between align-items-center text-muted pl-0 pr-0">
    <div class="footer-left">
      <p class="mb-0">
        © {{ 'now' | date: "%Y" }}
        <a href="{{ site.social.links[0] }}">{{ site.social.name }}</a>.
        {% if site.data.locales[lang].copyright.brief %}
        <span data-toggle="tooltip" data-placement="top"
          title="{{ site.data.locales[lang].copyright.verbose }}">{{ site.data.locales[lang].copyright.brief }}</span>
        {% endif %}
      </p>
    </div>

    <div class="footer-right">
      <p class="mb-0">
        {% capture _platform %}
          <a href="https://jekyllrb.com" target="_blank" rel="noopener">Jekyll</a>
        {% endcapture %}

        {% capture _theme %}
          <a href="https://github.com/cotes2020/jekyll-theme-chirpy" target="_blank" rel="noopener">Chirpy</a>
        {% endcapture %}

        {{ site.data.locales[lang].meta
          | default: 'Powered by :PLATFORM with :THEME theme.'
          | replace: ':PLATFORM', _platform | replace: ':THEME', _theme
        }}

      </p>
    </div>

  </div>

</footer>
```
{: file="_include/footer.html" }
{% endraw %}

코드를 보면 왼쪽(`class="footer-left"`)에는 블로그 소유자(나), 오른쪽에는 테마 제작자의 정보를 출력하도록 되어있는 걸 알 수 있다.   
딱히 수정하지 않아도 되지만, 나는 오른쪽(`class="footer-right"`)은 그냥 냅두고 왼쪽의 정보를 지워줄 것이다. 심플하게 지우고 싶은 정보를 주석처리 해주면 끝. `class="footer-right"` 의 `div` 블럭을 전부 주석처리 해주었다.

적용 결과 왼쪽의 'Powered by ~ '가 사라진 걸 볼 수 있다.
![footer_chirpy_edit](footer_chirpy_edit.png)
![footer_custom](footer_custom.png)


## 사이드 바 꾸미기

사이드 바 관련 UI는 `_sass/addon/commons.scss`의 `#sidebar {}` 항목에서 수정해 줄 수 있다.
나는 별건 안하고 사이드 바에 예쁜 사진을 넣어줬는데, `background`키의 값을 이미지의 경로로 바꿔 주면 된다.

원래는 이렇게 `--sidebar-bg`라는 지정한 색상을 나타내는 변수가 지정되어 있을 텐데,

```scss
background: var(--sidebar-bg);
```
{: .nolineno}
형식을 `url`로 바꿔 주고 안에 이미지의 경로를 입력해 주면 된다.
```scss
background: url(<img_path>);
```
{: .nolineno}
> `<xxxxxxx>` 의 값은 본인 환경에 맞게 수정

이미지를 넣지 않고 색만 바꿔주고 싶다면 `--sidebar-bg`의 값을 수정해 주면 되는데, 이 변수는 `_sass/colors/light-typography.scss`와 `_sass/colors/dark-typography.scss` 에서 변경할 수 있다.  
이름에서 알 수 있듯이 `light-typography.scss`는 밝은 배경일 때의 색 설정, `dark-typography.scss`는 어두운 배경일 때의 색 설정이다. 각각 바꿔주면 된다.

각 파일을 보면 `--sidebar-bg` 이외에도 사이드 바 관련해서 이하의 변수가 설정되어 있는 것을 볼 수 있는데, 
```scss
  --sidebar-muted-color: #a2a19f;
  --sidebar-active-color: #424242;
  --nav-cursor-color: #757575;
  --sidebar-btn-bg: white;
```
{: .nolineno}
이 변수들은 사이드 바에 있는 글씨의 색을 설정하는 변수다.  
사이드 바의 배경색상이나 설정한 이미지의 채도에 따라 이 값을 바꿔줘야 글씨가 잘 보일테니, 적당히 설정을 변경해 주자.

## 사이드 바 하단: 컨텍트 아이콘 표시 여부, 링크 수정
`_data/contact.yml` 에서 표시할 아이콘을 수정 할 수 있다.

기본적으로 이하의 7개의 정보가 이미 입력되어있는데
- github
- twitter
- email
- rss
- mastodon
- linkedin
- stack-overflow

초기 상태에서는 github, twitter, email, rss 가 활성화되어 있을 거다.  
자기가 사용하고 싶은 링크만 남기고 커멘드 아웃 해주면 된다. 이때, 원래 커멘드 아웃 되어있던 mastodon, linkedin, stack-overflow 같은 경우는 직접 url을 추가해 줘야한다.  
나는 github, twitter, rss, linkedin 로 남겨 놓음.

## Favicon 수정 [(Customize the Favicon)](https://chirpy.cotes.page/posts/customize-the-favicon/)

홈페이지의 심볼아이콘인, favicon을 변경해보자.   
초기상태의 chirpy 테마 favicon은 수박을 먹는 개미친구로 되어있다. 귀여움.

![chirpy_theme_favicon](favicon_chirpy.png){:width="20%" height="20%"}

이걸 내가 원하는 아이콘으로 바꿔줄 건데, 나는 내가 찍은 큐브 사진을 사용했다.

자기가 아이콘으로 하고 싶은 이미지를 선택했으면, [Real Favicon Generator](https://realfavicongenerator.net) 사이트에 들어가서 favicon 패키지를 생성해 주면 된다.  
`Select your Favicon image`를 누르면 이지미를 제출할 수 있는데, 최소 70x70 픽셀의 이미지를 제출하라고 되어있으니 그것만 주의해서 이미지를 제출해주자. 그러면 로딩바가 나오는데 좀 기다리면 자동으로 favicon 패키지를 생성해준다.

그럼 이런 페이지가 나오면서 데스트답에서 내가 설정한 아이콘이 어떻게 보여질지 미리보기를 보여주는데, 
![favicon_generator_priview](favicon_generator_priview.png){:width="90%" height="90%"}

여기서 아이콘의 사이즈, 아이콘 테두리의 사각 라운딩 정도, 배경색이 투명일 경우 배경색을 커스텀 할 수 있다.  
![favicon_generator_setting](favicon_generator_setting.png){:width="90%" height="90%"}

데스크 탑에서 뿐만 아니라 iOS Web Clip, Android Chrome, Windows Metro, macOS Safari 에서의 미리보기와 각각 커스텀도 가능하니 필요하면 설정을 하면 된다.

그런 설정들을 지나서 맨 하단에 도착하면 `Generate your Favicons and HTML code` 라는 버튼이 있는데, 이걸 클릭하면 설정한 favicon 들을 압축해서 다운받을 수 있다.   
별 설정하지 않고 그냥 다운 받았긴했지만, 참 유능한 사이트... 

다운 받은 파일`favicon_package_v0`을 열어보면 이하의 파일 9개가 들어있을 것이다.
```bash
.
├── favicon_package_v0
│   ├── android-chrome-192x192.png
│   ├── android-chrome-512x512.png
│   ├── apple-touch-icon.png
│   ├── browserconfig.xml
│   ├── favicon-16x16.png
│   ├── favicon-32x32.png
│   ├── favicon.ico
│   ├── mstile-150x150.png
│   └── site.webmanifest
```
{: .nolineno}
이 중에서 이 두 파일은 삭제해주자. 원래 테마에 들어있는 파일을 사용할 것이다.
- browserconfig.xml
- site.webmanifest

그 후 나머지 파일을 `assets/img/favicon` 로 옮긴다. 원래 있던 파일들을 전부 덮어쓰기하면 된다.

그 후 페이지를 새로고침하여 favicon 이 적용된 걸 확인하면 끝~   
캐시때문에 바로 안바뀔수 있으니 캐시 삭제하고 새로고침하면 바로 확인 할 수 있다.

![favicon_custom](favicon_custom.png){:width="40%" height="40%"}

***

# 포스트 (Post) 관련
포스팅 페이지 관련한 설정 

## 저자 정보 재정의 [(Authors info)](https://chirpy.cotes.page/posts/write-a-new-post/#author-information)

포스트를 발행하면 타이틀과 발행시간 밑에 'By저자'가 출력되는데, 이 저자 정보를 재정의하는 방법이다.  

![post_header](post_header.png){:width="90%" height="90%"}


`_data/authors.yml` 파일의 내용을

``` yml
<author_id>:
  name: <full name>
  twitter: <twitter_of_author>
  url: <homepage_of_author>
```
{: file="_data/authors.yml" }

> `<xxxxxxx>` 의 내용은 본인의 정보를 입력

로 세팅 해준 후, `해당 포스트의 yaml block`에 author라는 키값을 추가, 위의 설정한 author_id를 값으로 넣어주면 된다.   
파일을 열어보면 초기 상태로 테마 제작자의 정보가 입력되어 있을텐데, 수정을 해주면 된다.

```yaml
---
author: <author_id>
---
```

사실 이런 세팅을 해주지 않아도 'By저자'는 `_config.yml` 파일에 세팅한 

```yaml
social:
  name: <your_name>
```
{: file="_config.yml" }

의 정보를 가져와서 자동으로 입력이 되지만,  `_data/authors.yml` 에서 세팅한 내용으로 재정의를 해주면 twitter 계정 정보가 추가 되어서 검색이 되기 쉽고, 입력해준 url이 하이퍼 링크로 들어가서 저자를 클릭했을 때 내가 원하는 페이지로 연결해 줄 수 있다. 

나는 url에 깃헙 주소를 입력해 두었다.

## 이미지 파일 경로 [(Image path)](https://chirpy.cotes.page/posts/write-a-new-post/#image-path)

포스팅 할 때, 이미지를 넣다보면 
```markdown
![img-description](/assets/img/post01/image.png)
```
{: .nolineno }
이런식으로 작성하게 되는데, 이미지의 크기나 정렬, 스타일은 그때그때 따라 달라질 수 있으니 하나하나 설정하는 게 별로 불편하다고 생각이 되지 않지만, 파일 경로만큼은 입력이 귀찮다.
어짜피 대부분은 `/assets/img/...` 의 하위폴더에 위치할 것이기 때문에...

그런 이유로 `해당 포스트의 yaml block` 에 `img_path` 키를 추가하여
```yaml
---
img_path: /assets/img/post01/
---
```
> 이미지 경로는 임의 값, 본인 환경에 맞게 수정

이미지 파일의 절대 경로를 미리 입력해 두면, 글을 작성할 때에는 

```markdown
![img-description](image.png)
```
{: .nolineno }
> 이미지 파일이름은 임의 값, 본인 환경에 맞게 수정


이런 식으로 파일 이름만으로 이미지 경로를 설정할 수 있기 때문에 편하다. 이미지 파일을 저장하는 폴더 이름이 바뀌어도 `img_path` 값 하나만 수정해주면 되기 때문에 관리도 편하고.

내 경우는 포스팅 하나에 사용되는 이미지를 전부 `/assets/img/포스트의 이름과 같은 이름의 폴더/` 에 넣어놓는 스타일이라 매우 유용.

## 포스트 고정 [(Pinned post)](https://chirpy.cotes.page/posts/write-a-new-post/#pinned-posts)

특정 포스트를 고정하고 싶을 때, `해당 포스트의 yaml block` 에 `pin` 키를 추가하여 `true`로 설정하면 끝
```yaml
---
pin: true
---
```

![pinned_post](pinned_post.png){:width="90%" height="90%"}

이렇게 표시 되면서 Home 화면의 맨 위로 포스트가 고정 된다.

***

# 후기
jekyll 개발자 대단스