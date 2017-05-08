---
layout: post
title: "Ruby: Bỏ Chữ và Số Còn Lại Gì"
modified:
comments: true
categories: Programming
excerpt: "Đẩy Ruby tới giới hạn: Lập trình mà không dùng chữ cái cũng như chữ số"
tags:
 - Ruby
image:
  feature:
date: 2016-02-02T10:58:10+07:00
---

Trong CTF lần trước có bài `[Codegolf] Ruby Lab` với yêu cầu viết một chương trình mà không được phép sử dụng bất cứ một kí tự chữ (a, b, c...) hay số (0, 1, 2...) nào.

Mới đầu nghe đề bài thì có vẻ vô lý, nhưng khi tìm hiểu rồi mới thấy, thật đúng là không có gì là không thể với Ruby. Cùng xem sao nhé.

## 1. Tên biến

Trước hết, ta cần tên biến để lưu các giá trị mà chương trình cần sử dụng.

Do không được sử dụng các kí tự chữ và số, nên ta có thể sử dụng một hoặc nhiều dấu gạch dưới `_` để làm tên. Ngoài ra, Ruby cho phép tên biến bắt đầu bằng `$` hoặc `@`, vậy nên ta có thể thêm hai kí tự này phía trước những dấu `_`. Khi cần thì `$-_` cũng có thể được sử dụng để làm tên biến.

## 2. Giá trị số

Khi không dùng các kí tự số, thì ta lấy các gía trị `0`, `1`... ở đâu?

