
# GolangUK talks notes

![](https://i.imgur.com/E4FhipS.jpg)

## About us

![](https://i.imgur.com/YVQOUU9.jpg =450x)

From left to right:

- Hernán Kleiman - https://twitter.com/justiniano
- Sergio Moya - https://twitter.com/soyelsergillo
- Gonzalo Serrano - https://twitter.com/gonzaloserrano
- Ferran Orriols - https://twitter.com/ferranorriols
- Alberto Fernández - https://twitter.com/albertofem

#### Sponsored by Social Point

- https://twitter.com/socialpointeng
- http://www.socialpoint.es

#### Related

Ultimate Go Workshop hackpad https://hackmd.io/s/SJF9j9Wc

## Schedule

http://golanguk.com/schedule

![](https://i.imgur.com/ARdhPHX.jpg =400x)

## Keynote: SOLID Go design
![](http://i.imgur.com/1d1nzZ9.jpg)
> Dave Cheney - http://golanguk.com/speakers/#dave-cheney

:eight: :star: :star: :star: :star: :star: :star: :star: :star:

### The **SOLID** principles
Objective: design good programs.

- **S**ingle Responsability
  - Coupling & Cohesion
  - package names 
  	- good
		- net/http, os/exec, encoding/json
		- importing packages increases coupling
	- bad: server, private, common, util
  - UNIX tools are small sharp tools that together solve larger tasks. Go stdlib tries to mimic that, we all shall do.
 - **O**pen/closed principle
   - open for extension
   - closed for modification: struct embedding example; an embedded type does'nt know its embedded; you can't change the method of the embedded type from the one that embeds it it.
 - **L**iskov substitution principle
 	- Substitution powered by interfaces. Avoid concrete types if possible.
 	- Small interfaces are more powerful (e.g `io.Reader`)
 	  - a lot of the stdlib types implement Reader
	  - you can change one for the other and it will work
  - **I**nterface segregation principle
  	- Do not force clients to implements non-required methods
  	- Again, simpler/smaller is better.
	- Use the smaller interface possible that will do the work. E.g. need a `File` or you can generalize your method receiving instead a `ReadWriteCloser`? Or maybe you just call to `Write()`and you can change it for `Writer`? 
	- Related tool https://github.com/mvdan/interfacer
  - **D**ependency inversion principle
	- structure of your import graph
		- should be acyclic, otherwise compile error but also it would be a design issue

### Summary
- Interfaces let you apply the **SOLID** principles to Go programs.
- Focus on reuse. Forget frameworks, think about design.

## Applied Go kit
> Peter Bourgon - http://golanguk.com/speakers/#peter-bourgon

:six: :star: :star: :star: :star: :star: :star:

http://gokit.io - A toolkit for microservices

### microservices

- Microservices are terrible. gokit tried to make it less terrible. 2 year old project.
- Described by:
	- size: one dev can manage it 
	- data: implements a single bounded context <- we like this one
	- ops: built and deployed independently (12Factor)
	- Architecture: What microservices could be?: 
	  - monolithic -> microservices
	  - type1: rpc, CRUD-like
	  - type2: stream-oriented, event sourcing, pipelining, queues
	
> **Microservices solve ORGANIZATIONAL problems, but they  create TECHNICAL problems.**

It solves big teams org. If you have less than 5 devs you don't need them.
	
Problems:
  -  testing becomes really hard.
  - distributed transactions: another really hard problem.
  - deploys, monitoring, logging, distributed tracing, security...
  - Sean Treadway (Soundcloud) found around 40 issues with microservices

### go kit
  - is not a framework. It's there for help you.
  - similar to Twitter's Finangle (Scala)
   - philosophy: encourages good design
     - no global state
     - declarative
	 
#### code show
- without go-kit
  - `Service` interface
  - `ServeHTTP()` to implement the `http.Handler` interface
  - return json
  - add logging before returning, could use decorators etc!
  - add instrumentation, use Prometheus
  - result: bussiness logic mixed with tons of other stuff :cry: 
- with go-kit
  - move non-bussiness stuff to middlewares
- `Endpoint` func type
- middlewares (using decorator design pattern)
- you can also decorate your own `Service`
	  
#### transport

- HTTP, you can also use it with the Gorilla mux.
- gRPC https://github.com/go-kit/kit/blob/master/examples/addsvc/transport_grpc.go
- Thrift, others...

#### dist tracing

- Zipkin (Twitter's open source distributed tracing system)
- middleware to move trace ID from HTTP header to Context.

![](https://i.imgur.com/QnEYGkv.png =512x300)
		
## Idiomatic Go Tricks
> Mat Ryer - http://matryer.com

:four: :star: :star: :star: :star:

- Idiomatic:
  - adjective: using, containing, denoting expressions.
  - talk natural go language
- Go code usually has the same appearance
  - returns a tuple, {interface, error}
  - line of sight
    - quickly scan to see what it does

### tips

  - make happy return that last statement if possible
  - next time you write else, consider flipping the logic
  - single method interfaces 
    - type func to implement the interface
  - log blocks. Prettify them adding `------` separators.
  - return teardown functions. teardown functions basically cleanup the called method garbage.
  - good timing. Basically talked about deferring stop methods on timers. Not serious business here.
  - use type assertion in order to execute one or another logic in your program
  - mocking:
    - simple mocks. Create a mock that implements  only the methods you need from the interface you want to mock.
    - third party structs. If they don't provide an interface, just create it.
  - retrying. Do not bloat the `Try` function with arguments
  - empty struct implementations.
    - `struct{}` to group methods together
  - semaphores in goroutines.
    - buffered channels. How many concurrent goroutines you want. Special dedication to Mr **William Kennedy** who encouraged avoid use them at yesterday's workshop.
  - debug logs with a build tag

### Summary
How to become a native speaker:

- read the std lib
- write obvious code
- do not surprise your users
- simplicity

## Cloud in your Cloud
![](http://i.imgur.com/KqPoebX.jpg =512x)
> Matthew Cambell - http://golanguk.com/speakers/#matthew-campbell

:six: :star: :star: :star: :star: :star: :star: 

AKA How we build DigitalOcean

- their infra
  - several apps in dozen nodes
  - a fleet of 10K company nodes
  - millions of customer nodes 

![](http://i.imgur.com/GrsapRr.jpg =512x)

### how to build go code
- monorepo
	- cross-cutting concern changes in a single commit
	- they have custom lint tools that will open source soon to solve issues :smile:
	- quick security patches appliance
- pull requests + code review
- HTTP/JSON -> gRPC
	- schemas + codegen rule. Alternative http://swagger.io/specification/
- service discovery
	- 50% in the audience using Consul, some etcd and little ZooKeeper (he says it's a nightmare)
		- Consul understands regions (in fact, they have 11) for e.g if your run in a cloud env
	- he says SD is one if the biggest wins of Microservices arch
	- Consul scales for real: they have 10K nodes.
	- DSN SRV vs API
- metrics: Prometheus cluster
	- they mix it with Consul to gather metrics for new services
- deploy
	- commit to master generates 
	  - builds
	  - tests
	  	- monorepo makes them run integration tests between microservices which code is in the same repo
	  - docker images. 
	    - created on every build.
	- feature flagging
	  - no multiple branches, no multiple versions
	- incremental rollout with Chef
	  - no blue green deployments.
- monitoring
    - they use graphite + prometheus + grafana (he encourage the use of this last one)
    	- because prometheus is pull-oriented they don't crash a push system like they did with http://opentsdb.net/
- structured logging
    - ![](http://i.imgur.com/Y8B9Ksw.jpg =512x)
    - breaking logs down into something readable. JSON formatted log as example.
    - rsyslog + elasticsearch + kibana
    - alternatives: logdna, loggly, splunk
	- logging and metrics will converge
- dist tracing
	- still sucks
	- global transaction ID + an ID per microservice => complicated
	- Zipkin => he says **it sucks** (storage and UI) Instead use opentracing.
		- go and do a good one in go
- uptime monitoring
    - via Service Discovery.
    - Nagios in every instance.

## Advanced testing concepts for Go 1.7
![](http://i.imgur.com/2Ybb4mP.jpg =512x)
> Marcel Ban Lohuizen - Go team - http://golanguk.com/speakers/#marcel-van-lohuizen

:six: :star: :star: :star: :star: :star: :star:

- Go 1.7 is out (╯°. °）╯︵ ┻┻
> The testing package now supports the definition of tests with subtests and benchmarks with sub-benchmarks. This support makes it easy to write table-driven benchmarks and to create hierarchical tests. It also provides a way to share common setup and tear-down code. See the package documentation for details. -- from 1.7 release notes
- tests
- benchmarks
	- pre 1.7: table-driven tests
	  - pros: less duplication code, less time 
	  - problem: benchmarking non-friendly, you had to create funcs and call them from the tests
	- 1.7: table-driven benchmarks 
- error messages
	- 1.7. They added message formatting. `t.Errorf`
- error/fatal/skip
	- pre 1.7: first error stops the execution
	- 1.7: multiple errors reported with the subtest that throws it
- test/benchmark isolation: how to execute just a subtest that fails 
	- pre 1.7: you do it manually commenting etc
	- 1.7: 
	  - `go test --run={test_name}/"{regex}"`
	  - every slash separates a regex.
	  - Examples:
	    - `go test --run=TestTime/"in Europe"`
	    - `go test --run=TestTime/12:[0-9]`
- test names are unique, if you don't pass a name it will generate a sequence
- SetUp and TearDown: usually frameworks emulate XUnit
  - pre 1.7: 
    ```go
    func TestFoo(b * testing.B) {
      //common set-up code

      // test

      // common tear-down code
    }
    ```
  - 1.7: They encourage us to use the following approach:
  - in the Run func start with setup and defer the teardown.
- parallelism
  - how to separate parallel tests that you don't want to run together? (maybe they allocate similar stuff etc)
    - `tc := tc` inside a concurrent loop trick :-| hope go vet detects it soon :-|++
      - otherwise the loop beeing concurrent the next concurrent exec of the loop would overwrite tc and things would go nasty
  - teardown in parallel tests
    - sync parallel tests in a group putting them together inside a func runned by `r.Run()` and after that call put the teardown func 

## Developing Apps for Developing Countries with go-mobile
> Natalie Pistunovich - http://golanguk.com/speakers/#natalie-pistunovich

:four: :star: :star: :star: :star:

* experience for high consuming data applications, which are normal here but very data consuming in developing countries
* this implies costs for these developing countries that people simply cannot afford

### Introduction to Africa

* mercator projection: bad sizes for continents
* Galls-Peters projection: Africa is actually huge
* Africa is BIG
* Africa population is Europe and NA combined
* Africa has a 1250-3000 languages spoken, comparing with 225 spoken in Europe
* data consumption of Africa is expectdly lower than Africa and NA. Very very much lower.
* mobile consumption is not that much differenced
	* easier to bring mobile wireless network than wired connections, cable, etc.
* phone types distribution shows that Africa's population mostly own featurephone (no smart features, internet, etc.)

### Initatives to improve Africa's situation

* Google
	* Project Loon: Aerial WiFi
	* Project Link: Metro Fiber and WiFi
* Facebook:
	* Aquila Aircraft
	* https://info.internet.org/en/

### App Store stats

* in developing countries, most of applications are social applications (Messenger, WhatsApp, Skype, Facebook)
	* most people don't use app store apps, they download without going through the app store
* in Europe and NA most of applications are games and social

### Targeting apps for Africa

* Africa is diverse as hell: no surprise which such a big habitable area
* local demand: weather apps, sharing apps and educational apps.
* we need to write applications who consum less data: baterry and network transfer
* proposal for a solution: gomobile
	* multicore smartphone + Parallelism in GoMobile = Less data consumption
* native vs hybrid apps
	* both approaches have pros and cons

### Problems when developing apps for Africa

* freedom:
	* state-sponsor surveillance
	* regulation to control online media
	* detention and arrest of bloggers, etc.
* UX
	* there is not much done in this are, it's still in research and much to invent by now (rules are different than Europe, NA, etc.) 
	* symbols over words
* Localization
	* many languages (official languages are not very adapted to reality)
	* Different norms

## A beginners guide to Context
![](http://i.imgur.com/rLuHhS3.jpg =512x)
> Paul Crawford - http://golanguk.com/speakers/#paul-crawford 

:four: :star: :star: :star: :star:

- problems
  - each new request spawns it's own goroutine
  - goroutines don't have any "thread local" state
  - you have to care about cancellation
- solution: `Context` package
  - request scoped data
  - cancellation, deadlines & timeouts
  - it is safe for concurrent use
  - must be the first parameter, is a best practice
 
> 1.7 includes context in the core standard library

### derived contexts
  - package provides derive new Context values from existing ones
  - `Background()`: the top level context for incoming request
  - `WithCancel()`: return a copy of the parent with a new Done channel, this is closed when the cancel function is fire
  - `WithTimeout()`: returns a context with the deadline
  - ...
  
### more

- you can store whatever type you want in the context (we actually now that storing our type is not recommended at all)
  - examples:
    - `WithValues()`: add user session to the context
    - `WithTimeout()`: request to obtain the resource from an api
- In 1.7: `client.Do(request.WithContext(ctx))`
- `ctxthttp` helper package
  - context-aware HTTP requests.

#### demo
A search engine mixing results from DuckDuckGo and giphy

## Go from Dev to Prod
> Florin Pățan - http://golanguk.com/speakers/#marcel-van-lohuizen

:three: :star: :star: :star:

- logging: structured with useful info
- building: vendor your deps but not if you write a lib
- testing: do blackbox testing using a different package name for your test (e.g instead of `demo` use `demo_test`) so you don't have access to unexported symbols from your test.
- examples: use example files as documentation.
- profiling: 
  - use benchmarks
  - you can add benchmark results as a CI step, so if something goes slow than a threshold make the build fail
 - containers: use Scratch or Alpine
 - tracing: 
   - client: go-kit's opentracing, x/net/trace
   - server: https://github.com/tracer/tracer

## Design patterns in Microservices architectures and Gilmour
![](https://i.imgur.com/sOHTgfK.jpg =512x380)
> Piyush Verma - http://golanguk.com/speakers/#piyush-verma
> https://github.com/gilmour-libs/gilmour

:four: :star: :star: :star: :star:

- services vs servers
  - services solve a software problem and a management problem
- communication design patterns
  - Request/Response
    - example with a topic layer to identify endpoints
    - confirmed delivery: response or error
    - HTTP friendly
    - sender handles errors
  - async: signals and slots
    - events -> queue sys -> event processors
      - you can scale the processors horizontally
    - e.g: Kafka, RabbitMQ...
    - patterns: fan-out, broadcast, filtering (wildcards)...
- discovery and load balancing
  - replicated load balancer + health checking
  - Gilmour does Pub/Sub, there is no discovery
- error
  - detection
    - timeout: server or client side
  - handling
    - ignore, publish...
- composition: request piping

## static deadlock detection

> Nicholas Ng - http://golanguk.com/speakers/#nicholas-ng

:nine: :star: :star: :star: :star: :star: :star: :star: :star: :star: 

- concurrency
  - > don't communicate by sharing memory, share memory by communication
  - channels message passing
  - is complicated, bugs hard to find
- deadlocks
  - `all goroutines are asleep - deadlock`
  - what causes a deadlock in go?
      1. sender sends message, Receiver not ready
      2. sender blocks goroutine and wait until Receiver ready
      3. blocked goroutine goes to sleep
  - avoiding deadlocks
    - make sure sender/receiver are matching and compatible
    - then why are hard to avoid in practice?
  - go detects deadlocks at execution time and panics
  - "local deadlock" -> not all program is stuck
  - how to detect a deadlock without running the program?
- static deadlock detector: github.com/nickng/dingo-hunter
    - models the program gorutines as FSMs
    - builds a graph
    - checks for deadlocks in the graph
- modelling concurrency
  - learn from the masters: Hoare, Milner
  - *Session types* on pi-calculus

## Advanced Patterns with io.ReadWriter
![](http://i.imgur.com/PAB6Ixr.jpg =512x)
> Paul Bellamy - http://golanguk.com/speakers/#paul-bellamy

:eight: :star: :star: :star: :star: :star: :star: :star: :star:

- What can you find in `io`, `bufio`, `ioutil`:
  - `Reader`
  - `Writer`
  - `Copy`
  - `LimitReader`
  - `TeeReader`
  - etc
- composition is so easy
- a stream in go is a buffer that you use to read or write in chunks

### examples
- HTTP chunking. Transparently proxy a chunked HTTP in a stream.
  - `httputil.ChunkedReader`
    - problems: 
      - strips out the chunking data
      - can't validate MD5
  - `httputil.ChunkedReader` + `TeeReader`
- prefixing, split into sections
  - `os.Stdout` is a `Writer`
  - `bufio.NewScanner(input)`, `scanner.SplitFunc`, loop over `scanner.Scan`, use `scanner.Bytes`
  - input is a `Reader`
  - `io.Pipe` syncs a `Reader` and a `Writer`, if one does not "work" then it will **block** (which is handy)

## What every developer should know about logging
![](https://i.imgur.com/2WIFJuP.jpg =512x)
> Slawosz Slawinski - http://golanguk.com/speakers/#slawosz-slawinski

:two: :star: :star:

### logging 101

  - two levels
    - debug: lot information
    - info: understand application flow, what happens in application
  - logging infrastructure is complex

### why do we write logs
  - we want to know what our application is doing
  - good logging is the key to solve bugs
  - replace lot of third party monitoring services

### logging infrastructure
  - is not an easy task
  - workflow
    - application is logging to stdout
    - stdout is redirected to file
    - log forwarders are moving logs to centralized log server
    - log server allows to browse and analyze application bahaviours
    - its unix style, one simple step at the time
  - applications
    - logstash
    - kibana
    - summologic
    - splunk

### how to structure logs in microservices
  - ```[timestamp][RID(request id)=foobar][RD(request delta)=0.003][LD(local delta)=0.001][service=API][search][results=432]```
  - we should adapt log message structure to our application needs

### golang log libraries
  - package log
  - glog - logging library from google, allows more control on log levels

### tips and tricks
  - profile your logging system
  - log sampling: when your systems are produciong too many logs, collect only every Xth message, statiscally you will get enough logs to understand your app behaviour
  - security: never log user passwords, keys, name (does anyone do it?)
  - dapper: tracing system from google
http://research.google.com/pubs/pub36356.html

## Implementing software machines in Go & C
> Eleanor McHugh - http://golanguk.com/speakers/#eleanor-mchugh

:seven: :star: :star: :star: :star: :star: :star: :star:

### virtual machines

* many types of virtual machines: system virtualization, hardware emulation
* focus: abstract virtual machines
* inspired by hardware: discrete componentes, processors, storage, communications
* software machines:
	* stateful: state matters. With no state they don't have identity
	* timely: 
	* scriptable: you need to add complex behaviour, if not, it makes no sense to have a machine in the first place
* abstract virtual machine components:
	* stacks, CPU (registers), CPU cache, Heap, Buffers, Network and persistent store
	* timings in different components: depending on heap, buffer, network, etc. different magnitudes

### Memory

* store data and instructions
	* memory model:
		* Von Neumann
		* Harvard
		* Forth (split model)
		* Hybrid
* other parts (not covered)
	* addressing
	* protection
* creating a heap in Go
	* creating sliceHeaders using slice manipulations
	* public interface: 
		* `Bytes()`
		* `Serialise()`
		* `Overwrite()`
	* usage of unsafe: to manipulate bytes is actually legitimate
* array stack:
	* it's a functional data structure which is actually used

### CPU

* dispatch loops: 
	* reading opcodes from mememory
	* change state machine internal state 
	* switch interpreter
		* tokens are often single bytes
* we need stack in order to implement the CPU
* opcodes have mnemonics
* opcodes are stored in a stack to be execute in the dispatch loop
* direct call threading
	* each instruction is represented by a pointer to a function
	* instruction require a machine word
* indirect threading
	* used to represent local jumps in the program
	* https://en.wikipedia.org/wiki/Threaded_code#Indirect_threading
* timings:
	* clock pulse
	* synchronization: provides it across the processor

### Transport triggering

* https://en.wikipedia.org/wiki/Transport_triggered_architecture	
* register machine architecture
* exposes internal buses as components
* operations are side-effects of internal writes

### Random thoughts

* C VM: primitive!
* distances in physical: they matter in computing, analogies with physicist
* it's bad to have things not initialized by default (Rust structs must be initialized all fields by default, Golang not! You want nil pointers? Use Option)
* usefulness of functional data structures
* cactus stack: https://en.wikipedia.org/wiki/Parent_pointer_tree

## Grand Treatise of Modern Instrumentation and Orchestration
> Björn Rabenstein - http://golanguk.com/speakers/#bjoern-rabenstein

:four: :star: :star: :star: :star:

See https://prometheus.io

- inspired by Google's monitoring strategy: alert for high-level stuff and allow inspecting low level.
- USE method: **U**tilization, **S**aturation and **E**rror rates. Also, latency.
- RED method: **R**equest count (rate) **E**rror count (rate) and **D**uration
- Prometheus :moneybag: :bike: 
  - expvar-like API and other types
  - you have a server and query prometheus endpoints in services
  - it gives you the typical go VM stats: goroutine number, memmory, GC etc
  - it has a query lang called PromQL
  - it gives you percentiles etc
  - counters, gauges... Counters always go up, Gauges can go down (e.g 1,2,3,4,3,4,2,1)
  - there are exporter tools around (e.g Apache)

## GoBridge and the Go Community: Initiatives and Opportunities
> Carlisia Campos - http://golanguk.com/speakers/#carlisia-campos

:five: :star: :star: :star: :star: :star:

- http://bridgefoundry.org/
- https://golangbridge.org/
- must listen https://changelog.com/gotime-6/
- they need people, you can help!
  - code combat https://codecombat.com/about to help people learn Go. If interested talk with William Kennedy (@goinggodotnet)

## Managing and scaling Real-time Data Pipelines using Go
![](https://i.imgur.com/WBEq7IG.jpg)
> Jennie Lees - http://golanguk.com/speakers/#jennie-lees

:six: :star: :star: :star: :star: :star: :star:

- works at Riot Games, better known for develop the League Of Legends (LOL) videogame.
- high success: from 7 year old startup to XXX workers and millions of players
- reliability -> monitoring at scale
  - you have to trust your monitoring systems, false alarms do the contrary
- 15+ datacenters, 20K servers, 160k services, 6.5M metrics, 100K values/second, 1TB daily volume
- microservices for the win
  - Diverse tech choices
  - Containers and metal
  - Linux, Mac and Windows
  - In house or as a service
- 2 agents by host.
  - Discovery agent
  - Metrics agent
- Endpoints by host providing data and stats in Json format.

#### The pipeline

![](https://i.imgur.com/cWwb99a.jpg)

#### design goals

  - decouple
    - pure interface abstraction. 
    - no datastore details.
    - in order to make changes easily
  - resilient
    - batching
    - draining
    - stateless
    - they receive attacks
    - what if Kafka goes down?
  - HTTP native
    - HTTP + JSON everywhere
  - instrumented
    - Metrics are key. 
  - fast
    - 75K+ values/sec on a laptop

#### go
- channels
  - they put them everywhere
  - visualizing them is hard
  - event collector pipeline: generate events, transform them, send to Kafka
- design patterns
  - rate limiting
    - buffered channel
  - caching to avoid backpressure when readers block you
  - tickers with `time.After()`
  - pipelining 
    - they use an empty struct channel (but should have used `context.Done()`)
    - `sync.WaitGroup` with `Add(1)` in every step of the pipeline and `Wait()` at the end
  - batching
    - good for client APIs
    - take advantage of buffers
      - bottleneck if ticking is very low
      - know your data: for metrics, client draining: if server fails, discard old items

## Building Cloud Native applications with Go
> Mandy Waite - http://golanguk.com/speakers/#mandy-waite

:seven: :star: :star: :star: :star: :star: :star: :star:

- cloud native evolution
  - build software in virtualized envs
  - https://cncf.io
- arch: 
  - abstractions: containers, clusters, microservices
  - container managers: docker, rkt
- Kubernetes: 
  - http://kubernetes.io/docs/whatisk8s
  - container manager + microservices
  - Manage applications, not machines

#### The flow of Deployment
![](http://i.imgur.com/vOaB1wH.jpg =600x)

#### Kubernetes features

- immutability
  - means robustness
  - containers are **immutable**
  - various immutable server images (deployed using Spinnaker) - http://www.spinnaker.io
  - from [build -> package -> deploy] to [build -> package -> **construct** -> deploy]
    - construct the immutable graph that will be deployed
    - k8s is a declarative way of configuring the construct step
- config
  - in deploy time
    - credentials: pod structure
    - env-based: k/v store
    - no flags or env vars to avoid restarting the app
- deploy
  - with config / secrets
- provides a general resource type system
  - DSL, config based on YAML
  - describes: load balancers, autoscaling, networking, clustering, zones, vm stuff, disk stuff...
    - looks like AWS API without the As a Service part
- nested deployments: example with an app with a backend and a frontend sub-deployments
- packages: based on https://github.com/kubernetes/helm
  - > Helm is a tool for managing Kubernetes charts. Charts are packages of pre-configured Kubernetes resources
  - CHART files (config files) <- Helm <- Repository (S3 as example)
- it uses 
  - etcd for storing and distribute cluster stats and configurations
    - distributed K/V, Raft-based
  - prometheus for monitoring which is integrated with the k8s API 

#### Why golang
- designed for large, distributed, collaborative software projects
- static compilation
- strong alternative to C/C++ for cross platform simple and really functional CLI tools
- low level primitives
  - networking
  - process managing
  - syscalls
- critical mass
  - hipster
  - cool
  - trendy

## Building Mobile SDKs with GoMobile
![](https://i.imgur.com/9Nex3fp.jpg =512x)
> Nic Jackson - http://golanguk.com/speakers/#nic-jackson

:seven: :star: :star: :star: :star: :star: :star: :star:

#### Creating a client SDK in Go
- the horrors of generating cross platform code
  - c/c++: "god please no"
  - xamarin: better but tooling is a pain
  - javascript: uggh
  - go moblie: rather nice
    - go can interact with c
    - compile native binaries
- why do we not write dry code?
  - we do it because dont have the time
- how would we test our client?
  - no sandbox
  - binary protocol
  - tcp sockets

#### Generationg native frameworks with Go Mobile
- protobuf (rpc)
- use ```protoc``` to generate the go code
- create a simple rpc server
- GoMobile could generate a callback to call a native client function
- every in go can run native

#### Integration the generated SDK into an Android app
- live coding
- http://github.com/gokitter <-- source code

## Seven ways to profile Go applications
> Dave Cheney - http://golanguk.com/speakers/#dave-cheney

:eight: :star: :star: :star: :star: :star: :star: :star: :star:

1. **time**
 - GNU's (`/usr/bin/time` not `time`) `/usr/bin/time -v`
2. **go build** -x + toolexec (the latter prepends the command with the tool specified)
  - `go build -toolexec="/usr/bin/time" cmd/compile/internal/gc`
3. **GODEBUG** env variable
  - stats provided by Go
  - `env GODEBUG=gctrace=1` 
4. **Profiling**
  - tips
    - machine must be idle
    - watch out for power saving and thermal scaling
    - avoid virtual machines. too much noise.
    - don't use OSX < El Capitan
    - try to use dedicated performance test hardware
    - run multiple times (OS has several layers of cache and needs warmup etc)
    
4.1. **pprof**
  - descends from Google Performance Tools suite 
  - CPU profiling
  - memory profiling
    - heap allocations (in contrast to stack allocations which are "free", heap ones are costly)
    - Bill tip: don't use htop & friends
  - block profiling
    - aka backpressure measurement?
    - similar to CPU profile but records the amount of time a goroutine spent waiting shared resources.
  - do not mix profiling types! 
  - microbenchmarks
    - `go test -run=XXX -bench=IndexByte -cpuprofile=/tmp/c.p bytes`
      - XXX trick does the test just to run the benchmarks and not the tests
    - https://github.com/pkg/profile
      - good for profiling whole programs from start to end, you put the code in your program
  - reports can be exported to visual graphs
    - in the example, SVG version, the biggest box was a syscall for reading from the network. He said you can avoid to with buffering. 
5. **perf**
  - linux tool, integrate with toolexec
    - what we use on osx? Maybe [this apple tool](https://developer.apple.com/library/tvos/documentation/DeveloperTools/Conceptual/InstrumentsUserGuide) can help.
  - looks awesome
  - percent on the left is how expensive the op was
  - framepointers, Go >=1.7 at least. otherwise doesn't look very good.
6. **flame graph** from netflix
  - func calls are just displayed once and grouped by size
7. **Go tool trace**
  - goroutines
    -  creation/start/end
    -  blocking/unblocking
  -  network
  -  system calls

## Building your own log-based message queue in Go
> Victor Ruiz- http://golanguk.com/speakers/#víctor-ruiz
> https://texlution.com/

:eight: :star: :star: :star: :star: :star: :star: :star: :star:

Good read https://engineering.linkedin.com/distributed-systems/log-what-every-software-engineer-should-know-about-real-time-datas-unifying

- logs 
  - > append only, totally-ordered sequence of records ordered by time --LinkedIn eng blog
  - very simple but also very fast to read and write in disk
  - strong ordering: not all message queues assure it but logs do
  - DDBBs usually write to logs first before writing to the final data structure they use
- Kafka
  - super performant, battle tested, LinkedIn
  - O(1) reads and writes
    - see https://kafka.apache.org/08/design.html, "Constant Time Suffices"
  - distributed, HA, uses ZooKeeper
  - sometimes is too much for some cases -> **lets build one in Go**
- building blocks 
  - indexes
  - readers
    - file readers on demand implementing `io.Reader`
  - watcher: subscribers with channel
  - scanner: map in a buffered way to iterate over the log records
  - streamer: if you need higher throughput than the scanner
- result: BigLog https://github.com/ninibe/netlog/tree/master/biglog  
- demo
  - API
  - simple demo with curl client against the HTTP API
  - pub/sub example

## Real-Time Go
> Andreas Krennmair - http://golanguk.com/speakers/#andreas-krennmair

:four: :star: :star: :star: :star: 

- GC-based langs don't have good reputation in RT systems
- talk is about building a RT bidding system with Go
- RT definition
  - something executed very fast? nop
  - > system... finite and specified period
  - aka **deadline**
- types
  - hard: e.g patience analysis in medic systems
    - if deadline is missed => :bomb:
  - form: occasional deadline misses are tolerable although the result is not useful
  - soft: usefulness degrades with bigger deadline misses
    - e.g 3D FPS game lag (PWNT :-)
    - cam control systems in rooms
- RT bidding
  - automated 2nd price auctions
  - 100-120ms
  - Nash equilibrium, game theory algos
- Travel Audience (Amadeus company)
  - itermediary between publishers and users
  - they do RT bidding
  - segment users based on travel destinations
- "The Problem"
  - intersect user interest and available offers 
- iterations
  - C lang, 2K QPS + memcached
    - never worked really well
  - go version
    - some deadlines missed
    - solution: set an internal deadline of 75ms
    - use `time.After`, also `context.CancelFunc`
  - storage:
    - first memcached
    - then MongoDB
      - https://aphyr.com/posts/322-jepsen-mongodb-stale-reads
      - external DDBB adds net latency
    - then Redis
