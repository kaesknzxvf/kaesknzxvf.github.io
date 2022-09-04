---
# multilingual page pair id, this must pair with translations of this page. (This name must be unique)
#lng_pair: "id_study_flutter"
title: "Flutter 01 Simple Chat UI 응용 02 데이터 로컬 저장 및 읽기, 쓰기 "

# post specific
author: kae
# publish date
date: 2022-09-03 14:08:00 +0900
category: [Flutter, Simple Chat]
tags: [flutter, practice, programming]

# disable comments on this page
#comments_disable: true

# thumbnail image for post
# img: ":Flutter.jpeg"

# data path
img_path: /assets/img/2022-09-03-Flutter-Simple-Chat-UI-EX02/
# image:
  # path: /assets/img/2022-07-10-Flutter-Simple-Chat-UI-01/
  # width: 1000   # in pixels
  # height: 400   # in pixels
  # alt: image alternative text

# if pinned Posts
# pin: true

########################################################################

# seo
# if not specified, date will be used.
#meta_modify_date: 2022-02-10 08:11:06 +0900
# check the meta_common_description in _data/owner/[language].yml
#meta_description: ""

# optional
# if you enabled image_viewer_posts you don't need to enable this. This is only if image_viewer_posts = false
#image_viewer_on: true
# if you enabled image_lazy_loader_posts you don't need to enable this. This is only if image_lazy_loader_posts = false
#image_lazy_loader_on: true
# exclude from on site search
#on_site_search_exclude: true
# exclude from search engines
#search_engine_exclude: true
# to disable this page, simply set published: false or delete this file
#published: false

---

<!-- outline-start -->


<!-- outline-end -->

이 포스트는

