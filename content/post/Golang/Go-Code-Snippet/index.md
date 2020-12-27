---
title: Go Code Style
date: 2020-09-01
categories:
  - programing
markup: mmark
math: true
tags:
    - programing
---

### goroutine limiter 
```go
var count int32
resp := make([]*CommandResponse, len(req.Images))
//
waitForAllJob := make(chan struct{}, cfg.Server.MaxParallelNum)
defer close(waitForAllJob)
//
for i := 0; i < len(req.Images); i++ {
	waitForAllJob <- struct{}{}
	go func(index int, img *model.Image) {
		response := process(img)
		resp[index] = response
		<-waitForAllJob
		atomic.AddInt32(&count, 1)
	}(i, &req.Images[i])
}
//
for int32(len(req.Images)) != count {
}
```

### Terminal Color
```go
package main

import "fmt"

var (
  Info = Teal
  Warn = Yellow
  Fata = Red
)

var (
  Black   = Color("\033[1;30m%s\033[0m")
  Red     = Color("\033[1;31m%s\033[0m")
  Green   = Color("\033[1;32m%s\033[0m")
  Yellow  = Color("\033[1;33m%s\033[0m")
  Purple  = Color("\033[1;34m%s\033[0m")
  Magenta = Color("\033[1;35m%s\033[0m")
  Teal    = Color("\033[1;36m%s\033[0m")
  White   = Color("\033[1;37m%s\033[0m")
)

func Color(colorString string) func(...interface{}) string {
  sprint := func(args ...interface{}) string {
    return fmt.Sprintf(colorString,
      fmt.Sprint(args...))
  }
  return sprint
}

func main() {
  fmt.Println(Info("hello, world!"))
}
```