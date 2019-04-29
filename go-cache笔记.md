今天看了go-cache项目很短学到了这些：

１．time.Ticker gob time  这些熟悉了些

２．设计方面呢很简单其实就一个map 然后　janitor 定时清理expired 的entry

​	janitor 有一个stop 的channel 

​	然后这里有点意思的是，以前看别说go 里面没有finalize函数没办法在像java cpp 哪样对象清理的时候完成	一些事情。但实际上呢是可以的。。。。。

​	在这里面可以这样子

```go
func newCacheWithJanitor(de time.Duration, ci time.Duration, m map[string]Item) *Cache {
	c := newCache(de, m)
	// This trick ensures that the janitor goroutine (which--granted it
	// was enabled--is running DeleteExpired on c forever) does not keep
	// the returned C object from being garbage collected. When it is
	// garbage collected, the finalizer stops the janitor goroutine, after
	// which c can be collected.
	C := &Cache{c}
	if ci > 0 {
		runJanitor(c, ci)
        // 看这里　！！！
		runtime.SetFinalizer(C, stopJanitor)
	}
	return C
}

```

３．作者在尝试 高并发的情况下shared 这样不用锁全部的map 可能会快一丢丢

但是从ｂｅｎｃｈｍａｒｋ上面看并没有很明显。。。。。

理论上是有用的啦但是两次hash也有多花时间。。。

他第一次sharded时候用的hash 用的djb333据说比标准库快５x, 我感觉第一次hash 就是ｓｈａｒｅｄ的话呢

不需要抗碰撞太好的hash 其实就可以了。。。。S

