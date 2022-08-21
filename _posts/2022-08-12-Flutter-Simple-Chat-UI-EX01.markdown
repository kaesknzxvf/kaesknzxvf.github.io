---
# multilingual page pair id, this must pair with translations of this page. (This name must be unique)
#lng_pair: "id_study_flutter"
title: "Flutter 01 Simple Chat UI 응용 01 화면 간 데이터 전달, 메세지 입력 "

# post specific
author: kae
# publish date
date: 2022-08-12 15:16:00 +0900
category: [Flutter, Simple Chat]
tags: [flutter, practice, programming]

# disable comments on this page
#comments_disable: true

# thumbnail image for post
# img: ":Flutter.jpeg"

# data path
img_path: /assets/img/2022-08-12-Flutter-Simple-Chat-UI-EX01/
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

을 따라서 작성한 Flutter Simple Chat UI 을 이용하여 
- 화면 간의 데이터 전달 
- 메세지 입력
을 구현한 내용이다.

원문의 예제는 단순히 UI만 다루는 포스트이다 보니, 실제로 데이터를 전달하거나 저장하는 등 데이터를 처리하는 작업은 해주지 않는다. 그러므로 이번 포스트부터는 스스로 공부하며 추가한 코드이다.

관련 포스트
- [Base, Chat 화면 구현](https://kaesknzxvf.github.io/posts/Flutter-Simple-Chat-UI-01/)
- [Chat Detail 화면 구현](https://kaesknzxvf.github.io/posts/Flutter-Simple-Chat-UI-02/)
- 응용 01 화면 간 데이터 전달, 메세지 입력 (현재포스트)

***
# 환경

- MacBook Air (M1, 2020)
- macOS Monterey (v12.4)

- Flutter 3.0.3
- Dart 2.17.5

- Visual Studio Code 1.69.0

***

# Chat Detail <-> Chat 화면에 데이터 전달
## Chat Detail 헤더 UI에 값 전달
![Flutter Simple chat UI ex 01](SimulSc_ex_01.png){:width="30%" height="30%"}

지금은 Chat 화면의 대화상대 리스트의 어떤 개체를 선택해도 Chat Detail 화면은 위와 같이 고정 된 헤더를 가진 화면이 실행될 뿐이지만, 헤더에 내가 선택한 대화 상대의 정보가 뜨게 만들어 주고 싶다.  
즉, 내가 Chat 화면의 리스트 중에서 어떤 개체를 선택하였는지, Chat Detail 화면에 전달할 필요가 있다.

기본 개념 참고한 페이지
- [새로운 화면으로 데이터 보내기](https://flutter-ko.dev/docs/cookbook/navigation/passing-data)
  
위의 페이지는 보면 onTap 내에서 [Navigator.push](https://api.flutter.dev/flutter/widgets/Navigator/push.html) 를 이용하여 todo리스트의 데이터를 전달해주는 예제이다. 전달해주는 데이터만 다를 뿐, 기본 개념은 같으므로 이 예제를 보고 따라하면 문제 없이 데이터를 전달해 해줄 수 있다.

먼저, 작성한 simple chat ui 어플리케이션이 어떻게 생겨 먹었는지 그려보자.

![Flutter Simple chat UI ex 02](SimulSc_ex_02.jpg){:width="30%" height="30%"}

screen 인 `ChatPage` 안에 `conversationList` 위젯이 위치하고 있으며, 실질적인 리스트뷰의 출력이나 제스처 동작은 `conversationList` 에서 일어나고 있다는 것이 포인트다.

이제 코드를 보면 위젯인 `conversationList.dart` 에서 onTap 내에 이미  `Navigator.push` 메서드가 `ChatDetailPage()` 를 리턴하도록 되어 있는 것을 알수 있다.

```dart
      onTap: () {
        Navigator.push(context, MaterialPageRoute(builder: (context) {
          return ChatDetailPage();
        }));
      },
```
{: file="/lib/models/conversationList.dart" }


우리가 할 일은 `Navigator.push`메서드가 단순히 `ChatDetailPage()` 를 리턴하는 것이 아니고, `ChatDetailPage()`페이지에 매개변수를 전달하도록 코드를 변경하는 것이다.

`chatDetailPage` 페이지에 전달되어야 할 데이터는, String형태의 `imageUrl`과 `name` 이 2가지 이다. 각각 전달하는 것이 아니고, `ChatUsers` 객체를 전달해도 괜찮으나, 이번에는 그냥 각각 전달해 보겠다.

`conversationList`에서 인자를 넘겨주기 전에, `ChatDetailPage`에서 인자를 받을 수 있도록 정의 부터 해줘야 한다.

```dart
class ChatDetailPage extends StatefulWidget {
  //final ChatUsers chatuser;
  String cpname;
  String cpimageurl;

  ChatDetailPage({Key? key, required this.cpimageurl, required this.cpname})
      : super(key: key);

  @override
  _ChatDetailPageState createState() => _ChatDetailPageState();
}
```
{: file="/lib/screen/ChatDetailPage.dart" }

@override 위 쪽에, imageurl과 name가 들어 갈 변수를 정의해 준 후, 인자로 받을 수 있도록 생성자를 작성해 주었다. imageurl과 name 둘 다 필수적인 인자이니 `required` 지정.

다시 `conversationList`로 돌아와서 인자를 전달해 주었다.

```dart
      onTap: () {
        Navigator.push(
            context,
            MaterialPageRoute(
              builder: (context) => ChatDetailPage(
                cpimageurl: widget.imageUrl,
                cpname: widget.name,
              ),
            )
        ;
      },
```
{: file="/lib/models/conversationList.dart" }

여기까지 하면, `conversationList`의 imageurl과 name이 `chatDetailPage`로 넘어간 것이다! 이제 `chatDetailPage`의 해당 값(imageurl과 name)을 고정값이 아닌 선언했던 변수로 바꾸어 주면 끝!

- `CircleAvatar` 의 `backgroundImage` 키의 값을  `"https://randomuser.me/api/portraits/men/5.jpg"` 에서 `widget.cpimageurl`로 변경
- `Text` 의 `"Kriss Benwat"` 를 `widget.cpname` 로 변경

이하는 헤더(appBar) 전체 코드이다.

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
                  onPressed: () {
                    Navigator.pop(context);
                  },
                  icon: Icon(
                    Icons.arrow_back,
                    color: Colors.black,
                  ),
                ),
                SizedBox(
                  width: 2,
                ),
                CircleAvatar(
                  backgroundImage: NetworkImage(widget.cpimageurl), //변경
                  maxRadius: 20,
                ),
                SizedBox(
                  width: 12,
                ),
                Expanded(
                  child: Column(
                    crossAxisAlignment: CrossAxisAlignment.start,
                    mainAxisAlignment: MainAxisAlignment.center,
                    children: <Widget>[
                      Text(
                        widget.cpname, //변경
                        style: TextStyle(
                            fontSize: 16, fontWeight: FontWeight.w600),
                      ),
                      SizedBox(
                        height: 6,
                      ),
                      Text(
                        "Online",
                        style: TextStyle(
                            color: Colors.grey.shade600, fontSize: 13),
                      ),
                    ],
                  ),
                ),
                Icon(
                  Icons.settings,
                  color: Colors.black54,
                ),
              ],
            ),
          ),
        ),
      ),
