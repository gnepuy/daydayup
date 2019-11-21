[TOC]



## 1. gem xxx, path: '..'

### 1. Gemfile

```ruby
gem 'haha', path: '/Users/xiongzenghui/Desktop/haha'
```

### 2. 第一次 bundle install

查看生成的 Gemfile.lock

```
PATH
  remote: /Users/xiongzenghui/Desktop/haha
  specs:
    haha (0.1.0)

GEM
  remote: https://rubygems.org/
  specs:

PLATFORMS
  ruby

DEPENDENCIES
  haha!

BUNDLED WITH
   2.0.1
```

### 3. 修改 haha gem 之后, 第二次 bundle install

```
 ~/Desktop/test   master  bundle install
Using bundler 2.0.1
Using haha 0.1.0 from source at `/Users/xiongzenghui/Desktop/haha`
Bundle complete! 1 Gemfile dependency, 2 gems now installed.
Use `bundle info [gemname]` to see where a bundled gem is installed.
```

- Gemfile.lock **没有改变**
- 但起始 引用的 **haha 代码** 已经 **发生改变** 了



## 2. gem xxx, git: '..', branch: '..'

### 1. Gemfile

```ruby
gem 'haha', git: 'https://gitee.com/garywade/haha.git', branch: 'master'
```

### 2. 第一次 bundle install

```
 ~/Desktop/haha   master  bundle install
Using rake 10.5.0
Using bundler 2.0.1
Using diff-lcs 1.3
Using haha 0.1.0 from source at `.`
Using rspec-support 3.8.2
Using rspec-core 3.8.2
Using rspec-expectations 3.8.4
Using rspec-mocks 3.8.1
Using rspec 3.8.0
Bundle complete! 4 Gemfile dependencies, 9 gems now installed.
Use `bundle info [gemname]` to see where a bundled gem is installed.
```

- 发现并 **没有** 从设置的 **git repo** 仓库中，**重新** 下载 gem
- 而是读取的 **Gemfile.lock** 文件中记录的 haha 依赖方式, 直接进行本地安装

### 3. rm Gemfile.lock 之后, 再第二次 bundle install

rm Gemfile.lock

```
 ~/Desktop/test   master  rm Gemfile.lock
```

再第二次 bundle install

```
 ~/Desktop/test   master ●  bundle install
Fetching https://gitee.com/garywade/haha.git
Fetching gem metadata from https://rubygems.org/
Resolving dependencies...
Using bundler 2.0.1
Using haha 0.1.0 from https://gitee.com/garywade/haha.git (at master@404adcc)
Bundle complete! 1 Gemfile dependency, 2 gems now installed.
Use `bundle info [gemname]` to see where a bundled gem is installed.
```

- 1) **重新** 从 **https://gitee.com/garywade/haha.git** 下载 gem 
- 2) 根据 git 和 branch/最新 commit 确定下载的代码 **master@404adcc**

此时的 Gemfile.lock 已经发生 **改变** .

### 4. 后续不管怎么 bundle install, 都使用 ==Gemfile.lock== 记录的 ==master@404adcc== 下载 gem

```
 ~/Desktop/test   master ●  bundle install
Using bundler 2.0.1
Using haha 0.1.0 from https://gitee.com/garywade/haha.git (at master@404adcc)
Bundle complete! 1 Gemfile dependency, 2 gems now installed.
Use `bundle info [gemname]` to see where a bundled gem is installed.
```

```
 ~/Desktop/test   master ●  bundle install
Using bundler 2.0.1
Using haha 0.1.0 from https://gitee.com/garywade/haha.git (at master@404adcc)
Bundle complete! 1 Gemfile dependency, 2 gems now installed.
Use `bundle info [gemname]` to see where a bundled gem is installed.
```

- 无论 bundle install 多少次, 都是使用 **Gemfile.lock** 中记录的 **master@404adcc** 下载 gem
- 也就是说 haha 代码不再发生改变
- 只有 **删除 Gemfile.lock** 或者 **修改 Gemfile 中的 gem 版本** , 再执行 bundle install 才会下载 **git/branch 最新 commit** 的代码



## 3. gem xxx, git: '..', tag: '...'

### 1. Gemfile

```ruby
gem 'haha', git: 'https://gitee.com/garywade/haha.git', tag: 'v0.1.0'
```

### 2. 第一次 bundle install

```
 ~/Desktop/test   master ●  bundle install
Fetching https://gitee.com/garywade/haha.git
Fetching gem metadata from https://rubygems.org/
Resolving dependencies...
Using bundler 2.0.1
Using haha 0.1.0 from https://gitee.com/garywade/haha.git (at v0.1.0@99afda4)
Bundle complete! 1 Gemfile dependency, 2 gems now installed.
Use `bundle info [gemname]` to see where a bundled gem is installed.
```

- 发现会自动获取 **tag** 对应的 **最新** commit 代码
- 此时 **Gemfile.lock** 发生 **改变**

### 3. 后续不管怎么 bundle install, 包括 rm Gemfile.lock, 都直接使用 ==Gemfile.lock== 中记录的 ==tag==

```
 ~/Desktop/test   master  bundle install
Using bundler 2.0.1
Using haha 0.1.0 from https://gitee.com/garywade/haha.git (at v0.1.0@99afda4)
Bundle complete! 1 Gemfile dependency, 2 gems now installed.
Use `bundle info [gemname]` to see where a bundled gem is installed.
```

```
 ~/Desktop/test   master  bundle install
Using bundler 2.0.1
Using haha 0.1.0 from https://gitee.com/garywade/haha.git (at v0.1.0@99afda4)
Bundle complete! 1 Gemfile dependency, 2 gems now installed.
Use `bundle info [gemname]` to see where a bundled gem is installed.
```

```
 ~/Desktop/test   master  bundle install
Using bundler 2.0.1
Using haha 0.1.0 from https://gitee.com/garywade/haha.git (at v0.1.0@99afda4)
Bundle complete! 1 Gemfile dependency, 2 gems now installed.
Use `bundle info [gemname]` to see where a bundled gem is installed.
```



## 4. 总结 bundle install

- 1) 大部分情况下, **bundle install** 只会按照 **Gemfile.lock** 下载 **固定** 版本的 gem

- 2) 如果 **gem xxx, git:.., branch: 'dev'** 时, 会 **更新** gem 版本的情况
  - 1) 执行 bundle **update xxx** , 会 **更新 Gemfile.lock** 
  - 2) **rm** Gemfile.lock

- 3) 如果 **gem xxx, git:.., tag: '0.1.0'** 时, 会 **更新** gem 版本的情况
  - 1) 只有 **一种** 情况, 手动修改 **Gemfile** , 修改 **tag: 0.2.0** 引用 **新的 tag**



## 5. gem 依赖方式建议

| gem 依赖方式建议 | 适合使用的场景                       |
| ---------------- | ------------------------------------ |
| path             | **本地** 调试                        |
| git + **branch** | **远程** 调试, 并且代码 **频繁改动** |
| git + **tag**    | **稳定** (不需要频繁修改)            |



## 6. 一定要提交 Gemfile.lock 修改, 进行版本控制

```
git add Gemfile.lock
```

- 在其他的机器上, 执行 **bundle install** 情况下
- 都是按照 **Gemfile.lock** 中 **锁死** 的 **commit** 下载 gem



