### 接口转型返回临时对象，只有使用指针才能改变其状态

```go
type User struct{
    id int
    name string
}
func main(){
    u := User{1, "Tom"}
    var vi, pi interface{} = u, &u 
    // vi.(User).name = "jack"   // Error cant assgin to  vi.(User).name 
    pi.(*User).name = "jack"
    fmt.Println("%v\n", vi.(User))
    fmt.Println("%v\n", pi.(*User))
}


{1 Tom}
&{1 Jack}
```

### 只有tab data 都为nil 接口才是nil

```c
struct Iface
{
    Itab* tab;
    void* data;
}

struct Itab
{
	InterfaceType* inter;
	Type*	type;
	void (*fun[])(void);
}
//接口表存储元数据信息，　包括接口类型，　动态类型，　以及实现接口的方法指针。　无论是反射还是通过接口调用方法都会用到这些。
```



```go
var a interface{} = nil // tab = nil, data = nil
var b interface{} = (*int)(nil) // tab 包含 *int 类型信息, data = nil
type iface struct {
 itab, data uintptr
}
ia := *(*iface)(unsafe.Pointer(&a))
ib := *(*iface)(unsafe.Pointer(&b))
fmt.Println(a == nil, ia)
fmt.Println(b == nil, ib, reflect.ValueOf(b).IsNil())

true {0 0}
false {505728 0} true

```

### 接口技巧

```go
// 函数直接实现接口
type Tester interface {
 Do()
}
type FuncDo func()
func (self FuncDo) Do() { self() }
func main() {
 var t Tester = FuncDo(func() { println("Hello, World!") })
 t.Do()
}
```

### switch 批量判断　不支持fallthrough

### goroutine

```
runtime.Goexit() // 退出当前ｇｏｒｏｕｔｉｎｅ
sync.WaitGroutp{} // wait goroutine finished 
wg.Add(1) // add wait group
wg.Done()  // wait group -- if  wg == 0 then contine
runtime.Gosched() // 将当前goroutine暂停放回队列等待下次被调度

```

### chan

```go
// make(chan int) // 这样是0 buff 的ｃｈａｎｎｅｌ　是同步的
// make(chan int, 10) 这是可以缓冲１０个元素的ｃｈａｎｎｅｌ
// close(c) 关闭channel
// len(c) 返回缓冲元素数量
// cap(c) 返回缓冲区的大小

for v := range c{
    
}

// for range expr
for{
    if d , ok := <- c; ok{
        pass
    }else{
        break
    }
} 

//ok - idiom
// 向closed channnel 发送数据引发panic 错误，　接收立即返回零值。
// nil channel 无论收发都会被阻塞

var t chan<- int = c  //send-only
var t2 <-chan int = c // recieve only 
for{
    select {
    case v, ok = <- c : 
    case v, ok = <- b :
    case v, ok = <- a :
    default:
    	
	}
	if ok{
        
	}else{
        
	}
}
// select is not sync 

```

### closed channel 发出退出通知

```go
func main(){
    var wg sync.WaitGroup{}
    quit := make(chan bool)
    for i:= 0; i<2; i++{
        wg.Add(1)
        go func(id int){
            defer wg.Done()
            task := func(){
                println(id, time.Now().Nanosecond())
                time.Sleep(time.Second)
                for{
                    select{
                        case <-quit:
                            return 
                        default:
                            task()
                    }
                }
            }(i)
        }
      }
    time.Sleep(time.Second*5)
    close(quit)
    wg.Wait()
}
```

### 用select 实现timeout

```go
w := make(chan bool)
	c := make(chan int, 2)
	go func() {
		select {
		case v := <- c:
			fmt.Println(v)
		case <- time.After(time.Second*3):
			fmt.Println("timeout")
		}
		fmt.Println("!!!")
		w <- true
	}()

	<-w
```

### 初始化

```go
• 每个源⽂件都可以定义⼀个或多个初始化函数。
• 编译器不保证多个初始化函数执⾏次序。
• 初始化函数在单⼀线程被调⽤，仅执⾏⼀次。
• 初始化函数在包所有全局变量初始化后执⾏。
• 在所有初始化函数结束后才执⾏ main.main。
• ⽆法调⽤初始化函数。
```

### 指针

```go
// 分为　unsafe.Pointer 和　uintptr
// 其中uintptr 会被当做普通整数对象被回收
// 看下面这个例子
type data struct {
 x [1024 * 100]byte
}
func test() uintptr {
 p := &data{}
 return uintptr(unsafe.Pointer(p))
}
func main() {
 const N = 10000
 cache := new([N]uintptr)
 for i := 0; i < N; i++ {
 cache[i] = test()
 time.Sleep(time.Millisecond)
 }
}
$ go build -o test && GODEBUG="gctrace=1" ./test
gc607(1): 0+0+0 ms, 0 -> 0 MB 50 -> 45 (3070-3025) objects
gc611(1): 0+0+0 ms, 0 -> 0 MB 50 -> 45 (3090-3045) objects
gc613(1): 0+0+0 ms, 0 -> 0 MB 50 -> 45 (3100-3055) objects
```

### 指针循环应用问题

```go
垃圾回收器能正确处理 "指针循环引⽤"，但⽆法确定 Finalizer 依赖次序，也就⽆法调⽤
Finalizer 函数，这会导致⺫标对象⽆法变成不可达状态，其所占⽤内存⽆法被回收。
type data struct {
	d [1024*100]byte
	o *data
}

func test(){
	var a , b data
	a.o = &b
	b.o = &a
	runtime.SetFinalizer(&a, func(d *data) { fmt.Printf("a %p final.\n", d) })
	runtime.SetFinalizer(&b, func(d *data) { fmt.Printf("b %p final.\n", d) })
}
func main() {
	for {
		test()
		time.Sleep(time.Millisecond)
	}

}


$ go build -gcflags "-N -l" && GODEBUG="gctrace=1" ./test
gc11(1): 2+0+0 ms, 104 -> 104 MB 1127 -> 1127 (1180-53) objects
gc12(1): 4+0+0 ms, 208 -> 208 MB 2151 -> 2151 (2226-75) objects
gc13(1): 8+0+1 ms, 416 -> 416 MB 4198 -> 4198 (4307-109) objects

```