```
{: file="/lib/screen/ChatDetailPage.dart" }

결과물을 확인해 보면, 이렇게 클릭한 사람의 이미지와 이름이 뜨는 것을 볼 수 있다.

![Flutter Simple chat UI ex 03](SimulSc_ex_03.gif){:width="30%" height="30%"}

## Chat 화면 : 리스트에 마지막 메세지 전달

# Chat Detail 화면 : 메세지 입력

이제 채팅을 입력하는 화면에서 실제로   
1. 텍스트 박스([TextField](https://api.flutter.dev/flutter/material/TextField-class.html))에 메세지를 입력하고 
2. 송신 버튼([FloatingActionButton](https://api.flutter.dev/flutter/material/FloatingActionButton-class.html))을 누르면 
3. 메세지 로그에 입력한 메세지가 띄워지도록
해볼 것이다.

메세지를 입력하는 부분인 [TextField](https://api.flutter.dev/flutter/material/TextField-class.html) 클래스는 [`controller`](https://api.flutter.dev/flutter/material/TextField/controller.html)라는 키를 가지는데, 여기에 텍스틀를 제어하는 [TextEditingController](https://api.flutter.dev/flutter/widgets/TextEditingController-class.html) 인스턴스를 지정해 줄 수 있다. 

그러므로 [TextEditingController](https://api.flutter.dev/flutter/widgets/TextEditingController-class.html) 인스턴스를 선언해주고, `controller` 키에 만든 인스턴스를 지정하면 입력된 텍스트를 제어할 수 있게 된다.

그렇게 입력된 텍스트를, 송신 버튼을 누르면 메세지 리스트에 저장하고 싶다. 그러면 [FloatingActionButton](https://api.flutter.dev/flutter/material/FloatingActionButton-class.html) 클래스의 `onPressed` 키에 입력된 값을 메세지 리스트에 추가하는 코드를 써주면 된다.

일단 여기까지 입력된 텍스트가 제대로 변수에 저장이 되는지 확인해 보겠다.

먼저 `_ChatDetailPageState` 클래스에 [TextEditingController](https://api.flutter.dev/flutter/widgets/TextEditingController-class.html) 인스턴스를 선언

```dart
  final TextEditingController msgtextController = TextEditingController();
