## 测试脚手架 ##

这里我们准备写一个etcd的测试，etcd是一个分布式共识系统。我们希望你亲自输入每一行代码，即使你并不理解这些代码。这能帮助你很快上手，而且我们更新时，你也不会感到困惑。

我们先创建一个新的Leiningen工程：

```
$ lein new jepsen.etcdemo
Generating a project called jepsen.etcdemo based on the 'default' template.
The default template is intended for library projects, not applications.
To see other templates (app, plugin, etc), try `lein help new`.
$ cd jepsen.etcdemo
$ ls
CHANGELOG.md  doc/  LICENSE  project.clj  README.md  resources/  src/  test/
```

和其他新的clojure工程结构一样， project.clj是工程描述文件，resources目录放数据文件、被测数据库的配置。src目录放源代码，test放测试代码。注意jepsen.etcdemo整个目录都是一个Jepsen测试，下面的test子目录其实这里用不到。

我们先编辑project.clj，整个文件描述了工程的依赖关系和其他的元数据。添加一个:main命名空间，这样测试可以通过命令行运行。除了依赖于Clojure语言本身，我们还依赖于jepsen的库，以及被测的etcd库Verschlimmbesserung

```
(defproject jepsen.etcdemo "0.1.0-SNAPSHOT"
  :description "A Jepsen test for etcd"
  :license {:name "Eclipse Public License"
            :url "http://www.eclipse.org/legal/epl-v10.html"}
  :main jepsen.etcdemo
  :dependencies [[org.clojure/clojure "1.9.0"]
                 [jepsen "0.1.8"]
                 [verschlimmbesserung "0.1.3"]])
```

现在可以通过```lein run ```执行一下程序

```
$ lein run
Exception in thread "main" java.lang.Exception: Cannot find anything to run for: jepsen.etcdemo, compiling:(/tmp/form-init6673004597601163646.clj:1:73)
```

是的，我们还没有写任何代码。我们需要一个main函数对应project.clj里配置，这个main函数从命令行接受参数，然后执行测试, 打开src/jepsen/etcdemo.clj:

```
(ns jepsen.etcdemo)

(defn -main
  "Handles command line arguments. Can either run a test, or a web server for
  browsing results."
  [& args]
  (prn "Hello, world!" args))
```

Clojure默认调用-main函数，传入命令行输入的任何参数 — 就是lein run后面所有内容.  代码中通过 ```&```表示参数数量是可变的，通过```args```拿到具体的参数列表。我们在Hello, world后打印这些参数：

```
$ lein run hi there
"Hello, world!" ("hi" "there")
```

Jepsen内置了一些参数处理、执行测试、错误处理、日志等等的工具。例如，我们给jepsen.cli命名空间起个别名cli，并且将我们的main函数转为一个Jepsen测试runner:

```
(ns jepsen.etcdemo
  (:require [jepsen.cli :as cli]
            [jepsen.tests :as tests]))


(defn etcd-test
  "Given an options map from the command line runner (e.g. :nodes, :ssh,
  :concurrency, ...), constructs a test map."
  [opts]
  (merge tests/noop-test
         opts))

(defn -main
  "Handles command line arguments. Can either run a test, or a web server for
  browsing results."
  [& args]
  (cli/run! (cli/single-test-cmd {:test-fn etcd-test})
            args))
```

```cli/single-test-cmd```是jepson.cli提供的: 它解析测试的命令行参数，并调用内置的```:test-fn```，test-fn会返回一个map，里面有Jepsen执行测试所含的所有信息。这里，我们测试函数是etcd-test，从命令行获取参数，并填充到一个空用例```noop-test```中。

没有参数时，```cli/run!```会给出一个帮助信息，提示我们需要提供一个命令作为它的第一个参数。我们试试将test参数传进去：

