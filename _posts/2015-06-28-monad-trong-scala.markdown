---
layout: post
title: "Monad Trong Scala"
modified:
categories: Programming
excerpt: Cùng tìm hiểu xem Monad là gì, và ứng dụng trong Scala...
tags: ['Functional Programming', 'Scala']
image:
  feature:
date: 2015-06-28T15:43:39+07:00
---

## TL/DR

Trong Scala, một kiểu chỉ cần hỗ trợ `flatMap` là có thể xem như
là một **Monad**. Lợi ích trước mắt là có thể sử dụng trong **for
comprehension**.

Ví dụ thế này, ta có một đĩa chứa vài quả ổi. Thêm vào đó, ta biết cách
bổ từng quả ổi ra rồi bày lên đĩa. Khi đó, nếu cái đĩa ổi kia là một
**Monad** thì ta có thể dùng một cái máy kì lạ có tên là `flatMap`. Ta
nhét đĩa ổi vào cái máy này, bảo nó cách bổ ổi bày ra đĩa, thì nó sẽ tự
biết cách bổ hết các quả ổi có trong đĩa, rồi bày tất cả lên chung một
cái đĩa khác.

{% highlight scala %}
val đĩa_quả_ổi = Đĩa(Ổi("to"), Ổi("găng"), Ổi("sâu"))
def bổ_ổi(ổi: Ổi): Đĩa[Miếng_Ổi] = {
  val các_miếng_ổi = cắt(ổi)
  Đĩa(các_miếng_ổi)
}
val đĩa_miếng_ổi = đĩa_quả_ổi.flatMap{ bổ_ổi(_) }
{% endhighlight %}

## Monad là gì

### I. Định nghĩa

Có thể xem một monad là một giá trị được gắn kèm thêm ngữ cảnh. Ví dụ
như `Option[T]` thể hiện một giá trị của kiểu `T` với ngữ cảnh là có khả
năng lỗi sẽ xảy ra. Hay nếu kết quả trả về của một hàm là `List[T]` thì
ta biết hàm đó có thể có 0, 1, hoặc nhiều giá trị trả về.

Theo [Wikipedia][wiki], monad gồm ba thành phần:

 - Type constructor
 - Unit function
 - Binding operation

#### 1. Type constructor

Type constructor định nghĩa cách tạo một kiểu monadic từ một kiểu cụ thể
nào đó. Kiểu monadic này nhận một tham số kiểu, giống như generic trong
Java hay template của C++ vậy. Ví dụ: `Option[T]`, `List[T]` chưa phải
là một kiểu cụ thể, nhưng `Option[String]` hay `Option[Int]` là kiểu cụ
thể, nhận tham số kiểu là `String` và `Int` tương ứng.

#### 2. Unit function

Unit function là một hàm đính kèm thêm ngữ cảnh vào một giá trị cụ thể,
từ `A -> Monad[A]`. Kết quả thu được thường là giá trị "đơn giản nhất"
mà vẫn giữ được thông tin giá trị đầu vào.  Ví dụ đối với `Option`, ta
có `Some(_)`; với `List` sẽ là `List(_)`

#### 3. Binding Operation

Binding operation có kiểu `Monad[A] -> (A -> Monad[B]) -> Monad[B]`

Tham số đầu tiên là một giá trị có kiểu monadic, tham số thứ hai là một
hàm biến đổi từ kiểu mà monad trong tham số đầu tiên đóng gói sang một
kiểu monadic khác, kết quả thu được thuộc kiểu monadic kia. Binding
operation có thể xem như gồm bốn bước:

1. Các giá trị chứa trong monad tham số đầu tiên được lấy ra
2. Hàm số trong tham số thứ hai được áp dụng cho tất cả các giá trị vừa
   thu được, kết quả thu được cũng là các monad
3. Lấy các giá trị từ các monad vừa tính toán được ra
4. Từ các giá trị vừa lấy ra được, tạo thành một giá trị monad duy nhất

Binding operation trong Scala thông thường có tên gọi là `flatMap`.

Ngoài ra, monad còn phải tuân thủ các **luật monad**.

### II. Các luật Monad

#### 1. Left identify

Luật monad đầu tiên yêu cầu rằng nếu ta có một giá trị, thêm ngữ cảnh
với `unit function` rồi truyền nó vào `binding operation` cùng một hàm
thì sẽ tương đương với việc truyền trực tiếp giá trị ban đầu vào hàm đó.

Ví dụ với `Option` thì `Some(x).flatMap(f)` chính bằng `f(x)`, hay với
`List` thì `List(x).flatMap(f)` thì không khác gì `f(x)`.

{% highlight scala %}
scala> def f(x: Int) = List(x, -x)
f: (x: Int)List[Int]

scala> List(1).flatMap(f)
res14: List[Int] = List(1, -1)

scala> f(1)
res15: List[Int] = List(1, -1)
{% endhighlight %}

#### 2. Right identity

Luật thứ hai yêu cầu rằng, nếu ta có một giá trị monadic, truyền vào
`binding operation` cùng với tham số thứ hai là `unit function` thì kết
quả thu được phải là monad ban đầu.

Cụ thể với `Option` thì `x.flatMap(Some(_))` phải trả về `x`, cũng như
đối với `List` thì `x.flatMap(List(_))` cũng như `x`

{% highlight scala %}
scala> Some(10).flatMap(Some(_))
res19: Option[Int] = Some(10)

scala> None.flatMap(Some(_))
res20: Option[Nothing] = None

scala> List('a','b','c').flatMap(List(_))
res21: List[Char] = List(a, b, c)

scala> List().flatMap(List(_))
res22: List[Nothing] = List()
{% endhighlight %}

#### 3. Associativity (kết hợp)

