---
title: Fork And Exec
date: 2020-12-25
categories:
  - linux
markup: mmark
math: true
tags:
    - process
---

# Intro

When i view code of [*micro*](https://github.com/micro/micro).I found a interesting function *run*,so i deep into it. Concededly **Golang** is static programing language,which don't have dymanic load class function like **Java**.So this function probably implement by executing a command.Fortunately,*micro* has pretty clean project structure,i can find it easily.


Create Service instance:
 ```go
func (r *localRuntime) Create(resource runtime.Resource, opts ...runtime.CreateOption) error {
	  ...

		if len(options.Namespace) == 0 {
			options.Namespace = defaultNamespace
		}
		if len(options.Entrypoint) > 0 {
			s.Source = filepath.Join(s.Source, options.Entrypoint)
		}
		if len(options.Command) == 0 {
			options.Command = []string{"go"}

			// not all source will have a vendor directory (e.g. source pulled from a git remote)
			if _, err := os.Stat(filepath.Join(s.Source, "vendor")); err == nil {
				options.Args = []string{"run", "-mod", "vendor", "."}
			} else {
				options.Args = []string{"run", "."}
			}
		}

		
		// create new service
		service := newService(s, options)
    
    ...
}
 ```

 Start Service by forking:

 ```go
// Start starts the service
func (s *service) Start() error {
	s.Lock()
	defer s.Unlock()

	if !s.shouldStart() {
		return nil
	}

	// reset
	s.err = nil
	s.closed = make(chan bool)
	s.retries = 0

	if s.Metadata == nil {
		s.Metadata = make(map[string]string)
	}
	s.Status(runtime.Starting, nil)

	// TODO: pull source & build binary
	if logger.V(logger.DebugLevel, logger.DefaultLogger) {
		logger.Debugf("Runtime service %s forking new process", s.Service.Name)
	}

	p, err := s.Process.Fork(s.Exec)
	if err != nil {
		s.Status(runtime.Error, err)
		return err
	}
	// set the pid
	s.PID = p
	// set to running
	s.running = true
	// set status
	s.Status(runtime.Running, nil)
	// set started
	s.Metadata["started"] = time.Now().Format(time.RFC3339)

	if s.output != nil {
		s.streamOutput()
	}

	// wait and watch
	go s.Wait()

	return nil
}
 ```

 `s.Process.Fork(s.Exec)` this is why i write this article.I want to figure it out what different between `fork` and `exec`.

 # Fork And Exec

I found two answers from *StackOverflow*.They explained it very clearly.


    Note that fork() has been invented at the time when no threads were used at all, and a process had always had just a single thread of execution in it, and hence forking it was safe. With Go, the situation is radically different as it heavily uses OS-level threads to power its goroutine scheduling

This answer gvie a background of `fork`

The biggest different are:

Fork:

    The fork call basically makes a duplicate of the current process, identical in almost every way. Not everything is copied over (for example, resource limits in some implementations) but the idea is to create as close a copy as possible.

Exec:

    The exec call is a way to basically replace the entire current process with a new program. It loads the program into the current process space and runs it from the entry point.

The following diagram write by answer's author.

    +--------+
    | pid=7  |
    | ppid=4 |
    | bash   |
    +--------+
        |
        | calls fork
        V
    +--------+             +--------+
    | pid=7  |    forks    | pid=22 |
    | ppid=4 | ----------> | ppid=7 |
    | bash   |             | bash   |
    +--------+             +--------+
        |                      |
        | waits for pid 22     | calls exec to run ls
        |                      V
        |                  +--------+
        |                  | pid=22 |
        |                  | ppid=7 |
        |                  | ls     |
        V                  +--------+
    +--------+                 |
    | pid=7  |                 | exits
    | ppid=4 | <---------------+
    | bash   |
    +--------+
        |
        | continues
        V


Go back to *micro* code,implemention of `fork`:

```go
func (p *Process) Fork(exe *process.Binary) (*process.PID, error) {

	// create command
	cmd := exec.Command(exe.Package.Path, exe.Args...)

	cmd.Dir = exe.Dir
	// set env vars
	cmd.Env = append(cmd.Env, os.Environ()...)
	cmd.Env = append(cmd.Env, exe.Env...)

	// create process group
	cmd.SysProcAttr = &syscall.SysProcAttr{Setpgid: true}

	in, err := cmd.StdinPipe()
	if err != nil {
		return nil, err
	}
	out, err := cmd.StdoutPipe()
	if err != nil {
		return nil, err
	}
	er, err := cmd.StderrPipe()
	if err != nil {
		return nil, err
	}
	// start the process
	if err := cmd.Start(); err != nil {
		return nil, err
	}

	return &process.PID{
		ID:     fmt.Sprintf("%d", cmd.Process.Pid),
		Input:  in,
		Output: out,
		Error:  er,
	}, nil
}
```

That's it ,sorry my english.

# Ref

 - [differences-between-fork-and-exec](https://stackoverflow.com/questions/1653340/differences-between-fork-and-exec)
 - [How do I fork a go process?](https://stackoverflow.com/questions/28370646/how-do-i-fork-a-go-process/28371586#28371586)