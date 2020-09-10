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