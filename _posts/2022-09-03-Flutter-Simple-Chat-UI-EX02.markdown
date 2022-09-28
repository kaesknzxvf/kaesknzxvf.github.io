---
# multilingual page pair id, this must pair with translations of this page. (This name must be unique)
#lng_pair: "id_study_flutter"
title: "Flutter 01 Simple Chat UI 응용 02 데이터 로컬 저장 및 읽기,쓰기_유저정보편 (+상태창 추가) "

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

주요 내용/순서은 이하와 같다.
- 데이터 저장 환경 구축
  - 데이터 저장 방법 선택 
  - Hive를 사용한 로컬 데이터 베이스 구축 
    - 패키지 추가
    - 오리지널 클래스 정의
    - TypeAdapter 클래스 자동 생성
- `ChatPage` : 유저 추가, 삭제, 저장
  - 코드 정리, UI 수정
  - 채팅 유저 추가 및 저장
- `userDetailPage` : 상태창 생성, 유저 데이터 갱신
  - 상대의 프로필 사진 클릭 시, 상태창 표시
    - 갤러리서 선택한 이미지를 배경 이미지로 설정 

원문의 예제는 단순히 UI만 다루는 포스트이다 보니, 실제로 데이터를 전달하거나 저장하는 등 데이터를 처리하는 작업은 해주지 않는다. 그러므로 이번 포스트는 스스로 공부하며 추가한 코드이다.

