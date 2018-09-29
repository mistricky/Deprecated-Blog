# 【golang】并发遍历指定目录下的所有文件大小

这篇博文旨在写一个遍历指定目录下的所有文件大小的demo，最后打印出该目录所占的空间大小，还会拿没有使用goroutine的情况下， 计算所花费的时间。

先上一个没有使用goroutine的版本

```
package main

import (
	"sync"
	"time"
	"fmt"
	"path/filepath"
	"os"
	"log"
	"io/ioutil"

)

const (
	//goTest目录
	GO_TEST_DIR_PATH = "/Users/haodawang/Documents/tests"
)

var waitGroup sync.WaitGroup
var ch = make(chan struct{}, 1)
/* 
*	计算整个目录所占磁盘大小
*/

func dirents(path string) ([]os.FileInfo, bool) {
	entries, err := ioutil.ReadDir(path)
	if err != nil { 
		log.Fatal(err) 
		return nil, false
	}
	return entries, true
}

//递归计算目录下所有文件
func walkDir(path string, fileSize chan <- int64){
	fmt.Printf("\rwalk ... %s\n", path)
	entries, ok := dirents(path)
	if !ok { 
		log.Fatal("can not find this dir path!!") 
		return
	}
	for _, e := range entries{
		if e.IsDir() {
			walkDir(filepath.Join(path, e.Name()), fileSize)
		} else {
			fileSize <- e.Size()
		}
	}
}

func main(){
	//文件大小chennel
	fileSize := make(chan int64)
	//文件总大小
	var sizeCount int64
	//文件数目
	var fileCount int

	//计算目录下所有文件占的大小总和
	go func(){
		walkDir(GO_TEST_DIR_PATH, fileSize)
		defer close(fileSize)
	}()
	
	t := time.Now()
	for size := range fileSize {
		fileCount++
		sizeCount += size
	}
	fmt.Println("花费的时间为 " + time.Since(t).String())
	fmt.Printf("该目录大小为 %.1fGB\n文件总数为 %d\n", float64(sizeCount)/1e9, fileCount)
}
```

![这里写图片描述](https://img-blog.csdn.net/20180704174200126?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0hhb0RhV2FuZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

从上面的运行结果来看， 在我的tests目录下，有3.3GB的东西，然后运行时间有8s多。

接下来看一个goroutine的版本

```
package main

import (
	"sync"
	"time"
	"fmt"
	"path/filepath"
	"os"
	"log"
	"io/ioutil"

)

const (
	//goTest目录
	GO_TEST_DIR_PATH = "/Users/haodawang/Documents/tests"
)

var waitGroup sync.WaitGroup
var ch = make(chan struct{}, 255)
/* 
*	计算整个目录所占磁盘大小
*/

func dirents(path string) ([]os.FileInfo, bool) {
	entries, err := ioutil.ReadDir(path)
	if err != nil { 
		log.Fatal(err) 
		return nil, false
	}
	return entries, true
}

//递归计算目录下所有文件
func walkDir(path string, fileSize chan <- int64){
	defer waitGroup.Done()
	fmt.Printf("\rwalk ... %s\n", path)
	ch <- struct{}{} //限制并发量
	entries, ok := dirents(path)
	<- ch
	if !ok { 
		log.Fatal("can not find this dir path!!") 
		return
	}
	for _, e := range entries{
		if e.IsDir() {
			waitGroup.Add(1)
			go walkDir(filepath.Join(path, e.Name()), fileSize)
		} else {
			fileSize <- e.Size()
		}
	}
}

func main(){
	//文件大小chennel
	fileSize := make(chan int64)
	//文件总大小
	var sizeCount int64
	//文件数目
	var fileCount int

	//计算目录下所有文件占的大小总和
	waitGroup.Add(1)
	go walkDir(GO_TEST_DIR_PATH, fileSize)

	go func(){
		defer close(fileSize)
		waitGroup.Wait()
	}()
	
	t := time.Now()
	for size := range fileSize {
		fileCount++
		sizeCount += size
	}
	fmt.Println("花费的时间为 " + time.Since(t).String())
	fmt.Printf("该目录大小为 %.1fGB\n文件总数为 %d\n", float64(sizeCount)/1e9, fileCount)
}
```

![这里写图片描述](https://img-blog.csdn.net/20180704174519423?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0hhb0RhV2FuZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

从运行结果来看的话，花费了4s多，基本上的花费是上个版本的一半。

这里为了限制并发量，避免too many open file这样的错误，使用了一个临时的channel

```
var ch = make(chan struct{}, 255)
```

```
ch <- struct{}{} //限制并发量
entries, ok := dirents(path)
<- ch
```

