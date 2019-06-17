Với lập trình cross-platform việc quản lý state, luông điều khiển của ứng dụng khá là kho khăn khi chỉ sử dụng hàm `setState` để thay đổi state của ứng dụng (Trong cả ReactNative, Flutter đều cung cấp hàm setSate như là một hảm cơ bản để quản lý state và rebuild view khi state thay đổi).  Khi dùng setSate() :
* Chỉ có thể sử dụng trong StatefullWidget
* Một StatefullWidget nếu có nhiều state sẽ khiến widget đó phình to, khó quản lý
* Khó reuse , Khó test.
* Khi setState() được gọi thì toàn bọ view của widget sẽ được rebuild, ảnh hưởng đến performance của ứng dụng.

Nhưng mọi thứ sẽ dễ dàng hơn khi áp dụng các phương pháp Reactive Programming để phát triển ứng dụng.

#Reactive Programming là gì?
**Reactive Programming là lập trình với luồng dữ liệu.**
Nói cách khác mọi thứ có thể thay đổi, diễn ra như một sự kiện, dữ liệu đều có thể được triggered bởi một luồng dữ liệu.
Với reactive programming, một dứng dụng:
* Trở thành bất đồng bộ.
* Kiếm trúc ứng dụng được xây dựng quanh khái niệm Streams và listeners.
* Khi một event, biến thay đổi thì một thông báo sẽ được gửi đi trong Stream.
* Khi có một listener được đăng kí lắng nghe Stream. Nó sẽ được thông báo vào thực hiện các action thích hợp.

> Sẽ không còn tight coupling giữa các components.

Với một Widget phát stream, Widget đó không cần biết:
* Cái gì sẽ diễn ra tiếp theo.
* Ai sẽ sử dụng thông tin được gửi đi (Không có, một hoặc nhiều Widgets sử dụng... ).
* Nơi thông tin sẽ được sử dụng (không ở đâu, cùng một màn hình, mà hình khác hay nhiều màn hình cùng sử dụng..).
* Khi nào thì thông tin sẽ được sử dụng (ngay lập tức, sau một vài giây, không bao giờ... ).

Lợi ích của việc sử dụng reactive programming:
* Có thể tạo các component cho một hoặc nhiều hành động cụ thể, giúp cho tái sử dụng component dễ dàng hơn.
* Dễ dàng test các action.

Trong lập trình flutter thì để xây dựng một reactive app ta cần sử dụng Stream hoặc BLoc.

#Stream

Stream, đơn giản hoá khái niệm thì nó là một cái ống với 2 đầu. Thông tin sẽ đi vào từ một đầu và ra ở đầu bên kia.

Trong flutter:
* để điều khiển Stream chúng ta sử dụng ***StreamController***
* Để thêm dữ liệu vào *Stream*,  *StreamController*  sẽ mở một cổng vào được gọi là *StreamSink*, được truy cập qua thuộc tính ***sink***
* Để lấy giữ liệu ra thi ta sử dụng *await* hoặc hàm *listen()* (StreamSubcription) do Stream API cung cấp.
* Dữ liệu Stream có thể truyển tải là: một value, event, object, collection, map, error ... hay thậm chí là một stream khác (Con gì bơi, chúng tôi đều có =)). )

##StreamTransformer
StreamTransformer có nhiệm vụ xử lý dữ liệu bên trong `Stream`
* Là một hàm *capture*  data trong Stream.
* Xử lý data.
* StreamTransformer trả về một Stream

StreamTransformer xử lý các kiểu sau:
* filtering: filter data theo điều kiện.
* regrouping: gom nhóm data
* modification: thay đổi data
* inject data vào trong stream khác.
* ...

