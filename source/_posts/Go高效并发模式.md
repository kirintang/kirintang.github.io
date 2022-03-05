---
title: Go高效并发模式
date: 2022-03-05 18:28:38
tags: go
---

`Golang`中实现高效并发场景模式。

<!-- more -->

#### 一，`for select`无限循环模式

`for + select`实现多路复用，无条件`return`会一直执行`default`。

```go
for {
  select {
    case <-done:
    	return
  	default:
    	// TODO
  }
}
```

#### 二，`for range select`有限循环模式

```go
for _, s := range []int {
  select {
    case <-done:
    	return
    case resultCh <-s:
  }
}
```

#### 三，`select timeout`模式

设置超时时间，避免因网络等其他问题导致此次获取不到响应。

```go
func main() {
  result := make(chan string)
  timeout := time.After(3 * time.Second)
  go func() {
    time.Sleep(5 * time.Second)
    result <- "server result"
  }()
  for {
    select {
      case v := result:
      	fmt.Println(v)
      case <-timeout:
      	fmt.Println("timeout")
      	return
    	default:
      	fmt.Println("wait...")
      	time.Sleep(1 * time.Second)
    }
  }
}

// wait...
// wait...
// wait...
// timeout
```

#### 四，`Context的WithTimeout`函数超时取消

推荐使用`WithTimeout`而非`select+timeout`.

```go
func main() {
  ctx, stop := context.WithTimeout(context.Background(), 3 * time.Second)
  go func() {
    worker(ctx, "张三")
  }()
  go func() {
    worker(ctx, "李四")
  }()
  time.Sleep(5 * time.Second)
  stop()
  fmt.Println("...")
}

func worker(ctx context.Context, name string) {
  for {
    select {
      case <-ctx.Done():
      	fmt.Println("下班咯~")
      	return
      default:
      	fmt.Println(name, "摸鱼中~")
      time.Sleep(1 * time.Second)
    }
  }
}

// 张三 摸鱼中~
// 李四 摸鱼中 ~
// 张三 摸鱼中~
// 李四 摸鱼中 ~
// 张三 摸鱼中~
// 李四 摸鱼中 ~
// 下班咯~
// 下班咯~
// wait 2s
// ...
```

#### 五，`Pipline`模式

也称之为流水线模式，如生产手机：零件采购->组装->打包成品。

```go
func main() {
  coms := buy(10)
  phones := build(coms)
  packs := pack(phones)
  for p := range packs {
    fmt.Println(p)
  }
}

// 零件采购
func buy(n int) <-chan string {
  out := make(chan string)
  go func() {
    defer close(out)
    for i := 0; i <= n; i++ {
      out <-fmt.Sprint("零件", i)
    }
  }()
  return out
}

// 组装
func build(in <-chan string) <-chan string {
  out := make(chan string)
  go func() {
    defer close(out)
    for c := range in {
      out <- "组装()" + c + ")"
    }
  }()
  return out
}

// 打包
func pack(in <-chan string) <-chan string {
  out := make(chan string)
  go func() {
    defer close(out)
    for c := range in {
      out <- "打包(" + c + ")"
    }
  }()
  return out
}

/**
打包(组装()零件0))
打包(组装()零件1))
打包(组装()零件2))
打包(组装()零件3))
打包(组装()零件4))
打包(组装()零件5))
打包(组装()零件6))
打包(组装()零件7))
打包(组装()零件8))
打包(组装()零件9))
打包(组装()零件10))
*/
```

#### 六，扇入扇出模式

流水线模式可以看出，中间某个环节如果耗时比较严重的情况下，相应的会拖慢其他环节效率，为了提升性能，可以考虑扇入扇出模式，考虑给中间环节增加成本，如上给组装环节增加人手：

```go
func main() {
	coms := buy(10)
	phones1 := build(coms)
	phones2 := build(coms)
	phones3 := build(coms)
	phones := merge(phones1, phones2, phones3)
	packs := pack(phones)
	for p := range packs {
		fmt.Println(p)
	}
}

// 零件采购
func buy(n int) <-chan string {
	out := make(chan string)
	go func() {
		defer close(out)
		for i := 0; i <= n; i++ {
			out <- fmt.Sprint("零件", i)
		}
	}()
	return out
}

// 组装
func build(in <-chan string) <-chan string {
	out := make(chan string)
	go func() {
		defer close(out)
		for c := range in {
			out <- "组装()" + c + ")"
		}
	}()
	return out
}

// 打包
func pack(in <-chan string) <-chan string {
	out := make(chan string)
	go func() {
		defer close(out)
		for c := range in {
			out <- "打包(" + c + ")"
		}
	}()
	return out
}

// 扇入函数，把多个channel中的数据发送到一个channel中
func merge(ins ...<-chan string) <-chan string {
	var wg sync.WaitGroup
	out := make(chan string)
	p := func(in <-chan string) {
		defer wg.Done()
		for c := range in {
			out <- c
		}
	}

	wg.Add(len(ins))
	for _, cs := range ins {
		go p(cs)
	}
	go func() {
		wg.Wait()
		close(out)
	}()

	return out
}

/**
打包(组装()零件2))
打包(组装()零件3))
打包(组装()零件1))
打包(组装()零件5))
打包(组装()零件6))
打包(组装()零件4))
打包(组装()零件0))
打包(组装()零件7))
打包(组装()零件9))
打包(组装()零件10))
打包(组装()零件8))
*/
```

#### 七，未来模式(`Futures`)

我们发现有时候有的任务之间并不需要依赖，为了提高性能，有些独立的任务可以并发执行：

> `Futures`模式主协程不用等待子协程返回的结果，可以先去做其他的事，等将来子协程结果返回后再来取，如果子协程没有返回，则继续等待。

如以做饭为例：洗菜、烧水2个步骤之间没有依赖关系，可同时做：

```go
func main() {
	vegetablesCh := washVegetables()
	waterCh := boilWater()
	fmt.Println("安排好洗菜烧水了，休息一下")
	time.Sleep(2 * time.Second)
	fmt.Println("开始做饭了，看下菜和水好了吗")
	vegetables := <-vegetablesCh
	water := <-waterCh
	fmt.Println("准备好了，开始做饭:", vegetables, water)
}

// 洗菜
func washVegetables() <-chan string {
	vegetables := make(chan string)
	go func() {
		time.Sleep(5 * time.Second)
		vegetables <- "菜洗好了"
	}()

	return vegetables
}

// 烧水
func boilWater() <-chan string {
	water := make(chan string)
	go func() {
		time.Sleep(5 * time.Second)
		water <- "水烧好了"
	}()

	return water
}

/**
安排好洗菜烧水了，休息一下
开始做饭了，看下菜和水好了吗
准备好了，开始做饭: 菜洗好了 水烧好了
*/
```