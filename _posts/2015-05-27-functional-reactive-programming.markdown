---
layout: post
title: "Functional Reactive Programming"
modified:
comments: true
categories: Programming
excerpt: "FRP là gì? Nó có gì tốt hơn so với lập trình thông thường?"
tags: ['Rx', 'Javascript', 'FRP']
image:
  feature:
date: 2015-05-27T14:21:55+07:00
---

![FRP](
https://farm4.staticflickr.com/3757/19458545463_fca7dce9fd_o.png)

## Functional Reactive Programming là gì?

[Functional Reactive
Programming](http://en.wikipedia.org/wiki/Functional_reactive_programming) (FRP)
là mô hình lập trình hướng tới luồng dữ liệu và sự lan truyền thay đổi. Trong
FRP, ta có một loại dữ liệu thể hiện được "giá trị thay đổi theo thời gian", ta
có thể áp dụng các hàm cơ bản đặc trưng của functional programming (ví dụ như
map, reduce, scan...). Ví dụ, trong mô hình lập trình thông thường - imperative
programming - khi ta có `a = b + c`, giá trị của `a` thu được chính là tổng giá
trị của `b` và `c` tại thời điểm chạy câu lệnh, sau thời điểm đó giá trị `b`,
`c` có thể thay đổi nhưng sẽ không làm thay đổi giá trị của `a`. Đối với FRP thì
khác, giá trị của `a` luôn tự động cập nhật theo giá tri của `b` và `c`. Việc
này giống như bảng tính Excel vậy, trong một ô nếu ta ghi công thức `=A1+B1` thì
khi giá trị của ô `A1` hay `B1` được cập nhật, thì giá trị ô bảng tính vừa rồi
cũng được cập nhật theo.

## Observable

Về cơ bản, "giá trị thay đổi theo thời gian" trong FRP có hai loại giống như bất
cứ loại dữ liệu nào trong tự nhiên: rời rạc và liên tục.  Tuy nhiên, do bản chất
của máy tính, việc cài đặt hỗ trợ mô hình loại dữ liệu liên tục khó khăn hơn,
hầu hết các framework hiện nay mới hỗ trợ cho các cập nhật rời rạc, hướng sự
kiện. Trong mô hình này, dữ liệu được mô hình thành các tín hiệu chứa giá trị
hiện tại, thay đổi một cách rời rạc gọi là sự kiện. Các sự kiện này thực tế là
một dòng sự kiện bất đồng bộ. Như vậy dùng FRP là ta lập trình với dòng dữ liệu
bất đồng bộ.

Chúng ta đã quen với việc làm việc với dòng dữ liệu từ lâu. Nguồn xuất dòng dữ
liệu chính là producer, và consumer sử dụng dòng dữ liệu đó. Để giải quyết bài
toán produce - consumer này ta đã có 2 design pattern
[Iterator](http://en.wikipedia.org/wiki/Iterator_pattern) và
[Observable](http://en.wikipedia.org/wiki/Observer_pattern). Đối với Iterator,
consumer khi cần dữ liệu thì mới yêu cầu producer, giống như khi ta hết thực
phẩm mới ra siệu thị mua đồ vậy. Còn đối với Observable, consumer đăng ký với
producer, khi producer sẽ chuyển trực tiếp cho consumer ngay khi có sản phẩm.
Việc này tương tự như chúng ta đặt báo hàng tháng: sau khi đăng ký, mỗi khi đến
kỳ có báo mới là có nhân viên giao báo tới tận nhà chúng ta. Trong FRP, dòng sự
kiện là Observable.  Các observer đăng ký với observable, và phản ứng với các
item mà observable emit ra.

Ví dụ, một observable sẽ tạo ra một dòng event như sau:

```
-----1---3--3---4-2---X--2----|->

1, 2, 3, 4 là các giá trị được emit
X tương ứng với lỗi
| là báo hiệu dòng event đã kết thúc
---> thể hiện dòng thời gian
```

Nếu chỉ có như vậy thì FRP chẳng hơn gì ta lập trình bình thường sử dụng design
pattern Observable. Trong FRP, observable được mô hình như một collection bình
thường, từ đó ta có thể áp dụng các hàm cho collection, biến đối từ một
observable này sang một observable khác.

Xét một mảng thông thường

```
[1, 3, 3, 4]
```

Giờ xét một observable

```
--1---3-3-----4--|->
```

Có thể thấy, một observable có thể xem như là một collection, trong đó các giá
trị của nó chưa có sẵn, nhưng được cam kết sẽ có trong tương lai.

Với việc xem observable như một collection, ta có thể lập trình với một mức trừu
tượng cao hơn. Ta có thể định nghĩa các mối quan hệ giữa các sự kiện một cách rõ
ràng thay vì phải viết hàng loạt đoạn code cài đặt phức tạp.

## Ví dụ minh hoạ

Để có thể hiện ưu điểm của FRP, ta cùng xét một ví dụ. Giả sử ta cần cài đặt một
search box, request lên server ngay trong lúc người dùng đang gõ để gợi ý. Tuy
nhiên nếu như vậy, khi người dùng gõ "beautiful flower", ta phải gửi 16 truy
vấn, hầu hết đều không có tác dụng gì. Do đó, searchbox của ta sẽ không được gửi
request khi người dùng đang gõ liên tục. Ngoài ra, khi đang request mà người
dùng gõ thêm, thì không được phép hiện kết quả cũ, mà phải chờ kết quả mới.

Ở đây, mình chọn thư viện
[RxJS](https://github.com/Reactive-Extensions/RxJS) để làm ví dụ. Vậy
giải quyết bài toán này bằng FRP như thế nào?

Trước hết, cần biết rằng mọi thứ đều có thể là observable: đầu tiên ta cho sự
kiện người dùng bấm một nút trên bàn phím, làm thay đổi text trong search box là
một observable:

```
-xxx--x-xx-xx-xx-----|->
```

Để tạo observable từ event rất đơn giản:

```javascript
var keyPressObservable = Rx.Observable.fromEvent(inputRx, 'keyup');
```

Do event keypress không bắt được việc bấm backspace, vậy nên ở đây ta dùng tạm
keyup.

Lúc người dùng đang gõ liên tục, ta tạm thời không quan tâm. Ta có thể dùng
method [throttle](http://reactivex.io/documentation/operators/debounce.html) để
chỉ lọc những sự kiện mà ta cần. Throttle chỉ emit các item từ Observable nếu đã
qua một khoảng thời gian mà không emit một item nào khác.

```javascript
keyPressObservable.throttle(200);
```

```
keyPressObservable: -xxx--x-xx-xx-xx-----|->
                    vvvv throttle(200) vvvv
                    -----x--x--x--x--x---|->
```

Sau khi người dùng tạm dừng gõ, ta muốn biết người dùng đã gõ được những
gì. Ta sẽ [map](http://reactivex.io/documentation/operators/map.html)
từng event với giá trị mà người dùng đã gõ được.

```javascript
requestObservable = keypressObservable.throttle(200)
  .map(function (key) {
    return inputRx.val();
});
```

Như vậy ta đã có một dòng các text mà ta cần gửi truy vấn. Mỗi khi có một
request, ta cần gửi yêu cầu lên server, và thu nhận lại kết quả.  Trong Rx, ta
có thể dễ dàng chuyển đổi từ Promise, kết quả của truy vấn ajax, sang
observable: `var observable = Rx.Observable.fromPromise(promise)`, ta sẽ dùng nó
để tạo ra observable cho response.

Với mỗi request trong collection requestObservable, ta biến đổi thành một
`response = f(request)`, lại nghề của map rồi:

```javascript
var responseObservableOfObservable =
requestObservable.map(function(requestData) {
  return Rx.Observable.fromPromise(getResult(requestData));
}
```

Mỗi request ta sinh ra một observable của response, như vậy từ
requestObservable ban đầu, ta đã sinh ra một Observable của Observable,
một collection của các collection. Nhưng ta không quan tâm đến tập các
collection làm gì, ta chỉ quan tâm tới từng phần tử trong từng
collection đó thôi. Vì vậy, ta cần "là phẳng" nó đi, biến nó thành một
observable duy nhất.

```javascript
var responseObservableOfObservable =
requestObservable.map(function(requestData) {
  return Rx.Observable.fromPromise(getResult(requestData)).concatAll();
}
```

Việc map rồi concatAll gặp rất thường xuyên, vì vậy Rx hỗ trợ flatMap,
kết hợp của cả hai

```javascript
var responseObservableOfObservable =
requestObservable.flatMap(function(requestData) {
  return Rx.Observable.fromPromise(getResult(requestData));
}
```

Khi đang request mà có thêm một request mới, ta cần huỷ request trước
đi. Rx có method
[takeUntil](http://reactivex.io/documentation/operators/takeuntil.html)
để giải quyết vấn đề này

```
--1----2--3--4---5-6--7--8--9--|->
---------------a---------------|->
vvv         takeUntil          vvv
--1----2--3--4-|----------------->
```

Áp dụng vào bài toán của ta, khi người dùng tiếp tục bấm nút thì huỷ
request cũ:

```javascript
var responseObservable = requestObservable.flatMap(function (requestData) {
    return Rx.Observable.fromPromise(getResult(requestData))
        .takeUntil(keyPressObservable);
});
```

Khi đã có Observable cho response, ta chỉ cần
[subscribe](http://reactivex.io/documentation/operators/subscribe.html)
nó để hiển thị kết quả:

```javascript
responseObservable.subscribe(function (result) {
    showResult(outputRx, result);
}, function () {
    showResult(output, "There is some error");
});
```

Tổng hợp lại toàn bộ code cho searchbox là:

```javascript
var keyPressObservable = Rx.Observable.fromEvent(inputRx, 'keyup');
var requestObservable = keyPressObservable.throttle(200)
    .map(function (key) {
    return inputRx.val();
});
var responseObservable = requestObservable.flatMap(function (requestData) {
    return Rx.Observable.fromPromise(getResult(requestData))
        .takeUntil(keyPressObservable);
});

responseObservable.subscribe(function (result) {
    showResult(outputRx, result);
}, function () {
    showResult(outputRx, "There is some error");
});
```

Nếu loại bỏ hết các biến tạm ta chỉ còn

```javascript
var keyPressObservable = Rx.Observable.fromEvent(inputRx, 'keyup').throttle(200)
    .map(function(key) {
        return inputRx.val();
    })
keyPressObservable.flatMap(function(requestData) {
        return Rx.Observable.fromPromise(getResult(requestData)).takeUntil(
            keyPressObservable);
    }).subscribe(function(result) {
        showResult(outputRx, result);
    }, function() {
        showResult(outputRx, "There is some error");
    });
```

So sánh với việc cài đặt chức năng tương đương mà không dùng Rx

```javascript
var timeout = null;
var currentRequest = null;
input.keyup(function() {
    if (timeout) clearTimeout(timeout);
    if (currentRequest) currentRequest.abort();
    var text = input.val();
    timeout = setTimeout(function() {
        currentRequest = getResult(text)
            .done(function(result) {
                showResult(output, result);
            })
            .fail(function(jqXHR, status, error) {
                if (error === "abort") return;
                showResult(output, "There is some error");
            });
    }, 200);
});
```

Có thể thấy, với Rx, ta mô tả một cách tự nhiên bài toán vào trong code,
chỉ cần nói mình muốn gì là máy tính thực hiện. Còn trong ví dụ trên, ta
phải tự đặt timeout, huỷ request, hướng dẫn từng bước để máy tính thực
hiện theo ý ta.

Phần code trong bài này có thể thử [tại
đây](https://jsfiddle.net/xuanduc987/d79q1rj5/)


## Một số trang liên quan:

- [The introduction to Reactive Programming you've been
missing](https://gist.github.com/staltz/868e7e9bc2a7b8c1f754)
- [Duality and the End of
Reactive](http://channel9.msdn.com/Events/Lang-NEXT/Lang-NEXT-2014/Keynote-Duality)
- [ReactiveX](http://reactivex.io)
- [What is functional reactive
programming](http://stackoverflow.com/questions/1028250/what-is-functional-reactive-programming)
- [Rx for .NET and RxJava for
Android](http://futurice.com/blog/tech-pick-of-the-week-rx-for-net-and-rxjava-for-android)
- [Netflix JavaScript Talks - Async JavaScript with Reactive
Extensions](https://youtu.be/XRYN2xt11Ek)
- [Many resources about FRT at haskell
wiki](https://wiki.haskell.org/Functional_Reactive_Programming)
