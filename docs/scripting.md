---
permalink: /docs/scripting/index.html
layout: docs
---

박스 밖에 있는 휴봇은 많은 것을 할 수 없으나, 이것은 확장 가능하고, 스크립팅 가능한 로봇 친구 입니다. [여기에](/docs/#scripts.md)에 커뮤티에서 작성되고 관리되는 수백개의 스크립트가 있으며 당신것을 작성하기도 쉽습니다. You can create a custom script in hubot's `scripts` directory or [create a script package](#creating-a-script-package) for sharing with the community!

## Anatomy of a script

당신의 휴봇을 만들었을때, `scripts` 디렉토리도 같이 생성됩니다. 만약 그곳을 들여다 보았다면 당신은 몇개의 예제 스크립트를 볼 수 있을 것입니다. 

When you created your hubot, the generator also created a `scripts` directory. If you peek around there, you will see some examples of scripts. 이 스크립트들은 아래와 같은 것이 필요합니다:

* 휴봇 스크립트 로드 경로에 있는 스크립트 (`src/scripts` and `scripts` by default)
* `.coffee` or `.js` 확장자를 가진 파일
* export a function

우리에게 export a function 을 아래와 같은 것을 의미합니다:

```coffeescript
module.exports = (robot) ->
  # your code here
```

`robot` 파라미터는 당신의 로봇 친구의 인스턴스입니다. 여기서 우리는 놀라운 무언가를 작성하기 시작할 수 있습니다.

## Hearing and responding

채팅 봇일때부터 대부분의 공통적인 상호작용들은 메시지를 통해서 이루어 졌습니다. 휴봇은 대화방 안에 있는 메시지를 듣거나 직접적으로 응답 할 수 있습니다.
정규식과 파라미터로 넘어온 콜백 함수를 사용합니다. 예를 들면:

```coffeescript
module.exports = (robot) ->
  robot.hear /badger/i, (res) ->
    # your code here

  robot.respond /open the pod bay doors/i, (res) ->
    # your code here
```

이 `robot.hear /badger/` 는 언제든지 매칭된 문자가 있으면 콜백을 실행합니다. 예를 들어:

* Stop badgering the witness
* badger me
* what exactly is a badger anyways

이 `robot.respond /open the pod bay doors/i` 콜백은 오직 로봇의 이름이나 별명이 앞에 있는 메시지에 실행됩니다. 만약 로봇의 이름이 HAL이고  alias 가 / 라면, 이 콜백이 실행될 것입니다:

*  hal open the pod bay doors
* HAL: open the pod bay doors
* @HAL open the pod bay doors
* /open the pod bay doors

아래와 같은 경우 실행되지 않습니다:

* HAL: please open the pod bay doors
	* `respond`은 로봇의 이름 바로 텍스트가 따라 와야 합니다.
*  has anyone ever mentioned how lovely you are when you open the pod bay doors?
	* 로봇의 이름이 없어서 실행되지 않습니다.

## Send & reply

이 `res` 파라미터는 `Response` 의 인스턴스입니다(역사적으로, 이 파라미터는 `msg`였고 다른 스크립트는 이 방법으로 사용이 불가능합니다.). 이것과 함께 당신은 `res`가 넘어온 방에 메시지를 `send` 하거나, `이모티콘` 을 방에 보내거나(만약 어댑터가 이 기능을 지원한다면), 사람에게 `대답` 할 수 있습니다. 예를 들어:

```coffeescript
module.exports = (robot) ->
  robot.hear /badger/i, (res) ->
    res.send "Badgers? BADGERS? WE DON'T NEED NO STINKIN BADGERS"

  robot.respond /open the pod bay doors/i, (res) ->
    res.reply "I'm afraid I can't let you do that."

  robot.hear /I like pie/i, (res) ->
    res.emote "makes a freshly baked pie"
```

이 `robot.hear /badgers/` 콜백은 누가 이것을 말했는지 상관없이 "Badgers? BADGERS? WE DON'T NEED NO STINKIN BADGERS" 라는 메시지를 보냅니다.

만약 Dave 라는 사용자가 "HAL: open the pod bay doors" 라고 말했다면 `robot.respond /open the pod bay doors/i` 콜백은 "Dave: I'm afraid I can't let you do that." 라는 메시지를 보냅니다.

## Capturing data

지금까지 우리는 심심하거나 지루할때 사용할만한 정해진 응답을 하는 스크립트를 만들었습니다. `res.match` 는 수신된 메시지를 정규식에 대한 `매칭`된 결과를 가지고 있습니다. 이것은 단지 [JavaScript thing](http://www.w3schools.com/jsref/jsref_match.asp)인데, 정규식에 의해 전체 텍스트가 일치하는 배열을 가지고 있습니다.
만약 그룹을 캡쳐해서 포함한다면, 일반적으로 `res.match` 를 사용합니다. 예를 들어, 우리가 스크립트를 아래와 같이 수정한다면:

```coffeescript
  robot.respond /open the (.*) doors/i, (res) ->
    # your code here
```

만약 Dave 가 "HAL: open the pod bay doors" 라고 말한다면, `res.match[0]` 에는 "open the pod bay doors"가 있고, `res.match[1]` 에는 단지 "pod bay" 만 있습니다. 이제 우리는 좀 더 다이나믹한 것을 시작 할 수 있습니다:

```coffeescript
  robot.respond /open the (.*) doors/i, (res) ->
    doorType = res.match[1]
    if doorType is "pod bay"
      res.reply "I'm afraid I can't let you do that."
    else
      res.reply "Opening #{doorType} doors"
```

## Making HTTP calls

휴봇은 당신에게 이익이 되는 것을 통합하거나 서드파티 API 사용하는 HTTP 톨을 만들 수 있습니다. 이것은 `robot.http`에서 사용 가능한 [node-scoped-http-client](https://github.com/technoweenie/node-scoped-http-client) 인스턴스를 통해 할 수 있습니다. 간단한 케이스는 아래와 같습니다.:

```coffeescript
  robot.http("https://midnight-train")
    .get() (err, res, body) ->
      # your code here
```

post 방식은 아래와 같습니다:

```coffeescript
  data = JSON.stringify({
    foo: 'bar'
  })
  robot.http("https://midnight-train")
    .header('Content-Type', 'application/json')
    .post(data) (err, res, body) ->
      # your code here
```

`err`는 처리도중 에러가 발견될 때 발생합니다. 당신이 일반적으로 이것을 체크하고 핸들링하기 원한다면 아래와 같이 따라합니다.

```coffeescript
  robot.http("https://midnight-train")
    .get() (err, res, body) ->
      if err
        res.send "Encountered an error :( #{err}"
        return
      # your code here, knowing it was successful
```

`res` 는  [http.ServerResponse](http://nodejs.org/api/http.html#http_class_http_serverresponse) 의 인스턴스 입니다. node-scoped-http-client를 사용할 때 대부분의 경우 문제가 발생하지 않지만, `statusCode` 와 `getHeader` 에 관심가져야 합니다. HTTP 상태 코드를 체크하기 위해 `statusCode` 를 사용했는데 보통 200이 아니라면 무언가 나쁜 일이 발생한 것입니다. header 정보를 가져오기 위해 `getHeader` 사용할 수 있는데, 예를 들면 속도 제한을 체크한다면:

```coffeescript
  robot.http("https://midnight-train")
    .get() (err, res, body) ->
      # pretend there's error checking code here

      if res.statusCode isnt 200
        res.send "Request didn't come back HTTP 200 :("
        return

      rateLimitRemaining = parseInt res.getHeader('X-RateLimit-Limit') if res.getHeader('X-RateLimit-Limit')
      if rateLimitRemaining and rateLimitRemaining < 1
        res.send "Rate Limit hit, stop believing for awhile"

      # rest of your code
```

`body`는 문자로된 응답된 body 이지만, 당신이 더 관심있는 것은:

```coffeescript
  robot.http("https://midnight-train")
    .get() (err, res, body) ->
      # error checking code here

      res.send "Got back #{body}"
```

### JSON

만약 당신이 APIs 에 말한다면, 가장 쉬운방법은 JSON 을 이용하는 것인데 그 이유는 어떤 추가적인 디펜던시가 필요 없기 때문입니다. `robot.http`을 만들어 호출 할때, 당신은 보통 당신이 돌려주기를 기대하는 것에 대한 실마리를 `Accept` 헤더에 API로 전달하기 위한 셋팅을 해야 합니다. 한번 `body`를 받으면, 너는 이것을 `JSON.parse`로 구문 분석을 할 수 있습니다.

```coffeescript
  robot.http("https://midnight-train")
    .header('Accept', 'application/json')
    .get() (err, res, body) ->
      # error checking code here

      data = JSON.parse body
      res.send "#{data.passenger} taking midnight train going #{data.destination}"
```

이것은 API 가 오류가 발생하거나 JSON 이 아닌 일반 HTML 을 렌더하려고 시도할때 JSON 이 아닌 것으로 받을 가능성이 있습니다. 안전을 위해 당신은 `Content-Type`을 체크해야만 하고, 구문 분석하는 도중 어떤 에러라도 캐치해야 합니다.

```coffeescript
  robot.http("https://midnight-train")
    .header('Accept', 'application/json')
    .get() (err, res, body) ->
      # err & response status checking code here

      if response.getHeader('Content-Type') isnt 'application/json'
        res.send "Didn't get back JSON :("
        return

      data = null
      try
        data = JSON.parse body
      catch error
       res.send "Ran into an error parsing JSON :("
       return

      # your code here
```

### XML

XML APIs 는 어려운데 그 이유는 XML 구문 분석 라이브러리가 번들로 제공하지 않기 때문입니다. 이것을 자세히 보기에는 이 문서의 범위를 벗어나지만, 여기에 몇가지 라이브러리가 있으니 확인해 보세요:

* [xml2json](https://github.com/buglabs/node-xml2json) (simplest to use, but has some limitations)
* [jsdom](https://github.com/tmpvar/jsdom) (JavaScript implementation of the W3C DOM)
* [xml2js](https://github.com/Leonidas-from-XIV/node-xml2js)

### Screen scraping

API 가아닌 경우에는, 언제든지 스크린 스크랩핑이 가능합니다. 이것을 자세히 보기에는 이 문서의 범위를 벗어나지만, 여기에 몇가지 라이브러리가 있으니 확인해 보세요:

* [cheerio](https://github.com/MatthewMueller/cheerio) (familiar syntax and API to jQuery)
* [jsdom](https://github.com/tmpvar/jsdom) (JavaScript implementation of the W3C DOM)


### Advanced HTTP and HTTPS settings

이미 언급한 것 처럼, 휴봇은 HTTP와 HTTP 요청을 만드는 것을 [node-scoped-http-client](https://github.com/technoweenie/node-scoped-http-client) 을 이용해서 간단한 인터페이스를 제공합니다. 그 안에는 노드의 [http](http://nodejs.org/api/http.html) 와 [https](http://nodejs.org/api/https.html) 라이브러리가 포함하고 있지만 일반적인 종류의 인터렉션 구현을 위한 쉬운 DSL 을 제공하고 있습니다. 

만약 당신이 http와 https 의 옵션을 좀 더 직접적으로 제어하는 것이 필요하다면, 당신은 `robot.http`에 node-scoped-http-client에 전달되는 두번째 인수로 전달하면 이것은 http나 https 에 전달되게 될 것입니다:

```
  options =
    # don't verify server certificate against a CA, SCARY!
    rejectUnauthorized: false
  robot.http("https://midnight-train", options)
```

추가적으로, 만약 node-scoped-http-client 가 당신에 맞게 제대로 작동하지 않는다면 당신은 [http](http://nodejs.org/api/http.html) 와 [https](http://nodejs.org/api/https.html) 을 스스로 직접 사용하거나, [request](https://github.com/request/request)와 같은 어떤 다른 노드 라이브러리를 사용 할 수 있습니다.

## Random

일반적인 패턴은 명령어에 듣거나 응답하고, 랜덤한 재미있는 이미지를 보내거나 사용 가능한 배열에서 텍스트를 라인에 보낼 수 있습니다. 자바스크립트와 커피스크립트로 할 수 있지만 성가신 것을 휴봇에서는 편리한 메소드로 포함하고 있습니다.

```coffeescript
lulz = ['lol', 'rofl', 'lmao']

res.send res.random lulz
```

## Topic

휴봇은 어댑터가 지원을 한다면 토픽이 변경되는 것에 반응 할 수 있습니다.

```coffeescript
module.exports = (robot) ->
  robot.topic (res) ->
    res.send "#{res.message.text}? That's a Paddlin'"
```

## Entering and leaving

휴봇은 사용자가 들어오거나, 나가는 것을 어댑터가 지원한다면 볼 수 있습니다.

```coffeescript
enterReplies = ['Hi', 'Target Acquired', 'Firing', 'Hello friend.', 'Gotcha', 'I see you']
leaveReplies = ['Are you still there?', 'Target lost', 'Searching']

module.exports = (robot) ->
  robot.enter (res) ->
    res.send res.random enterReplies
  robot.leave (res) ->
    res.send res.random leaveReplies
```

## Environment variables

휴봇은 그가 실행되는 동안 다른 노드 프로그램과 같이 [`process.env`](http://nodejs.org/api/process.html#process_process_env) 를 이용해서 환경에 접근 할 수 있습니다. 이것은 `HUBOT_` 프리픽스로 시작하는 컨벤션과 함께 어떻게 스크립트를 실행할지 설정할 수 있습니다.

```coffeescript
answer = process.env.HUBOT_ANSWER_TO_THE_ULTIMATE_QUESTION_OF_LIFE_THE_UNIVERSE_AND_EVERYTHING

module.exports = (robot) ->
  robot.respond /what is the answer to the ultimate question of life/, (res) ->
    res.send "#{answer}, but what is the question?"
```

만약 이것이 정의되지 않았을때 휴봇 개발자에게 어떻게 정의되어야 하는지 주거나, 기본적인 어떤 것을 실행하도록 만들 수 있습니다. 만약 치명적인 에러(e.g. 휴봇 종료)가 발생했을 경우, 또는 만들고 있는 스크립트가 어떤 설정이 필요한지 말해주거나 하는 것은 스크립트 작성자에게 달려 있습니다. 가능하다면 어떠한 다른 설정 없이 작동하는 스크립트를 만드는 것을 선호합니다.

여기 우리가 기본적인 무언가를 하는 것이 있습니다:

```coffeescript
answer = process.env.HUBOT_ANSWER_TO_THE_ULTIMATE_QUESTION_OF_LIFE_THE_UNIVERSE_AND_EVERYTHING or 42

module.exports = (robot) ->
  robot.respond /what is the answer to the ultimate question of life/, (res) ->
    res.send "#{answer}, but what is the question?"
```

여기 만약 정의되지 않았다면 우리가 종료하는 것이 있습니다:

```coffeescript
answer = process.env.HUBOT_ANSWER_TO_THE_ULTIMATE_QUESTION_OF_LIFE_THE_UNIVERSE_AND_EVERYTHING
unless answer?
  console.log "Missing HUBOT_ANSWER_TO_THE_ULTIMATE_QUESTION_OF_LIFE_THE_UNIVERSE_AND_EVERYTHING in environment: please set and try again"
  process.exit(1)

module.exports = (robot) ->
  robot.respond /what is the answer to the ultimate question of life/, (res) ->
    res.send "#{answer}, but what is the question?"
```

그리고 마지막으로, 우리가 `robot.respond` 를 업데이트하는 것을 확인해 보세요:

```coffeescript
answer = process.env.HUBOT_ANSWER_TO_THE_ULTIMATE_QUESTION_OF_LIFE_THE_UNIVERSE_AND_EVERYTHING

module.exports = (robot) ->
  robot.respond /what is the answer to the ultimate question of life/, (res) ->
    unless answer?
      res.send "Missing HUBOT_ANSWER_TO_THE_ULTIMATE_QUESTION_OF_LIFE_THE_UNIVERSE_AND_EVERYTHING in environment: please set and try again"
      return
    res.send "#{answer}, but what is the question?"
```

## Dependencies

휴봇은 [npm](https://github.com/isaacs/npm)을 사용해서 이것의 의존성을 관리합니다. 추가적인 패키지를 추가하려면 `package.json`안에 `dependencies` 에 그것들을 추가하면 됩니다. 예를 들어 lolimadeupthispackage 1.2.3 를 추가 하려면 다음과 같습니다:

```json
  "dependencies": {
    "hubot":         "2.5.5",
    "lolimadeupthispackage": "1.2.3"
  },
```

만약 당신이 hubot-scripts 에서 스크립트를 사용한다면, `Dependencies` 문서에서 스크립트 설명서를 살펴보세요. 그것들은 `package.json` 에 복사해서 붙여넣을 수 있게 정리되어 있으나, 단지 유효한 JSON 형태로 콤마로 구분되어 있는지 확인이 필요합니다.

# Timeouts and Intervals

휴봇은 자바스크립트에 포함된 [setTimeout](http://nodejs.org/api/timers.html#timers_settimeout_callback_delay_arg) 를 이용해서 이후에 실행 할 수 있습니다.
이 콜백 메소드를 보면, 호출 되기 전에 일정 시간을 기다립니다:

```coffeescript
module.exports = (robot) ->
  robot.respond /you are a little slow/, (res) ->
    setTimeout () ->
      res.send "Who you calling 'slow'?"
    , 60 * 1000
```

추가적으로, 휴봇은  [setInterval](http://nodejs.org/api/timers.html#timers_setinterval_callback_delay_arg)를 이용해서 코드를 반복 할 수 있습니다.
이 콜백 메소드를 보면, 일정시간 마다 호출 됩니다:

```coffeescript
module.exports = (robot) ->
  robot.respond /annoy me/, (res) ->
    res.send "Hey, want to hear the most annoying sound in the world?"
    setInterval () ->
      res.send "AAAAAAAAAAAEEEEEEEEEEEEEEEEEEEEEEEEIIIIIIIIHHHHHHHHHH"
    , 1000
```

`setTimeout`와 `setInterval` 모두 timeout 또는 interval 이 만들어진 ID 를 리턴합니다. 이것은 `clearTimeout`과 `clearInterval` 에 사용 할 수 있습니다.

```coffeescript
module.exports = (robot) ->
  annoyIntervalId = null

  robot.respond /annoy me/, (res) ->
    if annoyIntervalId
      res.send "AAAAAAAAAAAEEEEEEEEEEEEEEEEEEEEEEEEIIIIIIIIHHHHHHHHHH"
      return

    res.send "Hey, want to hear the most annoying sound in the world?"
    annoyIntervalId = setInterval () ->
      res.send "AAAAAAAAAAAEEEEEEEEEEEEEEEEEEEEEEEEIIIIIIIIHHHHHHHHHH"
    , 1000

  robot.respond /unannoy me/, (res) ->
    if annoyIntervalId
      res.send "GUYS, GUYS, GUYS!"
      clearInterval(annoyIntervalId) ->
      annoyIntervalId = null
    else
      res.send "Not annoying you right now, am I?"
```

## HTTP Listener

휴봇은 HTTP 요청을 지원하기 위해 [express](http://expressjs.com) 웹 프레임워크를 포함하고 있습니다. 이것은 `EXPRESS_PORT` 나 `PORT` 환경 변수 (preferred in that order) 로 지정되어 있는 특정 포트위를 listen 하고 있고 기본 포트는 8080 입니다. express 응용 프로그램의 인스턴스는 `robot.router` 로 사용 가능 합니다. 이것은 `EXPRESS_USER` and `EXPRESS_PASSWORD` 로 지정된 username 과 password 로 보호될 수 있습니다. 이것은 자동으로 `EXPRESS_STATIC` 로 셋팅된 파일을 서비스 할 수 있습니다.

이것은 webhooks 에 밀어넣고, 채팅에 표시하기 위한 서비스를 위해 HTTP 엔드 포인트를 제공하는 용도로 가장 많이 사용합니다.

```coffeescript
module.exports = (robot) ->
  robot.router.post '/hubot/chatsecrets/:room', (req, res) ->
    room   = req.params.room
    data   = if req.body.payload? then JSON.parse req.body.payload else req.body
    secret = data.secret

    robot.messageRoom room, "I have a secret: #{secret}"

    res.send 'OK'
```

curl 로 테스트를 해보면; 또한 [error handling](#error-handling) 이하 부분을 보세요.

```shell
// raw json, must specify Content-Type: application/json
curl -X POST -H "Content-Type: application/json" -d '{"secret":"C-TECH Astronomy"}' http://127.0.0.1:8080/hubot/chatsecrets/general

// defaults Content-Type: application/x-www-form-urlencoded, must st payload=...
curl -d 'payload=%7B%22secret%22%3A%22C-TECH+Astronomy%22%7D' http://127.0.0.1:8080/hubot/chatsecrets/general
```

모든 엔트포인트 URL들은 `/hubot`와 동일한 문자로 시작해야 합니다(당신의 로봇의 이름과 관계 없이). 이러한 일관성은 URL이 (모든 봇 이름, URL 안전에) 유효하다는 것을 보증하고, 쉽게 webhooks (URL 을 복사 붙여넣어서) 을 설정할 수 있습니다.

## Events

휴봇은 스크립트간에 데이터를 주고받기 위한 이벤트에도 응답할 수 있습니다. 이것은 `robot.emit` 와 `robot.on` 함께 node.js [EventEmitter](http://nodejs.org/api/events.html#events_class_events_eventemitter) 로 캡슐화 되어 있습니다.

하나의 사용 예로 서비스와 상호작용을 처리한다음 그들이 온 이벤트에 다시 발송하는 하나의 스크립트를 가질 것입니다. 예를 들어, 우리는 GitHub의 post-commit hook 데이터가 올때 커밋 데이터를 방출하고, 그 다음에 다른 스크립트에서 행동을 취하는 스크립트가 있습니다.

```coffeescript
# src/scripts/github-commits.coffee
module.exports = (robot) ->
  robot.router.post "/hubot/gh-commits", (req, res) ->
    robot.emit "commit", {
        user    : {}, #hubot user object
        repo    : 'https://github.com/github/hubot',
        hash  : '2e1951c089bd865839328592ff673d2f08153643'
    }
```

```coffeescript
# src/scripts/heroku.coffee
module.exports = (robot) ->
  robot.on "commit", (commit) ->
    robot.send commit.user, "Will now deploy #{commit.hash} from #{commit.repo}!"
    #deploy code goes here
```

당신이 이벤트를 제공한다면, 휴봇 사용자나 룸 객체를 데이터에 포함시키는 것이 좋습니다. 
이것은 휴봇에게 대화방 내의 사용자나 대화방에 알림을 하는 것을 허용해 줍니다.

## Error Handling

어떤 코드든 완벽하지 않고, 에러와, 예외 상황이 발생하는 것을 예상할 수 있습니다. 이전에 처리되지 않은 예외 상황은 당신의 휴봇 인스턴스를 강제 종료 될 것입니다. 휴봇은 현재 어떤 스크립트의 예외상황에 대해 무언가 액션을 취하기 위한 `uncaughException` 핸들러를 포함하고 있습니다.

```coffeescript
# src/scripts/does-not-compute.coffee
module.exports = (robot) ->
  robot.error (err, res) ->
    robot.logger.error "DOES NOT COMPUTE"

    if res?
      res.reply "DOES NOT COMPUTE"
```

당신은 여기에 당신이 원하는 어떤것이든 할 수 있습니다. 그러나 당신이 별도의 주의사항과 에러 로깅, 특별한 처리를 원한다면 비동기 코드로 작성해야 합니다.
그렇지 않으면, 당신은 스스로 재귀오류와 알지 못하는 무언가를 발견하게 될 것입니다.

숨겨진 곳에 'error' 이벤트를 방출되고 에러 핸들러와 함께 이벤트를 소비하고 있습니다. 그 uncaughtException 핸들러는 [technically leaves the process in an unknown state](http://nodejs.org/api/process.html#process_event_uncaughtexception). 따라서 당신은 언제어디서든지 당신의 예외사항을 구해서 그 스스로 이벤트를 방출 해야 합니다. 그 첫번째 인자는 그 에러를 방출하고, 두번째 인자는 에러가 생성될때 추가적인 메시지입니다.

앞의 예시를 보면:

```coffeescript
  robot.router.post '/hubot/chatsecrets/:room', (req, res) ->
    room = req.params.room
    data = null
    try
      data = JSON.parse req.body.payload
    catch err
      robot.emit 'error', err

    # rest of the code here


  robot.hear /midnight train/i, (res)
    robot.http("https://midnight-train")
      .get() (err, res, body) ->
        if err
          res.reply "Had problems taking the midnight train"
          robot.emit 'error', err, res
          return
        # rest of code here
```

두번째 예제는, 가치가 있다고 생각되는 메시지를 사용자가 볼 것 입니다. 만약 당신이 사용자에게 답변을 하는 에러 핸들러를 가지고 있다면, 당신은 별도의 메시지를 추가할 필요 없이 `get()` 요청에서 제공하고 있는 에러 메시지를 그대로 되돌려 보낼 수 있습니다. 하지만 당신이 당신의 에러 리포트를 어떻게 공개하기를 원하는지에 달려있습니다. 

## Documenting Scripts

휴봇 스크립트는 그 파일들의 맨 위에 위치한 주석으로 문서화 할 수 있습니다. 예를 들면:

```coffeescript
# Description:
#   <description of the scripts functionality>
#
# Dependencies:
#   "<module name>": "<module version>"
#
# Configuration:
#   LIST_OF_ENV_VARS_TO_SET
#
# Commands:
#   hubot <trigger> - <what the respond trigger does>
#   <trigger> - <what the hear trigger does>
#
# Notes:
#   <optional notes required for the script>
#
# Author:
#   <github username of the original script author>
```

가장 중요하고 사용자가 직면하게 될 것은 `Commands`입니다. 로드 시점에, 휴봇은 각 스크립트의 `Commands` 부분을 찾고 모든 명령어를 생성합니다. 포함되어 있는 `help.coffee`는 사용자가 명령어를 통해 질문하거나 검색 할 수 있습니다. 따라서 문서화 하는 것은 명령어를 만들때 사용자로 하여 좀 더 많이 검색 될 수 있도록 합니다.

명령어가 문서화 될때, 여기에 좋은 사례가 있습니다:

* 한줄로 유지하라. Help 명령어는 정렬하기 때문에, 예기치 못하게 두번째 라인에 추가된 예외사항을 이해할 수 없을 것 입니다.
* 당신의 휴봇이 다른 이름을 가지고 있더라도 Hubot을 이용해 참조하십시오. 이것은 자동으로 일치하는 이름으로 변경 될 것입니다. 이것은 업데이트된 문서 없이 스크립트를 공유하기 쉽게 만들어 줍니다.
* `robot.respond` 문서는 항상 `hubot` 접두사와 함께 하십시오. 휴봇은 자동으로 당신의 로봇 이름이나 로봇의 별명을 가지고 있다면 그것으로 변경할 것입니다.
* man 페이지들을 문서화를 어떻게 하는지 확인 하세요. 특히, 괄호 표시는 옵션을 표시하고, '...' 는 인자의 어떤 숫자, 기타 등등.

다른 부분은 봇 개발과 관련되고, 특히 디펜던시, 변수 구성 그리고 메모입니다. [hubot-scripts](https://github.com/github/hubot-scripts)에 기여된 모든 스크립트들은 스크립트와 함께 실행하거나 연관된 모든 부분을 포함해야 합니다.


## Persistence

휴봇은 메모리에 저장소로 사용하고, 데이터를 검색 할 수 있는 키-값으로 이루어진 `robot.brain` 을 가지고 있습니다.

```coffeescript
robot.respond /have a soda/i, (res) ->
  # Get number of sodas had (coerced to a number).
  sodasHad = robot.brain.get('totalSodas') * 1 or 0

  if sodasHad > 4
    res.reply "I'm too fizzy.."

  else
    res.reply 'Sure!'

    robot.brain.set 'totalSodas', sodasHad+1
robot.respond /sleep it off/i, (res) ->
  robot.brain.set 'totalSodas', 0
  msg.reply 'zzzzz'
```

만약 스크립트가 사용자 데이터를 찾는 것이 필요하다면, 아이디, 이름, 이름의 부분이 매칭으로 한명 또는 많은 사용자를 찾기 위한 방법이 `robot.brain` 에 있습니다:
`userForName`, `userForId`, `userForFuzzyName`, and `usersForFuzzyName`.

```coffeescript
module.exports = (robot) ->

  robot.respond /who is @?([\w .\-]+)\?*$/i, (res) ->
    name = res.match[1].trim()

    users = robot.brain.usersForFuzzyName(name)
    if users.length is 1
      user = users[0]
      # Do something interesting here..

      res.send "#{name} is user - #{user}"
```

## Script Loading

여기에 스크립트들을 로드하기 위한 세가지 중요한 방법이 있습니다:

* 당신의 휴봇이 설치되어 있는 곳 아래에 `scripts/` 디렉토리 내에 있는 모든 스크립트 __bundled__
* `hubot-scripts.json`과 npm bozlwldp `hubot-scripts` 가 포함된 __community scripts__
* 외부 __npm packages__ 와 `external-scripts.son` 안에 정의되어 있는 스크립트들

스크립트들은 `scripts/` 디렉토리에서 알파벳 순서대로 로드되기 때문에, 당신은 일관된 순서로 로드되길 기대할 수도 있습니다. 예를 들면:

* `scripts/1-first.coffee`
* `scripts/_second.coffee`
* `scripts/third.coffee`

# Sharing Scripts

당신의 휴봇의 능력을 확장하는 새로운 스크립트를 만들면, 당신은 그것들을 세계로의 공유를 고려해야 합니다! 최소한, 당신은 당신의 스크립트를 패키징하고 [Node.js Package Registry](http://npmjs.org)에 제출 하는 것이 필요합니다. 당신은 또한 스크립트를 공유하기 위한 최상의 방법을 검토해야 합니다.

## See if a script already exists

Start by [checking if an NPM package](/docs/index.md#scripts) for a script like yours already exists.  If you don't see an existing package that you can contribute to, then you can easily get started using the `hubot` script [yeoman](http://yeoman.io/) generator.

## Creating A Script Package

Creating a script package for hubot is very simple.  Start by installing the `hubot` [yeoman](http://yeoman.io/) generator:


```
% npm install -g yo generator-hubot
```

Once you've got the hubot generator installed, creating a hubot script is similar to creating a new hubot.  You create a directory for your hubot script and generate a new `hubot:script` in it.  For example, if we wanted to create a hubot script called "my-awesome-script":

```
% mkdir hubot-my-awesome-script
% cd hubot-my-awesome-script
% yo hubot:script
```

At this point, the you'll be asked a few questions about the author for the script, name of the script (which is guessed by the directory name), a short description, and keywords to find it (we suggest having at least `hubot, hubot-scripts` in this list).

If you are using git, the generated directory includes a .gitignore, so you can initialize and add everything:

```
% git init
% git add .
% git commit -m "Initial commit"
```

You now have a hubot script repository that's ready to roll! Feel free to crack open the pre-created `src/awesome-script.coffee` file and start building up your script! When you've got it ready, you can publish it to [npmjs](http://npmjs.org) by [following their documentation](https://docs.npmjs.com/getting-started/publishing-npm-packages)!

# Listener Metadata

In addition to a regular expression and callback, the `hear` and `respond` functions also accept an optional options Object which can be used to attach arbitrary metadata to the generated Listener object. This metadata allows for easy extension of your script's behavior without modifying the script package.

The most important and most common metadata key is `id`. Every Listener should be given a unique name (options.id; defaults to `null`). Names should be scoped by module (e.g. 'my-module.my-listener'). These names allow other scripts to directly address individual listeners and extend them with additional functionality like authorization and rate limiting.

Additional extensions may define and handle additional metadata keys.

Returning to an earlier example:

```coffeescript
module.exports = (robot) ->
  robot.respond /annoy me/, id:'annoyance.start', (msg)
    # code to annoy someone

  robot.respond /unannoy me/, id:'annoyance.stop', (msg)
    # code to stop annoying someone
```

These scoped identifiers allow you to externally specify new behaviors like:
- authorization policy: "allow everyone in the `annoyers` group to execute `annoyance.*` commands"
- rate limiting: "only allow executing `annoyance.start` once every 30 minutes"
