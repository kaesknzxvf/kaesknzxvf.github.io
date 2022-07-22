---
# multilingual page pair id, this must pair with translations of this page. (This name must be unique)
lng_pair: "id_study_flutter"
title: "Flutter 01 Simple Chat UI"

# post specific
# if not specified, .name will be used from _data/owner/[language].yml
author: Kae
# multiple category is not supported
category: Flutter
# multiple tag entries are possible
tags: [flutter, practice, programming]
# thumbnail image for post
img: ":Flutter.jpeg"
# disable comments on this page
#comments_disable: true

# publish date
date: 2022-07-10 13:10:06 +0900
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

Flutter 가보자고

<!-- outline-end -->

이 포스트는

- [How to Build a Chat App UI With Flutter and Dart](https://www.freecodecamp.org/news/build-a-chat-app-ui-with-flutter/)

을 따라서 작성한 Flutter Simple Chat UI 에 관한 내용이다.

***
## 환경

- MacBook Air (M1, 2020)
- macOS Monterey (v12.4)

- Flutter 3.0.3
- Dart 2.17.5

- Visual Studio Code 1.69.0

***

## 메인 화면 생성


`/lib/main.dart` 파일 내용을

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(MyApp());
}

class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Flutter Demo',
      theme: ThemeData(
        primarySwatch: Colors.blue,
      ),
      debugShowCheckedModeBanner: false,
      home: Container(),
    );
  }
}
```
로 변경

`lib`폴더에 `screen`폴더를 추가 `homePage.dart` 파일을 생성

`/lib/screen/homePage.dart` 파일에 

```dart
import 'package:flutter/material.dart';

class HomePage extends StatelessWidget{
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: Container(
        child: Center(child: Text("Chat")),
      ),

    );
  }
}
```
을 입력

`/lib/main.dart` 의 `home: Container(),` 을
```
home: HomePage(),
```
로 변경

![Flutter Simple chat UI 실행 화면 01](/assets/img/2022-07-10-Flutter-Simple-Chat-UI/SimulSc_chat_01.png){:width="30%" height="30%"}

## 메인 화면: 하단바 추가

`/lib/screen/homePage.dart` 파일의 `Scaffold` 클래스의 내용을 변경해주자

```dart
return Scaffold(
      body: Container(
        child: Center(child: Text("Chat")),
      ),
      bottomNavigationBar: BottomNavigationBar(
        selectedItemColor: Colors.red,
        unselectedItemColor: Colors.grey.shade600,
        selectedLabelStyle: TextStyle(fontWeight: FontWeight.w600),
        unselectedLabelStyle: TextStyle(fontWeight: FontWeight.w600),
        type: BottomNavigationBarType.fixed,
        items: [
          BottomNavigationBarItem(
            icon: Icon(Icons.message),
            label: 'Chats',
          ),
          BottomNavigationBarItem(
            icon: Icon(Icons.group_work),
            label: 'Channels',
          ),
          BottomNavigationBarItem(
            icon: Icon(Icons.account_box),
            label: 'Profile',
          ),
        ],
      ),
    );
  }
  ```

  원문에는 `BottomNavigationBarItem` 에 타이틀 속성으로 텍스트를 입력하고 있는데, 버전 1.22 이후에는 title 대신 label 속성을 쓰도록 바뀌었다  
  [Bottom Navigation Title To Label](https://docs.flutter.dev/release/breaking-changes/bottom-navigation-title-to-label)

  ![Flutter Simple chat UI 실행 화면 02](/assets/img/2022-07-10-Flutter-Simple-Chat-UI/SimulSc_chat_02.png){:width="30%" height="30%"}

## Chatpage : 대화 목록 화면 구현


`lib/screen`폴더에 `chatPage.dart` 파일을 생성

`/lib/screen/chatPage.dart` 파일에 

```dart
import 'package:flutter/material.dart';

class ChatPage extends StatefulWidget {
  @override
  _ChatPageState createState() => _ChatPageState();
}