Thật may là Ruby có một vài [biến định nghĩa sẵn](http://ruby-doc.org/core-2.3.0/doc/globals_rdoc.html#label-Pre-defined+variables) mà ta có thể dùng được:

 - `$$` là biến chứa process ID của Ruby. Ta có thể đảm bảo là `$$` khác `0`, nên khi chia cho chính nó ta được `1`. Từ đó, kết hợp cới các phép toán `+`, `-`, `*`, `/`, `**` ta thu được các số khác khi cần.
 - `$.` chứa "The current input line number of the last file that was read", nói cách khác, khi chạy chương trình thì `$.` có gía trị bằng `0`

## 3. String

Để tạo ra các xâu kí tự, ta có thể lợi dụng toán tử [`<<`](http://ruby-doc.org/core-2.2.0/String.html#method-i-3C-3C): Khi toán hạng bên tay phải là một số `integer`, thì nó sẽ được xem như là code point và được chuyển hoá thành kí tự trước khi thêm vào `String` bên tay trái.

{% highlight ruby %}
'' << 97 << 98 << 99
# "abc"
{% endhighlight %}

Kết hợp vợi gía trị số ở phần trước, ta có thể tạo `String` tuỳ ý.

## 4. Các cấu trúc điều khiển

Theo [Structured program theorem](https://en.wikipedia.org/wiki/Structured_program_theorem), ta có thể viết mọi chương trình chỉ cần 3 loại cấu trúc điều khiển:

 - Tuần tự
 - Rẽ nhánh
 - Lặp

Với cấu trúc điều khiển **tuần tự**, ta đơn giản chỉ cần viết từng câu lệnh thành các dòng liên tiếp, hoặc cùng dòng thì ngăn cách nhau bởi dấu `;`.
Vậy nên, thứ ta cần xét ở đây chỉ còn là **rẽ nhánh** và **lặp**

### 4.1 Rẽ nhánh

Vì không thể dùng các kí tự chữ cái thông thường, nên câu lệnh `if else` quen thuộc của chúng ta bị ném ra ngoài cửa sổ. Thay vào đó, Ruby hỗ trợ [Ternary operation](https://en.wikipedia.org/wiki/Ternary_operation) `?:`, vậy nên hãy sử dụng chúng thôi:

{% highlight ruby %}
if condition
  do_x
else
  do_y
end

# tương đương với

condition ? do_x : do_y
{% endhighlight %}

### 4.2 Lặp

Để tạo vòng lặp mà không dùng đến các kí tự chữ và số, ta phải dựa vào đệ quy, sử dụng cú pháp *lambda*

Ví dụ đoạn chương trình sau in ra các số từ 1 đến 100:

{% highlight ruby %}
n = 0
print_number = -> {
  n += 1
  puts n
  n < 100 ? print_number[] : 0
}
{% endhighlight %}

Có thể thấy, để tránh phải sử dụng kí tự chữ, ta tạo `Proc` qua cú pháp `->`. Lợi dụng [`#[]`](http://ruby-doc.org/core-2.2.0/Proc.html#method-i-5B-5D) là alias cho `#call` đối với `Proc`. Ta có thể khiến nó tự gọi đệ quy chính mình.

Kết hợp những gì vừa giới thiệu vào chương trình trên ta có:

{% highlight ruby %}
_ = $$ / $$      # 1
__ = _ - _       # 0
@_ = _ + _       # 2
$_ = @_ + @_ + _ # 5

$__ = __
(___ = -> {      # print_number
  $__ += _       # tăng n lên 1
  puts $__
  $__ < ($_ * @_) ** @_ ? ___[] : __    # (5 * 2) ** 2 = 100
})[] # ngay lập tức gọi lambda
{% endhighlight %}

Đoạn chương trình trên vẫn dùng `puts`, vi phạm việc không được sử dụng các kí tự chữ cái. Để loại bỏ `puts` này, ta có phần tiếp theo.

## 5. Output

Như đã nói ở trên, để chương trình có thể in ra kết qủa, đoạn script bên trên đã sử dụng `puts`. Tuy nhiên nếu không sử dụng chữ cái thì in kết qủa thế nào đây???

Quay trở lại các [biến được định nghĩa sẵn của Ruby](http://ruby-doc.org/core-2.3.0/doc/globals_rdoc.html#label-Pre-defined+variables), ta có:

> $>: The default output for print, printf. $stdout by default.

Kết hợp với toán tử `<<`, ta có thể in kết qủa ra màn hình, ví dụ như sau:

{% highlight ruby %}
$> << 97
# 97
$> << ('' << 97)
# 'a'
{% endhighlight %}

Vậy đó, ta có thể in ra chữ hay số một cách dễ dàng.

## 6. Input

Đã có `output` rồi, vậy lấy `input` thế nào đây?

Mặc dù có `$<` giống như một file ảo chứa thất cả tham số đầu vào, nhưng việc lấy dữ liệu từ `$<` mà không dùng kí tự chữ hay số là không thể được.

Tuy nhiên, Ruby hỗ trợ gọi `system call` thông qua backtick `` ` ``

{% highlight ruby %}
puts `ls`
# Music
# Pictures
# Public
# Video
{% endhighlight %}

Như vậy, thông qua việc gọi `cat` hoặc `tee` qua `system call`, ta có thể lấy được đầu vào cho chương trình, dù chương trình không có khả năng `interactive`

{% highlight ruby %}
input = `tee`
input = `#{'tee'}`
input = `#{'' << 116 << 101 << 101}`
{% endhighlight %}

## Kết luận

Áp dụng các kĩ thuật ở trên, ta có thể viết được hầu hết mọi chương trình bằng Ruby, mà không phải dùng bất cứ một kí tự chữ hay số nào.

Sau đây là một đoạn chương trình in ra chuỗi `n` số Fibonaci, với `n` tuỳ ý.
Cụ thể chương trình hoạt động ra sao, các bạn cùng thử đọc xem nhé ;)

{% highlight ruby %}
_  = $$ / $$
__ =  _ +  _
$_ = __ + __
@_ = $_ * $_ * (_ + __)
$-_ = _ + _ + _
$-_ = $-_ * $-_ + _

$__ = -> __ {
  @__ = @_ - _
  (___ = -> {
    ('' << @__ += _) == __ ? @__ - @_ : ___[]
  })[]
}

@__ = -> ___ {
  (@___ = -> {
    $. += $__[___[_ - _]]
    $. *= $_ * __ + __
    (___ = ___[_.._ - __]) == '' ? $. / ($_ * __ + __) : @___[]
  })[]
}

___ = $-_ ** __ + _
____ = `#{'' << (___ + $_ * $_ - _) << ___ << ___}`[$...$.-__]
____ = @__[____]

$___ = @___ = _
$. = _ - _
(@__ =  -> (__) {
  __ -= _
  $> << @___ << ('' << $-_)
  $_ = @___
  @___ += $___
  $___ = $_
  __ > $. ? @__[__] : $.
})[____]

{% endhighlight %}

Thử chạy:

{% highlight sh %}
$ cat n
12
$ cat n | ruby fibonaci.rb
1
2
3
5
8
13
21
34
55
89
144
233

{% endhighlight %}

## Tham khảo

 - [Non-Alphanumeric Ruby for Fun and Not Much Else](http://threeifbywhiskey.github.io/2014/03/05/non-alphanumeric-ruby-for-fun-and-not-much-else/)
