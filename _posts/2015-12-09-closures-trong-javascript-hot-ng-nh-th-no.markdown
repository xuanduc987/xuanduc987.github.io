---
layout: post
title: "Closures Trong Javascript Hoạt động Như Thế Nào?"
modified:
categories: Programming
excerpt: Nguyên lý hoạt động và một số điểm đặc biệt của closures trong Javascript
tags: [javascript, closure, translate]
image:
  feature:
date: 2015-12-09T15:13:24+07:00
---

_Bài viết được dịch từ [blog](http://dmitryfrank.com/articles/js_closures) của
tác giả Dmitry Frank._

Tôi đã dùng closures vài lần rồi. Tôi học cách dùng chúng, nhưng không hiểu rõ
closures thực sự hoạt động như thế nào, thực chất điều gì xảy ra khi tôi sử
dụng chúng. Mà clousre là cái gì cơ chứ?
[Wikipedia](https://en.wikipedia.org/wiki/Closure_%28computer_programming%29)
cũng không giúp ích gì lắm. Khi nào thì closure được tạo ra và khi nào nó bị
xóa bỏ? Implement của nó trông như thế nào?

{% highlight javascript %}
"use strict";

var myClosure = (function outerFunction() {

  var hidden = 1;

  return {
    inc: function innerFunction() {
      return hidden++;
    }
  };

}());

myClosure.inc();  // returns 1
myClosure.inc();  // returns 2
myClosure.inc();  // returns 3

// Ok, tuyệt. Nhưng nó được cài đặt như thế nào,
// và điều gì đã diễn ra?
{% endhighlight %}

Và khi cuối cùng cũng hiểu được, tôi cảm thấy cực kì thích thú và quyết định
sẽ giải thích nó: ít nhất, giờ đây tôi sẽ không quên nó nữa. Bạn biết đấy, có
câu nói như thé này:

> Tell me and I forget. Teach me and I remember. Involve me and I learn.
>
> <cite>© Benjamin Franklin</cite>

Và khi đọc các giải thích về closures, tôi cố hình dung một cách trực quan xem
mọi thứ liên quan đến nhau như thế nào: object nào reference đến các object
khác, cái nào kế thừa từ cái nào, vân vân... Tôi tìm các hình minh họa như vậy
mà không có, vậy nên tôi quyết định tự mình vẽ luôn.

Tôi xem như rằng độc giả của bài viết này đã quen với JavaScript, biết
Global Object là cái gì, biết rằng hàm trong JavaScript là "first-class
objects", vân vân và vân vân...

## Scope chain

Khi bất kì một đoạn JavaScript nào được thực thi thì nó cần một chỗ nào đó để
chứa các biến địa phương của mình. Ta hãy gọi nó là *scope object* đi (nhiều
người sẽ gọi nó là `LexicalEnvironment`). Ví dụ, khi bạn gọi một hàm, và hàm
đó định nghĩa ra vài biến địa phương, thì những biến địa phương này được lưu
vào trong scope object. Bạn có thể xem nó như một object của JavaScript bình
thường, chỉ có một điểm khác biệt là bạn không thể tham chiếu đến toàn bộ
object này một cách trực tiếp.

Khái niệm về scope object này rất khác với nhiều ngôn ngữ khác, như C hay C++;
ở các ngôn ngữ này thì biến địa phương được lưu trong stack. Còn trong
JavaScript, scope object được cấp phát bộ nhớ trong heap (hay ít nhất là "hành
vi" của nó giống như vậy), do đó nó có thể tồn tại ngay cả khi hàm đã được trả
về. Tôi sẽ giải thích thêm sau.

Đúng như bạn nghĩ, scope object có thể có parent. Khi một đoạn code thử truy
cập tới một biến, thì trình thông dịch sẽ tìm property này trong scope
object hiện tại. Nếu property không tồn tại, trình thông dịch chuyển sang
tìm ở parent scope object. Và tiếp tục lần lượt đến khi thấy giá trị cần tìm,
hoặc nếu như không còn parent nữa. Ta gọi chuỗi các scope object này là *scope
chain*.

Việc phân giải một biến trên scope chain rất giống với kế thừa prototype
(protototypal inheritance) với một điểm khác biệt: nếu bạn truy cập một thuộc
tính không tồn tại của object bình thường, và prototype chain cũng không chứa
property này thì sẽ không có lỗi xảy ra: `undefined` sẽ được trả về. Nhưng
nếu bạn truy cập một property không tồn tại trên scope chain (truy cập một
biến không tồn tại), thì lỗi `ReferenceError` sẽ xảy ra.

Phần tử cuối cùng trong scope chain luôn luôn là Global Object. Ở code
JavaScript top-level thì scope chain chỉ bao gồm một phần tử duy nhất: Global
Object. Vì vậy, nếu bạn định nghĩa biến ở code top-level, thì chúng được định
nghĩa ở Global Object. Khi một hàm được gọi, scope chain chứa nhiều hơn một
object. Bạn có thể nghĩ rằng nếu hàm được gọi từ code top-level, thì scope
chain đảm bảo chỉ chứa đúng 2 scope object, tuy nhiên điều này không đúng! Có
thể có 2 hoặc nhiều hơn scope object, tùy thuộc vào hàm. Phần sau tôi sẽ phân
tích kĩ hơn.

## Top-level code

Ok, lý thuyết thế là đủ rồi, hãy thử một ví dụ cụ thể. Sau đây là một ví dụ
rất đơn giản:

`my_script.js`
{% highlight javascript %}
"use strict";

var foo = 1;
var bar = 2;
{% endhighlight %}

Chúng ta mới tạo hai biến ở code top-level. Như đã nói bên trên, đối với code
top-level, scope object chính là Global Object:

![](http://dmitryfrank.com/_media/articles/js_closure_1.png)

Trong hình trên, ta có execution context (chính là code top-level
`my_script.js`) tham chiếu tới scope object. Đương nhiên, trong thực tế thì
`Global Object` bao gồm rất nhiều property chuẩn cũng như property đối với
từng host, nhưng không được biểu diễn trong hình trên.

## Non-nested functions

Giờ, xét đoạn script sau:

`my_script.js`
{% highlight javascript %}
"use strict";
var foo = 1;
var bar = 2;

function myFunc() {
  //-- định nghĩa biến địa phương của hàm
  var a = 1;
  var b = 2;
  var foo = 3;

  console.log("inside myFunc");
}

console.log("outside");

//-- vả gọi nó:
myFunc();
{% endhighlight %}

Khi hàm `myFunc` được định nghĩa, định danh `myFunc` được thêm vào scope
object hiện thời (trong ví dụ này là Global Object), và định danh này tham
chiếu tới **function object**. Function object chứa code của hàm cũng như các
property khác của nó. Một property mà ta quan tâm là internal property
`[[scope]]`, tham chiếu tới **scope object hiện tại**, hay nói cách khác chính
là scope object đang active khi hàm được định nghĩa (trong trường hợp này, là
Global Object).

Vậy nên lúc mà `console.log("outside");` được thực thi thì ta có dạng như sau:

![](http://dmitryfrank.com/_media/articles/js_closure_2.png)

Nhắc lại một lần nữa: function object được tham chiếu bởi `myFunc` không chỉ
giữ code của hàm mà còn tham chiếu tới scope object đang hoạt động lúc hàm
được định nghĩa. **Đây là một điểm rất quan trọng**.

Và khi hàm được *gọi*, một scope object mới được tạo ra lưu giữ các biến địa
phương cho `myFunc` (và cả các tham số của nó nữa), và scope object mới này
*kế thừa* từ scope object được tham chiếu bởi hàm được gọi.

Do vậy, lúc `myFunc` thực sự được gọi, thì ta có sơ đồ như sau:

![](http://dmitryfrank.com/_media/articles/js_closure_3.png)

Cái chúng ta có ở đây là một *scope chain*: nếu ta thử truy cập một biến bên
trong `myFunc`, JavaScript sẽ thử tìm nó trong scope object đầu tiên:
`myFunc() scope`. Nếu tìm kiếm thất bại thì tiếp tục tới scope object tiếp
theo (ở đây là `Global object`) để tìm tiếp. Nếu property được yêu cầu không
nằm trong bất cứ scope object nào thì `ReferenceError` sẽ xảy ra.

Ví dụ, nếu ta truy cập `a` từ `myFunc`, ta sẽ thu được giá trị `1` từ scope
object đầu tiên `myFunc() scope`. Nếu ta truy cập `foo`, ta sẽ có giá trị `3`
cũng từ `myFunc() scope`: nó che đi property `foo` của `Global object`. Nếu ta
truy cập `bar`, ta thu được `2` từ `Global object`. Nó hoạt động gần như là kế
thừa prototype.

Một điểm quan trọng cần chỉ ra ở đây là những scope object này còn tồn tại khi
mà có tham chiếu đến chúng. Khi tham chiếu cuối cùng tới một scope object biến
mất thì scope object này sẽ bị garbage-collect.

Vậy, khi `myFunc()` trả về, không có gì tham chiếu đến `myFunc() scope` nữa,
và nó bị garbage-collect, ta thu được sơ đồ giống như lúc trước:

![](http://dmitryfrank.com/_media/articles/js_closure_2.png)

Từ giờ, tôi sẽ không cho function object vào đồ hình nữa, nếu không đồ hình sẽ
trở nên cực kì rối rắm. Bạn hãy luôn nhớ rằng: bất cứ tham chiếu tới một hàm
nào trong JavaScript đều tham chiếu đến function object, và function object
này lại tham chiếu tới scope object.

## Nested functions

Như đã thấy ở phần trước, khi hàm trả về, không có gì tham chiếu tới scope
object của nó nữa, và vì vậy nó bị garbage-collect. Nhưng nếu ta định nghĩa
một nested function (hàm lồng nhau) và trả nó như kết quả (hoặc lưu nó vào đâu
đó bên ngoài) thì sao? Bạn đã biết rằng: function object luôn luôn tham chiếu
tới scope object mà tại đó nó được tạo ra. Do đó, khi ta định nghĩa nested
function, nó tham chiếu tới scope object của hàm bên ngoài. Và nếu ta lưu
nested function đâu đó bên ngoài, thì scope object không bị garbage collect kể
cả khi hàm số bên ngoài trả về: vì vẫn còn tham chiếu tới nó! Xem đoạn code
sau đây:

`my_script.js`
{% highlight javascript %}
"use strict";

function createCounter(initial) {
  //-- định nghĩa biến địa phương đồi với hàm
  var counter = initial;

  //-- định nghĩa nested function. Mỗi hàm đều có
  //   một tham chiếu đến scope object hiện thời

  /**
    * Tăng internal counter thêm một giá trị được cho.
    * Nếu giá trị đầu vào không phải là số hữu hạn, hoặc ít hơn 1 thì 1 sẽ
    * được dùng
    */
  function increment(value) {
    if (!isFinite(value) || value < 1){
      value = 1;
    }
    counter += value;
  }

  /**
    * Trả về giá trị counter hiện tại.
    */
  function get() {
    return counter;
  }


  //-- trả về object chứa tham chiếu đến
  //   các nested function
  return {
    increment: increment,
    get: get
  };
}

//-- tạo ra object counter
var myCounter = createCounter(100);

console.log(myCounter.get());   //-- in ra "100"

myCounter.increment(5);
console.log(myCounter.get());   //-- in ra "105"
{% endhighlight %}

Khi ta gọi `createCounter(100);`, ta có sơ đồ như sau:

![](http://dmitryfrank.com/_media/articles/js_closure_4.png)

Để ý rằng `createCounter(100) scope` được tham chiếu bới nested function
`increment` và `get`. Nếu `createCounter()` không trả về gì cả, thì đương
nhiên, việc tự tham chiếu này không tính và scope vẫn bị garbage collect.
Nhưng do `createCounter()` trả về object chứa tham chiếu tới những hàm này,
nên ta có:

![](http://dmitryfrank.com/_media/articles/js_closure_5.png)

Bỏ chút thời gian để ngẫm nghĩ về nó nào: hàm `createCounter(100);` đã trả về,
nhưng scope của nó vẫn còn, có thể truy cập được thông qua các hàm bên trong,
và **chỉ** thông qua những hàm này. Thực sự là không thể truy cập object
`createCounter(100) scope` trực tiếp, ta chỉ có thể gọi
`myCounter.increment()` hoặc `myCounter.get()` mà thôi. Những hàm này có quyền
truy cập private tới scope của `createCounter`.

Giờ ta hãy thử gọi, ví dụ như `myCounter.get()` xem sao. Nhớ lại rằng khi bất
cứ hàm nào được gọi, thì một scope object được tạo ra, và được thêm vào scope
chain tham chiếu bởi hàm. Do vậy, khi `myCounter.get()` được gọi, ta có:

![](http://dmitryfrank.com/_media/articles/js_closure_6.png)

Scope object đầu tiên trong chuỗi của hàm `get()` là object rỗng `get()
scope`. Vậy nên, khi `get()` truy cập biến `counter`, JavaScript không thể tìm
nó trong object đầu tiên của scope object chain, chuyển đến scope object tiếp
theo, và dùng biến `counter` ở `createCounter(100) scope`. Rồi hàm `get()` chỉ
đơn thuần trả về kết quả đó.

Bạn có thể đã để ý thấy rằng object `myCounter` được thêm vào hàm
`myCounter.get()` như là `this` (biểu thị trong hình bằng mũi tên đỏ). Bởi vì
`this` không bao giờ nằm trong scope chain, bạn cần lưu ý điều này. Tôi sẽ
giải thích thêm ở phần sau.

Gọi `increment(5)` thì thú vị hơn một chút, do hàm này còn có tham số đầu vào:

![](http://dmitryfrank.com/_media/articles/js_closure_6_inc.png)

Bạn thấy đấy, tham số `value` được lưu tại scope object mới được tạo cho lần
gọi `increment(5)`. Khi hàm truy cập đến biến `value`, JavaScript ngay lập tức
xác định được nó ở object đầu tiên trong scope chain. Tuy nhiên khi hàm truy
cập `counter`, JavaScript không tìm thấy nó ở object đầu tiên trong scope
chain, chuyển tiếp tới scope object tiếp theo, và tìm thấy nó ở đây. Vì vậy
`increment()` thay đối biến `counter` ở `createCounter(100) scope`. Và gần như
không ai khác có thể thay đổi được giá trị của biến này. Đó là lí do tại sao
closure lại mạnh đến như vậy: object `myCounter` không thể bị tấn công.
Closure cực kì phù hợp để lưu những thứ riêng tư.

Để ý rằng tham số `initial` cũng được lưu ở scope object của
`createCounter()`, dù rằng nó không được sử dụng. Vậy nên ta có thể tiết kiệm
được một xíu bộ nhớ nếu ta bỏ đi `var counter = initial;` đổi tên `initial`
thành `counter` và sử dụng trực tiếp. Nhưng để rõ ràng, ta dùng `initial` và
`var counter`;

Cần nói rõ thêm rằng các scope này vẫn "sống". Khi hàm được gọi, scope chain
hiện thời không được copy cho hàm này: chỉ là scope object mới được thêm vào
scope chain mà thôi, và khi bất kì scope object nào đó trong chuỗi này bị thay
đổi bời một hàm nào đó, thì thay đổi này ngay lập tức có thể được quan sát
thấy bởi tất cả các hàm có scope object này trong scope chain của chúng. Khi
`increment()` thay đổi giá trị `counter` , lần gọi `get()` tiếp theo sẽ trả về
giá trị đã được cập nhật này.

Đó cũng chính là lí do vì sao ví dụ nổi tiếng sau chạy không đúng:

{% highlight javascript %}
"use strict";

var elems = document.getElementsByClassName("myClass"), i;

for (i = 0; i < elems.length; i++) {
  elems[i].addEventListener("click", function () {
    this.innerHTML = i;
  });
}
{% endhighlight %}

Trong vòng lặp trên có rất nhiều hàm được tạo ra, và tất cả chúng đều tham
chiếu đến cùng một scope object trong scope chain của mình. Chính vì vậy,
chúng sử dụng cùng một biến `i`, chứ không phải một bản sao riêng của nó. Để
xem thêm giải thích rõ hơn cho ví dụ này, xem link sau: [Don't make functions
within a loop](http://jslinterrors.com/dont-make-functions-within-a-loop).

## Function object giống nhau, scope object khác nhau

Giờ ta mở rộng ví dụ về couter cho vui nhé. Nếu ta tạo ra nhiều hơn một object
counter thì sao? Đơn giản:

`my_script.js`
{% highlight javascript %}
"use strict";

function createCounter(initial) {
  /* ... xem code từ ví dụ trước ... */
}

//-- tạo object counter
var myCounter1 = createCounter(100);
var myCounter2 = createCounter(200);
{% endhighlight %}

Khi cả `myCounter1` và `myCounter2` được tạo, ta có sơ đồ sau:

![](http://dmitryfrank.com/_media/articles/js_closure_7.png)

Hãy nhớ lại rằng từng funciton object có tham chiếu tới scope object. Vậy nên
ở ví dụ bên trên, `myCounter1.increment` và `myCounter2.increment` tham chiếu
đến function object có **code giống hệt nhau** và các giá trị property như
nhau(`name`, `length`, và [nhiều
nữa](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Function#Function_prototype_object)),
nhưng `[[scope]]` của chúng lại tham chiếu đến **scope object khác nhau**.

Đồ hình không vẽ các function object riêng biệt để đơn giản, nhưng chúng vẫn
có ở đó.

Một vài ví dụ khác:

{% highlight javascript %}
var a, b;
a = myCounter1.get();   // a equals 100
b = myCounter2.get();   // b equals 200

myCounter1.increment(1);
myCounter1.increment(2);

myCounter2.increment(5);

a = myCounter1.get();   // a equals 103
b = myCounter2.get();   // b equals 205
{% endhighlight %}

Đó chính là cách nó hoạt động, Khái niệm closure quả thật rất mạnh.

## Scope chain và "this"

Dù thích hay không thì `this` không được lưu như một phần của scope chain.
Thay vào đó, giá trị của `this` phụ thuộc vào *cách hàm được gọi*: bạn có thể
gọi cùng một hàm với nhiều giá trị `this` khác nhau.

### Invocation patterns

Về chủ đề này thì có thể viết được hẳn một bài viết riêng, nên tôi không đi
sâu nhưng nhìn chung có bốn cách gọi hàm:

#### Method invocation pattern

{% highlight javascript %}
"use strict";

var myObj = {
  myProp: 100,
  myFunc: function myFunc() {
    return this.myProp;
  }
};
myObj.myFunc();  //-- returned 100
{% endhighlight %}

Nếu invocation expression chứa refinement (dấu chấm, hoặc `[subscript]`), thì
hàm được gọi như là một phương thức. Vì vậy, trong ví dụ bên trên, `this` được
cho trong `myFunc` là tham chiếu tới `myObj`.

#### Function invocation pattern

{% highlight javascript %}
"use strict";

function myFunc() {
  return this;
}
myFunc();   //-- returns undefined
{% endhighlight %}

Khi không có refinement, thì nó lại phụ thuộc xem code có được chạy trong
strict mode hay không:

 - trong strict mode, thì `this` là `undefined`
 - trong non-strict mode, `this` trỏ tới Global Object

Do đoạn code trên có `"use strict";` nên được chạy trong strict mode,
`myFunc()` trả về `undefined`.

#### Constructor invocation pattern

{% highlight javascript %}
"use strict";

function MyObj() {
  this.a = 'a';
  this.b = 'b';
}
var myObj = new MyObj();
{% endhighlight %}

Khi hàm được gọi với `new` ở đằng trước, JavaScripts cấp phát object mới kế
thừa từ property `prototype` của hàm, và object mới được cấp phát này chính là
`this` của hàm.

#### Apply invocation pattern

{% highlight javascript %}
"use strict";

function myFunc(myArg) {
  return this.myProp + " " + myArg;
}

var result = myFunc.apply(
  { myProp: "prop" },
  [ "arg" ]
);
//-- result is "prop arg"
{% endhighlight %}

Ta cũng có thể truyền một giá trị tùy ý vào làm `this`. Trong ví dụ trên,
chúng ta dùng
[Function.prototype.apply()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Function/apply).
Xem giải thích cụ thể hơn tại:

 - [Function.prototype.call()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Function/call)
 - [Function.prototype.bind()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Function/bind)

## Sử dụng "this" trong nested function

Xét:

{% highlight javascript %}
"use strict";

var myObj = {

  myProp: "outer-value",
  createInnerObj: function createInnerObj() {

    var hidden = "value-in-closure";

    return {
      myProp: "inner-value",
      innerFunc: function innerFunc() {
        return "hidden: '" + hidden + "', myProp: '" + this.myProp + "'";
      }
    };

  }
};

var myInnerObj = myObj.createInnerObj();
console.log( myInnerObj.innerFunc() );
{% endhighlight %}

In ra `hidden: 'value-in-closure', myProp: 'inner-value'`

Đến lúc `myObj.createInnerObj()` được gọi, ta có sơ đồ như sau:

![](http://dmitryfrank.com/_media/articles/js_closure_8_this_1.png)

Và khi ta gọi `myInnerObj.innerFunc()`, nó trông như sau:

![](http://dmitryfrank.com/_media/articles/js_closure_8_this_2.png)

Ta có thể thấy rõ rằng `this` trong `myObj.createInnerObj()` trỏ đến `myObj`,
nhưng `this` trong `myInnerObj.innerFunc()` trỏ tới `myInnerObj`: cả hai hàm
đều được gọi với Method invocation pattern, như đã giải thích bên trên. Đó là
lí do tại sao `this.myProp` bên trong `innerFunc()` là `"inner-value"` chứ
không phải là `"outer-value"`.

Do đó, ta có thể lừa `innerFunc` dùng `myProp` khác như sau:

{% highlight javascript %}
/* ... see the definition of myObj above ... */

var myInnerObj = myObj.createInnerObj();
var fakeObject = {
  myProp: "fake-inner-value",
  innerFunc: myInnerObj.innerFunc
};
console.log( fakeObject.innerFunc() );
{% endhighlight %}

In ra: `hidden: 'value-in-closure', myProp: 'fake-inner-value'`

![](http://dmitryfrank.com/_media/articles/js_closure_8_this_2a.png)

Hoặc với `apply()` hoặc `call()`:

{% highlight javascript %}
/* ... see the definition of myObj above ... */

var myInnerObj = myObj.createInnerObj();
console.log(
  myInnerObj.innerFunc.call(
    {
      myProp: "fake-inner-value-2",
    }
  )
);
{% endhighlight %}

In ra: `hidden: 'value-in-closure', myProp: 'fake-inner-value-2'`

Tuy nhiên, đôi khi hàm bên trong cần truy cập tới `this` của hàm bên ngoài,
không phụ thuộc vào cách mà hàm được gọi. Có một idiom cho việc này: ta cần
lưu giá trị ta cần vào trong closure (chính là scope object hiện tại), như
sau: `var self = this`, và sử dụng `self` trong hàm bên trong, thay cho
`this`. Xét đoạn code sau:

{% highlight javascript %}
"use strict";

var myObj = {

  myProp: "outer-value",
  createInnerObj: function createInnerObj() {

    var self = this;
    var hidden = "value-in-closure";

    return {
      myProp: "inner-value",
      innerFunc: function innerFunc() {
        return "hidden: '" + hidden + "', myProp: '" + self.myProp + "'";
      }
    };

  }
};

var myInnerObj = myObj.createInnerObj();
console.log( myInnerObj.innerFunc() );
{% endhighlight %}

In ra: `hidden: 'value-in-closure', myProp: 'outer-value'`

Theo cách này, ta có sơ đồ sau:

![](http://dmitryfrank.com/_media/articles/js_closure_8_this_3.png)

Bạn thấy đấy, lần này `innerFunc()` cso thể truy cập tới giá trị `this` của
hàm bên ngoài, thông qua `self` lưu trong closure.

## Kết luận

Giờ ta hãy trả lời vài câu hỏi ở đầu bài viết nhé:

 - Closure là gì? - Là object tham chiếu tới cả function object và scope
   object. Thực ra thì tất cả hàm trong JavaScript đều là closure: không thể
   tham chiếu tới function object mà không có scope object.

 - Nó được tạo ra khi nào? - Do tất cả hàm trong JavaScript đều là closure,
   đáp án hiển nhiên là: khi bạn định nghĩa một hàm, bạn định nghĩa một
   closure. Do đó, nó được tạo khi hàm được định nghĩa. Nhưng bạn cần phân
   biệt giữa việc tạo closure và việc tạo ra scope object mới: *closure* (hàm
   và tham chiếu tới scope chain hiện tại) được tạo khi hàm được *định nghĩa*,
   nhưng một *scope object* mới đưuọc tạo ra (và được thêm vào scope chain của
   closure) khi hàm được *gọi*.

 - Khi nào thì nó bị xóa đi? - Cũng như bất kì object bình thường nào khác
   trong JavaScript, nó được garbage-collect khi không còn tham chiếu đến nó
   nữa.

Đọc thêm:

 - [JavaScript: The Good
   Parts](http://www.amazon.com/JavaScript-Good-Parts-Douglas-Crockford/dp/0596517742)
   của Douglas Crockford. Hiểu closure hoạt động thế nào thì tốt, nhưng quan
   trọng hơn là hiểu làm thế nào để sử dụng chúng đúng cách. Cuốn sách này rất
   ngắn gọn, và chứa đựng rất nhiều pattern hay và hữu dụng.
 - [JavaScript: The Definitive Guide](http://www.amazon.com/JavaScript-Definitive-Guide-Activate-Guides/dp/0596805527/ref=sr_1_1?s=books&ie=UTF8&qid=1441159889&sr=1-1&keywords=9780596805524) của David Flanagan. Đúng như tựa đề
   của nó, cuốn sách này giải thích ngôn ngữ rất chi tiết.
