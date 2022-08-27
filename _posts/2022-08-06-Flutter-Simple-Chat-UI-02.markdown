---
# multilingual page pair id, this must pair with translations of this page. (This name must be unique)
#lng_pair: "id_study_flutter"
title: "Flutter 01 Simple Chat UI 02 Chat Detail 화면"

# post specific
author: kae
# publish date
date: 2022-08-06 13:47:00 +0900
category: [Flutter, Simple Chat]
tags: [flutter, practice, programming]

# disable comments on this page
#comments_disable: true

# thumbnail image for post
# img: ":Flutter.jpeg"

# data path
img_path: /assets/img/2022-08-06-Flutter-Simple-Chat-UI-02/
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

을 따라서 작성한 Flutter Simple Chat UI 에 관한 내용이다.

관련 포스트
- [Base, Chat 화면 구현](https://kaesknzxvf.github.io/posts/Flutter-Simple-Chat-UI-01/)
- Chat Detail 화면 구현 (현재 포스트)
- [응용 01 (Chat Detail 화면) 화면 간 데이터 전달, 메세지 입력](https://kaesknzxvf.github.io/posts/Flutter-Simple-Chat-UI-EX01/)

***
# 환경

- MacBook Air (M1, 2020)
- macOS Monterey (v12.4)

- Flutter 3.0.3
- Dart 2.17.5

- Visual Studio Code 1.69.0

***

# Chatpage Detail : 화면 생성, 클릭 이벤트 적용

`/lib/screens/` 폴더에 `chatDetailPage.dart` 생성하여 이하의 베이스 코드를 작성해 준다.

```dart
import 'package:flutter/material.dart';

class ChatDetailPage extends StatefulWidget{
  @override
  _ChatDetailPageState createState() => _ChatDetailPageState();
}

class _ChatDetailPageState extends State<ChatDetailPage> {

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text("Chat Detail"),
      ),
      body: Container()
    );
  }
}
```
{: file="/lib/screens/chatDetailPage.dart" }

_ChatDetailPageState 클래스는   
AppBar: 텍스트  
body: 빈 컨테이너  
가 포함된 Scaffold 위젯을 반환한다.

conversationList에 표시된 개체 하나가 선택되었을 때, 이 화면을 활성화 해 주어야 하기 때문에, 

`/lib/models/conversationList.dart` 파일의 `GestureDetector`위젯, `onTap` 키의 값으로 해당 동작을 하는 코드를 추가해 줘야 한다.

그전에 잊지 말자 임포트
```dart
import 'package:flt_20220703_simple_chatapp/screen/chatDetailPage.dart';
```
{: file="/lib/models/conversationList.dart" }

```dart
    onTap: () {
        Navigator.push(context, MaterialPageRoute(builder: (context) {
          return ChatDetailPage();
        }));
      },
```
{: file="/lib/models/conversationList.dart" }

![Flutter Simple chat UI 실행 화면 chatdetail 01](SimulSc_chatdetail_01.gif){:width="30%" height="30%"}

그럼 이렇게 클릭 이벤트가 적용된다.


# Chatpage Detail UI 구현
## 헤더 : UI 변경

방금 만든 화면의 헤더에는 `Chat Detail`라는 텍스트만 써있지만, 보통은 대화를 나누는 사람의 정보가 들어간다. 그러니 대화 상대의 정보가 들어가는 UI를 만들어주자.

헤더 정보를 수정하고 싶은거니까 어디를 수정해야한다?   
`appBar`키의 값을 수정해야한다~

그냥 텍스트만 달랑 써주는 `title: Text("Chat Detail"),`을 지워주고 이하의 코드를 작성한다.

```dart
      appBar: AppBar(
        elevation: 0,
        automaticallyImplyLeading: false,
        backgroundColor: Colors.white,
        flexibleSpace: SafeArea(
          child: Container(
            padding: EdgeInsets.only(right: 16),
            child: Row(
              children: <Widget>[
                IconButton(
                  onPressed: (){
                    Navigator.pop(context);
                  },
                  icon: Icon(Icons.arrow_back,color: Colors.black,),
                ),
                SizedBox(width: 2,),
                CircleAvatar(
                  backgroundImage: NetworkImage("https://randomuser.me/api/portraits/men/5.jpg"),
                  maxRadius: 20,
                ),
                SizedBox(width: 12,),
                Expanded(
                  child: Column(
                    crossAxisAlignment: CrossAxisAlignment.start,
                    mainAxisAlignment: MainAxisAlignment.center,
                    children: <Widget>[
                      Text("Kriss Benwat",style: TextStyle( fontSize: 16 ,fontWeight: FontWeight.w600),),
                      SizedBox(height: 6,),
                      Text("Online",style: TextStyle(color: Colors.grey.shade600, fontSize: 13),),
                    ],
                  ),
                ),
                Icon(Icons.settings,color: Colors.black54,),
              ],
            ),
          ),
        ),
      ),
```
{: file="/lib/screens/chatDetailPage.dart" }

그럼 헤더에 `Chat Detail`이라는 텍스트 대신, 이하와 같은 UI가 보일 것이다.

![Flutter Simple chat UI 실행 화면 chatdetail 02](SimulSc_chatdetail_02.png){:width="30%" height="30%"}

## 바텀 : 텍스트 박스 추가

이제 하단에 메세지를 입력할 수 있는 텍스트 박스와 송신 버튼, 이미지 등을 추가하기 위한 버튼을 배치해 주자.  
현재의 코드에서는 `_ChatDetailPageState` 클래스의 바디가 여전히 컨테이너만을 가지고 있을 것이다. ( `body: Container())` )  
그러므로 `body` 의 값을 텍스트 박스, 버튼 개체를 표시해주는 코드로 변경한다.  
여기서는 이 개체들을 가로로 배치해 주고 싶기 떄문에 정렬(`Align`) 위젯을 사용, child 키의 값에 `Row` 위젯을 배치하여 그 안에 개채들을 넣어 주고 있다.

```dart
      body: Stack(
        children: <Widget>[
          Align(
            alignment: Alignment.bottomLeft,
            child: Container(
              padding: EdgeInsets.only(left: 10,bottom: 10,top: 10),
              height: 60,
              width: double.infinity,
              color: Colors.white,
              child: Row(
                children: <Widget>[
                  GestureDetector(
                    onTap: (){
                    },
                    child: Container(
                      height: 30,
                      width: 30,
                      decoration: BoxDecoration(
                        color: Colors.lightBlue,
                        borderRadius: BorderRadius.circular(30),
                      ),
                      child: Icon(Icons.add, color: Colors.white, size: 20, ),
                    ),
                  ),
                  SizedBox(width: 15,),
                  Expanded(
                    child: TextField(
                      decoration: InputDecoration(
                        hintText: "Write message...",
                        hintStyle: TextStyle(color: Colors.black54),
                        border: InputBorder.none
                      ),
                    ),
                  ),
                  SizedBox(width: 15,),
                  FloatingActionButton(
                    onPressed: (){},
                    child: Icon(Icons.send,color: Colors.white,size: 18,),
                    backgroundColor: Colors.blue,
                    elevation: 0,
                  ),
                ],
                
              ),
            ),
          ),
        ],
      ),
```
{: file="/lib/screens/chatDetailPage.dart" }

![Flutter Simple chat UI 실행 화면 chatdetail 03](SimulSc_chatdetail_03.png){:width="30%" height="30%"}

## 메세지 로그 : 메세지 추가

이제 메세지 로그가 띄워지는 화면에 그럴 듯한 메세지가 띄워지는 것 같은 UI를 만들기 위해 일단 메세지를 입력해 줄 것이다.

`/lib/models/` 폴더에 `chatMessageModel.dart` 생성하여 메세지 클래스를 작성해 준다.

```dart
import 'package:flutter/cupertino.dart';

class ChatMessage{
  String messageContent;
  String messageType;
  ChatMessage({required this.messageContent, required this.messageType});
}
```
{: file="/lib/models/chatMessageModel.dart" }

> 여기서도 `ChatMessage` 클래스와 [같은이유](https://dart.dev/null-safety/faq#how-does-required-compare-to-the-new-required-keyword)로, 원문과 같은 `@required` 어노테이션(Annotation)이 아닌 `requried` 로 대체해 주자 
{: .prompt-info}

다시 `chatDetailPage.dart` 파일로 돌아와서 표시될 메세지 리스트를 만들어 준다. 지금은 하드코딩(ㅋㅋ)을 해주지만 나중에는 수신된 메세지 리스트를 받아오면 될 것이다.

먼저 작성한 모델을 임포트 해준후 

```dart
import 'package:flt_20220703_simple_chatapp/models/chatMessageModel.dart';
```
{: file="/lib/screens/chatDetailPage.dart" }

이하의 리스트를 `_ChatDetailPageState` 클래스에 넣어주자. `messageContent`의 내용은 마음대로.

```dart
  List<ChatMessage> messages = [
    ChatMessage(messageContent: "Hello, Will", messageType: "receiver"),
    ChatMessage(messageContent: "How have you been?", messageType: "receiver"),
    ChatMessage(messageContent: "Hey Kriss, I am doing fine dude. wbu?", messageType: "sender"),
    ChatMessage(messageContent: "ehhhh, doing OK.", messageType: "receiver"),
    ChatMessage(messageContent: "Is there any thing wrong?", messageType: "sender"),
  ];
```
{: file="/lib/screens/chatDetailPage.dart" }

이제 데이터는 준비되었으니 이 데이터를 표시하는 UI를 추가해주자.  
이 메세지가 표시되는 곳은 `body`이지만, 바로 전 챕터에서 작성한 바텀의 윗부분에 배치해 주어야 하기 때문에 `Stack` 위젯 안쪽, 정렬(`Align`) 위젯 위쪽에, `children` 키를 추가하여 그 값으로 리스트 뷰를 빌드해 줘야한다.

```dart
        ListView.builder(
            itemCount: messages.length,
            shrinkWrap: true,
            padding: EdgeInsets.only(top: 10,bottom: 10),
            physics: NeverScrollableScrollPhysics(),
            itemBuilder: (context, index){
              return Container(
                padding: EdgeInsets.only(left: 16,right: 16,top: 10,bottom: 10),
                child: Text(messages[index].messageContent),
              );
            },
          ),
```
{: file="/lib/screens/chatDetailPage.dart" }

그럼 화면이 이렇게 데이터가 보여지게 된다.

![Flutter Simple chat UI 실행 화면 chatdetail 04](SimulSc_chatdetail_04.png){:width="30%" height="30%"}

## 메세지 로그 : 송신자, 수신자 UI 작성

메세지가 띄워지긴 헀지만, 채팅 페이지 같지는 않다. 이 메세지 내용들의 송신자, 수신자를 구분할 수 있도록 말풍선 UI안에 메세지가 띄워질 수 있도록 만들어 주자.

방금 전 추가해 주었던 `ListView`의 `itemBuilder`키를 손봐줄 것이다.
현재는 단순히 텍스트를 표시해주고 있는 것을 알 수 있는데 (`child: Text(messages[index].messageContent),`), 이곳을 `Text`위젯이 아닌 `Align`위젯으로 변경해 준 후, 배치할 요소들을 넣어줄 것이다.

```dart
        ListView.builder(
          itemCount: messages.length,
          shrinkWrap: true,
          padding: EdgeInsets.only(top: 10,bottom: 10),
          physics: NeverScrollableScrollPhysics(),
          itemBuilder: (context, index){
            return Container(
              padding: EdgeInsets.only(left: 16,right: 16,top: 10,bottom: 10),
              child: Align(
                alignment: (messages[index].messageType == "receiver"?Alignment.topLeft:Alignment.topRight),
                child: Container(
                decoration: BoxDecoration(
                    borderRadius: BorderRadius.circular(20),
                    color: (messages[index].messageType  == "receiver"?Colors.grey.shade200:Colors.blue[200]),
                ),
                padding: EdgeInsets.all(16),
                child: Text(messages[index].messageContent, style: TextStyle(fontSize: 15),),
                ),
              ),
            );
          },
        ),
```
{: file="/lib/screens/chatDetailPage.dart" }

![Flutter Simple chat UI 실행 화면 chatdetail 05](SimulSc_chatdetail_05.png){:width="30%" height="30%"}

핫 로드를 해주면 이런 완성 화면이 나온다!

***
# 후기
작성자가 [Flutter chat app](https://instaflutter.com/app-templates/flutter-chat-app/) 이라는 페이지를 소개하고 있는데 이런 채팅 앱의 템플릿을 판매하는 사이트니 참고할 사람은 참고하시길.

개별 스크린 설계, 클릭 이벤트 같은 기본적인 UI 구현을 익히기 좋은 예제 같다.  
이 예제를 통해 만든 UI를 기반으로 화면 간의 값 전달이나 실제로 메세지를 주고 받을 수 있는 소켓 프로그래밍을 하면 결과가 눈에 보이면서 재밌을 것 같으니 해보도록 하겠다.