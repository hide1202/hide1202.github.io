---
title: Jekyll과 minimal-mistakes 테마를 이용한 블로그 생성
categories:
  - Life
tags:
  - life
---

# 발단
기술적인 내용을 써보기 위해서 블로그를 시작하고 싶다는 생각은 항상 가지고 있었다. 이것저것 한참을 고민하다가 결국 Github Pages 기능과 Jekyll을 이용해서 구축해보기로 했다.

# 테마 결정
하지만 Jekyll(의 철자는 뭔가 계속 헷갈린다)의 수많은 테마를 보자니, 또 결정장애가 발생했다. 구글링을 하다보니 Github의 Star가 많은 minimal-mistakes 로 우선 시작해보기로 했다.

# 준비
- Github Pages 호스팅을 위해서는 특별한 이름의 리파지토리가 필요하다. `#{GITHUB_ID}.github.io` 라는 이름으로 리파지토리를 생성해야 한다. (Organization일 경우 Organization 이름을 넣으면 된다)
- Ruby가 필요하다. 구축은 macOS Sierra 를 이용했으며, [Homebrew](https://brew.sh/index_ko.html)를 이용하면 Ruby를 손쉽게 설치할 수 있다. 
- Gem을 이용해서 Jekyll 을 설치한다.

# 테마 페이지 가져오기

> 아래 내용은 [minimal-misatakes의 퀵가이드](https://mmistakes.github.io/minimal-mistakes/docs/quick-start-guide/) 를 참조해도 된다.

[minimal-mistakes](https://github.com/mmistakes/minimal-mistakes) 에서 소스를 가져온다. Fork를 해도 되지만 그냥 소스를 다운받아서 옮기면 된다. 옮기기 전에 쓸데없는 것들은 삭제하자. 삭제해야 할 리스트는 아래와 같다.
- `.editorconfig`
- `.gitattributes`
- `.github`
- `/docs`
- `/test`
- `CHANGELOG.md`
- `minimal-mistakes-jekyll.gemspec`
- `README.md`
- `screenshot-layouts.png`
- `screenshot.png`

`Gemfile`을 아래 코드로 대체한다.
```ruby
source "https://rubygems.org"

gem "github-pages", group: :jekyll_plugins

group :jekyll_plugins do
  gem "jekyll-paginate"
  gem "jekyll-sitemap"
  gem "jekyll-gist"
  gem "jekyll-feed"
  gem "jemoji"
end
```
Gem을 통해 `bundler` 를 설치하자. 이제 Jekyll 명령어를 `bundle exec jekyll build` 와 같이 실행 할 수 있다. 로컬에서 확인을 위해서 `bundle exec jekyll serve` 명령어를 사용하면 된다.

# Troubleshooting
macOS Sierra에서는 `nokogiri` 설치(의존에 의해 간접설치됨)에서 오류가 발생할 수 있다. 그럼 아래 명령어를 통해 `nokogiri`를 설치하면 된다.
- `sudo gem install nokogiri -- --use-system-libraries=true --with-xml2-include=/Applications/Xcode.app/Contents/Developer/Platforms/MacOSX.platform/Developer/SDKs/MacOSX10.12.sdk/usr/include/libxml2/`

# 포스트 쓰기
포스트는 특정 제목으로 작성되어야 한다. `YYYY-MM-DD-{TITLE}.md` 로 작성하면 된다.
내용은 크게 두 부분으로 나눌 수 있다. 메타데이터를 위한 yaml 부분과 본문을 위한 마크다운 부분이다. yaml은 md 파일의 머릿말에 적으면 된다. 아래가 그 예제이다.
```yaml
---
title: Jekyll과 minimal-mistakes 테마를 이용한 블로그 생성
categories:
  - Life
tags:
  - life
---
```

# 카테고리, 태그 적용
`minimal-mistakes`카테고리와 태그를 지원은 하지만 기본 설정되어 있지는 않다. `minimal-mistakes`의 [깃헙 페이지의 docs 폴더](https://github.com/mmistakes/minimal-mistakes/tree/master/docs) 를 참고하면 다양한 예제들을 가져올 수 있다. 해당 예제들은 데모로 제공되는 페이지의 코드들이다. 이 폴더의 `_pages` 하위 폴더의 `category-archive.html`와 `tag-archive.html`을 그대로 가져와서 `_pages`를 만들고 옮겨놓으면 된다.

# 기타 적용
`minimal-mistakes`의 데모를 참조하고 사용하고 싶은 기능이 있으면, 앞서 언급했던 [깃헙 페이지의 docs 폴더](https://github.com/mmistakes/minimal-mistakes/tree/master/docs) 의 코드를 참고하면 된다.

# 마치며
시행착오를 많이 격으면서 Jekyll 블로그를 구축했던 터라, 중간중간 빠진 부분이 있을 수 있다. 빠진 부분은 `minimal-mistakes`, `Jekyll` 의 공식 가이드를 참고하면 된다. 구축 전에 꼭 공식 가이드를 읽어보고 한번 차근차근 머릿속으로 정리해보고 나서 시도해보는게 좋을 것 같다. 다른 테마를 적용하길 원한다면 해당 테마의 가이드를 꼼꼼히 읽으면 큰 어려움 없이 적용할 수 있을 것이다.