- [How to Build a Chat App UI With Flutter and Dart](https://www.freecodecamp.org/news/build-a-chat-app-ui-with-flutter/)

을 따라서 작성한 Flutter Simple Chat UI 을 응용하여 좀 더 그럴듯한 채팅앱을 구현하기 위한 과정을 다룬다.

주요 내용은 이하와 같다.
- Chat Detail 화면의 채팅 데이터 저장 및 읽기, 쓰기
- Chat 화면의 유저 데이터 저장 및 읽기, 쓰기
 
원문의 예제는 단순히 UI만 다루는 포스트이다 보니, 실제로 데이터를 전달하거나 저장하는 등 데이터를 처리하는 작업은 해주지 않는다. 그러므로 이번 포스트부터는 스스로 공부하며 추가한 코드이다.

관련 포스트
- [Base, Chat 화면 구현](https://kaesknzxvf.github.io/posts/Flutter-Simple-Chat-UI-01/)
- [Chat Detail 화면 구현](https://kaesknzxvf.github.io/posts/Flutter-Simple-Chat-UI-02/)
- [응용 01 (Chat Detail 화면) 화면 간 데이터 전달, 메세지 입력](https://kaesknzxvf.github.io/posts/Flutter-Simple-Chat-UI-EX01/)
- 응용 02 데이터 로컬 저장 및 읽기, 쓰기 (현재포스트)

***
# 환경

- MacBook Air (M1, 2020)
- macOS Monterey (v12.4)

- Flutter 3.0.3
- Dart 2.17.5

- Visual Studio Code 1.69.0

***

# Chat Detail 화면의 채팅 데이터
## 데이터 저장 방법 선택 
현재는 메세지을 입력해서 보내도, 채팅창을 한 번 나갔다 들어오면 리셋 되어, 초기 상태로 되돌아 온다.  
그렇기 때문에 내가 보낸 메세지 데이터를 로컬에 저장해서, 채팅창을 나갔다 들어와도 예전의 메세지 로그가 남아 있도록 해 볼 것이다.

- **고려한 방법 1. shared_preferences 플러그인 → 사용×**

  [shared_preferences](https://pub.dev/packages/shared_preferences) 플러그인을 사용하면 [Key-Value 데이터를 로컬 디스크에 저장](https://flutter-ko.dev/docs/cookbook/persistence/key-value)할 수 있는데, 주의할 점이

  - 오직 원시 타입만 사용 가능: int, double, bool, string, stringList
  - 대용량 데이터 저장을 위해 설계되지 않음

  라고 되어 있어서, 지금은 오리지널 클래스인 `ChatMessage`의 리스트를 저장해야하기에 이 플러그 인의 사용은 포기

- **고려한 방법 2. 로컬 파일 → 사용×**

  너무... 원시적이고 데이터 관리가 귀찮으므로 미래를 위해 기각!

- **고려한 방법 3. 로컬 데이터 베이스 → 사용○**

  사용할 수 있는 로컬 데이터 베이스에는 여러 종류가 있는데, 여기에 대해서는 [Top 5 Local Database Solutions for Flutter Development](https://levelup.gitconnected.com/top-5-local-database-solutions-for-flutter-development-6351cd494070)라는 아주 좋은 글이 있었다.

  글에서 소개한 내용을 간단히 요약하면,

  - [SQLite](https://flutter-ko.dev/docs/cookbook/persistence/sqlite)  
    - [sqflite](https://pub.dev/packages/sqflite) 플러그인 : 가장 유명하고 많이 사용하는 플러그인 
    - [Floor](https://pub.dev/packages/floor) 플러그인 : sqflite에 의존, sqflite이 지원하는 모든 것을 지원 + 다른 기능도 추가
    - [Drift (Moor)](https://pub.dev/packages/drift) 플러그인 : sqflite에 의존, querying API 를 포함해 매우 강력한 툴, [공식페이지](https://drift.simonbinder.eu/)
  - [Hive](https://pub.dev/packages/hive)  
    NoSQL, Key-Value 데이터베이스, 속도가 빠름, 암호화 강력  
    이상적인 use case  
    - 소규모
    - 유연한 애플리케이션 : NoSQL 이기 때문에 '테이블'이 아닌 '박스'라는 개념이 사용되며, 어떠한 구조도 될 수 있는 '박스'는 동일한 유형의 개체에 대한 유연한 데이터 열이 필요한 경우 매우 유용
  - [Sembast](https://pub.dev/packages/sembast)  
    NoSQL 데이터베이스, 데이터 관리 방식은 Hive와 비슷하지만, 아직 Hive에 미치지 못하는 기능, 성능을 가짐

  > ※　데이터베이스 서버 구축 or 클라우드 서비스 데이터 베이스 이용 → 추후 수정 가능성 있음  
  >> 진짜로 채팅을 할 수 있도록 만드려면 송수신자가 서로 다른 네트워크에서 접근 할 수 있는 데이터 베이스가 필요할 테니, 앱이 그 수준까지 완성 되면 데이터 관리 방법을 꿔야 할 것이다... 그때 한 번 해보려고 한다. 

앞으로 응용해서 만드려고 하는 채팅 앱의 경우, 다수의 유저를 염두에 두고 만드는 것도 아니며, 오로지 공부 목적이기에 데이터의 규모가 그리 커지지도 않을 것이다. 그리고 NoSQL DB를 한번도 안 써봐서 써보고 싶다ㅎ   
위와 같은 이유로 이번엔 `Hive`를 이용하여 로컬 데이터 베이스를 구축해 보려한다.

## Hive를 사용한 로컬 데이터 베이스 구축 

이번 장은 [【Flutter】Hiveの使い方](https://zenn.dev/naoya_maeda/articles/b67f53af377395)를 참고해서 작성했다. 설명이 아주아주 친절하다.

기록 순서는 이하와 같다.

1. 필요한 패키지 추가
2. 오리지널 클래스 `ChatMessage` 정의
3. `ChatMessage`의 TypeAdapter 클래스 자동 생성
4. (Hive 관계 없음) 코드 정리 : `ChatDetailPage.dart` 에 포함되어있는 메세지 로그 리스트 코드 분리
5. `ChatDetailPage.dart` 메세지 로그 저장

### 패키지 추가

`pubspec.yaml`파일 `dependencies`와 `dev_dependencies`에 필요한 패키지의 최신 버전을 추가한다.
```yaml
dependencies:
  hive: ^2.2.3
  hive_flutter: ^1.1.0
dev_dependencies:
  hive_generator: ^1.1.3
  build_runner: ^2.2.0
```
{: file="/pubspec.yaml" }

> 2022/9/3시점, [hive](https://pub.dev/packages/hive)의 최신 버전은 2.2.3, [hive_flutter](https://pub.dev/packages/hive_flutter)의 최신 버전은 1.1.0, [hive_generator](https://pub.dev/packages/hive_generator)의 최신 버전은 1.1.3, [build_runner](https://pub.dev/packages/build_runner)의 최신 버전은 2.2.0 이다. 최신 버전은 각자 확인하고 변경해서 추가하자.
{: .prompt-warning}

> 개발할 때에만 필요하고, 릴리즈 할 때에는 필요 없는 패키지는 `dev_dependencies`에 포함시킨다. 
{: .prompt-tip}

여기서 (`ChatMessage`같은)오지지널 클래스의 TypeAdapter를 작성하기 위한 `hive_generator`와  `build_runner`는 개발할 때에만 필요하기 떄문에, `dev_dependencies`에 포함시킨 것.

### 오리지널 클래스 정의
저장하고 싶은 것은 메세지 로그이므로 생성해 두었던 `chatMessageModel.dart`를 이하와 같이 수정해 줄 것이다.

```dart
import 'package:flutter/cupertino.dart';
import 'package:hive/hive.dart'; //추가
part 'chatMessageModel.g.dart'; //추가

@HiveType(typeId: 1) //추가
class ChatMessage {
  @HiveField(0) //추가
  String messageContent;

  @HiveField(1) //추가
  String messageType;

  ChatMessage({required this.messageContent, required this.messageType});
}
```
{: file="/lib/models/chatMessageModel.dart" }

추가해준 코드의 설명과 추가한 이유는 이하와 같다.

- part 'chatUsersModel.g.dart';  
  `part`는 `part of`와 한쌍으로 프로그램을 분할 할 때 사용하는 구문으로, 파일에 선언해주면 (이번의 경우) `chatUsersModel.dart` 와 `chatUsersModel.g.dart` 는 하나의 파일에 정의한 것과 같이 취급된다. `chatUsersModel.g.dart`파일은 TypeAdapter 클래스가 정의된 파일로, 아직 생성해 주지 않았으므로 빌드에러가 날 것이다. (바로 다음 단계에서 생성해 줄 것)   
  ※ 추후 생성할 `chatUsersModel.g.dart`파일을 확인하면 `part of 'chatUsersModel.dart'` 가 선언되어 있는 것을 확인 할 수 있다.
- @HiveType(typeId:`0~255의 숫자`)  
  Hive가 TypeAdapter 생성시, 각 클래스들을 식별 할 수 있도록 지정해 주는 것이며, 들어갈 수 있는 숫자 범위는 `0~255`이다.
- @HiveField(`0~255의 숫자`)  
  Hive가 TypeAdapter 생성시, 각 필드들을 식별 할 수 있도록 지정해 주는 것이며, 들어갈 수 있는 숫자 범위는 `0~255`이다.

### TypeAdapter 클래스 자동 생성

위에서 입력해 둔
```dart
part 'chatMessageModel.g.dart'; 
```
{: file="/lib/models/chatMessageModel.dart" }

의 `chatMessageModel.g.dart`를 생성하는 작업을 해줄 것이다.

> 반드시 생성하고자 하는 클래스에 `part '해당클래스명.g.dart';` 를 먼저 추가해준 후 진행해야한다! 그렇지 않으면 생성되지 않음
{: .prompt-warning}

VSCode 터미널(현재의 flutter 프로젝트 경로)에서 이하와 같은 커멘드를 입력

```bash
flutter packages pub run build_runner build
```
그러면
```plaintext
[INFO] Generating build script...
[INFO] Generating build script completed, took 181ms

[INFO] Initializing inputs
[INFO] Reading cached asset graph...
[INFO] Reading cached asset graph completed, took 31ms

[INFO] Checking for updates since last build...
[INFO] Checking for updates since last build completed, took 428ms

[INFO] Running build...
[INFO] 1.0s elapsed, 0/1 actions completed.
[INFO] 4.0s elapsed, 0/1 actions completed.
[INFO] Running build completed, took 4.3s

[INFO] Caching finalized dependency graph...
[INFO] Caching finalized dependency graph completed, took 19ms

[INFO] Succeeded after 4.3s with 2 outputs (2 actions)
```
이런 로그가 출력되면서, `chatMessageModel.dart`와 같은 폴더에 `chatMessageModel.g.dart`이 생성된 것을 확인할 수 있다.

![Flutter Simple chat UI ex 12](SimulSc_ex_12.png){:width="50%" height="50%"}


### 코드 정리 : 메세지 로그 리스트뷰 코드 분리

지금 `ChatDetailPage.dart` 화면에 UI랑 데이터 처리 코드가 섞여 너무 정신사나워서, 메세지 로그를 표시하는 리스트뷰 부분 코드를 조금 정리했다.

`chatPage.dart`의 유저 목록을 표시하는 리스트뷰 `conversationList.dart`를 분리한 것 처럼, `ChatDetailPage.dart`도 body에 메세지로그를 표시하는 Listview를 `messageList.dart` 라는 widget으로 분리했다.

```dart
import 'package:flutter/material.dart';

class MessageList extends StatefulWidget {
  String messageContent;
  String messageType;
  MessageList({required this.messageContent, required this.messageType});
  @override
  _MessageListState createState() => _MessageListState();
}

class _MessageListState extends State<MessageList> {
  @override
  Widget build(BuildContext context) {
    return Container(
      padding: EdgeInsets.only(left: 16, right: 16, top: 10, bottom: 10),
      child: Align(
        // messages[index].messageType -> widget.messageType
        alignment: (widget.messageType == "receiver"
            ? Alignment.topLeft
            : Alignment.topRight),
        child: Container(
          decoration: BoxDecoration(
            borderRadius: BorderRadius.circular(20),
            // messages[index].messageType -> widget.messageType
            color: (widget.messageType == "receiver"
                ? Colors.grey.shade200
                : Colors.blue[200]),
          ),
          padding: EdgeInsets.all(16),
          // messages[index].messageType -> widget.messageType
          margin: (widget.messageType == "receiver"
              ? EdgeInsets.only(left: 0, right: 64, top: 0, bottom: 0)
              : EdgeInsets.only(left: 64, right: 0, top: 0, bottom: 0)),
          child: Text(
            // messages[index].messageContent -> widget.messageContent
            widget.messageContent, 
            style: TextStyle(fontSize: 15),
          ),
        ),
      ),
    );
  }
}
```
{: file="/lib/widgets/messageList.dart" }

기존 파일에는  해당 부분 삭제하고 인수 전달만...

```dart
//추가
import 'package:flt_simple_chat_ex/widgets/messageList.dart';

...

      body: Column(
        children: <Widget>[
          Flexible(
            fit: FlexFit.tight, 
            child: SingleChildScrollView(
              physics: BouncingScrollPhysics(), 
              controller: _scrollController, 
              child: ListView.builder(
                itemCount: messages.length,
                shrinkWrap: true,
                padding: EdgeInsets.only(
                    top: 10), 
                physics: NeverScrollableScrollPhysics(),
                itemBuilder: (context, index) {
                  // 수정, 인수로 전달
                  return MessageList(
                    messageContent: messages[index].messageContent,
                    messageType: messages[index].messageType,
                  );
                },
              ),
            ),
          ),

...
```
{: file="/lib/models/chatMessageModel.dart" }

### 메세지 로그 저장
사용할 클래스도 만들었고 코드도 정리했으니, 이제 본격적으로 메세지를 저장해보자.

사용하고자 하는 파일에 패키지 임포트
```dart
import 'package:hive_flutter/hive_flutter.dart';
```
{: file="/lib/screen/ChatDetailPage.dart" }

이제 `_ChatDetailPageState` 클래스에서 박스를 만들어 열어주자!

```dart
```
{: file="/lib/screen/ChatDetailPage.dart" }

Dart의 `late`선언은 반드시 값이 필요한 (null이면 안되는, non-nullable) 변수지만, 선언과 동시에 초기화를 하지 않을 때 사용한다. 변수의 초기화를 뒤로 밀어줄 수 있는 것.

# Chat 화면 유저 데이터
## 각 유저의 최근 메세지 표시
이렇게 채팅 데이터를 저장해 주면, Chat 화면에 출력 되는 각 유저의 최근메세지를 갱신해 줄 수 있게 된다! (현재는 임의의 데이터로 되어있다)


## 채팅 대상 유저 추가, 삭제







***

# 참고하기 좋은 사이트
- [How to save data locally in Flutter](https://pusher.com/tutorials/local-data-flutter/)  
  위에서 언급한 로컬에 데이터 저장할 때 고려한 방법 3개(①shared_preferences ②로컬 파일 ③로컬 데이터베이스 : sqflite플러그인사용)에 대한 튜토리얼이 있다. 

***
# 후기
