# 功能实现方法整理

#### golang初始化流程

![img](https://upload-images.jianshu.io/upload_images/2733312-9d0856df3b5faf00.png?imageMogr2/auto-orient/strip|imageView2/2/w/811/format/webp)

每个包中，按文件名排序，先常量，再全局变量，最后`init()`方法，如果涉及包之间的调用，最里层的包先进行上述的流程。

如果一个全局变量中依赖另一个全局变量，则被依赖的那个全局变量先初始化。

#### Map

```go
type Fields map[string]interface{}	// 定义
a := Fields{key: value}	// 类型可以不make 直接初始化
a[key] = value
// 做完make操作之后就相当于另外定义了一个变量 变量的初始化是这样的
b := make(Fields, 10)
b[key] = value

Fields := make(map[string]interface, 10)	// 变量必须要make 或者不用:= 分开写 var Fields map[string]interface{}
Fields[key] = value

// 匿名实例化
Fields := map[string]interface{
    "console": xxxx,
    "json": xxxx
}    

// 遍历map
for k, v := range Fields {
}

// 删除（仅适用于Map，数组删除元素得使用切片）
delete(Fields, key)
```

map make时会自动扩容。

#### 数组

数组和Map基本同理

```go
// 不make
type Fields []string
a := Fileds{"xxx", "ooo"}
a = append(a, "iou")

// make
Fields := make([]string, 10)
Fields = append(Fields, value)	// 必须要有Fields接收

// 匿名实例化
Fields := []string{"xxx", "foo"}

// 遍历数组（与Map不同的是，i获取到的是数组索引）
for i, v := range Fields {
}
```

#### 获取Map长度

```go
len(map)
```

#### 数组定义时实例化

```go
// 既然实例化，那就一定可以赋值给一个东西
level := []byte("trace")
fmt.Println(string(level))
```

#### 判断一个接口有没有实现另一个借口

```go
_ StdLogger = &log.Logger{} 
```

即：判断`log.Logger`有没有实现`StdLogger`这个接口 如果没有实现则会报编译错误

#### 方法名小写

方法名小写本意上是让该方法不被暴露在外面，但自己项目中的同级目录还是可以正常访问这个私有方法 私有只是对别人（别的包）私有

#### 任意长参数

args ...interface

args...

#### 原子操作设置/获取值

```go
sync.StoreUint32((*uint32)(&logger.Level), uint32(level))
Level(atomic.LoadUint32((*uint32)(&logger.Level)))
```

#### 结构体如何得到变量

1 结构体有定义

1.1 声明了：var xxx struct

1.2 但没有单独声明：var abc = xxx{成员变量1: xxx, 成员变量2: xxx}，成员变量可省略

2 结构体无定义（那么必须让定义和声明放在一起）

```go
var user struct{Name string; Age int}
user.Name = "xxx"
user.Age = 11
```

#### 取结构体地址的两种情况

1 非空值

1.1 已定义且声明：abc := &结构体

1.2 已定义但未声明：abc := &结构体{成员变量1: xxx, 成员变量2: xxx}

2 空值（仅获得地址）

```go
abc := new(结构体)
```

#### * &

地址类型再地址类型（**）是取值

#### 体会两段代码区别

```go
func (l *Level) hh() {
	*l = 4
	fmt.Println(l)
}

func main() {
	var oho Level = 3
	oho.hh()
	fmt.Println(oho)
}
```

输出4和4

```go
type Triangle struct {
	Bottom float32
	Height float32
}

func (t *Triangle) Area() float32 {
	t.Bottom = 4
	return (t.Bottom * t.Height) / 2
}

func main() {
	r := Triangle{6, 8}
	fmt.Println(r.Bottom)
	fmt.Println(r.Area())
}
```

输出6和16

也就是说直接由int变过来的类型可以通过指针全局的修改值 但结构体中的变量不能全局的改，在方法中改动只在改方法中有效，因为在`Area`方法中这样的语法是错误的：`*t.Bottom = 4`，因为`t.Bottom`是具体的值，这是整体，前面不能再用*号取值。

#### 切片slice

[low:high:max]

len = high - low

cap = max - low

#### 结构体序列化

结构体的定义中使用tag。

```go
data, _ := json.Marshal(m)	// m为结构体实例
fmt.Println("json result: ", string(data))
```

#### time.Now()的输出当前系统时间

```shell
2021-09-13 21:16:19.600797907 +0800 CST m=+0.000773350
```

可以将这个时间进行格式化：

```go
time.Now().Format(time.RFC3339)
```

#### time.IsZero()

```go
t:= time.Date(0001, 1, 1, 00, 00, 00, 00, time.UTC) 
IsZeroOrnot:= t.IsZero()	// true
```

#### 直接初始化空的

```go
map[string]interface{}{}
struct{}{}
```

#### fmt

```go
fmt.Print()
fmt.Println()
fmt.Printf()

// 这三个 是将字符串输出到另一个字符串中
fmt.Sprint()
fmt.Sprintln()
fmt.Sprintf()

// 这三个 是把字符串输出到文件中 当然这个文件参数也可以是其实例，比如os.Stdout和os.Stderr
// 使用os.Stderr时可保证字符串永远输出在终端，而使用os.Stdout则不能保证这一点，可能会因为使用重定向而使输出到文件中
fmt.Fprint()
fmt.Fprintln()
fmt.Fprintf()

f1 := fmt.Errorf() error	// 他与Sprint很像，只不过这里的返回值直接是error类型
f1.Error() // Error()方法可以将error类型转化为string类型
```

#### 关系

io.Writer是一个接口，os.File是他的一个实现类，而os.Stdin，os.Stdout，os.Stderr是*os.File类型的，所以这样的话就可以使os.Stdxxx和io.Writer赋值。

#### 字符串与byte相互转换

```go
data := []byte(str)
string(data)
```

#### 字符串与int之间转换

```go
strconv.Itoa(int) string
strconv.Atoi(str) (int, err)	// 如果不能转 err中存放报错信息，int为0
```

#### 数组排序

```go
sort.Strings(keys)
```

#### 一个数组添加到另一个数组中

```go
fixedKeys = append(fixedKeys, keys...)
```

#### 讲一下*bytes.Buffer

Buffer如果里面有东西的话，想再写是写不进去的，可以通过Reset()方法。

一般情况下，拿到buffer实例，首先进行Reset，Reset之后可将该buffer赋值给其他变量例如X，写完之后通过defer设置变量X为nil之后再进行Reset，方便下一次写。

结合sync.pool：

```go
buffer = bufPool.Get()
defer func() {
	newEntry.Buffer = nil
	buffer.Reset()        //// Reset作用是：方便下一次写入，如果buffer不进行reset，下一次是写不进去的
	bufPool.Put(buffer)   //// 再把reset之后的buffer放回池中 就不至于每次都要调用对象值生成函数
}()
buffer.Reset()	//// 拿到buffer对象后 先进行reset

newEntry.Buffer = buffer
newEntry.write()
```

```go
b *bytes.Buffer
b.WriteByte(' ')
b.WriteString(key)
```

#### strings包

```go
strings.ToUpper(str)
strings.ToLower(str)

strings.TrimSuffix(str, substr)	// 去除str尾部的substr 不能去除则仍然返回str
```

#### 全局变量和局部变量

- 全局变量实例化时不用:=，该符号只用于局部变量初始化

- 全局变量实例化但不使用不会报错，局部变量不使用会报错

#### 错误和异常

这俩分别对应error和panic

错误是指在可能发生错误的地方发生了错误，比如打开文件，数据库连接等等，异常是指在不可能发生错误的地方发生了错误。

# 功能实现设计思想

#### 多态的表现形式1

结构体中的接口变量，例如`Formatter:    new(TextFormatter)`，这样可以通过设置的方式使得Formatter获得其他具体的实现`Formatter = SetFormatter(Formatter的具体实现类 例如&log.JsonFormatter{})`

#### 多态的表现形式2

一个接口、多个实现类，不把实现类直接赋值给接口，而是在函数中用一个接口类型的参数接收。

同种行为的不同实现方式。

#### 生成新对象

```go
func (entry *Entry) WithContext(ctx context.Context) *Entry {
	return &Entry{Logger: entry.Logger, Data: dataCopy, Time: entry.Time, err: entry.err, Context: ctx}
}
```

这里就不是修改原本entry中的值了，而是生成新的entry对象，原本entry中的字段值不会得到修改，需要重新复制到新的变量才能看到修改：

```
context := entry.WithContext(xxx)
```

此时context变量里存放的是预期值。

#### 结构体中的变量

一个功能模块一般有一个结构体，其中无非包含两种变量：内置类型变量和自定义类型变量。我们可以在结构体初始化的时候设置默认值，另外可以使用Setter供用户设置他们自己的值（Golang中无需Getter 指针就行），然后就可以在当前模块中调用这个类型变量的方法，或者作为方法形参传入对应类型的值 类型拿过来就够了。

一定要在自己项目的代码内部初始化结构体并通过New暴露出去，这样用户从始至终只会拿到一个属于代码内部的结构体对象实例

所以要分清一个模块中，哪些是设置默认值/Setter，哪些是主要逻辑。

#### 你要暴露出去什么

给用户访问的方法，需要用户提供的形参，其类型用户应该访问的到。

暴露出去的东西与什么有关？——通过返回结构体实例，拿到相关方法。每个模块中应有一个New，通常返回结构体实例。

考虑一个模块给其他模块访问时，基本目前掌握的方法有两种情况——1.接口；2.New

#### Go语言省略self/this

```go
r := Triangle{6, 8}
```

Triangle是一个结构体，当对其实例化时，不需要像Python或Java那样给类中变量赋值时使用self.xxx=xxx或this.xxx=xxx，只需要在实例化时这样做即可。这种情况是结构体定义了但未单独声明。

#### 封装1

logger模块大量调用entry模块中的方法，这就导致logger模块中的很多地方都会出现entry.xxx这样的语句，你自己频繁访问别人 这肯定不好，不如logger中只访问一次entry，拿到entry的实例，后面在调用entry里面的方法，就用这个实例去调用。

#### 封装2

有些情况下，我们会使用到内置的结构体中的方法，比如sync.Pool和sync.Mutex等等，我们可以将他们这两个实现类重写，重写之后就是考虑如何将他们暴露出去以供其他地方访问，这里就分两种情况。如果在模块内部，可以通过结构体变量的方式暴露；再一个可以考虑接口，不过大多数情况下使用接口一般都想往多态那里靠。

#### 关于结构体变量

把变量写在结构体内有两个好处：

- 这个变量在本模块中是统一的，变量仅会随着本模块代码逻辑而得到修改
- 即使这个变量在本模块内被修改，随着代码逻辑，进入下一个模块时，仍然可以访问到那个被修改过的变量来服务这个模块

#### sync.Pool的应用场景

使用Pool的意义在于：仅仅第一次拿对象的时候会调用对象值生成函数来生成一个对象，除此之外，通过Put方法将这个对象在放回池中，Get方法执行时永远不会再创建新的对象，这就减轻了GC的负担。

sync.Pool中保存的任何项都可能随时不做通知的释放掉，所以不适合用于像socket长连接或数据库连接池。

适用于那种随着请求很多，需要重复使用某个对象的场景。

#### 我所见过的 sync.Pool中存放的东西

*Entry（logrus见到的）

*bytes.Buffer

#### 参数为接口类型

如果一个方法接收一个接口类型参数，但还是需要知道传过来的到底是什么类型，可以使用反射。

#### 向控制台输出的若干方法

- fmt.Println等等
- fmt.Fprintln等等
- os.Stderr.Write([]byte)/os.Stdout.Write([]byte)

#### 方法如何取得参数

一个方法可以有两个渠道取得参数

- 形参

- 从类似于如下代码所示的，`_unknownLevelColor`所代表的值：

  ```go
  _unknownLevelColor.Add(l.String())
  func (c Color) Add(s string) string
  ```

这样参数c能拿到`_unknownLevelColor`

#### 接口的实现

- 接口的实现仍然是接口：有点像功能的扩展，接口B实现了接口A，就相当于将接口A的功能添加了其他东西成了接口B
- 接口的实现先是接口，后是结构体，如何体现多态？例如`WriteSyncer`接口实现了`io.Writer`接口，`os.Stderr`却可以直接赋值给`WriteSyncer`接口，因为`WriteSyncer`接口继承了`io.Writer`的功能
- 虽然接口的实现可以是接口，但在阅读源码时注意接口的实现要精确到方法

#### Functional Options Pattern

- 简单情况下，可定义一个函数类

```go
type ConfigOption func(*Logger)
// 返回该函数类型
func WithConsoleOPT() ConfigOption {
	return func(l *logger) {
		l.isConsole = true
	}
}
```

- 高级的用法，把那个函数类抽象成一个接口，暴露`apply()`方法

```go
type Option interface {
	apply(*Logger)
}
type optionFunc func(*Logger)
func (f optionFunc) apply(log *Logger) {
	f(log)
}
// 返回该接口类型
func WrapCore() Option {
	return optionFunc(func(log *Logger) {
		log.core = log.xxx
	})
}
```

#### 根据不同情况执行不同函数

使用Map

```go
map[string]func(zapcore.EncoderConfig) (zapcore.Encoder, error) {
    "console": func() {},
    "json": func() {},
}
```

为了可扩展性，也可提供一个Register函数，可以往这个Map里加内容。