관련 포스트
- [Base, Chat 화면 구현](https://kaesknzxvf.github.io/posts/Flutter-Simple-Chat-UI-01/)
- [Chat Detail 화면 구현](https://kaesknzxvf.github.io/posts/Flutter-Simple-Chat-UI-02/)
- [응용 01 (Chat Detail 화면) 화면 간 데이터 전달, 메세지 입력](https://kaesknzxvf.github.io/posts/Flutter-Simple-Chat-UI-EX01/)
- 응용 02 데이터 로컬 저장 및 읽기,쓰기_유저정보편 (+상태창 추가) (현재포스트)
- [응용 03 데이터 로컬 저장 및 읽기,쓰기_메세지로그편](https://kaesknzxvf.github.io/posts/Flutter-Simple-Chat-UI-EX03/)

***
# 환경

- MacBook Air (M1, 2020)
- macOS Monterey (v12.4)

- Flutter 3.0.3
- Dart 2.17.5

- Visual Studio Code 1.69.0

***
# 데이터 저장 환경 구축
## 데이터 저장 방법 선택 
현재는 메세지을 입력해서 보내도, 채팅창을 한 번 나갔다 들어오면 리셋 되어, 초기 상태로 되돌아 온다.  
그렇기 때문에 내가 보낸 메세지 데이터를 로컬에 저장해서, 채팅창을 나갔다 들어와도 예전의 메세지 로그가 남아 있도록 해 볼 것이다.

- **고려한 방법 1. shared_preferences 플러그인 → 사용×**

  [shared_preferences](https://pub.dev/packages/shared_preferences) 플러그인을 사용하면 [Key-Value 데이터를 로컬 디스크에 저장](https://flutter-ko.dev/docs/cookbook/persistence/key-value)할 수 있는데, 주의할 점이

  - 오직 원시 타입만 사용 가능: int, double, bool, string, stringList
  - 대용량 데이터 저장을 위해 설계되지 않음

  라고 되어 있어서, 지금은 오리지널 클래스인 `ChatUsers`과 `ChatMessage`의 리스트를 저장해야하기에 이 플러그 인의 사용은 포기

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

이 앱에서는`ChatUsers`과 `ChatMessage`의 저장소를 만들어 줄건데 (설명은`ChatUsers`로 작성), 순서는 이하와 같다.

1. 필요한 패키지 추가
2. 오리지널 클래스 `ChatUsers` 정의
3. `ChatUsers`의 TypeAdapter 클래스 자동 생성

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

여기서 (`ChatUsers`같은)오지지널 클래스의 TypeAdapter를 작성하기 위한 `hive_generator`와  `build_runner`는 개발할 때에만 필요하기 떄문에, `dev_dependencies`에 포함시킨 것.

### 오리지널 클래스 정의
저장하고 싶은 것은 메세지 로그이므로 생성해 두었던 `chatUsersModel.dart`를 이하와 같이 수정해 줄 것이다.

```dart
import 'package:flutter/cupertino.dart';
import 'package:hive/hive.dart'; //추가
import 'package:uuid/uuid.dart'; //추가

part 'chatUsersModel.g.dart'; //추가

var uuid = Uuid();

@HiveType(typeId: 1) //추가
class ChatUsers {
  @HiveField(0) //추가
  String id; //추가
  @HiveField(1) //추가
  String name;
  @HiveField(2) //추가
  String messageText;
  @HiveField(3) //추가
  String imageURL;
  @HiveField(4) //추가
  String time;

  ChatUsers(
      {String? id,
      required this.name,
      required this.messageText,
      required this.imageURL,
      required this.time})
      : id = id ?? uuid.v4();
}
```
{: file="/lib/models/chatUsersModel.dart" }

추가해준 코드의 설명과 추가한 이유는 이하와 같다.


- String id;   
  데이터 베이스에 저장을 할 때, key가 될 속성이 필요하기 때문에, 추가해 주었다. `id`속성은 `uuid`플러그인을 이용해 생성자 실행 시에 자동으로 할당되도록 해주었다. SQL데이터베이스로 말하면 prime key같은 역할을 할 속성, name등 다른 속성으로 식별해 줄 수도 있지만, 이름은 겹칠 수도 있고...name 외의 속성도 딱히 prime key 역할은 못할 듯   
  ※ 그렇다고 진짜 hive에서 키로 설정할 속성을 (겹치면 절대 안되는)식별키로 인식하는건 아님 주의! 내가 아직 SQL식 사고에서 못벗어나서 만들어 준 것 뿐...! 진짜로 이렇게 사용하는 건지는 모르겠음  
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
part 'chatUsersModel.g.dart'; 
```
{: file="/lib/models/chatUsersModel.dart" }

의 `chatUsersModel.g.dart`를 생성하는 작업을 해줄 것이다.

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
이런 로그가 출력되면서, `chatUsersModel.dart`와 같은 폴더에 `chatUsersModel.g.dart`이 생성된 것을 확인할 수 있다.

![Flutter Simple chat UI ex 12](SimulSc_ex_12.png){:width="50%" height="50%"}

앞서 설명했듯,`chatMessageModel.dart`도 똑같이 생성해 주었음!

# ChatPage : 유저 추가, 삭제, 저장

## 코드 정리, UI 수정

기존 코드에서 수정할 것도 있고 여러가지가 추가되기 때문에 `chatPage.dart`의 코드를 수정, 추가해 주었다.
- 스크롤하면 페이지 타이틀까지 스크롤되는 거 수정
- Add new 버튼이 실제로 눌릴 수 있게 제스쳐 위젯 추가
- User list 를 초기화 할 수 있는 버튼 추가
- ListView의 User를 사이드 스와이프로 제거 ([Dismissible](https://docs.flutter.dev/cookbook/gestures/dismissible)) 할 수 있도록 추가
  
이하 수정한 `chatPage.dart` 전문

```dart
import 'package:flutter/material.dart';
import 'package:flt_simple_chat_ex/models/chatUsersModel.dart';
import 'package:flt_simple_chat_ex/widgets/conversationList.dart';

class ChatPage extends StatefulWidget {
  @override
  _ChatPageState createState() => _ChatPageState();
}

class _ChatPageState extends State<ChatPage> {
  List<ChatUsers> chatUsers = [
    ChatUsers(
        name: "Jane Russel",
        messageText: "Awesome Setup",
        imageURL: "https://randomuser.me/api/portraits/men/1.jpg",
        time: "Now"),
    ChatUsers(
        name: "Glady's Murphy",
        messageText: "That's Great",
        imageURL: "https://randomuser.me/api/portraits/women/1.jpg",
        time: "Yesterday"),
    ChatUsers(
        name: "Jorge Henry",
        messageText: "Hey where are you?",
        imageURL: "https://randomuser.me/api/portraits/men/2.jpg",
        time: "31 Mar"),
    ChatUsers(
        name: "Philip Fox",
        messageText: "Busy! Call me in 20 mins",
        imageURL: "https://randomuser.me/api/portraits/women/2.jpg",
        time: "28 Mar"),
    ChatUsers(
        name: "Debra Hawkins",
        messageText: "Thankyou, It's awesome",
        imageURL: "https://randomuser.me/api/portraits/men/3.jpg",
        time: "23 Mar"),
    ChatUsers(
        name: "Jacob Pena",
        messageText: "will update you in evening",
        imageURL: "https://randomuser.me/api/portraits/women/3.jpg",
        time: "17 Mar"),
    ChatUsers(
        name: "Andrey Jones",
        messageText: "Can you please share the file?",
        imageURL: "https://randomuser.me/api/portraits/men/4.jpg",
        time: "24 Feb"),
    ChatUsers(
        name: "John Wick",
        messageText: "How are you?",
        imageURL: "https://randomuser.me/api/portraits/women/4.jpg",
        time: "18 Feb"),
  ];

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: Column(
        //delete: SingleChildScrollView - 스크롤하면 페이지 타이틀까지 스크롤되는 거 수정
        crossAxisAlignment: CrossAxisAlignment.start,
        children: <Widget>[
          SafeArea(
            child: Padding(
              padding: EdgeInsets.only(left: 16, right: 16, top: 10),
              child: Row(
                mainAxisAlignment: MainAxisAlignment.spaceBetween,
                children: <Widget>[
                  Text(
                    "Conversations",
                    style: TextStyle(fontSize: 23, fontWeight: FontWeight.bold),
                  ),
                  Row(children: <Widget>[
                    Container( // User list 를 초기화 할 수 있는 버튼 추가
                      padding:
                          EdgeInsets.only(left: 8, right: 8, top: 2, bottom: 2),
                      height: 30,
                      decoration: BoxDecoration(
                        borderRadius: BorderRadius.circular(30),
                        color: Colors.green[50],
                      ),
                      child: GestureDetector(
                        onTap: () async {
                          print("Clear user list");
                        },
                        child: Row(
                          children: <Widget>[
                            Icon(
                              Icons.cancel,
                              color: Colors.green,
                              size: 20,
                            ),
                            SizedBox(
                              width: 2,
                            ),
                            Text(
                              "Clear",
                              style: TextStyle(
                                  fontSize: 14, fontWeight: FontWeight.bold),
                            ),
                          ],
                        ),
                      ),
                    ),
                    SizedBox(
                      width: 4,
                    ),
                    Container(
                      padding:
                          EdgeInsets.only(left: 8, right: 8, top: 2, bottom: 2),
                      height: 30,
                      decoration: BoxDecoration(
                        borderRadius: BorderRadius.circular(30),
                        color: Colors.pink[50],
                      ),
                      child: GestureDetector( // Add new 버튼이 실제로 눌릴 수 있게 제스쳐 위젯 추가
                        onTap: () async {
                          print("ADD NEW PRESSED");
                        },
                        child: Row(
                          children: <Widget>[
                            Icon(
                              Icons.add,
                              color: Colors.pink,
                              size: 20,
                            ),
                            SizedBox(
                              width: 2,
                            ),
                            Text(
                              "Add New",
                              style: TextStyle(
                                  fontSize: 14, fontWeight: FontWeight.bold),
                            ),
                          ],
                        ),
                      ),
                    )
                  ])
                ],
              ),
            ),
          ),
          Padding(
            padding: EdgeInsets.only(top: 16, left: 16, right: 16),
            child: TextField(
              decoration: InputDecoration(
                hintText: "Search...",
                hintStyle: TextStyle(color: Colors.grey.shade600),
                prefixIcon: Icon(
                  Icons.search,
                  color: Colors.grey.shade600,
                  size: 20,
                ),
                filled: true,
                fillColor: Colors.grey.shade100,
                contentPadding: EdgeInsets.all(8),
                enabledBorder: OutlineInputBorder(
                    borderRadius: BorderRadius.circular(20),
                    borderSide: BorderSide(color: Colors.grey.shade100)),
              ),
            ),
          ),
          Flexible(
            //add: Flexible,SingleChildScrollView - 스크롤하면 페이지 타이틀까지 스크롤되는 거 수정
            fit: FlexFit.tight,
            child: SingleChildScrollView( 
              physics: BouncingScrollPhysics(),
              child: ListView.builder(
                itemCount: chatUsers.length,
                shrinkWrap: true,
                padding: EdgeInsets.only(top: 16),
                physics: NeverScrollableScrollPhysics(),
                itemBuilder: (context, index) {
                  return Dismissible(
                    //ListView의 User를 사이드 스와이프로 제거 (Dismissible)
                    key: Key(chatUsers[index].toString()), 
                    onDismissed: (DismissDirection direction) {
                      ScaffoldMessenger.of(context).showSnackBar(SnackBar(
                          content: Text('${chatUsers[index].name} dismissed')));
                      setState(() {
                        chatUsers.removeAt(index);
                      });
                    },
                    background: Container(
                      color: Colors.red,
                    ),
                    child: ConversationList(
                      name: chatUsers[index].name,
                      messageText: chatUsers[index].messageText,
                      imageUrl: chatUsers[index].imageURL,
                      time: chatUsers[index].time,
                      isMessageRead: (index == 0 || index == 3) ? true : false,
                    ),
                  );
                },
              ),
            ),
          ),
        ],
      ),
    );
  }
}
```
{: file="/lib/screen/chatPage.dart" }

수정된 `chatPage` UI (유저 전부 삭제한 상태)

![Flutter Simple chat UI ex 28](SimulSc_ex_28.png){:width="30%" height="30%"}

## 채팅 유저 추가 및 저장

이제 저저...흉!측한 `List<ChatUsers> chatUsers`를 날려버리고 코드를 예쁘게 만들어줄 차례다. 먼저 그 과정을 편하게 만들어줄 버튼 2개를 만들었다.

- `Add New`버튼을 누르면 유저이름을 입력 다이얼로그 창이 뜨고, 유저 이름을 입력할 경우에 그 유저가 추가
- `Clear`버튼을 누르면 리스트에 있는 모든 유저가 삭제

유저의 상태를 앱을 다시 시작해도 기억할 수 있도록 위에 구축해 놓은 `hive`를 사용해 저장할 것이다.
그런데, 예쁘게 코드를 짜기 위해서는 `hive`와 더불어 위젯의 상태 관리를 하는 `provider`에 대해서도 알아야 했다.

`hive`같은 데이터 베이스는 뭐가 중요하다? 무결성이 중요하다~ 언제, 어디서든지, 누가 봤을 때도 같은 상태의 데이터가 보여야한다. A위젯에서 보는 데이터는 오늘데이터고, B위젯에서 보는 데이터는 어제 데이터면 안된다. 이렇게 일관된 데이터에 모든 위젯이 접근하기 위해서는 상태관리가 필수 인데, 기존 Dart의 `Stateless Widget`과 `Stateful Widget`위젯으로 상태 관리를 하면, 부모 위젯/자식 위젯끼리 데이터를 일일히 전달하고 받고 어쩌고 난리난리를 피워야하기 때문에, 이 때 유용한 것이 `provider`이다.

공유가 필요한 데이터의 `provider`를 만들어주고 거기에 해당 데이터와 제어할 수 있는 동작을 담아두면 `provider`가 정한 범위 내 속하는 어떠한 위젯이든 신선한 데이터에 아주 편하게 다이렉트로 접근을 할 수 있게 된다.

`provider`에도 다양한 패키지가 있던데 나는 이번에 [flutter_state_notifier](https://pub.dev/packages/flutter_state_notifier)를 사용했다. 필요한 패키지는 적재적소에 알아서 추가해 주자. 

- [flutter_state_notifier](https://pub.dev/packages/flutter_state_notifier) : `main.dart`
- [provider](https://pub.dev/packages/provider) : `provider`사용하는 곳에 필요
- [flutter_hive](https://pub.dev/packages/hive_flutter) : `provider`에서 접근하므로 `provider`에 필요

`provider`를 사용하기까지!

① `provider`를 정의한다  
② `provider`를 생성한다(범위지정)  
③ `provider`를 사용한다  

말은 쉽다. 하나씩 해보자...

① `provider`를 정의한다
   
우리는 user에 대한 DB, message에 대한 DB를 만들어 줘야 한다.

![Flutter Simple chat UI ex 29](SimulSc_ex_29.png){:width="30%" height="30%"}

그러니까 `providers`라는 폴더를 만들어서 `users.dart` 와 `messages.dart`라는 파일을 만들어 주었다. 여기서는 `users.dart`을 기준으로 기록해두겠다. `messages.dart`도 구조는 비슷하다.

`users.dart`에는 크게 상태를 관리하는 클래스 `UserListPrState`와 제어를 위한 클래스 `UserListController`가 있다.
user 정보를 획득만 하고 싶으면 `UserListPrState`를 통해서, user 정보를 수정하거나 삭제하거나 하고 싶으면 `UserListController`를 통해서 하면 된다는 뜻. 

따라서 `UserListPrState`에서는 hive에 저장된 유저정보를 읽어들이고, 변경된 상태가 있으면 저장할 수 있는 기능을, `UserListController`에는 필요한 제어 함수를 작성하면 된다.

```dart
import 'package:hive_flutter/hive_flutter.dart';
import 'package:flt_simple_chat_ex/models/chatUsersModel.dart';

Box<ChatUsers>? userList = Hive.box('users_log');

class UserListPrState {
  List<ChatUsers> users;

  UserListPrState({
    required this.users,
  });

  factory UserListPrState.initial() {
    return UserListPrState(users: userList!.values.toList());
  }

  UserListPrState copyWith({
    List<ChatUsers>? users_cp,
  }) {
    return UserListPrState(
      users: users_cp ?? users,
    );
  }
}

class UserListController extends StateNotifier<UserListPrState> {
  UserListController() : super(UserListPrState.initial());

  // 유저 추가
  void addUser(ChatUsers user) {
    final _users = [...state.users, user];
    state = state.copyWith(users_cp: _users); // 상태 변경
    userList!.put(user.id, user); // 데이터베이스 저장
  }

  // 유저 삭제 (1명)
  void deleteUser(ChatUsers user) {
    final _users = state.users.where((m) => m.name != user.name).toList();
    state = state.copyWith(users_cp: _users); // 상태 변경
    userList!.delete(user.id); // 데이터베이스 저장
  }

  // 유저 전부 삭제
  void clearUser() async {
    state = state.copyWith(users_cp: []);
    await userList!.clear();

  }
}
```
{: file="/lib/providers/users.dart" }


  필요한 함수가 있으면 `UserListController`에 마구마구 추가해주면 된다.
  여기서 맨 처음에 `Box<ChatUsers>? userList = Hive.box('users_log');` 라는 걸 선언해 주었는데, 'users_log'라는 이름의 user정보를 담는 박스가 곧 데이터베이스가 되는 것이다. 따라서 변수 `userList`는 곧 데이터베이스!

  처음에 box를 생성할 때는 반드시 
  1. 초기화를 해주고 
  2. 위에서 생성한 TypeAdapter 등록해주고 
  3. openbox를 해줘야하는데
  
  그걸 이렇게 `main.dart`에서 해주면 된다.

```dart
void main() async {
  await Hive.initFlutter(); //초기화
  Hive.registerAdapter(ChatUsersAdapter()); // TypeAdapter 등록
  Hive.registerAdapter(ChatMessageAdapter()); // TypeAdapter 등록

  await Hive.deleteFromDisk();

  await Hive.openBox<ChatUsers>('users_log'); // openbox
  await Hive.openBox<ChatMessage>('messages_log'); // openbox

  runApp(
    MyApp(),
  );
}
```
{: file="/lib/main.dart" }

  openbox를 해준 다음에는 `users.dart`에서 처럼 `Hive.box`만으로 접근할 수 있게 된다.


② `provider`를 생성한다(범위지정)

`provider`의 범위는 그 `provider`를 생성한 그 부모위젯에 속하는 모든 자식위젯이 된다. 그래서 일반적으로는 앱 가장 상위에 있는 위젯에 생성하는 것 같다 (구글링해보니까 대부분 그렇게 쓰던데 아닐 수도 있음)

나는 주제에 다른데 써보겠다고 혼자 머리싸메다가 많은 일을 겪고 결국 `main.dart`에 `MaterialApp()`에 생성해 주었다ㅎ 사서 고생을 한다.

```dart
class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    // provider 생성
    return MultiProvider(
      providers: [
        StateNotifierProvider<MessageListController, MessageListPrState>(
          create: (_) => MessageListController(),
        ),
        StateNotifierProvider<UserListController, UserListPrState>(
          create: (_) => UserListController(),
        ),
      ],
    // provider 생성
      builder: (context, child) {
        return MaterialApp(
            title: 'Flutter Demo',
            theme: ThemeData(
              primarySwatch: Colors.blue,
            ),
            debugShowCheckedModeBanner: false,
            home: HomePage());
      },
    );
  }
}
```
{: file="/lib/main.dart" }

이렇게 생성된 `provider`는 `MaterialApp`을 포함한 이하에 존재하는 모든 위젯에서 접근할 수 있게 된다. (`MyApp`에서는 접근 못함 주의!)  
`MultiProvider`는 여러 프로바이더를 관리할 수 있게 해주는 위젯인데, 앞으로 얼마나 늘어날지 몰라서 그냥 썼다.

③ `provider`를 사용한다
   
이제 필요한 위젯에서 접근하거나 제어해주면 된다.
지금은 주로 `chatPage.dart`에서 `users`에, `chatDetailPage.dart`에서 `messages`접근하게 될텐데, 메세지 로그의 저장은 다음 포스팅에 기록하기로 하고, 이번엔 `chatPage.dart`에서 유저를 추가하고 삭제(+그 정보는 저장)하는 걸 해보자. 

`_ChatPageState` 클래스에서 user의 상태를 취득하고 제어(추가, 삭제)를 할 수 있도록 먼저 프로바이더에 접근해준다.

```dart
 @override
  Widget build(BuildContext context) {
    final List<ChatUsers> _ustate = context.watch<UserListPrState>().users; // 상태 취득
    final _uctlr = context.read<UserListController>(); // 제어
```
{: file="/lib/screen/chatPage.dart" }

- 유저를 등록 : `Add new` 버튼을 눌렀을 때, `_uctlr.addUser(_user);`를 실행

```dart
                    Container(
                      padding:
                          EdgeInsets.only(left: 8, right: 8, top: 2, bottom: 2),
                      height: 30,
                      decoration: BoxDecoration(
                        borderRadius: BorderRadius.circular(30),
                        color: Colors.pink[50],
                      ),
                      child: GestureDetector(
                        //add, GestureDetector
                        onTap: () async {
                          print("ADD NEW PRESSED");
                          late final _user;

                          DateTime now = DateTime.now();
                          String nowstring =
                              new DateFormat.yMMMd('en_US').format(now);

                          String? tempusername = await InputDialog(context);
                          if (tempusername == "" || tempusername == null) {
                            print("User Name is Empty!");
                            tempusername = null;
                          } else {
                            print("Save to Database");
                            String thisusername = tempusername;
                            setState(() {
                              _user = ChatUsers(
                                  name: thisusername,
                                  statusmsg: " ",
                                  messageText: " ",
                                  imageURL:
                                      "https://randomuser.me/api/portraits/men/6.jpg",
                                  time: nowstring,
                                  bgdimage: File("초기 셋팅할 String 타입 이미지경로 입력"));

                              _uctlr.addUser(_user);

                              _textFieldController.clear();
                            });
                          }
                          _uctlr.printUser();
                        },
                        child: Row(
                          children: <Widget>[
                            Icon(
                              Icons.add,
                              color: Colors.pink,
                              size: 20,
                            ),
                            SizedBox(
                              width: 2,
                            ),
                            Text(
                              "Add New",
                              style: TextStyle(
                                  fontSize: 14, fontWeight: FontWeight.bold),
                            ),
                          ],
                        ),
                      ),
                    )

```
{: file="/lib/screen/chatPage.dart" }

※ Dart의 `late`선언은 반드시 값이 필요한 (null이면 안되는, non-nullable) 변수지만, 선언과 동시에 초기화를 하지 않을 때 사용한다. 변수의 초기화를 뒤로 밀어줄 수 있는 것. 

- 등록된 모든 유저를 삭제 : `Clear` 버튼을 눌렀을때 `_uctlr.clearUser();`를 실행

```dart
                    Container(
                      padding:
                          EdgeInsets.only(left: 8, right: 8, top: 2, bottom: 2),
                      height: 30,
                      decoration: BoxDecoration(
                        borderRadius: BorderRadius.circular(30),
                        color: Colors.green[50],
                      ),
                      child: GestureDetector(
                        onTap: () async {
                          _uctlr.clearUser(); //모든 유저를 삭제
                        },
                        child: Row(
                          children: <Widget>[
                            Icon(
                              Icons.cancel,
                              color: Colors.green,
                              size: 20,
                            ),
                            SizedBox(
                              width: 2,
                            ),
                            Text(
                              "Clear",
                              style: TextStyle(
                                  fontSize: 14, fontWeight: FontWeight.bold),
                            ),
                          ],
                        ),
                      ),
                    ),
```
{: file="/lib/screen/chatPage.dart" }

- 지정한 유저를 삭제 : 리스트뷰의 특정 유저를 옆으로 스와이프를 했을 때, `_ustate.removeAt(index);` 실행
  
  이때, `_uctlr`이 아닌 `_ustate`를 사용하는데 좀 위화감이 있긴한데, (유저의 id를 획득해 와서 유저를 특정해서 그 유저를 지우는 것보다) 리스트뷰의 index(몇 번째 유저인지)를 이용하여 컨트롤을 해주는게 더 코드가 간단해서 이렇게 해줬다.

```dart
              ListView.builder(
                itemCount: _ustate.length,
                shrinkWrap: true,
                padding: EdgeInsets.only(top: 16),
                physics: NeverScrollableScrollPhysics(),
                itemBuilder: (context, index) {
                  return Dismissible(
                    //사이드 스와이프로 삭제
                    key: UniqueKey(),
                    onDismissed: (DismissDirection direction) {
                      ScaffoldMessenger.of(context).showSnackBar(SnackBar(
                          content: Text('${_ustate[index].name} dismissed')));
                      setState(() {
                        _ustate.removeAt(index);
                      });
                    },
                    background: Container(
                      color: Colors.red,
                    ),
                    child: ConversationList(
                      id: _ustate[index].id,
                      name: _ustate[index].name,
                      messageText: _ustate[index].messageText,
                      imageUrl: _ustate[index].imageURL,
                      time: _ustate[index].time,
                      isMessageRead: (index == 0 || index == 3) ? true : false,
                      user: _ustate[index],
                    ),
                  );
                },
              ),
```
{: file="/lib/screen/chatPage.dart" }

이런식으로 사용해주면 되는 것이다............  
중간에 DB에 저장은 되는데 UI에 업데이트가 안되는 미치겠는 상황이 있었는데, 앱을 restart 했더니 해결되었다...(빡)

결과, 요렇게 동작을 한다

![Flutter Simple chat UI ex 29](SimulSc_ex_29.gif){:width="30%" height="30%"}

# userDetailPage : 유저 데이터 갱신, 상태창 생성
## 상대의 프로필 사진 클릭 시, 상태창 표시

일단은 프로필을 구현한 `conversationList.dart`에 가서, 라운드한 프로필 모양에 따라 터치 이벤트를 발생시켜주면 되는데, 방법에는 여러가지가 있다.

방법1. `InkWell`사용 : 프로필 사진의 원을 포함하는 사각형 모양으로 터치 반응
```dart
                InkWell(
                    onTap: () {
                      print("CircleAvatar touch");
                    },
                    child: CircleAvatar(
                      backgroundImage: NetworkImage(widget.imageUrl),
                      maxRadius: 30,
                    ),
                  ),
```
{: file="/lib/widgets/conversationList.dart" }

방법2. `GestureDetector`사용 : 프로필 사진의 원 모양으로 터치 반응
```dart
                Column(children: <Widget>[
                    SizedBox(height: 30.0),
                    GestureDetector(
                      onTap: () {
                        print("CircleAvatar touch");
                      },
                      child: CircleAvatar(
                        backgroundImage: NetworkImage(widget.imageUrl),
                        maxRadius: 30,
                      ),
                    )
                  ]),
```
{: file="/lib/widgets/conversationList.dart" }

방법3. `RawMaterialButton`사용 : 프로필 사진의 원 모양으로 터치 반응
```dart
                Stack(
                    children: <Widget>[
                      CircleAvatar(
                        backgroundImage: NetworkImage(widget.imageUrl),
                        maxRadius: 30,
                      ),
                      RawMaterialButton(
                        onPressed: () {
                          print("CircleAvatar touch");
                        },
                        child: Container(
                          width: 60.0, // CircleAvatarのradiusの2倍
                          height: 60.0,
                        ),
                        shape: new CircleBorder(),
                        elevation: 0.0,
                      ),
                    ],
                  ),
```
{: file="/lib/widgets/conversationList.dart" }

하여튼 `CircleAvatar`을 `child`로 두는 포함하는 버튼/제스처 위젯을 사용하든, `Stack`으로 `CircleAvatar`과 버튼 위젯을 겹쳐주든 하면 된다. 이외에도 방법은 여러가지가 있을 듯. 나는 방법1 사용.

이제, 터치하면 새로운 페이지로 상태창을 띄워줄것이다 지금 위에 코드에서 `print("CircleAvatar touch");`라고 되어있는 부분에 실제 코드를 넣어주면 된다.  
상태창을 그려줄 `userDetailPage.dart`를 `screen`폴더에 추가해 주었다.  
그리고 상태창에서 유저의 여러 속성에 접근하고 수정할 거라서 `chatDetailPage`에서처럼 유저의 속성을 `id`, `imageUrl`같이 각각 전달하지 않고 그냥 `ChatUsers` 클래스 전체를 전달했다. (그김에 `chatDetailPage`도 동일하게 수정)


```dart
import 'package:flutter/material.dart';

class UserDetailPage extends StatefulWidget {
  ChatUsers user;

  UserDetailPage({
    Key? key,
    required this.user,
  }) : super(key: key);

  @override
  _UserDetailPageState createState() => _UserDetailPageState();
}

class _UserDetailPageState extends State<UserDetailPage> {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        elevation: 0,
        automaticallyImplyLeading: false,
        backgroundColor: Colors.white.withOpacity(0.5),
        flexibleSpace: SafeArea(
          child: Container(
            padding: EdgeInsets.only(right: 16),
            child: Container(
              color: Colors.transparent,
              child: Row(
                children: <Widget>[
                  IconButton(
                    onPressed: () {
                      Navigator.pop(context);
                    },
                    icon: Icon(
                      Icons.arrow_back_ios_new,
                      color: Colors.black,
                    ),
                  ),
                ],
              ),
            ),
          ),
        ),
      ),
    );
  }
}
```
{: file="/lib/screen/userDetailPage.dart" }

만들어준 `userDetailPage.dart` 를 프로필 사진의 제스처에 연결해 주면 된다.

```dart
                  InkWell(
                    onTap: () {
                       //추가
                      Navigator.of(context).push(MaterialPageRoute(
                        builder: (context) {
                          return UserDetailPage(
                            cpuser: widget.user,
                          );
                        },
                      ));
                      //추가 END
                    },
                    child: CircleAvatar(
                      backgroundImage: NetworkImage(widget.user.imageURL), //수정, widget.imageurl ->  widget.user.imageURL
                      maxRadius: 30,
                    ),
                  ),
