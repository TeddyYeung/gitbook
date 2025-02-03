# fastlane 설치하기 (rbenv → 최신 Ruby → Bundler → Fastlane)

Fastlane 사용 시 꼭 읽어봐야 하는 자료&#x20;

* [![](https://t1.daumcdn.net/tistory_admin/favicon/tistory_favicon_32x32.ico)\[fastlane\] 1. fastlane이란](https://ios-development.tistory.com/255)
* [\[iOS 앱 배포 준비\] 애플에 배포하기 위한 4가지 개념](https://ios-development.tistory.com/246)
* [Fastlane match를 사용한 iOS 인증서 동기화](https://millo-l.github.io/ReactNative-fastlane-match/)

#### **Fastlane 설치 및 환경 구성 가이드**

Fastlane은 iOS 및 Android 앱 배포 자동화를 지원하는 강력한 도구입니다.\
설치 방법으로는 **Homebrew**와 **Bundler**방식이 있지만, 안정적인 종속성 관리를 위해 **Bundler**를 통한 설치가 더 권장됩니다.

#### **Fastlane 설치 방법 비교: Homebrew vs Bundler** <a href="#fastlane-homebrew-vs-bundler" id="fastlane-homebrew-vs-bundler"></a>

| **항목**             | **Homebrew**                 | **Bundler**             |
| ------------------ | ---------------------------- | ----------------------- |
| **설치 방식**          | 시스템 전체에 Fastlane 설치          | 프로젝트별로 Gemfile을 통해 설치   |
| **종속성 관리**         | 전역 Gem 환경에 의존 (프로젝트 간 충돌 가능) | 프로젝트 격리 (Gemfile로 관리)   |
| **환경 재현성**         | 동일 환경 보장 어려움                 | Gemfile.lock으로 환경 재현 가능 |
| **시스템 Ruby 사용 여부** | 기본적으로 사용                     | 원하는 Ruby 버전 사용 가능       |
| **권장 여부**          | 단순 테스트용 적합                   | 실무 프로젝트에 권장             |

따라서 **실무 환경에서는 Bundler**를 통한 설치를 진행합니다.

#### **Ruby가 Bundler 설치에 필요한 이유** <a href="#ruby-bundler" id="ruby-bundler"></a>

Bundler는 Ruby 기반 Gem 관리 도구입니다. 따라서 **Ruby 인터프리터**가 설치되어 있어야 Bundler를 사용할 수 있습니다.

#### **rBenv를 통한 최신 Ruby 설치가 필요한 이유** <a href="#rbenv-ruby" id="rbenv-ruby"></a>

* **macOS 시스템 Ruby**는 기본적으로 `/usr/bin/ruby` 경로에 설치되어 있습니다. 하지만 시스템 Ruby는 macOS 업데이트에 종속되기 때문에 다음과 같은 문제점이 발생할 수 있습니다:
  * **버전 관리의 어려움:** macOS와 함께 제공되는 Ruby 버전은 오래된 경우가 많아 최신 Gem과 호환되지 않을 수 있습니다.
  * **권한 문제:** 시스템 디렉터리(`/usr/bin`)에 설치된 Ruby에서는 Gem 설치 시 권한 문제가 발생할 수 있습니다.
  * **종속성 충돌:** 다른 프로젝트 간의 Gem 버전 충돌이 발생할 가능성이 큽니다.

이러한 문제를 피하기 위해 **rbenv**를 사용하여 최신 Ruby를 설치하고, Bundler를 통해 Fastlane을 설치하는 방법을 권장합니다.

***

### **설치 순서: rbenv → 최신 Ruby → Bundler → Fastlane** <a href="#rbenv-ruby-bundler-fastlane" id="rbenv-ruby-bundler-fastlane"></a>

***

### 1) rbenv 설치  <a href="#id-1-rbenv" id="id-1-rbenv"></a>

Bundler와 Gemfile을 활용해 Fastlane의 의존성을 관리하기 위해 먼저 Bundler 설치가 필요합니다. \
Bundler는 최신 버전의 Ruby가 설치된 환경에서 정상적으로 작동하므로, 아래 단계에 따라 먼저 rbenv를 설치합니다.

```bash
# Homebrew 업데이트
brew update

# rbenv 및 ruby-build 설치
brew install rbenv ruby-build

# rbenv 설치 확인
rbenv versions

```

위 단계를 완료하면 최신 버전의 Ruby를 설치 진행 가능합니다.

### 2) Ruby 최신 버전 설치하기 <a href="#id-2-ruby" id="id-2-ruby"></a>

```bash
# Ruby 버전 목록 확인
rbenv install -l

# 최신 버전 설치 (예: 3.2.2)
rbenv install 3.2.2

# 설치된 버전 확인
# 출력 예시:
# -MacBookPro ~ % rbenv versions system * 3.2.2 (set by /Users/{user_repo}/.rbenv/version)
# 위처럼 3.2.2가 표시되면 설치가 잘 된 것입니다! 🎉
rbenv versions

# Ruby 글로벌 버전 설정
rbenv global 3.2.2
```

이제 Ruby 최신 버전 사용을 시작할 수 있습니다. 😉

#### 3) 쉘 설정 파일에 rbenv PATH 추가

#### - rbenv를 정상적으로 작동하도록 PATH를 추가합니다.

```bash
echo 'export PATH="$HOME/.rbenv/bin:$PATH"' >> ~/.zshrc
echo 'eval "$(rbenv init -)"' >> ~/.zshrc
source ~/.zshrc
```

***

### 3) Bundler 설치 <a href="#id-3-bundler" id="id-3-bundler"></a>

1.  **Bundler 설치**

    Ruby 환경이 준비되면 Bundler를 설치해 프로젝트의 Gem 의존성을 관리합니다.

    <pre class="language-bash"><code class="lang-bash"><strong>gem install bundler
    </strong></code></pre>
2. **프로젝트 초기화 및 의존성 추가**

* 아래 코드 실행 후 `Gemfile`이라는 파일이 생성됩니다.

```bash
# 프로젝트 루트 디렉터리로 이동 후 초기화
cd /path/to/your/project
bundle init
```

3. **Gemfile 수정**\
   \- 생성된 `Gemfile`을 열어 아래 내용을 추가합니다.

<pre class="language-ruby"><code class="lang-ruby"><strong>source "https://rubygems.org"
</strong>gem "fastlane"
</code></pre>

4. **의존성 업데이트**\
   \- 변경된 내용을 적용하고 의존성을 설치합니다.\
   \- 실행 후 `Gemfile.lock` 파일이 생성됩니다. \
   &#x20;  이 두 파일(`Gemfile`과 `Gemfile.lock`)은 버전 관리 시스템(Git 등)에 추가해야 합니다.

```bash
bundle update
```

4. Bundler 설치\
   \- 위와 같이 최신 Ruby 환경이 준비되었다면 `Bundler`를 설치하여 프로젝트 의존성을 관리합니다.

```bash
gem install bundler
```

***

### **4 ) Fastlane 설치** <a href="#id-4-fastlane" id="id-4-fastlane"></a>

Fastlane을 설치합니다.

```bash
sudo gem install fastlane -NV
```
