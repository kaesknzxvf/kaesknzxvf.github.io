---
# multilingual page pair id, this must pair with translations of this page. (This name must be unique)
#lng_pair: "id_study_flutter"
title: "Flutter 01 Simple Chat UI 응용 03 데이터 로컬 저장 및 읽기,쓰기_메세지 로그편 "

# post specific
author: kae
# publish date
date: 2022-09-19 20:17:00 +0900
category: [Flutter, Simple Chat]
tags: [flutter, practice, programming]

# disable comments on this page
#comments_disable: true

# thumbnail image for post
# img: ":Flutter.jpeg"

# data path
img_path: /assets/img/2022-09-20-Flutter-Simple-Chat-UI-EX03/
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
- `ChatDetailPage` : 유저 별 메세지 로그 저장, 불러오기
  - 코드 정리 : 메세지 로그 리스트뷰 코드 분리
  - 메세지 로그 저장/불러오기 시의 주의 점
  - 해당 유저의 메세지 로그 불러오기
  - 메세지 로그 저장
  - UI 변경, 기능 추가 (옵션)
    - 받은 메세지 조작 : receiver 버튼, 기능 추가
    - 메세지 초기화 : clear 버튼, 기능 추가
    - 개별 메세지에 대한 기능 추가 (개별 메세지 복사, 수정, 삭제)
- `ChatPage` : 유저 데이터 갱신
  - 각 유저의 최근 메세지 표시
 
원문의 예제는 단순히 UI만 다루는 포스트이다 보니, 실제로 데이터를 전달하거나 저장하는 등 데이터를 처리하는 작업은 해주지 않는다. 그러므로 이번 포스트는 스스로 공부하며 추가한 코드이다.