```
$ lein run test
13:04:30.927 [main] INFO  jepsen.cli - Test options:
 {:concurrency 5,
 :test-count 1,
 :time-limit 60,
 :nodes ["n1" "n2" "n3" "n4" "n5"],
 :ssh
 {:username "root",
  :password "root",
  :strict-host-key-checking false,
  :private-key-path nil}}

INFO [2018-02-02 13:04:30,994] jepsen test runner - jepsen.core Running test:
 {:concurrency 5,
 :db
 #object[jepsen.db$reify__1259 0x6dcf7b6a "jepsen.db$reify__1259@6dcf7b6a"],
 :name "noop",
 :start-time
 #object[org.joda.time.DateTime 0x79d4ff58 "2018-02-02T13:04:30.000-06:00"],
 :net
 #object[jepsen.net$reify__3493 0xae3c140 "jepsen.net$reify__3493@ae3c140"],
 :client
 #object[jepsen.client$reify__3380 0x20027c44 "jepsen.client$reify__3380@20027c44"],
 :barrier
 #object[java.util.concurrent.CyclicBarrier 0x2bf3ec4 "java.util.concurrent.CyclicBarrier@2bf3ec4"],
 :ssh
 {:username "root",
  :password "root",
  :strict-host-key-checking false,
  :private-key-path nil},
 :checker
 #object[jepsen.checker$unbridled_optimism$reify__3146 0x1410d645 "jepsen.checker$unbridled_optimism$reify__3146@1410d645"],
 :nemesis
 #object[jepsen.nemesis$reify__3574 0x4e6cbdf1 "jepsen.nemesis$reify__3574@4e6cbdf1"],
 :active-histories #<Atom@210a26b: #{}>,
 :nodes ["n1" "n2" "n3" "n4" "n5"],
 :test-count 1,
 :generator
 #object[jepsen.generator$reify__1936 0x1aac0a47 "jepsen.generator$reify__1936@1aac0a47"],
 :os
 #object[jepsen.os$reify__1176 0x438aaa9f "jepsen.os$reify__1176@438aaa9f"],
 :time-limit 60,
 :model {}}

INFO [2018-02-02 13:04:35,389] jepsen nemesis - jepsen.core Starting nemesis
INFO [2018-02-02 13:04:35,389] jepsen worker 1 - jepsen.core Starting worker 1
INFO [2018-02-02 13:04:35,389] jepsen worker 2 - jepsen.core Starting worker 2
INFO [2018-02-02 13:04:35,389] jepsen worker 0 - jepsen.core Starting worker 0
INFO [2018-02-02 13:04:35,390] jepsen worker 3 - jepsen.core Starting worker 3
INFO [2018-02-02 13:04:35,390] jepsen worker 4 - jepsen.core Starting worker 4
INFO [2018-02-02 13:04:35,391] jepsen nemesis - jepsen.core Running nemesis
INFO [2018-02-02 13:04:35,391] jepsen worker 1 - jepsen.core Running worker 1
INFO [2018-02-02 13:04:35,391] jepsen worker 2 - jepsen.core Running worker 2
INFO [2018-02-02 13:04:35,391] jepsen worker 0 - jepsen.core Running worker 0
INFO [2018-02-02 13:04:35,391] jepsen worker 3 - jepsen.core Running worker 3
INFO [2018-02-02 13:04:35,391] jepsen worker 4 - jepsen.core Running worker 4
INFO [2018-02-02 13:04:35,391] jepsen nemesis - jepsen.core Stopping nemesis
INFO [2018-02-02 13:04:35,391] jepsen worker 1 - jepsen.core Stopping worker 1
INFO [2018-02-02 13:04:35,391] jepsen worker 2 - jepsen.core Stopping worker 2
INFO [2018-02-02 13:04:35,391] jepsen worker 0 - jepsen.core Stopping worker 0
INFO [2018-02-02 13:04:35,391] jepsen worker 3 - jepsen.core Stopping worker 3
INFO [2018-02-02 13:04:35,391] jepsen worker 4 - jepsen.core Stopping worker 4
INFO [2018-02-02 13:04:35,397] jepsen test runner - jepsen.core Run complete, writing
INFO [2018-02-02 13:04:35,434] jepsen test runner - jepsen.core Analyzing
INFO [2018-02-02 13:04:35,435] jepsen test runner - jepsen.core Analysis complete
INFO [2018-02-02 13:04:35,438] jepsen results - jepsen.store Wrote /home/aphyr/jepsen/jepsen.etcdemo/store/noop/20180202T130430.000-0600/results.edn
INFO [2018-02-02 13:04:35,440] main - jepsen.core {:valid? true}

Everything looks good! ヽ(‘ー`)ノ
```

我们看到Jepsen起了一堆worker -- 每个worker在数据库上负责执行一系列操作 -- 然后nemesis服务，进行故障注入。我们没有告诉这些进程做什么，所以他们很快就退出了。Jepsen将测试结果写入store目录，并会打印一份简要的分析。

noop-test默认使用n1,n2, … n5的节点。如果你的节点名字不一样，测试程序会连接失败。这个没有问题！你只需要在命令行下更改节点名称。

```
$ lein run test --node foo.mycluster --node 1.2.3.4
```

或者传入一个文件名，在文件里列出所有节点，每行对应一个节点。如果你用的是AWS集群，本地home目录下默认会有一个叫nodes的文件，可以传入这份文件。

```
$ lein run test --nodes-file ~/nodes
```

如果仍然ssh失败，你要检查本地的ssh agent是否持有所有节点的key。```ssh some-db-node```应该能免密登录。你也可以在命令行下覆盖默认的用户名、密码以及证书。用```lein run test --help```看一下帮助。

```
$ lein run test --help
#object[jepsen.cli$test_usage 0x7ddd84b5 jepsen.cli$test_usage@7ddd84b5]

  -h, --help                                             Print out this message and exit
  -n, --node HOSTNAME             [:n1 :n2 :n3 :n4 :n5]  Node(s) to run test on
      --nodes-file FILENAME                              File containing node hostnames, one per line.
      --username USER             root                   Username for logins
      --password PASS             root                   Password for sudo access
      --strict-host-key-checking                         Whether to check host keys
      --ssh-private-key FILE                             Path to an SSH identity file
      --concurrency NUMBER        1n                     How many workers should we run? Must be an integer, optionally followed by n (e.g. 3n) to multiply by the number of nodes.
      --test-count NUMBER         1                      How many times should we repeat a test?
      --time-limit SECONDS        60                     Excluding setup and teardown, how long should a test run for, in seconds?
```

这篇教程里，我们会通过```lein run test …```命令重跑测试。每次执行测试，Jepsen会在store目录下创建一个新的子目录。你可以在store/latest目录下看到最新结果：

```
$ ls store/latest/
history.txt  jepsen.log  results.edn  test.fressian
```

history.txt列出了测试过程中执行的所有操作 — 现在我们的是空的，因为noop测试没有执行任何操作。jepsen.log复制了测试时控制台打印的所有日志。result.edn有测试的分析，就是每轮测试结束时看到的内容。最后，test.fressian有测试的原始数据，包括完整的机读历史和分析，以便后续做一些特定的分析。

Jepsen也内置了一个web页面，可以看到执行结果。我们需要在main函数中加入这个功能：

```
(defn -main
  "Handles command line arguments. Can either run a test, or a web server for
  browsing results."
  [& args]
  (cli/run! (merge (cli/single-test-cmd {:test-fn etcd-test})
                   (cli/serve-cmd))
            args))
```

这里，我们要通过merge合并多个命令。

```
$ lein run serve
13:29:21.425 [main] INFO  jepsen.web - Web server running.
13:29:21.428 [main] INFO  jepsen.cli - Listening on http://0.0.0.0:8080/
```

