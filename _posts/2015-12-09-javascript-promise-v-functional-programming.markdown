---
layout: post
title: "Javascript, Promise và Functional Programming"
modified:
categories: Programming
excerpt: Async với promise, cùng mối liên hệ giữa promise và các khái niệm của functional programming
tags: [javascript, promise, functional programming]
image:
  feature:
date: 2015-12-09T15:00:22+07:00
---


## I. Mở đầu

Trong lập trình ta sẽ bắt gặp hai loại function: sync (đồng bộ - synchronous) và
async (bất đồng bộ - asynchronous). Với sync function, chương trình phải đợi
từng tác vụ hoàn thành trước khi bắt đầu một tác vụ khác. Và, ngược lại, với
async function, ta có thể thực hiện nhiều tác vụ một cách đồng thời.

Đối với Javascript, để gọi một async function, lập trình viên thường phải sử
dụng [continuation-passing style][cps], truyền callback mỗi khi gọi function,
dẫn đến một thứ gọi là [callback hell][callback-hell].

Sử dụng [promise][promise] cùng tư duy functional programming sẽ giúp lập trình
viên dễ dàng nắm bắt được luồng xử lý của chương trình khi dùng async function.

## II. Async với callback

Chắc hẳn mọi người đều biết đến những đoạn code kiểu này:

{% highlight javascript %}
$.getJson('...', function (data) {
   // do something with data
   ...
});

{% endhighlight %}

Với phương pháp này, mỗi khi gọi một async function, lập trình viên truyền thêm
một tham số: một function được gọi khi async function kia kết thúc.

Ở đây, ta không quan tâm giá trị `getJson` trả về mà chỉ quan tâm đến `side
effect`: đó là việc callback mà ta truyền vào được gọi. Mỗi khi sử dụng một
`async function` như `getJson`, luồng thực hiện cũng như luồng tư duy của lập
trình viên bị "ngắt quãng". Để hiểu được chương trình, lập trình viên cần xem
kết quả trả về được sử dụng như thế nào *trong tương lai* ở hàm `callback` rồi
lại tiếp tục với các dòng lệnh tiếp theo. Khi đọc và viết code, lập trình viên
*chạy chương trình* trong đầu, vậy nên khi sử dụng nhiều callback, đặc biệt là
callback lồng nhau thì chương trình trở nên cực kì khó đọc.

Xét một ví dụ chương trình đơn giản mô phỏng việc mua gạo, thực hiện từng bước
như sau:

{% highlight text %}
B1: Mua gạo
B2: Rửa nồi
B3: Vo gạo
B4: Cắm cơm
B5: Ăn
{% endhighlight %}

Nếu dùng callback thì ta sẽ có:

{% highlight javascript %}
var an, camCom, muaGao, noi, ruaNoi, tien, voGao;

tien = 3000;

noi = {
  name: "noi",
  cleaned: false
};

muaGao = function(tien, success, error) {
  if (tien < 2000) {
    error("Khong du tien");
    return;
  }
  console.log("Dang mua gao");
  return setTimeout(function() {
    return success({
      name: "gao",
      cleaned: false
    });
  }, 1000);
};

voGao = function(gao, success, error) {
  if ((gao == null) || gao.name !== "gao") {
    error("Khong co gao");
    return;
  }
  if (gao.cleaned) {
    success(gao);
    return;
  }
  console.log("Dang vo gao");
  return setTimeout(function() {
    gao.cleaned = true;
    return success(gao);
  }, 1000);
};

ruaNoi = function(noi, success, error) {
  if ((noi == null) || noi.name !== "noi") {
    error("Khong co noi");
    return;
  }
  if (noi.cleaned) {
    success(noi);
    return;
  }
  console.log("Dang rua noi");
  return setTimeout(function() {
    noi.cleaned = true;
    return success(noi);
  }, 1000);
};

camCom = function(noi, gao, success, error) {
  if ((noi == null) || noi.name !== "noi") {
    error("Khong co noi");
    return;
  }
  if (!noi.cleaned) {
    error("Chua rua noi");
    return;
  }
  if ((gao == null) || gao.name !== "gao") {
    error("Khong co gao");
    return;
  }
  if (!gao.cleaned) {
    error("Chua vo gao");
    return;
  }
  console.log("Dang cam com");
  return setTimeout(function() {
    return success("com");
  }, 1000);
};

