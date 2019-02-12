# Jekyll 블로그 생성하기

## 1. Jekyll 설치

### 1.1 ruby 버전 확인

```bash
ruby -v
```

루비가 만약 2.X대가 아니라면 2.X이상의 버전으로 업그레이드

```bash
brew upgrade ruby
```

### 1.2 nodejs 설치

```bash
brew install nodejs
```

### 1.3 Jekyll 설치

```bash
sudo gem install jekyll
```

버전확인

```bash
jekyll -v
```

### 1.4 jekyll 실행

bundler 설치

```bash
gem install jekyll bundler
```

블로그 디렉토리로 이동한 뒤 의존성 패키지들을 체크하고, 업데이트

```bash
bundle install
```

```bash
bundle exec jekll serve
```