Luật monad cuối cùng là khi ta 'chain' các binding operation với nhau
thì kết quả không phụ thuộc vào cách ta đặt dấu ngoặc: binding hai hàm
liên tiếp giống như binding vào một hàm số có thể xác định từ hai hàm
ban đầu.

Một cách hình thức `x.flatMap(f).flatMap(g)` tương đương với
`x.flatMap(f(_).flatMap(g))`

Ví dụ:

{% highlight scala %}
scala> def f(x:Int) = List(x, -x)
f: (x: Int)List[Int]

scala> def g(x:Int) = List(2*x)
g: (x: Int)List[Int]

scala> List(1,2).flatMap(f).flatMap(g)
res24: List[Int] = List(2, -2, 4, -4)

scala> List(1,2).flatMap(f(_).flatMap(g))
res25: List[Int] = List(2, -2, 4, -4)
{% endhighlight %}

### III. Scala for comprehensions

Trong scala, for comprehensions được biến đổi sử dụng `flatMap`

Ví dụ:

{% highlight scala %}
for(
  val x <- m1
  val y <- m2
) yield x+y
{% endhighlight %}

Tương đương với

{% highlight scala %}
m1.flatMap(x => m2.flatMap(y => List(x+y)))
{% endhighlight %}

Tuy nhiên, flatMap cuối cùng hơi thừa, nên thực tế Scala biến đổi thành:

{% highlight scala %}
m1.flatMap(x => m2.map(y => x+y))
{% endhighlight %}

## Ví dụ áp dụng Monad `Option` trong Scala

Trong phần này, ta sẽ xét một ví dụ để thấy được tác dụng của Monad
`Option` cũng như **binding operation** `flatMap`.

Giả sử ta có `Person` như sau:

{% highlight scala %}
object Person {

  val persons = List("P", "MP", "MMP", "FMP", "FP", "MFP", "FFP") map { Person(_) }

  private val mothers = Map(
    Person("P") -> Person("MP"),
    Person("MP") -> Person("MMP"),
    Person("FP") -> Person("MFP"))

  private val fathers = Map(
    Person("P") -> Person("FP"),
    Person("MP") -> Person("FMP"),
    Person("FP") -> Person("FFP"))

  def mother(p: Person): Option[Person] = mothers.get(p)

  def father(p: Person): Option[Person] = fathers.get(p)
}

case class Person(name: String) {
  def mother: Option[Person] = Person.mother(this)
  def father: Option[Person] = Person.father(this)
}
{% endhighlight %}

Như vậy, ta có một danh sách các người ["P", "MP", "MMP", "FMP", "FP",
"MFP", "FFP"], với mỗi người ta có thể xem thông tin về cha hoặc mẹ của
người đó: nếu có sẽ trả về `Some(person)` còn không sẽ trả về `None`

Giờ nếu ta muốn tìm ông ngoại của một người, ta có thể viết:

{% highlight scala %}
def maternalGrandfatherNoMonad(p: Person): Option[Person] =
  p.mother match {
    case Some(m) => m.father
    case None => None
  }
{% endhighlight %}

Còn nếu ta muốn kiểm tra xem cả ông nội và ông ngoại của một người có
nằm trong cơ sở dữ liệu không thì:

{% highlight scala %}
def bothGrandfathersNoMonad(p: Person): Option[(Person, Person)] =
  p.father match {
    case None => None
    case Some(f) =>
      f.father match {
        case None => None
        case Some(ff) =>
          p.mother match {
            case None => None
            case Some(m) =>
              m.father match {
                case None => None
                case Some(fm) => Some(ff, fm)
              }
          }
      }
  }
{% endhighlight %}

Với mỗi truy vấn tìm cha hoặc mẹ của một người mà lỗi, trả về `None` thì
tất cả hàm của ta cũng phải trả về `None`. Khi không có monad, ta phải
dùng pattern matching nhiều lần, xét tất cả các trường hợp có thể xảy
ra, hàm thu được vừa dài, vừa khó đọc.

May mắn là chúng ta có monad. Khi dùng monad, hai hàm của ta có thể được
đơn giản hoá thành:

{% highlight scala %}
def maternalGrandfather(p: Person) : Option[Person] =
  p.mother.flatMap { _.father }

def bothGrandfathers(p: Person) : Option[(Person, Person)] =
  p.father.flatMap(f =>
    f.father.flatMap(ff =>
      p.mother.flatMap(m =>
        m.father.flatMap(fm =>
          Some(ff, fm)
        )
      )
    )
  )
{% endhighlight %}

Nếu dùng for comprehensions thì `bothGrandfathers` còn dễ đọc hơn

{% highlight scala %}
def bothGrandfathersFor(p: Person): Option[(Person, Person)] =
  for (
    f <- p.father;
    ff <- f.father;
    m <- p.mother;
    fm <- m.father
  ) yield (ff, fm)
{% endhighlight %}

# Tài liệu tham khảo

- [Learn you a Haskell][lyah]
- [Monads in Scala][mis]
- [Haskell/Understanding monads][um]
- [What exactly makes Option a monad in Scala?][wmom]
- [Monads - Another way to abstract computations in Scala][awac]

[wiki]: https://en.wikipedia.org/wiki/Monad_(functional_programming)#Formal_definition
[lyah]: http://learnyouahaskell.com
[mis]: http://scabl.blogspot.com/2013/02/monads-in-scala-1.html
[um]: https://en.wikibooks.org/wiki/Haskell/Understanding_monads
[wmom]: http://stackoverflow.com/questions/25361203/what-exactly-makes-option-a-monad-in-scala
[awac]: http://debasishg.blogspot.com/2008/03/monads-another-way-to-abstract.html
