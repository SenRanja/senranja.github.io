---
title: goland goroutine, workers and chan
date: 2024-08-24 11:52:24
tags:
top: 30
categories:
- Go
---

### Code Snippet

    package main

    import (
        "fmt"
        "sync"
    )

    func worker(ports chan int, wg *sync.WaitGroup) {
        for p := range ports {
        # P receive one value from the chan ports
            fmt.Println("p", p)
            wg.Done()
        }
    }

    func main() {
        ports := make(chan int, 100)
        var wg sync.WaitGroup

        for i := 0; i < cap(ports); i++ {
            go worker(ports, &wg)
        }

        for i := 1; i < 1024; i++ {
            wg.Add(1)
            ports <- i
        }

        wg.Wait()
        close(ports)
    }

### 口诀

1. Init Async Tasks, wg

    Init make chain 和 var wg
    ports := make(chan int, 100)
    var wg sync.WaitGroup

2. 俩循环，一 纯go worker，二 add 1 扔任务：
   
   cap go worker
   ---
   wg.Add(1)
   ports <- i

3. 单worker内 Done 
   
    for p := range ports {
            fmt.Println("p", p)
            wg.Done()
        }

4. 俩终结，等与关
   
    wg.Wait()
	close(ports)




