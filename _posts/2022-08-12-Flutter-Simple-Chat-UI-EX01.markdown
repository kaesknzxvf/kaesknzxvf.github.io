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

# Chat Detail 화면에 데이터 전달
## 헤더 : UI에 값 전달
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
```
{: file="/lib/screen/ChatDetailPage.dart" }

결과물을 확인해 보면, 이렇게 클릭한 사람의 이미지와 이름이 뜨는 것을 볼 수 있다.

![Flutter Simple chat UI ex 03](SimulSc_ex_03.gif){:width="30%" height="30%"}

# Chat Detail 화면 : 메세지 입력



***
# 후기