an = function(com) {
  if (com !== "com") {
    console.error("Khong co com");
    return;
  }
  return console.log("Nom nom");
};

muaGao(tien, function(gao) {
  return ruaNoi(noi, function(noiSach) {
    return voGao(gao, function(gaoSach) {
      return camCom(noiSach, gaoSach, function(com) {
        return an(com);
      }, function(error) {
        return console.log(error);
      });
    }, function(error) {
      return console.log(error);
    });
  }, function(error) {
    return console.log(error);
  });
}, function(error) {
  return console.log(error);
});
{% endhighlight %}

Với một loạt callback lồng nhau, chương trình trở nên khó đọc và khó hiểu.

## III. I `promise` it will help

Trong phần này, ta cùng xem sử dụng `promise` giúp cải thiện đoạn chương trình
trên như thế nào.

### 1. Điểm lại một số khái niệm về functional programming

`Functor` hay `Monad` đều có thể xem như giá trị gắn kèm thêm ngữ cảnh (có thể
đọc thêm về monad ở bài viết [trước][monad]). Các kiểu dữ liệu này ta có thể tưởng
tượng như một hộp chứa dữ liệu bên trong.

Với `Functor`, ta có thể `map` các giá trị bên trong chiếc hộp này.

Type signature: `map :: (a -> b) -> Functor a -> Functor b`

Ví dụ:

{% highlight ruby %}
[1, 2, 3].map {|x| x*x}
 => [1, 4, 9]
{% endhighlight %}

`Monad` thì sẽ là `flatMap`

Type signature: `flatMap :: Monad a -> (a -> Monad b) -> Monad b`

Ví dụ:

{% highlight ruby %}
[1, 2, 3].flat_map {|x| [x, 3*x]}
 => [1, 3, 2, 6, 3, 9]
{% endhighlight %}

### 2. Promise và functional programming

Ta kí kiệu `Promise[T]` là một `Promise` bọc bên ngoài một giá trị có kiểu là
`T`. Khi một function trả về `Promise[T]`, ta hiểu function đó muốn nói rằng "À,
hiện giờ tôi chưa thể trả về kết quả ngay được, nhưng tôi *hứa* sau này sẽ trả
về kết quả kiểu `T`, trừ khi có lỗi xảy ra".

Với kết quả trả về là `Promise[T]` thì sẽ có 2 khả năng xảy ra:

 - Mọi chuyện đều êm đẹp, ta thu được giá trị kiểu `T`
 - Có lỗi xảy ra, ta thu được thông báo `error`

Ta cùng xét xem `map` và `flatMap` đối với `Promise[T]` như thế nào nhé!

{% highlight javascript %}
function map(promise, transform) {
  promise.then(function() {
    transform(data)
  });
}
{% endhighlight %}

Ở đây nếu `promise` resolve về một giá trị `data` thì function `map` sẽ trả về
một `promise` resolve về giá trị `transform(data)`. Và ngược lại, nếu `promise`
có lỗi `error`, thì kết quả của `map` vẫn chính là `promise` này.

{% highlight javascript %}
function flatMap(promise, transform) {
  promise.then(function() {
    transform(data)
  });
}
{% endhighlight %}

Có thể thấy, `flatMap` giống hệt `map` bên trên. Lí do là ở function truyền vào
`onFulfilled` của `promise.then`, nếu function này trả về một giá trị nào đó thì
kết quả thu được của `promise.then` sẽ là một `promise` được fulfilled bởi giá
trị vừa thu được; còn nếu `onFulfilled` trả về một `promise`, thì kết quả của
`promise.then` sẽ chính là `promise` vừa được trả về.

### 3. Thử dùng `promise`

Chương trình "nấu cơm ăn" bên trên nếu viết dưới dạng functional programming sẽ
có dạng như sau:

{% highlight text %}
gao = muaGao(tien)
noiSach = ruaNoi(noi)
gaoSach = gao.flatMap(x -> voGao(x))
com = noiSach.flatMap (ns) ->
  gaoSach.flatMap (gs) ->
    camCom(ns, gs)