## Các kiểu Stream
Có 2 loại Stream
**Single-subscription Streams**
Kiểu Stream này chỉ cho phép có 1 duy nhất 1 listener, kể cả khi huy các đăng ký trước đó.
```dart
import 'dart:async';

void main() {
  //
  // Initialize a "Single-Subscription" Stream controller
  //
  final StreamController ctrl = StreamController();
  
  //
  // Initialize a single listener which simply prints the data
  // as soon as it receives it
  //
  final StreamSubscription subscription = ctrl.stream.listen((data) => print('$data'));

  //
  // We here add the data that will flow inside the stream
  //
  ctrl.sink.add('my name');
  ctrl.sink.add(1234);
  ctrl.sink.add({'a': 'element A', 'b': 'element B'});
  ctrl.sink.add(123.45);
  
  //
  // We release the StreamController
  //
  ctrl.close();
}
```
**Broadcast Streams**
Loại Stream này cho phép nhiều listener. Khi có 1 listener mới được đăng kí, nó sẽ nhận được các event tại tời điểm nó đăng kí.
```dart
import 'dart:async';

void main() {
  //
  // Initialize a "Broadcast" Stream controller of integers
  //
  final StreamController<int> ctrl = StreamController<int>.broadcast();
  
  //
  // Initialize a single listener which filters out the odd numbers and
  // only prints the even numbers
  //
  final StreamSubscription subscription = ctrl.stream
					      .where((value) => (value % 2 == 0))
					      .listen((value) => print('$value'));

  //
  // We here add the data that will flow inside the stream
  //
  for(int i=1; i<11; i++){
  	ctrl.sink.add(i);
  }
  
  //
  // We release the StreamController
  //
  ctrl.close();
}
```
#RXDart
RXDart package gồm những class kế thừa Dart Stream Api và tuân thủ các tiêu chuẩn của ReactiveX.
Mối tương quan giữa Dart và RxDart như sau:
![alt text](/uploads/fde0/d8bd/image.png)
Trong RxDart cung cấp 3 object chính, các Object này cơ bản là một Broadcast StreamController nhưng nó trả về 1 Observable thay vì Stream.:
**PublishSubject**
![alt text](https://www.didierboelens.com/images/S.PublishSubject.png)
1 Listener khi đăng kí PublishSubject sẽ nhận được các event tại thời điểm nó đăng kí
**BehaviorSubject**
![alt text](https://www.didierboelens.com/images/S.BehaviorSubject.png)
1 Listener khi đăng kí BehaviorSubject sẽ nhận được các event cuối cùng trước thời điểm mà nó đăng kí.
**ReplaySubject**
![alt text](https://www.didierboelens.com/images/S.ReplaySubject.png)
1 Listener khi đăng kí ReplaySubject sẽ nhận được tất cả các event mà Stream phát ra.
**Lưu ý**
Khi không cần lắng nghe các Streams nữa thì ta nên hủy lắng nghe chúng.
* StreamSubscription: Khi không cần lắng nghe Stream thì hãy **cancel** đăng kí.
* StreamController: Khi không cần dùng nữa, hãy **close** nó.
* PublishSubject, BehaviorSubject, ReplaySubject: Khi không cần lắng nghe nữa, hãy **close**  chúng.

#Stream trong fullter
Trong fullter, ta sử dụng StreamBuilder widget để lắng nghe Stream.
```flutter
StreamBuilder<T>(
    key: ...optional, the unique ID of this Widget...
    stream: ...the stream to listen to...
    initialData: ...any initial data, in case the stream would initially be empty...
    builder: (BuildContext context, AsyncSnapshot<T> snapshot){
        if (snapshot.hasData){
            return ...the Widget to be built based on snapshot.data
        }
        return ...the Widget to be built if no data is available
    },
)
```
Ví dụ sau sẽ dùng Stream xây ứng dụng *counter* và không còn dùng setState để cập nhật view khi count thay đổi như ứng dụng mặc định.
```flutter
import 'dart:async';
import 'package:flutter/material.dart';

class CounterPage extends StatefulWidget {
  @override
  _CounterPageState createState() => _CounterPageState();
}

class _CounterPageState extends State<CounterPage> {
  int _counter = 0;
  final StreamController<int> _streamController = StreamController<int>();

  @override
  void dispose(){
    _streamController.close();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text('Stream version of the Counter App')),
      body: Center(
        child: StreamBuilder<int>(
          stream: _streamController.stream,
          initialData: _counter,
          builder: (BuildContext context, AsyncSnapshot<int> snapshot){
            return Text('You hit me: ${snapshot.data} times');
          }
        ),
      ),
      floatingActionButton: FloatingActionButton(
        child: const Icon(Icons.add),
        onPressed: (){
          _streamController.sink.add(++_counter);
        },
      ),
    );
  }
}
```
Trong phần tiếp theo, chúng ta sẽ tìm hiểu cách sử dụng BloC pattern trong fullter nhé.

#Tài liệu tham khảo
https://www.didierboelens.com/2018/08/reactive-programming---streams---bloc/