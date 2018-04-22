---
layout: post
title: "Using agnoster theme in zsh on iTerm"
tags: [java]
comments: true
---

## Prerequisite
* Unix-based operating system (Mac OS X or Linux)
* Zsh should be installed (v4.3.9 or more recent). (confirm it via `zsh --version`),
* curl or wget should be installed
* git should be installed

## Using agonoster theme in zsh on iTerm
1. Installation zsh
  * via curl
`sh -c "$(curl -fsSL https://raw.github.com/robbyrussell/oh-my-zsh/master/tools/install.sh)"`
  * via wget
`sh -c "$(wget https://raw.github.com/robbyrussell/oh-my-zsh/master/tools/install.sh -O -)"`

2. Edit ~/.zshrc
`vi ~/.zshrc` 명령어로 파일을 연뒤 `ZSH_THEME`부분을 다음과 같이 바꿔준다. `ZSH_THEME="agnoster"`

3. iTerm - Preferences - Profile - Colors - Solarized Dark
Solarized Dark iterm theme은 [여기](https://github.com/altercation/solarized/tree/master/iterm2-colors-solarized)에서 받을 수 있다.

4. Download one of the [patched fonts from Vim-Powerline](http://powerline.readthedocs.org/en/master/installation/osx.html)

5. iTerm - Preferences - Profile Tab - Text - Non-ASCII Font - [4에서 다운받은 폰트로 설정]

6. iTerm의 기본 shell을 zsh로 설정하기
iTerm - Preferences - Profiles Tab - General section set Command to: /bin/zsh —login
[참고사이트](http://stackoverflow.com/questions/1276703/how-to-make-zsh-run-as-a-login-shell-on-mac-os-x-in-iterm)


#### 참고
* https://github.com/robbyrussell/oh-my-zsh/