com.map(x -> an(x))
{% endhighlight %}

Áp dụng với `promise` cùng `map` và `flatMap` bên trên ta được:

{% highlight javascript %}
pGao = muaGao(tien);
pNoiSach = ruaNoi(noi);
pGaoSach = pGao.then(function(x) {
  return voGao(x);
});
pCom = pNoiSach.then(function(ns) {
  return pGaoSach.then(function(gs) {
    return camCom(ns, gs);
  });
});
pCom.done(function(x) {
  return an(x);
}, function(e) {
  return console.log(e.message);
});
{% endhighlight %}

Kết quả cuối cùng ta có chương trình

{% highlight javascript %}
var an, camCom, muaGao, noi, pCom, pGao, pGaoSach, pNoiSach, ruaNoi, tien, voGao;

tien = 3000;

noi = {
  name: "noi",
  cleaned: false
};

muaGao = function(tien) {
  var deferred;
  deferred = Q.defer();
  if (tien < 2000) {
    deferred.reject(new Error("Khong du tien"));
    return deferred.promise;
  }
  console.log("Dang mua gao");
  setTimeout(function() {
    return deferred.resolve({
      name: "gao",
      cleaned: false
    });
  }, 1000);
  return deferred.promise;
};

voGao = function(gao) {
  var deferred;
  deferred = Q.defer();
  if ((gao == null) || gao.name !== "gao") {
    deferred.reject(new Error("Khong co gao"));
    return deferred.promise;
  }
  if (gao.cleaned) {
    deferred.resolve(gao);
    return deferred.promise;
  }
  console.log("Dang vo gao");
  setTimeout(function() {
    gao.cleaned = true;
    return deferred.resolve(gao);
  }, 1000);
  return deferred.promise;
};

ruaNoi = function(noi) {
  var deferred;
  deferred = Q.defer();
  if ((noi == null) || noi.name !== "noi") {
    deferred.reject(new Error("Khong co noi"));
    return deferred.promise;
  }
  if (noi.cleaned) {
    deferred.resolve(noi);
    return deferred.promise;
  }
  console.log("Dang rua noi");
  setTimeout(function() {
    noi.cleaned = true;
    return deferred.resolve(noi);
  }, 1000);
  return deferred.promise;
};

camCom = function(noi, gao) {
  var deferred;
  deferred = Q.defer();
  if ((noi == null) || noi.name !== "noi") {
    deferred.reject(new Error("Khong co noi"));
    return deferred.promise;
  }
  if (!noi.cleaned) {
    deferred.reject(new Error("Chua rua noi"));
    return deferred.promise;
  }
  if ((gao == null) || gao.name !== "gao") {
    deferred.reject(new Error("Khong co gao"));
    return deferred.promise;
  }
  if (!gao.cleaned) {
    error("Chua vo gao");
    return deferred.promise;
  }
  console.log("Dang cam com");
  setTimeout(function() {
    return deferred.resolve("com");
  }, 1000);
  return deferred.promise;
};

an = function(com) {
  if (com !== "com") {
    console.error("Khong co com");
    return;
  }
  return console.log("Nom nom");
};

pGao = muaGao(tien);

pNoiSach = ruaNoi(noi);

pGaoSach = pGao.then(function(x) {
  return voGao(x);
});

pCom = pNoiSach.then(function(ns) {
  return pGaoSach.then(function(gs) {
    return camCom(ns, gs);
  });
});

pCom.done(function(x) {
  return an(x);
}, function(e) {
  return console.log(e.message);
});
{% endhighlight %}

Toàn bộ code của bài viết có thể xem tại hai link jsfiddle sau:

- http://jsfiddle.net/xuanduc987/5krbvcgh/
- http://jsfiddle.net/xuanduc987/af6ntod3/

[cps]: https://www.wikiwand.com/en/Continuation-passing_style
[callback-hell]: http://callbackhell.com/
[promise]: https://github.com/kriskowal/q/wiki/API-Reference
[monad]: https://viblo.asia/xuanduc987/posts/jlA7GKoVGKZQ

