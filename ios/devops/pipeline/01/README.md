[TOC]



## 1、gitlac-ci.yml 语法

### 1. 简单示例

```yml
stages:
  - build
  - test

job1:
  stage: test
  script:
  - sh test.sh
  artifacts:
    paths:
    - data_test/

job2:
  stage: build
  script:
  - echo 'suceed test' >> txt1
```

- 上面的代码块定义 **两个阶段（stages）**

  - build 阶段
  - test 阶段

- stages 中的阶段, 按照 **顺序** 从上往下进行

- 接下来定义了 两个 job1、job2

  - job1 属于 **test 阶段** 的工作
  - job2 属于 **build 阶段** 的工作
  - 所以 **先执行 job2** 完毕, 再执行 job1

- 每一个 job 下面的 2个 关键字

  | 关键字 | 作用 |
  | ------ | ---- |
  |    stage    |  定义当前这个 job ， **属于哪个阶段**    |
  | script| 定义当前这个 job ，**要执行的命令或脚本代码** |

### 2. before_script , after_script

#### 1. global

```yml
before_script:
  - global before script
```

#### 2. job

```yml
job:
  before_script:
    - execute this instead of global before script
  script:
    - my command
  after_script:
    - execute this after my script
```

### 3. stages

- 定义 pipeline 的 **全部阶段（stage）**
- 前一个执行成功，才会执行下一个 stage
- 任何阶段内任意 job 执行 **失败** 都会导致 **pipeline 失败**
- 如果 **未定义 stages** , 则 **默认** 存在 **build、test、deploy** 三个阶段
- 如果 **未定义 stage** , 则 **默认** 使用 **test** 阶段

```yml
stages:
  - build
  - test
  - deploy
```

### 4. stage

- 定义一个 job **属于哪个 stage (阶段)**
- 默认 **属于 test 阶段**

```yml
stages:
  - build
  - test
  - deploy

job 1:
  stage: build
  script: make build dependencies

job 2:
  stage: build
  script: make build artifacts

job 3:
  stage: test
  script: make test

job 4:
  stage: deploy
  script: make deploy
```

### 5. script

- job 内唯一一项 **必须** 的关键字设置

- 配置 gitlab runner 要执行的 **shell 命令**

- 可单行、也可多行

```yml
job1:
  script: "bundle exec rspec"

job2:
  script:
    - uname -a
    - bundle exec rspec
```

### 6. only , except

- only : 定义在 哪个分支、tag 上的修改 **触发** 执行
- except : 定义 哪个分支、tag 修改  **不触发** 任务的执行

#### 1. 只执行 `issue-` 开头的 refs，所有 branch 跳过

```yml
job:
  # use regexp
  only:
    - /^issue-.*$/
  # use special keyword
  except:
    - branches
```

#### 2. 执行 gitlab-org/gitlab-ce 上的所有分支，除了 master

```yml
job:
  only:
    - branches@gitlab-org/gitlab-ce
  except:
    - master@gitlab-org/gitlab-ce
```

### 7. tags

- tags 允许指定 **已经关联的 runner** 来执行任务

#### 1. 由定义为 ruby 和 postgres 的 runner 来执行任务

```yml
job:
  tags:
    - ruby
    - postgres
```

#### 2. 也可以为 不同的 job , 指定 不同的 runner

例如 windos 和 mac上 的任务， 由 不同的 runner 执行

```yml
windows job:
  stage:
    - build
  tags:
    - windows
  script:
    - echo Hello, %USERNAME%!

osx job:
  stage:
    - build
  tags:
    - mac
  script:
    - echo "Hello, $USER!"
```

### 8. allow_failure

- allow_failure 关键字, **允许该项 job 失败** , 后不影响其他任务的继续执行

```yml
job1:
  stage: test
  script:
    - execute_script_that_will_fail
  allow_failure: true
```

### 9. when

- when 用于当前一阶段任务故障或者忽略故障的 job 任务
- 主要用于清理发生故障时的后续措施
  1.  `on_success` - 默认参数，前一阶段任务成功时才执行
  2.  `on_failure` - 前一阶段任务故障时才执行
  3.  `always` - 无论前一阶段任务是否成功都执行
  4.  `manual` - 手动设置，比较复杂，略过

```yml
stages:
  - build
  - cleanup_build
  - test
  - deploy
  - cleanup

build_job:
  stage: build
  script:
    - make build

cleanup_build_job:
  stage: cleanup_build
  script:
    - cleanup build when failed
  when: on_failure

test_job:
  stage: test
  script:
    - make test

deploy_job:
  stage: deploy
  script:
    - make deploy
  when: manual

cleanup_job:
  stage: cleanup
  script:
    - cleanup after jobs
  when: always
```

- 上面的例子当 **build 任务失败** 时才会执行 **cleanup_build** 任务
- 无论之前的任务是否成功，**最后都会执行 clean_up** 的任务
- 当执行到 deploy 任务时，会触发手动设置

### 10. artifacts

- artifacts 指任 **务成功后** 可供 **下载的附件**，可在 pipeline-CI 中下载

- `artifacts:paths` - 该路径下的文件作为附件，只可指定仓库内的路径
- `artifacts:name` - 下载附件时的名称，可利用内置变量作为附件的名字
- `artifacts:when` - 指定当job成功或失败时才会上传附件
- `artifacts:expire_in` - 指定附件上载后保存的时间，默认永久在Gitlab保存
- `artifacts:untracked` - 指定上传Git未跟踪的文件

```yml
job:
  artifacts:
    name: "$CI_JOB_NAME-$CI_COMMIT_REF_NAME"
    paths:
      - binaries/
    when: on_failure
    expire_in: 1 week
    untracked: true
```