```
{: file="/lib/widgets/conversationList.dart" }

이제 프로필 사진을 클릭하면 이런 빈 화면이 뜰텐데, 이 상태창은 취향에 맞게 디자인 해주면 된다.  

![Flutter Simple chat UI ex 30](SimulSc_ex_30.png){:width="30%" height="30%"}


나는 이런식으로 만들어 주었다. 상태메세지는 원래 유저 속성에 없었기 때문에 UI작업하면서 `String statusmsg`라는 변수타입/명으로 추가해 주었다.

![Flutter Simple chat UI ex 31](SimulSc_ex_31.png){:width="30%" height="30%"}

> #수정 보류중인 오류  
> 이름이랑 상태메세지 정렬이 맘에 안들어서 나중에 수정

상태창 UI의 버튼 목록
- Appbar의 역화살표 아이콘 : `ChatPage`로 돌아가기
- Appbar의 펜모양 아이콘 : 아직 아무 기능 없음
- Appbar의 더하기 아이콘 : 배경 이미지를 변경 (다음 소챕터에 자세히 작성)  
- 이름 옆의 펜 아이콘 : 이름 수정
- 상태메세지 옆의 펜 아이콘 : 상태메세지 수정
- 하단의 메세지 아이콘 : `ChatDetailPage`로 이동
- 하단의 전화 아이콘 : 아직 아무 기능 없음
- 하단의 정보 아이콘 : 아직 아무 기능 없음

아이콘은 [Flutter Icon Class](https://api.flutter.dev/flutter/widgets/Icon-class.html) 참고.

UI중에 지금까지랑은 크게 다른건 Appbar의 백그라운드를 body뒤로 숨긴것 정도 이다. `Scaffold`위젯에서 `extendBodyBehindAppBar` 옵션은 `true`로 해주면 됨! 그 외에는 딱히 신경쓴건 없고 그냥 배치 구성만 잘 짜면 된다. 아주 제일 머리아프지...

```dart
 ... // 생략
    return Scaffold(
        extendBodyBehindAppBar: true, //앱바 배경 숨기기
        appBar: AppBar(

 ... // 생략
```
{: file="/lib/screen/userDetailPage.dart" }

그런데 위의 코드대로라면 유저의 이름을 수정한 후에 `ChatPage`로 돌아갔을 때 리스트뷰에 갱신내용이 반영이 되어있지 않은 것을 확인 할 수 있을 것이다. 그건`conversationList`에서 `UserDetailPage`로 `Navigator.of(context).push`할 때, 미래의 값을 기다려 줌(`await`)으로써 값이 수정되면 상태를 갱신(`setState`)하는 것으로 해결할 수 있다.

```dart
                  InkWell(
                    onTap: () async { //추가, async
                      await Navigator.of(context).push(MaterialPageRoute( //추가, await
                        builder: (context) {
                          return UserDetailPage(
                            cpuser: widget.user,
                          );
                        },
                      ));
                      setState(() {}); //추가
                    },
                    child: CircleAvatar(
                      backgroundImage: NetworkImage(widget.user.imageURL), 
                      maxRadius: 30,
                    ),
                  ),
```
{: file="/lib/widgets/conversationList.dart" }


### 갤러리서 선택한 이미지를 배경 이미지로 설정 

유저의 속성(`/lib/models/chatUsersModel.dart`)에 배경 이미지 속성(`File bgdimage`) 추가해주고,

```dart
class ChatUsers {
  @HiveField(0) 
  String id; // hive의 key로 지정하기 위해 추가
  @HiveField(1)
  String name;
  @HiveField(2)
  String statusmsg; // 상태 메세지
  @HiveField(3)
  String messageText;
  @HiveField(4)
  String imageURL;
  @HiveField(5)
  String time;
  @HiveField(6)
  File bgdimage; //배경 이미지

  ChatUsers(
      {String? id,
      required this.name,
      required this.messageText,
      required this.statusmsg,
      required this.imageURL,
      required this.time,
      required this.bgdimage})
      : id = id ?? uuid.v4();

  ... // 생략
}
```
{: file="/lib/models/chatUsersModel.dart" }

`/lib/modelschatUsersModel.g.dart` 파일 갱신을 위해서 터미널에서 명령어 실행 
```html
flutter packages pub run build_runner build
```

그런데 여기서 문제가 발생할 것이다... 왜냐하면, `Hive`는 File Type을 지원하지 않기 떄문이다. 코드를 수정한 후에 `Add new`로 유저를 추가해 주면, 이러한 「너 File의 Typeadapter를 추가하지 않았다. 추가해줘라.」라는 에러를 만나게 될 것이다. 

```error
[VERBOSE-2:ui_dart_state.cc(198)] Unhandled Exception: HiveError: Cannot write, unknown type: _File. Did you forget to register an adapter?
```
그리고 덩달아 나머지 속성들에 대한 저장도 제대로 되지 않는다. (아예 `'user_log'`박스가 제대로 동작하지 않는다)  File의 Typeadapter를 알아서 작성하라는 거냐... 싶지만 인터넷에는 웬만한 찾아보면 있다.  

내가 참고한 글은 [【Flutter】Hiveの対応しているオブジェクト以外も格納できるようにする](https://zenn.dev/kirisimacreate/articles/6badb7d8ee58f8) 라는 글인데, 이 글에서 처럼 `file_apdater.dart`라는 파일을 만들어 주고, 만들어준 파일에 정의한 `FileAdapter`를 `main.dart`에 등록해 주면 된다. 그럼 에러가 없이 깔끔하게 실행되는 것을 확인 할 수 있다.

이제 해결했으니 본론으로 돌아오겠다.

배경 이미지 선택은 갤러리에서 사진을 가져오는 방식을 사용할 것이기 때문에, `/ios/Runner/info.plist`에 카메라, 앨범에 액세스 권한을 추가해주었다.(카메라는 아직 권한만 추가하고 미구현), 권한을 주지 않고 앨범에 접근하려고 하면 앱이 강제종료된다.

```plist
	<string>Access to take a photo by camera</string>
	<key>NSAppleMusicUsageDescription</key>
	<string>Access to pick a photo</string>
	<key>NSPhotoLibraryUsageDescription</key>
	<string>Access to pick a photo</string>
```
{: file="/ios/Runner/info.plist" }

상태창 UI(`/lib/screen/userDetailPage.dart`)에 버튼을 만들어 주고 버튼을 누르면 갤러리에 접근할 수 있도록 해준다.

```dart
                    IconButton(
                        onPressed: () async {
                          print("Add background Image");
                          
                          File _tempimg = await getImageFromGallery();
                          setState(() {
                            _uctlr.editUserbgd(widget.cpuser.id, _tempimg); //해당 id유저의 bgdimage 속성값을 변경
                          });
                        },
                        alignment: Alignment.centerRight,
                        icon: const Icon(
                          Icons.add,
                          color: Colors.black,
                        ),
                      )
```
{: file="/lib/screen/userDetailPage.dart" }

여기서 `getImageFromGallery()`에 `await`선언을 해주었는데, 이유는 `getImageFromGallery()`에서 반환된 값을 받아온 후에 파일 값을 갱신해야하기 때문이다.
`await`선언을 해주지 않고 그냥 `_tempimg`에 반환값을 받아오려고 해도 어짜피 에러가 난다 `getImageFromGallery()`는 반환값이 `Future<File>`타입이기 때문이다. 

갤러리에 접근해서 이미지를 가져오는 함수(`getImageFromGallery()`)는 이렇게 생겼다. 
[Image Picker](https://pub.dev/packages/image_picker)라는 플러그인을 사용해서 가져오니 `pubspec.yaml`에 `dependencies`추가해주고 사용할 위젯에 임포트 해주었다.

```dart
  File _bgdimage = File("초기 셋팅할 String 타입 이미지경로 입력");
  final ImagePicker picker = ImagePicker();

  Future<File> getImageFromGallery() async {
    final pickedFile = await picker.pickImage(source: ImageSource.gallery);

    if (pickedFile != null) {
      setState(() {
        _bgdimage = File(pickedFile.path);
      });
      return _bgdimage;
    } else {
      print('No image selected');
      return _bgdimage;
    }
```
{: file="/lib/screen/userDetailPage.dart" }

이 때 `getImageFromGallery` 함수는 (배경 이미지)파일 값을 받는 비동기(`Future<File>`)처리를 해주었다. 
`pickImage`가 사용자가 갤러리에서 이미지를 선택하기까지 (혹은 선택을 취소할 때까지) 대기를 타다가 작업이 완료되면, 그제서야 값을 가져올수 있는 상태(`await`)이기 떄문이다. (이것 때문에 `_tempimg`에 값을 받아올 때도 `await` 필요)

처음에는 `_tempimg`를 거치지 않고 `_bgdimage` 변수를 그대로 `editUserbgd(widget.cpuser.id, _bgdimage)` 이런식으로 넣어버렸었는데, 그러면 비동기화 때문에 `_bgdimage` 값이 변경되기 전에 재빌드가 먼저 되어버리고, 그 뒤에 `_bgdimage` 값이 변경되어서 배경 이미지가 바뀌지 않았다.

UI에는 이렇게 `BoxDecoration`위젯을 사용해 주면 된다.

```dart
 ... // 생략
          body: Container(
            //color: Colors.orange, //decoration과 함께 쓸 수 없는 속성
            decoration: BoxDecoration(
              image: DecorationImage(
                  image: FileImage(widget.cpuser.bgdimage),
                  fit: BoxFit.fill,
                  opacity: 0.3),
            ),
 ... // 생략
```
{: file="/lib/screen/userDetailPage.dart" }

완성된 상태창, 수정 데모

![Flutter Simple chat UI ex 32](SimulSc_ex_32.gif){:width="30%" height="30%"}

> #수정 보류중인 오류  
> 상태메세지 바꿀 때 뜨는 다이얼로그를 이름 바꿀 때 쓰는 다이얼로그와 같이 써서, 타이틀이랑 힌트가 상황에 맞지 않게 뜨는 거 요수정



***
# 다음 포스팅 내용

이제 유저 정보는 데이터 베이스에 저장이 된다! 상태창에서 정상적으로 이름, 상태메세지, 배경이미지를 바꿀 수 있다.  
다음엔 유저 정보를 저장해 준 것처럼, 유저별로 메세지 로그를 저장해 주고 마지막에 대화했던 메세지 내용을 `ChatPage`에 표시해 주는 것을 해 볼 것이다.

***

# flutter의 상태 관리

## StatefulWidget(setState)
Dart에서 기본적으로 제공하는 상태 관리 위젯과 함수
## Redux
자세히 공부 안함 이런게 있다는 것만 앎, 나중에 공부하면 추가하기
## Stream + InheritedWidget/Scoped Model (BLoC)
자세히 공부 안함 이런게 있다는 것만 앎, 나중에 공부하면 추가하하기
## [Provider](https://pub.dev/packages/provider)
provider package
- [change_notifier](https://pub.dev/packages/change_notifier)
- [state_notifier](https://pub.dev/packages/state_notifier) ([flutter_state_notifier](https://pub.dev/packages/flutter_state_notifier))  
  change_notifier 을 좀 더 개선한 느낌, 이번에 이거 사용

Provider 사용법에서 디지게 애먹었다 아놔.... 분명 제대로 쓴 거 같은데 계속 범위 외라고 떠서 해결방법 까지 한 200번의 성공의 어머니를 겪음

`ChatDetailPage()`가`MaterialPageRoute`에서 빌드되고 있다는 것의 의미를 한!!!!!!참을 이해를 못해서 `StateNotifierProvider`를 이상한 곳에서 공급해 주고있는데, 왜 에러가 나는지 모르겠어서 진짜 미치고 돌아버리는 줄 알았다. 이론을 전혀 모르고 그냥 맨땅에 머리를 박으니까 이런데서 시간이 무지하게 들어간다. 지금 생각하면 이거가지고 이렇게 고생을 한 게 어이없어 보이지만, 덕분에 Provider의 범위에 대해서는 공부가 되었으니 오케입니다

해결하면서 도움이 많이 된 사이트
- [Provider설명과 예제](https://dev-yakuza.posstree.com/ko/flutter/provider/)
- [Flutter(Provider+StateNotifier+freezed)](https://note.com/mukae9/n/n91b3301ebccf)
- [Simple app state management](https://docs.flutter.dev/development/data-and-backend/state-mgmt/simple#changenotifierprovider)

## [Riverpod](https://riverpod.dev/ko/docs/getting_started)
riverpod package 
- [flutter_riverpod](https://pub.dev/packages/flutter_riverpod)

GetX, Provider, BLoC처럼 상태관리를 위한 패키지 (Provider의 확장판)

참고 사이트
- [Riverpod설명과 예제](https://prod.velog.io/@leeeeeoy/Flutter-Riverpod-사용해보기-1)


***

# 참고하기 좋은 사이트
- [How to save data locally in Flutter](https://pusher.com/tutorials/local-data-flutter/)  
  위에서 언급한 로컬에 데이터 저장할 때 고려한 방법 3개(①shared_preferences ②로컬 파일 ③로컬 데이터베이스 : sqflite플러그인사용)에 대한 튜토리얼이 있다. 
- [flutter study](https://www.flutter-study.dev/create-app/provider)
- [flutter provider 사용법 기초](https://zenn.dev/naoya_maeda/articles/01e51841d6fb8b)

***
# 후기
글로 기록하면 한번에 톽딱!땋! 한 것처럼 보이는데 사실 쪼랩이라서 뭣만하면 모르는 게 나오기 때문에 시행착오를 굉장히 겪는다... 돌아버려 진짜,,, 이번거 완전 오래 걸렸고,,, 그래도 많이 배웠다.  
그리고 쓰다보니 생각보다 긴데, 나중에 좀 내용을 나누든가 해야겠다. 원래 메세지 로그 저장도 여기에 같이 쓰려다 너무 길어져서 나눈 건데, 그래도 기네