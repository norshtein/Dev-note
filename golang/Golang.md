# Golang

## Tips

- Go的导出规则：首字母大写导出，小写不导出。

- 向nil map中增加元素会导致panic, 向nil slice中则不会:

  ```go
  package main

  import "fmt"

  func main(){
  	m := make(map[int]int)
  	var mm map[int]int
  	fmt.Println(m == nil) //false
  	fmt.Println(mm == nil) //true
  	m[0] = 1
  	fmt.Println(m[0]) //1
  	/*
  	mm[0] = 1 panic: cannot assignment to entry in nil map
  	fmt.Println(mm[0])
  	*/

  	s := make([]string,0)
  	var ss []string
  	fmt.Println(s ==nil) //false
  	fmt.Println(ss ==nil) //true
  	s = append(s,"aa")
  	fmt.Println(s[0]) //aa
  	ss = append(ss,"aaa")
  	fmt.Println(ss[0]) //aaa
  }

  ```

- 可比较指”可以使用== 和 != 运算符进行比较“，若结构体的全部成员可比较，则结构体可比较。可比较的结构体可以作为map的key类型。

  ```go
  package main

  import (
  	"fmt"
  )

  type Man struct{
  	name string
  	age uint8
  }
  func main(){
  	tom := Man{name: "tom", age: 12}
  	bob := Man{name: "bob", age: 13}
  	score := make(map[Man]int)
  	score[tom] = 99
  	score[bob] = 60
  	fmt.Println(tom == bob) //false
  	fmt.Println(score[tom]) //99
  	fmt.Println(score[bob]) //60
  }
  ```

  之所以可以这么搞，是因为go的map类似于c++中的unordered_map，内部使用hash来实现。如果是c++中的map，则进一步要求在该类型上定义了偏序关系（如小于符号）。

- 结构体匿名成员类型。由于外层的结构体不仅获得了匿名成员类型的所有成员，也获得了该类型导出的所有方法，所以可以将一个有简单行为的对象，组合成有复杂行为的对象。

- 匿名函数。可以实现C++中 static variable 的效果，与js中的匿名函数非常类似。注意下面两种写法的区别：

  ```go
  package main

  import "fmt"

  func nextSquare() func() int{
  	i := 0
  	return func() int{
  		i++
  		return i * i
  	}
  }
  func main(){
  	for i := 0;i < 10;i++{
  		fmt.Println(nextSquare()()) // 10 个 1
  	}
  }
  ```

  ```go
  package main

  import "fmt"

  func nextSquare() func() int{
  	i := 0
  	return func() int{
  		i++
  		return i * i
  	}
  }
  func main(){
  	f := nextSquare()
  	for i := 0;i < 10;i++{
  		fmt.Println(f()) //1,4,9,...,100
  	}
  }
  ```

  反常识之处：函数值(function object)记录了状态；变量的生命周期不由作用域确定（nextSquare返回后，i仍然隐式存在于f中）(因为变量被分配在了堆上而不是栈上)

  I thought that when a function exits, all of its local references disappear.

  Except for those locals which are closed over in a closure. Those do not disappear, even when the function to which they are local has returned.

- Clousure:

  - https://github.com/golang/go/wiki/CommonMistakes#using-goroutines-on-loop-iterator-variables


  - Any variables used in the closure must be guaranteed to live as long as the closure does. 
  - The enclosing scope variable is looked up when the nested functions are later called
  - https://stackoverflow.com/questions/3145893/how-are-closures-implemented

  Clousure还是比较tricky的，由于不会汇编，所以我也是知其然不知其所以然，网上也没有解释内部实现的文章。一个合理的猜测见Experience。