class _ChatPageState extends State<ChatPage> {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: SingleChildScrollView(
        child: Center(child: Text("Chat")),
      ),
    );
  }
}
```
을 입력

`/lib/screen/homePage.dart` 파일에

```dart
import 'package:flt_20220703_simple_chatapp/screen/chatPage.dart';
```
임포트 해주고

```dart
    return Scaffold(
      body: Container(
        child: Center(child: Text("Chat")),
      ),
```
를
```dart
    return Scaffold(
      body: ChatPage(),
```
로 변경

![Flutter Simple chat UI 실행 화면 03](/assets/img/2022-07-10-Flutter-Simple-Chat-UI/SimulSc_chat_03.png){:width="30%" height="30%"}

변한건 별로 없어 보이지만, chat이라고 텍스트만 쓰여져 있던 화면이 아닌, 새로운 레이어로 덮혀진 상태라는 걸 알 수 있다.

### 헤더를 만들어주자
이제부터 본격적으로 chat page ui를 만들어 주는 작업

`/lib/screen/chatPage.dart` 파일에 `_ChatPageState` 클래스 `build`위젯 내용을 수정해서 대화 목록을 표시할 것임

```dart
    return Scaffold(
      body: SingleChildScrollView(
        child: Center(child: Text("Chat")),
      ),
    );
```
을

```dart
    return Scaffold(
      body: SingleChildScrollView(
        physics: BouncingScrollPhysics(),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: <Widget>[
            SafeArea(
              child: Padding(
                padding: EdgeInsets.only(left: 16,right: 16,top: 10),
                child: Row(
                  mainAxisAlignment: MainAxisAlignment.spaceBetween,
                  children: <Widget>[
                    Text("Conversations",style: TextStyle(fontSize: 32,fontWeight: FontWeight.bold),),
                    Container(
                      padding: EdgeInsets.only(left: 8,right: 8,top: 2,bottom: 2),
                      height: 30,
                      decoration: BoxDecoration(
                        borderRadius: BorderRadius.circular(30),
                        color: Colors.pink[50],
                      ),
                      child: Row(
                        children: <Widget>[
                          Icon(Icons.add,color: Colors.pink,size: 20,),
                          SizedBox(width: 2,),
                          Text("Add New",style: TextStyle(fontSize: 14,fontWeight: FontWeight.bold),),
                        ],
                      ),
                    )
                  ],
                ),
              ),
            ),
          ],
        ),
      ),
    );
```
로 수정

- body: SingleChildScrollView  
  chatPage 의 본문을 전체적으로 스크롤 할 수 있도록 
- physics: BouncingScrollPhysics()  
  사용자의 스크롤이 끝/시작부분에 도달할 때 바운싱 효과를 내어, 끝 부분에 도달했음을 알기 쉽게 함
- children: < Widget > [Text(), Container()]  
  헤더를 표시할 텍스트 위젯과 컨테이너
- child: Column  
  SingleChildScrollView 의 모든 하위 항목은 수직으로 표시함


![Flutter Simple chat UI 실행 화면 04](/assets/img/2022-07-10-Flutter-Simple-Chat-UI/SimulSc_chat_04.png){:width="30%" height="30%"}

### 검색창을 만들어주자
Column 위젯의 내용을 변경 children:SafeArea(), 다음에

```dart
Padding(
  padding: EdgeInsets.only(top: 16,left: 16,right: 16),
  child: TextField(
    decoration: InputDecoration(
      hintText: "Search...",
      hintStyle: TextStyle(color: Colors.grey.shade600),
      prefixIcon: Icon(Icons.search,color: Colors.grey.shade600, size: 20,),
      filled: true,
      fillColor: Colors.grey.shade100,
      contentPadding: EdgeInsets.all(8),
      enabledBorder: OutlineInputBorder(
          borderRadius: BorderRadius.circular(20),
          borderSide: BorderSide(
              color: Colors.grey.shade100
          )
      ),
    ),
  ),
),
```
를 추가

여기까지의 화면 

![Flutter Simple chat UI 실행 화면 05](/assets/img/2022-07-10-Flutter-Simple-Chat-UI/SimulSc_chat_05.png){:width="30%" height="30%"}


### 대화 목록을 만들어 주자

대화 목록의 인스턴스를 저장하는 클래스(객체) 모델을 먼저 만들어야 함  
`./lib` 폴더에 `./models` 라는 폴더를 생성  
`./models` 안에 `chatUsersModel.dart` 라는 파일을 생성
`/lib/models/chatUsersModel.dart` 파일에

```dart
import 'package:flutter/cupertino.dart';

class ChatUsers{
  String name;
  String messageText;
  String imageURL;
  String time;
  ChatUsers({required this.name,required this.messageText,required this.imageURL,required this.time});
}
```
와 같이 ChatUsers 라는 클래스를 작성  
ChatUsers 객체에는 사용자 이름, 텍스트 메세지, 이미지 URL, 시간이 저장됨

원문에는 변수 들 앞에 반드시 값이 필요하다는 의미로 `@required` 선언을 하고 있는데, null safty 문제로 `requried` 쓰도록 바뀌었다

[How does @required compare to the new required keyword?](https://dart.dev/null-safety/faq#how-does-required-compare-to-the-new-required-keyword)



그 다음 다시 `/lib/screen/chatPage.dart` 로 돌아와서 

```dart
import 'package:flt_20220703_simple_chatapp/models/chatUsersModel.dart';
```
해주고, 방금 만든 클래스를 이용해 사용자 목록을 만들어 주자

```dart
List<ChatUsers> chatUsers = [
    ChatUsers(name: "Jane Russel", messageText: "Awesome Setup", imageURL: "images/userImage1.jpeg", time: "Now"),
    ChatUsers(name: "Glady's Murphy", messageText: "That's Great", imageURL: "images/userImage2.jpeg", time: "Yesterday"),
    ChatUsers(name: "Jorge Henry", messageText: "Hey where are you?", imageURL: "images/userImage3.jpeg", time: "31 Mar"),
    ChatUsers(name: "Philip Fox", messageText: "Busy! Call me in 20 mins", imageURL: "images/userImage4.jpeg", time: "28 Mar"),
    ChatUsers(name: "Debra Hawkins", messageText: "Thankyou, It's awesome", imageURL: "images/userImage5.jpeg", time: "23 Mar"),
    ChatUsers(name: "Jacob Pena", messageText: "will update you in evening", imageURL: "images/userImage6.jpeg", time: "17 Mar"),
    ChatUsers(name: "Andrey Jones", messageText: "Can you please share the file?", imageURL: "images/userImage7.jpeg", time: "24 Feb"),
    ChatUsers(name: "John Wick", messageText: "How are you?", imageURL: "images/userImage8.jpeg", time: "18 Feb"),
  ];
```

### 개별 대화를 위한 개별 클래스 위젯을 만들자

`./lib` 폴더에 `./widgets` 라는 폴더를 생성  
`./models` 안에 `conversationList.dart` 라는 파일을 생성
`/lib/models/conversationList.dart` 파일에

```dart
import 'package:flutter/material.dart';

class ConversationList extends StatefulWidget{
  String name;
  String messageText;
  String imageUrl;
  String time;
  bool isMessageRead;
  ConversationList({required this.name,required this.messageText,required this.imageUrl,required this.time,required this.isMessageRead});
  @override
  _ConversationListState createState() => _ConversationListState();
}

class _ConversationListState extends State<ConversationList> {
  @override
  Widget build(BuildContext context) {
    return GestureDetector(
      onTap: (){
      },
      child: Container(
        padding: EdgeInsets.only(left: 16,right: 16,top: 10,bottom: 10),
        child: Row(
          children: <Widget>[
            Expanded(
              child: Row(
                children: <Widget>[
                  CircleAvatar(
                    backgroundImage: NetworkImage(widget.imageUrl),
                    maxRadius: 30,
                  ),
                  SizedBox(width: 16,),
                  Expanded(
                    child: Container(
                      color: Colors.transparent,
                      child: Column(
                        crossAxisAlignment: CrossAxisAlignment.start,
                        children: <Widget>[
                          Text(widget.name, style: TextStyle(fontSize: 16),),
                          SizedBox(height: 6,),
                          Text(widget.messageText,style: TextStyle(fontSize: 13,color: Colors.grey.shade600, fontWeight: widget.isMessageRead?FontWeight.bold:FontWeight.normal),),
                        ],
                      ),
                    ),
                  ),
                ],
              ),
            ),
            Text(widget.time,style: TextStyle(fontSize: 12,fontWeight: widget.isMessageRead?FontWeight.bold:FontWeight.normal),),
          ],
        ),
      ),
    );
  }
}
```
추가

여기서는 chatUsersModel.dart 에 만든 객체의 변수 + 메세지 유형을 표시 할 bool 값을 파라미터로 사용하고, 그 값이 포함된 템플릿을 반환함

`/lib/screen/chatPage.dart` 의 ListView 위젯 안에서 필요한 파라미터를 전달하여, conversationList 위젯을 호출해야 함

언제나 그렇듯...먼저 임포트를 해주고
```dart
import 'package:flt_20220703_simple_chatapp/widgets/conversationList.dart';
```
Column 위젯의 내용을 변경 children:SafeArea(), Padding(), 다음에

```dart
ListView.builder(
  itemCount: chatUsers.length,
  shrinkWrap: true,
  padding: EdgeInsets.only(top: 16),
  physics: NeverScrollableScrollPhysics(),
  itemBuilder: (context, index){
    return ConversationList(
      name: chatUsers[index].name,
      messageText: chatUsers[index].messageText,
      imageUrl: chatUsers[index].imageURL,
      time: chatUsers[index].time,
      isMessageRead: (index == 0 || index == 3)?true:false,
    );
  },
),

```
를 추가해 주자 

그리고 핫 로드를 하면 이런 화면이 된다


![Flutter Simple chat UI 실행 화면 06](/assets/img/2022-07-10-Flutter-Simple-Chat-UI/SimulSc_chat_06.png){:width="30%" height="30%"}


## 채팅 세부 정보 화면 구현



***
## 후기

호냐라라