```
{: file="/lib/screen/ChatDetailPage.dart" }

확인을 위해 `debugPrint`로 콘솔에 변수 값을 출력해줘봄...

```dart
                  Expanded(
                    child: TextField(
                      decoration: InputDecoration(
                          hintText: "Write message...",
                          hintStyle: TextStyle(color: Colors.black54),
                          border: InputBorder.none),
                      controller: msgtextController, //추가
                    ),
                  ),
                  SizedBox(
                    width: 15,
                  ),
                  FloatingActionButton(
                    onPressed: () {
                      ChatMessage newMsg = ChatMessage(
                          messageContent: msgtextController.text,
                          messageType: "sender"); //추가, 변수에 입력값 저장
                      debugPrint('Input text : ${newMsg.messageContent}');  //추가, 확인을 위해 콘솔에 변수값 출력
                      messages.add(newMsg); //추가, 메세지 리스트에 새로운 메세지 추가
                    },
                    child: Icon(
                      Icons.send,
                      color: Colors.white,
                      size: 18,
                    ),
                    backgroundColor: Colors.blue,
                    elevation: 0,
                  ),
```
{: file="/lib/screen/ChatDetailPage.dart" }

![Flutter Simple chat UI ex 04](SimulSc_ex_04.gif){:width="30%" height="30%"}

위의 gif 가장 하단에 보이는 검정색 바탕 화면이 콘솔화면인데, 시뮬레이터에 입력한 값이 제대로 출력되는 것을 볼 수 있다.

이 값을 메세지 로그(바디)에 출력해 주기 위해서는 버튼을 눌렀을 때, 리스트 뷰를 갱신하는 `setState()` 함수를 호출하면 된다!

```dart
                  FloatingActionButton(
                    onPressed: () {
                      ChatMessage newMsg = ChatMessage(
                          messageContent: msgtextController.text,
                          messageType: "sender"); 
                      debugPrint('Input text : ${newMsg.messageContent}'); 
                      setState(() {
                        messages.add(newMsg); 
                      }); //추가, 버튼을 누를 때 마다 리스트뷰 갱신
                    },
```
{: file="/lib/screen/ChatDetailPage.dart" }

결과물을 이하와 같다. 

![Flutter Simple chat UI ex 05](SimulSc_ex_05.gif){:width="30%" height="30%"}

## 자잘한 수정

제대로된 채팅 앱처럼 만들기 위해서는 그 외에도 여러가지 자잘한 작업이 필요한데,

1. 메세지 송신 후, 메세지 필드 비우기  
   화면을 보면 송신 후에도 입력했던 텍스트가 아직 텍스트 필드에 남아 있는 것을 알 수 있다.  
   이때도 역시 텍스트 필드에 있는 값을 제어하는 것이므로, [TextEditingController](https://api.flutter.dev/flutter/widgets/TextEditingController-class.html)에 구현되어있는 `clear()` 함수 호출하면 쉽게 텍스트 필드를 비워 줄 수 있다.

2. 텍스트 필드 멀티라인 입력  
   텍스트 필드에 메세지를 입력해보면, 텍스트 필드의 길이를 초과하는 메세지를 입력해도 개행이 되지 않는 것을 알 수 있다. 이것을 텍스트 필드의 길이에 맞춰 자동으로 개행이 되도록(=멀티라인 입력이 가능하도록) 수정해 주었다.

3. 바텀 높이 고정 해제, 맥스 높이 설정  
   2에서 기왕 멀티라인이 입력되게 해주었는데, 바텀의 높이가 60으로 고정이 되어있어서 멀티라인이 입력되는게 잘 보이지 않는다. 그러므로, 고정되어있는 `Container`의 높이를 해제해 주고, `ConstrainedBox`위젯을 추가해 텍스트 필드로 인해 최대로 커질 수 있는 높이를 300으로 설정해 주었다.

   ![Flutter Simple chat UI ex 10-03 origin](SimulSc_ex_10-03-01.png){:width="30%" height="30%"}

   {: align="center"}
   수정 전

   ![Flutter Simple chat UI ex 10-03 edit](SimulSc_ex_10-03-02.png){:width="30%" height="30%"}

   {: align="center"}
   수정 후

4. 송신메세지와 수산메세지 박스 정렬 수정  
   이건 단순히 UI수정이므로, 취향에 따라 수정해 주면 된다.  
   장문의 메세지일 경우, 원래는 송신메세지 박스와 수신메세지 박스가 모두 화면에 꽉 차는 상태인데, 나는 수신메세지의 경우 오른쪽에 좀 간격을 남기고,  송신메세지의 경우 왼쪽에 좀더 간격을 남기도록 메세지 박스에 마진을 설정해 주었다.
   
   ![Flutter Simple chat UI ex 10-04 origin](SimulSc_ex_10-04-01.png){:width="30%" height="30%"}

   {: align="center"}
   수정 전

   ![Flutter Simple chat UI ex 10-04 edit](SimulSc_ex_10-04-02.png){:width="30%" height="30%"}

   {: align="center"}
   수정 후

5. 송신버튼 크기 고정  
   이것도 단순 UI 수정이므로 취향에 따라... 
   원문대로라면 송신 버튼이 `FloatingActionButton`위젯으로 설정되어있어서, 텍스트 필드의 크기에 따라서 백그라운드의 크기가 멋대로 바뀔것이다. 매우 마음에 들지 않으므로 좌측의 버튼처럼 송신 버튼의 모양도 `SizedBox`로 감싸서 높이와 크기를 고정해 주었다.  
   (3번의 스크린 샷에서 버튼이 고정되어있는 이유는 이미 수정 후에 스크린샷을 찍었기 때문)

6. 송신버튼,첨부버튼 위치 고정  
   이것도 단순 UI 수정이므로 취향에 따라... 
   마찬가지로 원문대로라면 좌측의 첨부 버튼, 송신 버튼 모두 텍스트 필드의 높이가 커짐에 따라서 위로 올라갈것이다.(높이 가운데 정렬이 됨) 나는 이걸 하단에 고정 시켜 주고 싶어서 정렬 `Row`위젯에 정렬을 추가해 주었다.  
   (3번의 스크린 샷에서 버튼이 고정되어있는 이유는 이미 수정 후에 스크린샷을 찍었기 때문)

7. 입력이 있을 때만 메세지 전송  
   입력이 없을 때, 송신버튼을 눌러도 반응하지 않도록, `FloatingActionButton`위젯의 `onPressed`함수에 조건문을 추가해 주었다.
   > 단, 지금은 스페이스 바나 엔터도 입력으로 취급한다. 나중에 걸러주는 코드를 추가할 예정...

8. 메세지 로그 스크롤  
   메세지 로그를 표시하는 `Listview` 위젯이 스크롤이 되지 않아서 메세지 내용의 길이가 스크린 화면을 벗어나면 더 이상 내용을 볼 수가 없다. 그래서 메세지 로그를 표시하는 리스트 뷰 위젯을 `SingleChildScrollView` 위젯으로 감싸주었다.

9.  메세지 입력 시, 메세지 로그 가장 하단(최신)으로 이동  
    - 스크롤을 제어하기 위한 스크롤 컨트롤러를 선언해 준 후, 메세지 로그 부분에 스크롤 컨트롤러 추가  
    - 스크롤의 가장 마지막 부분으로 이동하는 `_scrollToLatest`라는 함수를 작성
    - `FloatingActionButton`위젯의 `onPressed`함수에서 작성한 `_scrollToLatest`를 호출
  
    ![Flutter Simple chat UI ex 10-09](SimulSc_ex_10-09.gif){:width="30%" height="30%"}

    > #수정 보류중인 오류  
    > 스크롤이 없다가 생기는 경우, (처음 한 줄만) 스크롤이 제대로 이동을 안함 

    여기까지의 `ChatDetailPage.dart` 전체 코드 
  
    ```dart
    import 'package:flutter/foundation.dart';
    import 'package:flutter/material.dart';
    import 'package:flt_simple_chat_ex/models/chatMessageModel.dart';

    class ChatDetailPage extends StatefulWidget {
    String cpname;
    String cpimageurl;

    ChatDetailPage({Key? key, required this.cpimageurl, required this.cpname})
        : super(key: key);

    @override
    _ChatDetailPageState createState() => _ChatDetailPageState();
    }

    class _ChatDetailPageState extends State<ChatDetailPage> {
    List<ChatMessage> messages = [
        ChatMessage(messageContent: "Hello, Will", messageType: "receiver"),
        ChatMessage(messageContent: "Hello, Jane", messageType: "sender"),
    ];

    final TextEditingController msgtextController = TextEditingController();
    //9. 추가, 스크롤 컨트롤러
    final ScrollController _scrollController = ScrollController();

    //9. 추가, 리스트 뷰의 가장 마지막(최신부분)으로 스크롤 이동
    void _scrollToLatest() {
        if (_scrollController.position.minScrollExtent ==
            _scrollController.position.maxScrollExtent) {
        // 스크롤이 없을 때는, 스크롤 이동 하지 않음
        } else {
            WidgetsBinding.instance.addPostFrameCallback((_) {
                _scrollController.jumpTo(_scrollController.position.maxScrollExtent);
                }
            );
        }
    }

    @override
    Widget build(BuildContext context) {
        return Scaffold(
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
                    onPressed: () {
                        Navigator.pop(context);
                    },
                    icon: Icon(
                        Icons.arrow_back,
                        color: Colors.black,
                    ),
                    ),
                    SizedBox(
                    width: 2,
                    ),
                    CircleAvatar(
                    backgroundImage: NetworkImage(widget.cpimageurl),
                    maxRadius: 20,
                    ),
                    SizedBox(
                    width: 12,
                    ),
                    Expanded(
                    child: Column(
                        crossAxisAlignment: CrossAxisAlignment.start,
                        mainAxisAlignment: MainAxisAlignment.center,
                        children: <Widget>[
                        Text(
                            widget.cpname,
                            style: TextStyle(
                                fontSize: 16, fontWeight: FontWeight.w600),
                        ),
                        SizedBox(
                            height: 6,
                        ),
                        Text(
                            "Online",
                            style: TextStyle(
                                color: Colors.grey.shade600, fontSize: 13),
                        ),
                        ],
                    ),
                    ),
                    Icon(
                    Icons.settings,
                    color: Colors.black54,
                    ),
                ],
                ),
            ),
            ),
        ),
        body: Stack(
            children: <Widget>[
            SingleChildScrollView(
                //8. 추가, 메세지 로그 스크롤 뷰
                physics: BouncingScrollPhysics(), //8. 추가, 메세지 로그 스크롤 뷰
                controller: _scrollController, //9. 추가, 스크롤 컨트롤러
                child: ListView.builder(
                itemCount: messages.length,
                shrinkWrap: true,
                padding: EdgeInsets.only(top: 10, bottom: 70),
                physics: NeverScrollableScrollPhysics(),
                itemBuilder: (context, index) {
                    return Container(
                    padding:
                        EdgeInsets.only(left: 16, right: 16, top: 10, bottom: 10),
                    child: Align(
                        alignment: (messages[index].messageType == "receiver"
                            ? Alignment.topLeft
                            : Alignment.topRight),
                        child: Container(
                        decoration: BoxDecoration(
                            borderRadius: BorderRadius.circular(20),
                            color: (messages[index].messageType == "receiver"
                                ? Colors.grey.shade200
                                : Colors.blue[200]),
                        ),
                        padding: EdgeInsets.all(16),
                        //4. 추가, 송신메세지(오른쪽 정렬, 왼쪽 간격 남기기)와 수산메세지(왼쪽 정렬, 오른쪽 간격 남기기) 박스 정렬 수정
                        margin: (messages[index].messageType == "receiver"
                            ? EdgeInsets.only(
                                left: 0, right: 64, top: 0, bottom: 0)
                            : EdgeInsets.only(
                                left: 64, right: 0, top: 0, bottom: 0)),
                        //4. 추가 END
                        child: Text(
                            messages[index].messageContent,
                            style: TextStyle(fontSize: 15),
                        ),
                        ),
                    ),
                    );
                },
                ),
            ),
            Align(
                alignment: Alignment.bottomLeft,
                child: ConstrainedBox(
                //3. 추가, 바텀의 맥스 높이 설정
                constraints: BoxConstraints(
                    maxHeight: 300,
                ),
                child: Container(
                    padding: EdgeInsets.only(
                        left: 10, bottom: 20, top: 0), //수정(임시), bottom10 top10
                    //height: 60, //3. 삭제, 바텀 고정 높이 해제
                    width: double.infinity,
                    color: Colors.white,
                    child: Row(
                    crossAxisAlignment: CrossAxisAlignment
                        .end, //6. 추가, 텍스트 필드가 길어져도 첨부, 송신버튼 하단으로 고정
                    children: <Widget>[
                        GestureDetector(
                        onTap: () {},
                        child: Container(
                            height: 30,
                            width: 30,
                            decoration: BoxDecoration(
                            color: Colors.lightBlue,
                            borderRadius: BorderRadius.circular(30),
                            ),
                            child: Icon(
                            Icons.add,
                            color: Colors.white,
                            size: 20,
                            ),
                          ),
                        ),
                        SizedBox(
                        width: 15,
                        ),
                        Expanded(
                        child: TextField(
                            decoration: InputDecoration(
                                hintText: "Write message...",
                                hintStyle: TextStyle(color: Colors.black54),
                                border: InputBorder.none),
                            controller: msgtextController,
                            keyboardType:
                                TextInputType.multiline, //2. 추가, 멀티라인 입력 가능하도록
                            maxLines: null, //2. 추가, 멀티라인 제한 없음
                          ),
                        ),
                        SizedBox(
                        width: 15,
                        ),
                        //5. 추가, FloatingActionButton을 SizedBox로 감싸주고 height, width 설정
                        SizedBox(
                        height: 30,
                        width: 50,
                        child: FloatingActionButton(
                            onPressed: () {
                            if (msgtextController.text.isNotEmpty) {
                                //7. 추가, 입력이 있을 때만 전송
                                ChatMessage newMsg = ChatMessage(
                                    messageContent: msgtextController.text,
                                    messageType: "sender");
                                debugPrint('Input text : ${newMsg.messageContent}');
                                setState(() {
                                    messages.add(newMsg);
                                    msgtextController.clear(); //1. 추가, 버튼을 누른 후 텍스트 필드 클리어
                                });
                                _scrollToLatest(); //9. 추가, 메세지 송신시, 가장 메세지 로그의 가장 최신 부분으로 이동
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
                      ],
                    ),
                  ),
                ),
              ),
            ],
          ),
        );
      }
    }
    ```
    {: file="/lib/screen/ChatDetailPage.dart" }

10.  메세지 입력 시, 텍스트 필드 크기에 따라 메세지 로그 범위도 변경  
    텍스트 필드의 높이가 높아지면, 메세지 로그 부분이 텍스트 필드에 맞춰서 위로 밀려났으면 좋겠는데, 지금은 그저 텍스트 필드가 커지는 만큼 메세지 로그가 가려져서 안보일 뿐이다.  
    수정해준 작업은 이하와 같다.  

     - 메세지 로그와 바텀(입력부분)을 포함하는 `body`의 위젯의 종류를 `Stack`에서 `Column`으로 변경
     - 그에 따라 변경된 자식 위젯들의 레이아웃을 수정하기 위해, 메세지 로그를 표시하는 `SingleChildScrollView` 위젯 밑에 `Flexible` 위젯을 하나더 깔아 줌
     - 위젯의 크기를 취득, 갱신하는 `_getWidgetInfo` 와 `_updateWidgetInfo` 함수를 작성
     - 텍스트 필드의 입력을 감지해서 바텀 부분의 위젯 높이가 변경될 경우 스크롤을 업데이트하도록 수정

     ![Flutter Simple chat UI ex 10-10](SimulSc_ex_10-10.gif){:width="30%" height="30%"}

     > #수정 보류중인 오류  
     > 9번과 마찬가지로 맨 처음 개행이 될 때, (처음 한 문자만) 스크롤이 제대로 이동을 안함 



***

참고하기 좋은 사이트

- [Flutter 화면 배치에 쓰는 기본 위젯 정리](https://pogon.tistory.com/entry/Flutter-화면-배치에-쓰는-기본-위젯-정리)

***
# 후기