관련 포스트
- [Base, Chat 화면 구현](https://kaesknzxvf.github.io/posts/Flutter-Simple-Chat-UI-01/)
- [Chat Detail 화면 구현](https://kaesknzxvf.github.io/posts/Flutter-Simple-Chat-UI-02/)
- [응용 01 (Chat Detail 화면) 화면 간 데이터 전달, 메세지 입력](https://kaesknzxvf.github.io/posts/Flutter-Simple-Chat-UI-EX01/)
- [응용 02 데이터 로컬 저장 및 읽기,쓰기_유저정보편 (+상태창 추가)](https://kaesknzxvf.github.io/posts/Flutter-Simple-Chat-UI-EX02/)
- 응용 03 데이터 로컬 저장 및 읽기,쓰기_메세지로그편 (현재포스트)

***
# 환경

- MacBook Air (M1, 2020)
- macOS Monterey (v12.4)

- Flutter 3.0.3
- Dart 2.17.5

- Visual Studio Code 1.69.0

***
# ChatDetailPage : 유저 별 메세지 로그 저장, 불러오기

먼저 유저 별 메세지 로그를 저장하고, 기타 기능을 구현하기 위해서 ChatMessage 모델의 속성을 조금 추가해 주었다.

- userid : 현재 채팅을 하고있는 유저의 `id`속성을 받는 속성
- messageid : 개별 메세지의 식별을 위한 속성, `ChatUsers` 모델의 `id`와 마찬가지로 `uuid`플러그인을 이용해 생성자 실행 시에 자동으로 할당되도록 해주었다.

```dart
import 'package:hive/hive.dart'; 
import 'package:uuid/uuid.dart';

part 'chatMessageModel.g.dart';

var uuid = Uuid();

@HiveType(typeId: 2)
class ChatMessage {
  @HiveField(0) //추가
  String userid; //추가

  @HiveField(1) //추가
  String messageid; //추가

  @HiveField(2) 
  String messageContent;

  @HiveField(3) 
  String messageType;


  ChatMessage(
      {required this.userid,
      String? messageid,
      required this.messageContent,
      required this.messageType})
      : messageid = messageid ?? uuid.v4();

}
```
{: file="/lib/models/chatMessageModel.dart" }

수정 후에는 `TypeAdapter` 갱신해주는 걸 잊지 말자
```html
flutter packages pub run build_runner build
```

## 코드 정리 : 메세지 로그 리스트뷰 코드 분리

지금 `ChatDetailPage.dart` 화면에 UI랑 데이터 처리 코드가 섞여 너무 정신사나워서, 메세지 로그를 표시하는 리스트뷰 부분 코드를 조금 정리했다.

`chatPage.dart`의 유저 목록을 표시하는 리스트뷰 `conversationList.dart`를 분리한 것 처럼, `ChatDetailPage.dart`도 body에 메세지로그를 표시하는 Listview를 `messageList.dart` 라는 widget으로 분리했다.

```dart
import 'package:flutter/material.dart';

class MessageList extends StatefulWidget {
  String userid;
  ChatMessage currentmsg;

  MessageList(
      {required this.userid,
      required this.currentmsg});

  @override
  _MessageListState createState() => _MessageListState();
}

class _MessageListState extends State<MessageList> {
  @override
  Widget build(BuildContext context) {
    return Container(
      padding: EdgeInsets.only(left: 16, right: 16, top: 10, bottom: 10),
      child: Align(
        alignment: (widget.currentmsg.messageType == "receiver"
            ? Alignment.topLeft
            : Alignment.topRight),
        child: Container(
          decoration: BoxDecoration(
            borderRadius: BorderRadius.circular(20),
            color: (widget.currentmsg.messageType == "receiver"
                ? Colors.grey.shade200
                : Colors.blue[200]),
          ),
          padding: EdgeInsets.all(16),
          margin: (widget.currentmsg.messageType == "receiver"
              ? EdgeInsets.only(left: 0, right: 64, top: 0, bottom: 0)
              : EdgeInsets.only(left: 64, right: 0, top: 0, bottom: 0)),
          child: Text(¥
            widget.currentmsg.messageContent,
            style: TextStyle(fontSize: 15),
          ),
        ),
      ),
    );
  }
}
```
{: file="/lib/widgets/messageList.dart" }

기존 파일에서는 해당 부분 삭제하고 인수 전달만 해주었다.

```dart
//추가
import 'package:flt_simple_chat_ex/widgets/messageList.dart';

... // 생략
      body: Column(
        children: <Widget>[
          Flexible(
            fit: FlexFit.tight, 
            child: SingleChildScrollView(
              physics: BouncingScrollPhysics(), 
              controller: _scrollController, 
              child: ListView.builder(
                itemCount: _mstate_currentuser.length,
                shrinkWrap: true,
                padding: EdgeInsets.only(
                    top: 10), 
                physics: NeverScrollableScrollPhysics(),
                itemBuilder: (context, index) {
                  // 수정, 인수로 전달
                  return MessageList( 
                    userid: widget.cpuser.id,
                    currentmsg: ChatMessage(
                        userid: widget.cpuser.id,
                        messageid: _mstate_currentuser[index].messageid,
                        messageContent:
                            _mstate_currentuser[index].messageContent,
                        messageType: _mstate_currentuser[index].messageType),
                  );
                  // 수정, 인수로 전달
                },
              ),
            ),
          ),
... // 생략
```
{: file="/lib/screens/chatDetailPage.dart" }

이제 메세지 로그에 대한 처리는 `messageList.dart`에서 할 것이다.

## 메세지 로그 저장/불러오기 시의 주의 점
코드도 정리했고 사용할 기능도 대충 만들었으니 이제 본격적으로 메세지를 저장해보자. 근데 사실... 유저 리스트 저장하는 것과 별 다를 것 없다. 
(이전 포스팅에서 provider에 대한 걸 다뤘으니 당연히 `/providers/messages.dart`가 있고, TypeAdapter가 등록, 범위 지정이 되어있다고 가정한다.) 메세지 로그를 담은 박스의 이름은 `'message_log'`인데, provider에서 이 박스를 요리 다루고 조리 다루면 된다.
메세지에 대한 읽기 및 쓰기 작업이 이루어 지는 위젯은  주로 `chatDetailPage.dart`과 `messageList.dart`일텐데, 필요에 따라 

```dart
import 'package:flt_simple_chat_ex/providers/messages.dart';
import 'package:provider/provider.dart';
... // 생략
  @override
  Widget build(BuildContext context) {
    final _mstate = context.watch<MessageListPrState>().messages;
    final _mctlr = context.read<MessageListController>();
... // 생략
```
{: file="/lib/screens/chatDetailPage.dart" }

이런 식으로 provider에 접근해서 메세지를 읽고 써주면 된다.

단, 지금 구현한 코드에서 유저 목록을 다룰 때와는 다르게 반드시 고려해야 할 점이 있는데, 바로 유저 목록(`'user_log'`박스)은 그 박스 자체로 하나의 개체로 볼 수 있지만, 메세지 로그(`'message_log'`박스)의 경우 모든 유저의 메세지를 구별없이 전부 담고 있는 하나의 혼돈의 박스라는 점이다.   
그러니까 각 유저의 채팅 창에는 `'message_log'`에 있는 모든 메세지를 전부 출력할게 아니라 그 채팅 창의 유저가 누군지 확인해서 해당 유저와 관련된 메세지만을 표시해야 된다는 소리다. 이걸 위해 `ChatMessage` 모델에 `userid` 속성을 추가한 것이다. (나도... 아주 이상한 코드라는 건 안다... 알긴 아는데 능력 부족으로 일단 밀고 나갔다)

개판 5분 전이지만 일단 가보자고...

## 해당 유저의 메세지 로그 불러오기

채팅창을 열었을 때, `checkUserMsg(widget.cpuser.id)` 라는 함수를 사용하여 리스트뷰를 빌드했다.  
messages provider의 `MessageListPrState`에 `checkUserMsg`라는 함수를 만들어서 현재 열어둔 채팅창의 userid를 가진 메세지들만으로만 구성된 리스트를 반환하도록 해주었다. (`MessageListPrState`에 구현한 이유는, 딱히 데이터베이스에 항목을 추가하거나 삭제하는게 아니고 단지 선별해서 읽어올 뿐이기 때문)

```dart
... // 생략
    final _mstate_currentuser =
        context.watch<MessageListPrState>().checkUserMsg(widget.cpuser.id);
... // 생략
      body: Column(
        children: <Widget>[
          Flexible(
            fit: FlexFit.tight,
            child: SingleChildScrollView(
              physics: BouncingScrollPhysics(),
              controller: _scrollController,
              child: ListView.builder(
                itemCount: _mstate_currentuser.length, // 수정
                shrinkWrap: true,
                padding: EdgeInsets.only(
                    top: 10),
                physics: NeverScrollableScrollPhysics(),
                itemBuilder: (context, index) {
                  return MessageList(
                    userid: widget.cpuser.id,
                    currentmsg: ChatMessage(
                        userid: widget.cpuser.id,
                        messageid: _mstate_currentuser[index].messageid, // 수정
                        messageContent:
                            _mstate_currentuser[index].messageContent, // 수정
                        messageType: _mstate_currentuser[index].messageType), // 수정
                  );
                },
              ),
            ),
          ), // Flexible
... // 생략
```
{: file="/lib/screen/ChatDetailPage.dart" }

`checkUserMsg()`는 이렇게 생겼다.

```dart
... // 생략
class MessageListPrState {
... // 생략
  List<ChatMessage> checkUserMsg(String userid) {
    List<ChatMessage> usermsg = [];
    for (int i = 0; i < messages.length; i++) {
      if (messages[i].userid == userid) {
        usermsg.insert(usermsg.length, messages[i]);
      }
    }
    return usermsg;
  }
... // 생략
```
{: file="/lib/provider/messages.dart" }

이렇게하면 최적화는 개나 준 코드 완성!

유저가 생길 때마다 메세지 박스를 열어 준 다음, ChatDetailPage에 접근하면 그 박스를 던져주고 연산 없이 읽고 쓸 수 있게 해줘야 한다... 는 걸 머리로는 알고 있지만, 그냥 코드가 생겨 먹은 대로 쓰려고 짱구를 굴리다 보니 이런 요오상한 코드가 탄생했다.

사실 그렇게 해주려고 이것저것 찾아보고 provider안에서 openBox해주도록 짜봤는데, provider의 Scope를 거의 최상위 위젯으로 두고 있어서, openBox하는 타이밍이 main()이 아니면 에러가... ㅎ...  
원하는 기능이 그럴 듯하게 구현이 되기는 했지만, 이건 너무 눈가리고 아웅하는 식이라 hive를 좀 더 동적으로 다룰 수 있는 방법이 있나 찾아보고, 시도해 볼 예정이다. 나의 실력에 RIP,,,  

## 메세지 로그 저장

`ChatDetailPage`에서 `send` 버튼을 누를 때, `addMessage(newMsg)` 라는 함수를 실행 

```dart
... // 생략
                    SizedBox(
                      height: 30,
                      width: 50,
                      child: FloatingActionButton(
                        heroTag: "btn_send", 
                        onPressed: () {
                          if (msgtextController.text.isNotEmpty) {
                            ChatMessage newMsg = ChatMessage(
                                userid: widget.cpuser.id,
                                messageContent: msgtextController.text,
                                messageType: "sender");
                            debugPrint('Input text : ${newMsg.messageContent}');
                            setState(() {
                              _mctlr.addMessage(newMsg); // 추가
                              msgtextController.clear();
                              latest_msg = msgtextController.text;
                            });
                            _scrollToLatest();
                          }
                        },
                        child: Icon(
                          Icons.send,
                          color: Colors.white,
                          size: 18,
                        ),
                        backgroundColor: Colors.blue,
                        elevation: 0,
                      ),
                    ),
... // 생략
```
{: file="/lib/screen/ChatDetailPage.dart" }

messages provider `MessageListController`의 `addMessage`함수는 이렇게 생겼다. (이건 그냥 `addUser`와 똑같이 생겼음)

```dart
... // 생략
  void addMessage(ChatMessage message) {
    final _messages = [...state.messages, message];
    state = state.copyWith(messages: _messages);
    messageList!.put(message.messageid, message);
  }
... // 생략
```
{: file="/lib/provider/messages.dart" }

기본적인 읽기, 쓰기기는 이렇다. 여기까지 했으면, (비록 효율적이지는 않지만) 유저 별로 메세지의 저장이 가능한 기본적인 기능을 하는 것처럼 보이는 채팅앱이 완성된다.

![Flutter Simple chat UI ex 03-04](SimulSc_ex_03-04.gif){:width="30%" height="30%"}
 
(데모에서는 `ChatPage`로 돌아갔을 때, 유저 이름 밑에 유저와의 마지막 메세지가 표시되도록 하고 있는데, 이에 대해서는 이 포스팅 마지막 장에 기술)

## UI 변경, 기능 추가 (옵션)

이번 장은 그냥 나 편하자고 이하의 UI 변경, 기능 추가를 한 내용이다.   
- 받은 메세지 조작 : receiver 버튼, 기능 추가
- 메세지 초기화 : clear 버튼, 기능 추가
- 개별 메세지에 대한 기능 추가 (개별 메세지 복사, 수정, 삭제)

### 받은 메세지 조작 : receiver 버튼, 기능 추가

원래는 내가 보내는 버튼만 있는게 맞지만, 여기서는 그럴듯하게 채팅창을 모방하기 위해 내가 받는 메세지도 조작할 수 있도록 `send`버튼 왼쪽 옆에 `receive`버튼을 하나 더 추가해 주었다. 텍스트 필드에 글을 입력하고 `receive`버튼을 누르면 해당 메세지가 나에게 온 메세지처럼 표시하는 기능을 한다.

기본적으로는 `send`버튼의 포멧을 그대로 사용하고, 전달하는 `messageType`을 `receiver` 해주면 된다. (+ Icon 변경)

```dart
... // 생략
                    SizedBox(
                      height: 30,
                      width: 50,
                      child: FloatingActionButton(
                        heroTag: "btn_receive", //heroTag 중복에러 방지 (heroTag: null도 가능)
                        onPressed: () {
                          if (msgtextController.text.isNotEmpty) {
                            ChatMessage newMsg = ChatMessage(
                                userid: widget.cpuser.id,
                                messageContent: msgtextController.text,
                                messageType: "receiver"); //수정
                            setState(() {
                              _mctlr.addMessage(newMsg);
                              msgtextController
                                  .clear(); 
                              latest_msg = msgtextController.text;
                            });
                            _scrollToLatest(); 
                          }
                        },
                        child: Icon(
                          Icons.send_outlined,
                          color: Colors.white,
                          size: 18,
                        ),
                        backgroundColor: Colors.blue,
                        elevation: 0,
                      ),
                    ),
... // 생략
```
{: file="/lib/screens/chatDetailPage.dart" }

만약 receiver버튼 추가하고 이런 에러가 난다면, 

```error
════════ Exception caught by scheduler library ═════════════════════════════════
The following assertion was thrown during a scheduler callback:
There are multiple heroes that share the same tag within a subtree.

Within each subtree for which heroes are to be animated (i.e. a PageRoute subtree), each Hero must have a unique non-null tag.
In this case, multiple heroes had the following tag: <default FloatingActionButton tag>
Here is the subtree for one of the offending heroes: Hero
    tag: <default FloatingActionButton tag>
    state: _HeroState#a0c4c
```

`heroTag`를 설정하지 않은 `FloatingActionButton`이 같은 위젯에 여러개 있어서 그런 것이니, 각 `FloatingActionButton`위젯에 `herotag`를 설정해 주자. (모든 `FloatingActionButton`에 `heroTag: null`이라고 해줘도 되긴 됨)

결과, 이러한 UI 된다. 왼쪽 버튼을 누르면 내가 받은 메세지가 되고, 오른쪽 버튼을 누르면 내가 보낸 메세지가 된다.

![Flutter Simple chat UI ex 03-01](SimulSc_ex_03-01.png){:width="50%" height="50%"}

### 메세지 초기화 : clear 버튼, 기능 추가

`ChatPage`의 유저 목록 Clear와 마찬가지로 `AppBar`에 메세지 로그의 모든 메세지을 지우는 버튼을 추가했다. (+덤으로 설정 버튼이 그냥 `Icon`으로만 되어있길래 누를 수 있도록 `IconButton`으로 변경해 주었음, 기능은 추가X)
```dart
... // 생략
    final _mctlr = context.read<MessageListController>();
... // 생략
                IconButton( // 메세지 로그 Clear
                  onPressed: () {
                    print("Click Clear Button");
                    _mctlr.clearUserMsg(widget.cpuser.id); //추가
                  },
                  alignment: Alignment.centerRight,
                  icon: const Icon(
                    Icons.clear_all,
                    color: Colors.black,
                  ),
                ),
                IconButton( // Setting
                  onPressed: () {
                    print("Click Setting Button");
                  },
                  alignment: Alignment.centerRight,
                  icon: const Icon(
                    Icons.settings,
                    color: Colors.black54,
                  ),
                ),
... // 생략
```
{: file="/lib/screens/chatDetailPage.dart" }

추가 후의 UI는 이런 모습.

![Flutter Simple chat UI ex 03-02](SimulSc_ex_03-02.png){:width="50%" height="50%"}

기능 구현을 위해서 message provider(`/providers/messages.dart`)의 `MessageListController`에  `clearUserMsg` 라는 함수를 추가해 주었다. 동작방식은 `userid`를 인수로 받아, 기존의 메세지 리스트(`messages`)에서 `userid`를 가진 메세지를 솎아낸 메세지 리스트를 만들어(`_messages`), 기존의 메세지 리스트(`messages`)를 해당 메세지 리스트(`_messages`)로 교체하는 것이다.   
hive box에서의 삭제는 좀 더 쉽다. `.delete`함수를 사용하여 `userid`를 가진 개체를 지워주면 된다.

```dart
... // 생략

  void clearUserMsg(String userid) {
    final _messages = state.messages.where((m) => m.userid != userid).toList();

    state = state.copyWith(messages: _messages);
    messageList!.delete(userid);
  }
... // 생략
```
{: file="/lib/providers/messages.dart" }

위의 두 기능에 대한 데모. 잘 구현되었다는 걸 확인할 수 있다.

![Flutter Simple chat UI ex 03-03](SimulSc_ex_03-03.gif){:width="30%" height="30%"}

### 개별 메세지에 대한 기능 추가 (개별 메세지 복사, 수정, 삭제)

개별 메세지를 롱터치하면 메뉴가 나오고, 메뉴를 통해 해당 메세지를 복사, 수정, 삭제가 가능하도록 기능을 추가했다.

메세지를 롱터치 했을 때, 메뉴가 나오게 하는 방법은 [Flutter: Showing a Context Menu on Long Press](https://www.kindacode.com/article/flutter-showing-a-context-menu-on-long-press/)와 [stackoverflow](https://stackoverflow.com/questions/55069228/flutter-popupmenubutton-onlongpressed)의 코드를 참고했다. 

사실 두 사이트에 있는 솔루션 그대로 코드를 작성하면 내가 원하는 대로 움직이지를 않길래 (메뉴 박스가 롱터치한 위치에서 제대로 나오지 않고 x축 또는 y축이 고정되어있었음.. 반쪽짜리 기능 됨) 입맛대로 고쳐서 원하는 대로 움직이게 만들긴 했는데, 얼레벌레 성공해 버려서 솔직히 [RelativeRect](https://api.flutter.dev/flutter/rendering/RelativeRect-class.html) 클래스의 동작 방식을 아직 완전히 이해하지는 못했다. 두 사각형을 그려서 상대적인 위치를 계산해 박스를 출력하는것 같긴한데... 나중에 문서 좀 제대로 읽어봐야 겠다. 일단 내가 작성한 얼레벌레 코드는 이렇다.

```dart
... // 생략
class _MessageListState extends State<MessageList> {
... // 생략
  Offset _tapPosition = Offset.zero;

  void _getTapPosition(TapDownDetails details) {
    print("on Tap Down");
    final RenderBox referenceBox = context.findRenderObject() as RenderBox;

    setState(() {
      _tapPosition = referenceBox.globalToLocal(details.globalPosition);
    });
  }

  final widgetKey = GlobalKey();

  RelativeRect _getRelativeRect(GlobalKey key) {
    return RelativeRect.fromSize(
        _getWidgetGlobalRect(key), const Size(10000, 200));
  }

  Rect _getWidgetGlobalRect(GlobalKey key) {
    final RenderBox renderBox =
        key.currentContext!.findRenderObject() as RenderBox;
    var offset = renderBox.localToGlobal(Offset.zero);

    final RenderObject? overlay =
        Overlay.of(context)?.context.findRenderObject();

    return Rect.fromLTWH(_tapPosition.dx, offset.dy, renderBox.size.width,
        renderBox.size.height);
  }

  void _showContextMenu(BuildContext context, GlobalKey key) async {
    final _mctlr = context.read<MessageListController>(); // 동작 제어를 위해 미리 추가

    final result = await showMenu(
        context: context,
        position: _getRelativeRect(key),
        items: [
          const PopupMenuItem(
            value: 'copy',
            child: Text('Copy Text'),
          ),
          const PopupMenuItem(
            value: 'edit',
            child: Text('Edit Text'),
          ),
          const PopupMenuItem(
            value: 'delete',
            child: Text('Delete Text'),
          ),
        ]);

    switch (result) {
      case 'copy':
        debugPrint('Copy Text');
        // 복사 동작 추가 부분
        break;
      case 'edit':
        debugPrint('Edit Text');
        // 수정 동작 추가 부분
        break;
      case 'delete':
        debugPrint('Delete Text');
        // 삭제 동작 추가 부분
        break;
    }
  }

  @override
  Widget build(BuildContext context) {
    return GestureDetector(
      // 추가
      key: widgetKey,
      onTapDown: (details) {
        _getTapPosition(details); 
      },
      onLongPress: () {
        _showContextMenu(context, widgetKey);
      },
      // 추가 END
      child: Container(
... // 생략
```
{: file="/lib/providers/messages.dart" }

#### 복사  

`services.dart` 라는 패키지의 `Clipboard.setData()`라는 함수를 통해서 쉽게 구현할 수 있다.

```dart
... // 생략
import 'package:flutter/services.dart'; // 추가
... // 생략
  void _showContextMenu(BuildContext context) async {
... // 생략
    switch (result) {
      case 'copy':
        debugPrint('Copy Text');
        Clipboard.setData(ClipboardData(text: widget.currentmsg.messageContent)); // 추가
        break;
... // 생략
```
{: file="/lib/widgets/messageList.dart" }

#### 수정  

먼저 message provider `MessageListController`에 `editMessage()`라는 함수를 작성해 주었다. 버튼이 눌리면 텍스트를 수정할 수 있는 팝업을 실행(`InputDialog`), 팝업창에서 텍스트의 수정이 발생한 경우, `editMessage()`함수를 실행해 주었다. 

```dart
... // 생략
  void _showContextMenu(BuildContext context) async {
    final _mctlr = context.read<MessageListController>();
... // 생략
    switch (result) {
... // 생략
      case 'edit':
        debugPrint('Edit Text');
        String? inputtext = await InputDialog(context);
        _textFieldController.clear();

        if (inputtext == "" ||
            inputtext == null ||
            inputtext == widget.currentmsg.messageContent) {
          print("Text not changed!");
        } else {
          String edittext = inputtext;
          _mctlr.editMessage(widget.currentmsg, edittext);
          setState(() {
            widget.currentmsg.messageContent = edittext; //화면에서도 곧 바로 수정
          });
        }
        break;
... // 생략
```
{: file="/lib/widgets/messageList.dart" }

`InputDialog()`함수는 별다를 것 없이 생겼는데, 이번에는 기존의 텍스트를 수정하는 거니까 텍스트 필드에 기존에 메세지 내용이 디폴트로 들어가 있고 그것을 수정하도록 `TextEditingController`의 포멧을 조금 변경해 주었다.

```dart
... // 생략
class _MessageListState extends State<MessageList> {
  late TextEditingController _textFieldController;

  @override
  void initState() {
    super.initState();
    _textFieldController = TextEditingController(
        text: widget.currentmsg.messageContent); // 기존 텍스트가 텍스트필드의 초기값
  }

  Future<String?> InputDialog(BuildContext context) async {
    final ButtonStyle flatButtonStyle = TextButton.styleFrom(
      backgroundColor: Color.fromARGB(255, 100, 100, 9100),
      padding: EdgeInsets.all(0),
    );
    _textFieldController = TextEditingController(
        text: widget.currentmsg.messageContent); // 기존 텍스트가 텍스트필드의 초기값

    return showDialog(
        context: context,
        builder: (context) {
          return AlertDialog(
            title: Text('메세지 수정'),
            content: TextField(
              controller: _textFieldController,
              decoration:
                  InputDecoration(hintText: widget.currentmsg.messageContent),
            ),
            actions: <Widget>[
              ElevatedButton(
                child: const Text("취소"),
                onPressed: () => Navigator.pop(context),
              ),
              ElevatedButton(
                child: const Text('완료'),
                onPressed: () =>
                    Navigator.pop(context, _textFieldController.text),
              ),
            ],
          );
        });
  }
... // 생략
```
{: file="/lib/widgets/messageList.dart" }

`editMessage()`에서는 `messageid`속성을 이용하여 해당 메세지를 식별하여 해당 메세지의 `messageContent`를 입력 받은 텍스트로 수정한다.

```dart
... // 생략
class MessageListController extends StateNotifier<MessageListPrState> {
... // 생략
  void editMessage(ChatMessage message, String edittext) {
    for (int index = 0; index < [...state.messages].length; index++) {
      if ([...state.messages][index].messageid == message.messageid) {
        [...state.messages][index].messageContent = edittext;
        messageList!.put(
            message.messageid,
            ChatMessage(
                userid: message.userid,
                messageid: message.messageid,
                messageContent: edittext,
                messageType: message.messageType));
        break;
      } else {
        continue;
      }
    }
  }

... // 생략
```
{: file="/lib/providers/messages.dart" }

#### 삭제  

message provider `MessageListController`에 `deleteMessage()`라는 함수를 작성해 주고, 버튼이 눌리면 함수를 실행해 주었다. 

```dart
... // 생략
  void _showContextMenu(BuildContext context) async {
    final _mctlr = context.read<MessageListController>();
... // 생략
    switch (result) {
... // 생략
      case 'delete':
        debugPrint('Delete Text');
        _mctlr.deleteMessage(widget.currentmsg);
        break;
... // 생략
```
{: file="/lib/widgets/messageList.dart" }

`deleteMessage()`에서는 `messageid`속성을 이용하여 해당 메세지만 배제한 메세지 리스트를 만들어낸다.

```dart
... // 생략
class MessageListController extends StateNotifier<MessageListPrState> {
... // 생략
  void deleteMessage(ChatMessage message) {
    final _messages =
        state.messages.where((m) => m.messageid != message.messageid).toList();

    state = state.copyWith(messages: _messages);
    messageList!.delete(message.userid);
  }
... // 생략
```
{: file="/lib/providers/messages.dart" }

이 기능들의 실행 데모는 다음과 같다.

![Flutter Simple chat UI ex 03-05](SimulSc_ex_03-05.gif){:width="30%" height="30%"}

# ChatPage : 유저 데이터 갱신

## 각 유저의 최근 메세지 표시
이렇게 유저별 채팅 데이터를 저장해 주면, `ChatPage`화면에 출력 되는 각 유저의 최근메세지를 갱신하는 의미가 생긴다. 유저 리스트 UI를 표시하는 `conversationList.dart`에서 유저 이름 밑에 표시하는 텍스트를 수정하면 된다.

```dart
... // 생략
  Widget build(BuildContext context) {
    // 추가
    final _mstate_currentuser_len =
        context.watch<MessageListPrState>().checkUserMsg(widget.user.id).length; 
    bool msgtext_null = _mstate_currentuser_len == 0; 
    widget.user.messageText = msgtext_null
        ? " "
        : _mstate_currentuser[_mstate_currentuser_len - 1].messageContent;
    // 추가 END
    return GestureDetector(
      onTap: () async {
        await Navigator.of(context).push(MaterialPageRoute(
          builder: (context) {
            return ChatDetailPage(
              cpuser: widget.user,
            );
          },
        ));
        // 추가
        if (mounted) {
          setState(() {
          });
        }
        // 추가 END
      },
      child: Container(
        padding: EdgeInsets.only(left: 16, right: 16, top: 10, bottom: 10),
        child: Row(
          children: <Widget>[
            Expanded(
              child: Row(
                children: <Widget>[
                  InkWell(
                    ... // 생략
                  ),
                  Expanded(
                    child: Container(
                      color: Colors.transparent,
                      child: Column(
                        crossAxisAlignment: CrossAxisAlignment.start,
                        children: <Widget>[
                          Text(
                            widget.user.name,
                            style: TextStyle(fontSize: 16),
                          ),
                          SizedBox(
                            height: 6,
                          ),
                          Text(
                            widget.user.messageText, // <- 여기에 나타다는 글씨를 수정
                            style: TextStyle(
                                fontSize: 13,
                                color: Colors.grey.shade600,
                                fontWeight: widget.isMessageRead
                                    ? FontWeight.bold
                                    : FontWeight.normal),
                          ),
                        ],
                      ),
                    ),
                  ),
... // 생략
```
{: file="/lib/widget/conversationList.dart" }

`msgtext_null`을 이용한 연산을 굳이 추가한 이유는, 유저를 생성했을 때 예외 처리가 필요하기 때문이다. 유저를 생성한 직 후라면 당연하지만 해당 유저와 관련된 메세지는 없을 것이다. 그러므로 해당 유저에 대한 메세지가 없을 때는, `" "`를 넣어 주어서 아무 것도 입력되지 않은 것 처럼 보여주도록 코드를 작성했다. 메세지가 있을 때는, 해당 유저와 나눈 메세지 리스트 중 가장 마지막에 있는 항목을 가져오도록 하면된다.

또한,`ChatDetailPage`에서 마지막 메세지가 수정된 경우에는 `conversationList` 위젯도 상태 갱신을 해주어야하기 때문에, `onTap{}` 에 `setState()`를 추가해 주었다.

- **if (mounted) {}** 조건문을 건 이유

상태 갱신을 위해서 `setState()`만을 추가해 주면, 이런 에러가 뜬다.

```error
[VERBOSE-2:ui_dart_state.cc(198)] Unhandled Exception: setState() called after dispose(): _ConversationListState#8ffd2(lifecycle state: defunct, not mounted)
This error happens if you call setState() on a State object for a widget that no longer appears in the widget tree (e.g., whose parent widget no longer includes the widget in its build). This error can occur when code calls setState() from a timer or an animation callback.
The preferred solution is to cancel the timer or stop listening to the animation in the dispose() callback. Another solution is to check the "mounted" property of this object before calling setState() to ensure the object is still in the tree.
This error might indicate a memory leak if setState() is being called because another object is retaining a reference to this State object after it has been removed from the tree. To avoid memory leaks, consider breaking the reference to this object during dispose().
#0      State.setState.<anonymous closure> (package:flutter/src/widgets<…>
```

내용을 보면 아마 `onTap{}`에서 비동기 처리 중에 상태 갱신을 위해 `setState()`를 실행 했기 때문에 예외가 발생한 것 같다. 에러에서 `Another solution is to check the "mounted" property of this object before calling setState() to ensure the object is still in the tree.`라고 해결책을 주었길래, `mounted`를 체크하는 조건문을 추가해 주었더니 에러는 사라졌다. 당장 에러는 사라졌지만 사실 여전히 문제는 남아있고, 더 좋은 해결 방법이 있을 거다... 일단 보류하고 수정하면 포스팅을 업데이트 할 예정.

> #수정 보류중인 오류  
> 메세지를 입력한 후 `ChatPage`로 돌아오지 않고 `ChatDetailPage`에서 바로 마지막 메세지를 수정하는 후, `ChatPage`로 돌아오는 경우 수정 전의 메세지가 표시 된다. `onTap`의 비동기 처리로 인해 provider에서 데이터가 업데이트 되기 전에 `ChatPage`의 상태 갱신이 먼저 이루어지기 때문으로 보인다.

완성된 화면

![Flutter Simple chat UI ex 03-06](SimulSc_ex_03-06.gif){:width="30%" height="30%"}

***

# 다음 포스팅 내용
기본기능은 얼추 다 넣은 것 같은데,  
- provider/hive 사용 방식 개선
- 수정 보류 중인 오류들 개선
- 재밌어 보이는 플러그 인이 있으면 적용
- 진짜 메세지를 주고 받을 수 있도록
  
혹시 새 프로젝트를 짠다면  
- 구체적으로 뭘 만들기 정하기
- UX/UI 공부를 해서 좀더 사용자 친화적인 프로그램 만들기
를 해보고 싶다.

UI에 대해서는 미적감각이 거의 없다 싶은 사람이고 UX/UI는 관련지식이 1도 없기 때문에, 이론 공부부터 해야한다. 공부하면 어떻게든 되겠지 허허

***
# 후기
음... 공부를 더 하도록... 이런식의 얼레벌레 코드를 어따 써