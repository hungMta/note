# Reactive Programming Stream, BLoC pattern trong Flutter (p2).

Trong phần trước mình đã trình bày về Streams. Phần này mình tiếp tục giới thiệu về BLoC trong flutter.

## BLoC Pattern
BLoc nằm trong phần Business Logic của ứng dụng.

* Tất cả các phần sử lý logic sẽ được cài đặt trong một hoặc nhiều BLoc
* Presentation layer (UI component) chỉ quan tâm đến view và không sử lý logic trong tầng này

Áp dụng BLoC pattern sẽ thuận lợi cho việc tái sử dụng BloC khi phát triền nhiều platform: web application, mobile application, back-end.

### BLoC là gì?
![alt text](https://www.didierboelens.com/images/streams_bloc.png)
* Widgets gửi event đến BLoC qua Sinks
* Widgets được thông báo bởi BLoC qua streams

Sau đây là ví dụ cho Counter Applicaiton khi sử dụng BLoC
```swift
void main() => runApp(new MyApp());

class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return new MaterialApp(
        title: 'Streams Demo',
        theme: new ThemeData(
          primarySwatch: Colors.blue,
        ),
        home: BlocProvider<IncrementBloc>(
          bloc: IncrementBloc(),
          child: CounterPage(),
        ),
    );
  }
}

class CounterPage extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    final IncrementBloc bloc = BlocProvider.of<IncrementBloc>(context);

    return Scaffold(
      appBar: AppBar(title: Text('Stream version of the Counter App')),
      body: Center(
        child: StreamBuilder<int>(
          stream: bloc.outCounter,
          initialData: 0,
          builder: (BuildContext context, AsyncSnapshot<int> snapshot){
            return Text('You hit me: ${snapshot.data} times');
          }
        ),
      ),
      floatingActionButton: FloatingActionButton(
        child: const Icon(Icons.add),
        onPressed: (){
          bloc.incrementCounter.add(null);
        },
      ),
    );
  }
}

class IncrementBloc implements BlocBase {
  int _counter;

  //
  // Stream to handle the counter
  //
  StreamController<int> _counterController = StreamController<int>();
  StreamSink<int> get _inAdd => _counterController.sink;
  Stream<int> get outCounter => _counterController.stream;

  //
  // Stream to handle the action on the counter
  //
  StreamController _actionController = StreamController();
  StreamSink get incrementCounter => _actionController.sink;

  //
  // Constructor
  //
  IncrementBloc(){
    _counter = 0;
    _actionController.stream
                     .listen(_handleLogic);
  }

  void dispose(){
    _actionController.close();
    _counterController.close();
  }

  void _handleLogic(data){
    _counter = _counter + 1;
    _inAdd.add(_counter);
  }
}
```
Như ví dụ trên ta có thể thấy được phần Business Logic được tách biệt thành `IncrementBloc` class riêng với `CounterPage` UI class. Việc này giúp cho test logic dễ dàng hơn và ta cũng có thể sử dụng lại logic ở bất cứ đâu khi chỉ cần đăng ký sử dụng và lắng nghe stream qua `bloc.incrementCounter` và `.outCounter`.

Dù giúp ta triển khai một kiến trúc phân tách khá rõ ràng nhưng ta có thể thấy  implement một bloc khá là khiến ta bối rối nếu như không quen với reactive programming. May mắn thay ta có thể sử dụng một thư viện hỗ trợ [*] khác được xây dựng dựa trên Bloc core.
(mình thấy kiến trúc khá giống với Redux Saga bên react native)

# Tham khảo
https://github.com/adam-p/markdown-here/wiki/Markdown-Cheatsheet

[*] Thư viện giúp triển khai BLoc : https://felangel.github.io/bloc/#/coreconcepts