- Interface:

  程序：

  ```go
  package main

  import (
  	"fmt"
  	"reflect"
  	"strings"
  )

  type Type int
  func (t Type)Foo() string{
  	return "foo"
  }
  func (t* Type)Bar() string{
  	return "bar"
  }

  type AnotherType Type
  func (at AnotherType) FooBar() string{
  	return "foobar"
  }

  type SType struct{
  	num int
  }
  func (t SType)Foo() string{
  	return "foo"
  }
  func (t* SType)Bar() string{
  	return "bar"
  }

  type AnotherSType struct {
  	SType
  }
  func (at AnotherSType) FooBar() string{
  	return "foobar"
  }

  func Print(x interface{}){
  	v := reflect.ValueOf(x)
  	t := v.Type()
  	fmt.Printf("type %s\n", t)
  	for i := 0; i < v.NumMethod(); i++ {
  		methType := v.Method(i).Type()
  		fmt.Printf("func (%s) %s%s\n", t, t.Method(i).Name,
  			strings.TrimPrefix(methType.String(), "func"))
  	}
  }

  func main(){
  	t := Type(1)
  	pt := &t
  	Print(t)
  	Print(pt)
  	fmt.Println()

  	at := AnotherType(1)
  	pat := &at
  	Print(at)
  	Print(pat)
  	fmt.Println()

  	st := SType{1}
  	pst := &st
  	Print(st)
  	Print(pst)
  	fmt.Println()

  	ast := AnotherSType{SType{1}}
  	past := &ast
  	Print(ast)
  	Print(past)
  	fmt.Println()
  }

  /* Output:
  type main.Type
  func (main.Type) Foo() string
  type *main.Type
  func (*main.Type) Bar() string
  func (*main.Type) Foo() string

  type main.AnotherType
  func (main.AnotherType) FooBar() string
  type *main.AnotherType
  func (*main.AnotherType) FooBar() string

  type main.SType
  func (main.SType) Foo() string
  type *main.SType
  func (*main.SType) Bar() string
  func (*main.SType) Foo() string

  type main.AnotherSType
  func (main.AnotherSType) Foo() string
  func (main.AnotherSType) FooBar() string
  type *main.AnotherSType
  func (*main.AnotherSType) Bar() string
  func (*main.AnotherSType) Foo() string
  func (*main.AnotherSType) FooBar() string
  */
  ```

  可能比较confusing的一点是T类型的指针，自动具有了T类型的方法。这是一个feature, [官方文档](https://golang.org/ref/spec#Method_sets)是这么说的：

  ```
  A type may have a method set associated with it. The method set of an interface type is its interface. The method set of any other type T consists of all methods declared with receiver type T. The method set of the corresponding pointer type *T is the set of all methods declared with receiver *T or T (that is, it also contains the method set of T). 
  ```

  [这里](https://golang.org/doc/faq#different_method_sets)是为什么，[这里](https://golang.org/doc/faq#methods_on_values_or_pointers)是应该用pointer receiver还是value receiver，需要注意的是”保持一致性“：

  ```
  Next is consistency. If some of the methods of the type must have pointer receivers, the rest should too, so the method set is consistent regardless of how the type is used. See the section on method sets for details.
  ```

  这里我有一点不明白，见[这个问题](https://stackoverflow.com/questions/49937748/in-golang-is-it-necessary-change-all-method-of-a-type-to-have-pointer-receivers)。

  Pre: 我的观点是对于实现了多个接口的类型，如果某些接口中的某些方法需要使用pointer receiver，则将该接口包含的方法全部设为使用pointer receiver；对于其他接口，若其中的所有方法使用value receiver都能够良好工作，则对于这部分方法仍保持使用value receiver，否则将会剥夺该类型被转换这些接口的能力（若将这部分函数也设为pointer receiver，则只有该类型的指针拥有被转换为这些接口的能力）。

  Now: 我觉得GilesW 的回答很有道理


## Exercise

- Exercise 4.14

- Exercise 7.7 

  因为flag.CommandLine.Var会调用传入的第一个参数的String()方法来作为default value, Celsius的String()方法里打印了°C。

## Experience

以下内容均为个人猜测，**不保证其正确性**，但与实践相符。

- Page 147: 这里说”调用者的stack未被修改“，应解释为：

  由于[]string的底层实现类似下面的结构体：

  ```go
  type Slice strcut {
      pointer *[x]string
      length int
      capacity int
  }
  ```

  在调用outline函数时，将该Context下的一份stack的拷贝作为实参传入，在下一层的outline函数中，虽然修改了pointer指向的底层数组，但上一层的length和capacity并未改变，所以仍可以作为实参传给下一个兄弟节点。下一个兄弟节点写入底层数组的字符串会覆盖掉先前的兄弟节点的写入。

- 关于go里的clousure的一个合理解释：可以理解为go里的clousure的捕获方式与C++里lambda的capture reference的捕获方式相同。这样的话就可以解释下面的程序：

  ```go
  package main

  import "fmt"

  func generateClousure() []func(){
  	fn := []func(){}
  	for i := 0;i < 5;i++{
  		ii := i
  		fn = append(fn,func(){
  			fmt.Println(ii)
  			fmt.Println(i)
  		})
  	}
  	return fn
  }

  func main(){
  	fns := generateClousure()
  	for i := range fns{
  		fns[i]()
  	}
  }
  /* Output:
  0
  5
  1
  5
  2
  5
  3
  5
  4
  5
  */
  ```

  五个function object共享i的引用，而ii由于是循环体内变量，五个function object引用的是不同的内存地址，所以有这种输出。

  以及下面的程序：

  ```go
  package main

  import (
  	"time"
  	"fmt"
  )

  func main(){
  	i := 1
  	time.AfterFunc(5*time.Second,func(){
  		fmt.Printf("In clousure: i %d\n",i)
  		fmt.Println("In clousure: add 1 to i")
  		i++
  	})
  	fmt.Printf("In main: i %d\n",i)
  	fmt.Println("In main: add 1 to i")
  	i++
  	fmt.Printf("In main: i %d\n",i)
  	time.Sleep(10*time.Second)
  	fmt.Printf("In main: i %d\n",i)
  }

  /* Output:
  In main: i 1
  In main: add 1 to i
  In main: i 2
  In clousure: i 2
  In clousure: add 1 to i
  In main: i 3
  */
  ```

  用capture reference也非常容易解释。