- 上面的例子会在 **任务失败** 时, 上传 `binaries/` 下未被跟踪的文件
- 附件以 job-ref 为名字
- 保留一周的时间

### 11. include

- 把 其他文件中的内容 **包含** 进来
- 类似于 **Makefile 中的 include**

#### 1.  ==local== git repo

```yml
include:
  - project: 'my-group/my-project'
    ref: master
    file: '/templates/.gitlab-ci-template.yml'

  - project: 'my-group/my-project'
    ref: v1.0.0
    file: '/templates/.gitlab-ci-template.yml'

  - project: 'my-group/my-project'
    ref: 787123b47f14b552955ca2786bc9542ae66fee5b
    file: '/templates/.gitlab-ci-template.yml'
```

#### 2.  ==remote== git repo


```yml
include:
  - remote: 'https://gitlab.com/awesome-project/raw/master/.gitlab-ci-template.yml'
```

### 12. variables

- 用于定义 全局 或 私有 变量
- 同时内置了一些内置变量

```yml
variables:
  DATABASE_URL: "postgres://postgres@postgres/my_database"
```

### 99. 参考

- https://www.jianshu.com/p/b69304279c5f
- https://www.jianshu.com/p/3c0cbb6c2936



## 2、以某个仓库的 gitlab ci yml 配置为例

### 1. 仓库顶层目录下的 `.gitlab-ci.yml`

```yml
stages:
  - prepare
  - lint
  - test
  - report

variables:
  SOME_REPORT_VAR: "xxxxxxx"

before_script:
  - export LANG=en_US.UTF-8
  - export LANGUAGE=en_US:en
  - export LC_ALL=en_US.UTF-8
  - export PATH=$PATH:/usr/local/bin/
  - source /Users/xxx/.bash_profile
  - source $(rvm env ruby-2.4.1@global --path)
  - rm -rf toolbox
  - git clone git@xxx.com:xxx/toolbox.git -b dev --depth=1 --single-branch
  - cd toolbox
  - bundle install
  - cd ../

after_script:
  - echo "Script stop working...."

include:
  - local: .gitlab/jobs/approve.gitlab-ci.yml
  - local: .gitlab/jobs/lint.gitlab-ci.yml
  - local: .gitlab/jobs/test.gitlab-ci.yml
```

### 2. approve 模块

#### 1. stage 定义: `.gitlab/jobs/approve.gitlab-ci.yml`

```yml
approvePrepare:
  stage: prepare
  only:
    - merge_requests
  script:
    - pwd
    - chmod a+x .gitlab/scripts/approve.sh
    - sh .gitlab/scripts/approve.sh
  tags:
    - Team-iOS
```

#### 2. 具体执行的脚本

没有

### 3. lint 模块

#### 1. stage 定义: `.gitlab/jobs/lint.gitlab-ci.yml`

```yml
lintPrepare:
  stage: prepare
  script:
    - pwd
    - chmod a+x .gitlab/scripts/lint.sh
    - sh .gitlab/scripts/lint.sh prepare
  only:
    - merge_requests
  tags:
    - Team-iOS

lintProject:
  stage: lint
  script:
    - pwd
    - chmod a+x .gitlab/scripts/lint.sh
    - sh .gitlab/scripts/lint.sh lint
  only:
    - merge_requests
  artifacts:
    paths: [./lint_result.txt]
    expire_in: 1 days
    when: always
  tags:
    - Team-iOS

lintReport:
  stage: report
  script:
    - pwd
    - chmod a+x .gitlab/scripts/lint.sh
    - sh .gitlab/scripts/lint.sh report
  tags:
    - Team-iOS
  only:
    - merge_requests
```

#### 2. 具体执行的脚本

```shell
#!/bin/bash

case $1 in
prepare)
  echo "lint_task -- prepare"
  cd toolbox
  bundle exec fastlane hello
  cd ..
  ;;
lint)
  echo "lint_task -- lint"
  echo "Gitlab Lint Stage Start Executed." >> ./lint_result.txt
  cd toolbox
  bundle exec fastlane hello
  cd ..
  ;;
report)
  echo "lint_task -- report"
  cd toolbox
  bundle exec fastlane hello
  cd ..
  ;;
esac
```

### 4.  unittest 模块

#### 1. stage 定义: `.gitlab/jobs/test.gitlab-ci.yml`

```yml
testPrepare:
  stage: prepare
  script:
    - pwd
    - chmod a+x .gitlab/scripts/test.sh
    - sh .gitlab/scripts/test.sh prepare
  only:
    - merge_requests
  tags:
    - Team-iOS

testProject:
  stage: test
  coverage: '/`\d+(?:\.\d*)?\%`$/'
  script:
    - pwd
    - chmod a+x .gitlab/scripts/test.sh
    - sh .gitlab/scripts/test.sh test
  artifacts:
    paths: [./test_result.txt]
    expire_in: 1 days
    when: always

  only:
    - merge_requests
  tags:
    - Team-iOS

testReport:
  stage: report
  script:
    - pwd
    - chmod a+x .gitlab/scripts/test.sh
    - sh .gitlab/scripts/test.sh report
  tags:
    - Team-iOS
  only:
    - merge_requests
```

#### 2. 具体执行的脚本

```shell
case $1 in
prepare)
  echo "test_task -- prepare"
  cd toolbox
  bundle exec fastlane hello
  cd ..
  ;;
test)
  echo "test_task -- test"
  echo "Test Shell executed." >> ./test_result.txt
  cd toolbox
  bundle exec fastlane hello
  cd ..
  ;;
report)
  echo "test_task -- report"
  cd toolbox
  bundle exec fastlane hello
  cd ..
  ;;
esac
```

### 5. 这个仓库的一个 MR pipeline 执行效果

![](Snip20191126_3.png)

