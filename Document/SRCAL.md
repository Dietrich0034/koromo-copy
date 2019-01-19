# SRCAL Script CDL

## 1. 소개

`Simple Robust CAL Script Crawler Description Language` 이하 `SRCAL-CDL`은 `Koromo Copy`에서 크롤러의 동작을 정의하기 위한 스크립트 언어입니다.
최대한 간단한 방법으로 크롤러를 정의하기위해 언어를 최대한 단순화 시켰습니다.
이 언어를 사용하면 거의 모든 종류의 크롤러의 동작을 정의할 수 있습니다.

## 2. 설계 및 문법

```
EBNF: SRCAL-CDL

script   -> block

comment  -> ##.*?
line     -> comment
          | expr
          | expr comment
          | e

expr     -> func
          | var = variable
          | runnable
          
block    -> [ block ]
         -> line block
         -> e
         
name     -> [_a-zA-Z]\w*
          | $name            ; Inernal functions

number   -> [0-9]+
string   -> "([^\\"]|\\")*"
const    -> number
          | string
         
var      -> name

variable -> var
          | function
          | variable [ number ]
          | const

argument -> variable
          | variable, argument
function -> name ( )
          | name ( argument )

runnable -> loop (var = variable "to" variable) block
          | foreach (var : variable)            block
          | if (variable)                       block
          | if (variable)                       block else block

```

## 3. 스크립트 상호작용

### 3.1. CAL

`cal` 함수는 `SRCAL`의 핵심 함수로 `XPath`가 혼합된 일련의 문자열입니다.
자세한 내용은 `Custom Crawler` 문서를 통해 확인해주세요.

https://github.com/dc-koromo/koromo-copy/blob/master/Document/CustomCrawler.md#2-cal-%EB%AC%B8%EB%B2%95

### 3.2. 상호작용 API

이 API들은 `Koromo Copy`의 스크립트 엔진이 제공하는 상호작용 함수입니다.
이 함수들은 `$~~~`와 같이 `$`가 접두어로 붙은 함수들입니다.

|  이름 | 내용 |
|------|-----|
|$LoadPage(url)|특정 URL을 다운로드하고, 현재 분석하는 문서를 다운로드한 문서로 바꿉니다. 이전 페이지는 저장되지 않기때문에, 필요한 문자열은 미리 저장해두어야 합니다.|
|$AppendImage(url, filename)|다운로드할 이미지와 이미지의 저장 경로를 저장합니다. 다운로드의 대상이 이미지가 아니여도 상관없습니다. Referer은 현재 URL로 지정됩니다. 저장 경로는 사용자설정에 대한 상대주소입니다.|
|$RequestDownload()|모든 요청을 끝내고 저장된 URL을 모두 다운로드합니다.|

필요하다면 새로운 함수가 언제든지 추가될 수 있습니다.

### 3.3. 상호작용 상수

`SRCAL`은 반드시 설정해야하는 상수인 `Attributes`가 있습니다.
이는 스크립트를 식별하기 위한 상수이므로 반드시 설정되어야합니다.

| 이름 | 내용 |
|------|-----|
|$ScriptName|스크립트의 이름입니다.|
|$ScriptVersion|스크립트의 버전입니다.|
|$ScriptAuthor|스크립트의 작성자입니다.|
|$ScriptFolderName|스크립트의 기본 생성 폴더 이름입니다.|
|$ScriptRequestName|다운로드 상태에 표시될 접두사입니다.|
|$URLSpecifier|스크립트 실별용 URL입니다. 이 URL이 포함된 URL이라면 해당 스크립트로 식별됩니다.|
|$UsingDriver|Selenium Driver를 사용할지에 대한 여부입니다.|

다음은 스크립트 엔진이 제공하는. 크롤러를 정의하는데 유용한 상수들입니다.

| 이름 | 내용 |
|------|-----|
|$RequestURL|현재 분석하고있는 URL을 가져옵니다.|
|$Infinity|무한대입니다. loop에서 사용될 경우 예외가 발생할 때 까지 반복합니다.|

필요하다면 새로운 상수가 언제든지 추가될 수 있습니다.

### 3.4. 어노테이션

어노테이션은 반복문을 병렬화하기 위해 사용합니다.

## 4. 스크립트 자동 생성 도구

`Koromo Copy`는 `SRCAL-CDL`의 정형화된 몇 가지 틀을 제공합니다.
제공되는 틀은 다음과 같습니다.

```
1. 한 페이지에서 여러개의 이미지를 다운로드할 경우
2. 한 페이지에서 다른 페이지들을 불러오고, 불러온 페이지에서 여러개의 이미지를 다운로드할 경우
3. 한 페이지에서 다른 페이지들을 불러오고, 불러온 페이지에서 또 다른 하위 페이지들을 불러온 뒤, 불러온 하위 페이지에서 여러개의 이미지를 다운로드할 경우
4. 구글 블로그 (blogspot)의 동의 여부가 필요할 경우

위 네 가지의 크롤링 유형외엔 다른 유형은 제공되지 않습니다.
```

## 5. 예제

### 5.1. 단부루 다운로더

```
##
## Koromo Copy SRCAL Script
##
## Danbooru Downloader
##

##
## Attributes
##
$ScriptName = "danbooru-pages"
$ScriptVersion = "0.1"
$ScriptAuthor = "dc-koromo"
$ScriptFolderName = "danbooru"
$ScriptRequestName = "danbooru"
$URLSpecifier = "https://danbooru.donmai.us/"
$UsingDriver = 0

##
## Procedure
##
request_url = $RequestURL
max_page = $Infinity

loop (i = 1 to max_page) [
    $LoadPage(url(request_url, "&page=", i))
    sub_urls = cal("/html[1]/body[1]/div[1]/div[3]/div[1]/section[1]/div[3]/div[1]/article[{1+i*1}]/a[1], #attr[href], #front[https://danbooru.donmai.us")

    foreach (sub_url : sub_urls) [
        $LoadPage(sub_url)
        if equal(cal("/html[1]/body[1]/div[1]/div[3]/div[1]/section[1]/div[2]/span[1]/a[1], #attr[id]"), "image-resize-link") [
            image_url = cal("/html[1]/body[1]/div[1]/div[3]/div[1]/section[1]/div[2]/span[1]/a[1], #attr[href]")
        ] else [
            image_url = cal("/html[1]/body[1]/div[1]/div[3]/div[1]/section[1]/section[1]/img[1], #attr[src]")
        ]
        file_name = split(image_url, '/')[-1]
        $AppendImage(image_url, file_name)
    ]

    if equal($LatestImagesCount, 0) [ 
        exit loop
    ]
]

$RequestDownload()
```