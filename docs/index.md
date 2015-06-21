---
title: Hubot
permalink: /docs/index.html
layout: docs
---

개인적으로 공부하기 위해 휴봇 Document 를 번역/의역한 문서입니다.

## 휴봇 시작하기

당신은 [node.js and npm](https://docs.npmjs.com/getting-started/installing-node) 가 필요합니다.. 한번 설치가 되어 있다면, 우리는 아래 명령어로 hubot generator 설치가 가능합니다:

    %  npm install -g yo generator-hubot

위 명령어는 우리에게 `hubot` [yeoman](http://yeoman.io/) generator 를 설치해 줍니다. 
이제 우리는 새로운 디렉토리를 만들고 새로운 휴봇 인스턴스를 생성할 수 있습니다.
예를 들어 우리가 myhubot 이라 불리는 봇을 만들기 원한다면 아래와 같이 생성합니다:


    % mkdir myhubot
    % cd myhubot
    % yo hubot

이 시점에서, 당신은 봇을 만들기 위한 몇가지 질문과 어느 [adapter](/docs/adapters.md)를 사용할건지에 대해 답변해야 합니다. Adapters 는 휴봇이 다양한 채팅 프로그램과 연결하기 위한 방법입니다.

만약 당신이 git 을 사용하면, 생성된 디렉토리 안에 .gitignore가 포함되어 있을 것이고, 그래서 당신은 아래와 같이 모든 파일을 추가하고 초기화 할 수 있습니다:

    % git init
    % git add .
    % git commit -m "Initial commit"

만약 당신이 휴봇 설정을 하기 위해 대화형 프롬프트 없이 자동으로 당신의 휴봇이 빌드되기를 선호한다면, 당신은 `yo hubot` 명령어를 아래와 같은 옵션들에 따라 추가할 수 있습니다:

| Option                                      | Description
| ------------------------------------------- | -----------------------------------------------------
| `--owner="Bot Wrangler <bw@example.com>"`   | Bot owner, e.g. "Bot Wrangler <bw@example.com>"
| `--name="Hubot"`                            | Bot name, e.g. "Hubot"
| `--description="Delightfully aware robutt"` | Bot description, e.g. "Delightfully aware robutt"
| `--adapter=campfire`                        | Bot adapter, e.g. "campfire"
| `--defaults`                                | Declare all defaults are set and no prompting required

당신은 이제 당신만의 휴봇을 가지게 되었습니다! npm dependencies 설치, 스크립트 로딩 그리고 당신의 휴봇을 실행할 수 있는 편리한 명령어가 `bin/hubot` 명령어로 있습니다.

휴봇은 데이터를 유지하기위해 Redis 가 필요합니다. 그래서 당신의 컴퓨터에 휴봇을 실행하기 이전에 localhost 에 Redis를 설치해야 합니다. 만약 휴봇을 Redis 없이 단지 테스트 하기를 원한다면, 당신은 `hubot-scripts.json` 파일에서 `redis-brain.coffee` 를 지울 수 있습니다.

    % bin/hubot
    Hubot>

위 명령어는 대게 유용한 개발을 위해 [shell adapter](/docs/adapters/shell.md)를 사용해서 휴봇을 시작합니다. 

콘솔에 기록된 `Hubot>`; 이것은 명령어에 응답하는 당신의 휴봇 이름입니다.
예를 들어, 사용 가능한 명령어들을 리스트에 출력하기 위해 아래와 같이 사용 가능합니다 - (주의:현재 help 를 치면 아무것도 나오지 않는데 이부분은 확인이 필요합니다):

    % bin/hubot
    hubot> hubot help
    hubot adapter - Reply with the adapter
    hubot animate me <query> - The same thing as `image me`, except adds a few parameters to try to return an animated GIF instead.
    hubot echo <text> - Reply back with <text>
    hubot help - Displays all of the help commands that hubot knows about.
    hubot help <query> - Displays all help commands that match <query>.
    hubot image me <query> - The Original. Queries Google Images for <query> and returns a random top result.
    hubot map me <query> - Returns a map view of the area returned by `query`.
    hubot mustache me <query> - Searches Google Images for the specified query and mustaches it.
    hubot mustache me <url> - Adds a mustache to the specified URL.
    hubot ping - Reply with pong
    hubot pronounce <phrase> in <language> - Provides pronounciation of <phrase> (<language> is optional)
    hubot pug bomb N - get N pugs
    hubot pug me - Receive a pug
    hubot the rules - Make sure hubot still knows the rules.
    hubot time - Reply with current time
    hubot translate me <phrase> - Searches for a translation for the <phrase> and then prints that bad boy out.
    hubot translate me from <source> into <target> <phrase> - Translates <phrase> from <source> into <target>. Both <source> and <target> are optional
    hubot youtube me <query> - Searches YouTube for the query and returns the video embed link.
    ship it - Display a motivation squirrel

당신은 명확하게 하기 위해 bin/hubot `--name` 옵션을 가지고 문자를 입력해서 당신의 휴봇의 이름을 바꾸는 것이 좋습니다:

    % bin/hubot --name myhubot
    myhubot>

당신의 휴봇은 이제 `myhubot`로 응답합니다. 이것은 대소문자를 구별하고, `@`가 이전에 붙거나   `:` 가 뒤에 붙어도 상관 없습니다. 이것들은 동일한 명령어 입니다:

    MYHUBOT help
    myhubot help
    @myhubot help
    myhubot: help

## Scripts

휴봇의 힘은 스크립트에서 옵니다. 수백개의 스크립트들이 커뮤니티에 작성되어 있습니다. 
`hubot-scripts <your-search-term>` 를 위한 것들을 [NPM registry](https://www.npmjs.com/browse/keyword/hubot-scripts)에서 검색합니다. 예를 들어 아래와 같이 github 와 관련된 스크립트를 찾을 수 있습니다:

```
$ npm search hubot-scripts github
NAME                  DESCRIPTION
hubot-deployer        Giving Hubot the ability to deploy GitHub repos to PaaS providers hubot hubot-scripts hubot-gith
hubot-gh-release-pr   A hubot script to create GitHub's PR for release
hubot-github          Giving Hubot the ability to be a vital member of your github organization
…
```

NPM 패키지에서 스크립트를 사용하기 위해서:

1. `npm install --save <package-name>` 을 실행하여 패키지에 종속시키고 이것을 인스톨 합니다.
2. `external-scripts.json`에 설치한 패키지 이름을 추가합니다..
3. 당신은 스크립트를 설정하거나 설치를 위한 추가적인 정보를 찾기 위해  `npm home <package-name>`을 실행하여 스크립트 홈페이지를 열 수 있습니다.

또한 당신은 당신의 스크립트를 `scripts/` 디렉토리 밑에 추가할 수 있습니다. 그곳에 위치한 모든 스크립트는 당신의 휴봇과 함께 사용할 수 있게 자동으로 로드됩니다. 휴봇을 커스터마이징 하기 위하면 [당신의 스크립트를 작성하기](/docs/scripting.md) 를 더 읽어 보십시오.

## Adapters

휴봇은 여러가지 채팅 서버를 지원하기 위해 어댑터 패턴을 사용합니다. [사용 가능한 어댑터 리스트](/docs/adapters.md)에 그것들을 구성하기 위한 자세한 방법이 있습니다.

## Deploying

당신은 휴봇을 공식적으로 지원하는 Heroky에 배포할 수 있습니다.
추가적으로 당신은 UNIX와 비슷한 시스템이나 Windows 에 배포할 수 있습니다.
공식적으로 지원되지 않는 Windows로 배포에 대한 부분을 유의하시기 바랍니다.

* [Deploying Hubot onto Heroku](/docs/deploying/heroku.md)
* [Deploying Hubot onto UNIX](/docs/deploying/unix.md)
* [Deploying Hubot onto Windows](/docs/deploying/windows.md)

## Patterns

사용자 정의 스크립트를 사용하여 신속하게 커스터마이즈 할 수 있습니다.. 당신의 휴봇에게 새로운 기술을 가르칠때 유용한 트릭 [docs/patterns.md](/docs/patterns.md)를 읽어 보십시오.