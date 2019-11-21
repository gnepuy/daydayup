[TOC]



## 1. gem 'haha', git: '..', tag: '..'

### 1. git repo 没有更新 tag 时, 执行 bundle update xxx

第一次 bundle update haha

```
 ~/Desktop/test   master  bundle update haha
Fetching https://gitee.com/garywade/haha.git
Fetching gem metadata from https://rubygems.org/
Resolving dependencies...
Using bundler 2.0.1
Using haha 0.1.0 from https://gitee.com/garywade/haha.git (at v0.1.0@99afda4)
Bundler attempted to update haha but its version stayed the same
Bundle updated!
```

第二次 bundle update haha

```
 ~/Desktop/test   master  bundle update haha
Fetching https://gitee.com/garywade/haha.git
Fetching gem metadata from https://rubygems.org/
Resolving dependencies...
Using bundler 2.0.1
Using haha 0.1.0 from https://gitee.com/garywade/haha.git (at v0.1.0@99afda4)
Bundler attempted to update haha but its version stayed the same
Bundle updated!
```

结论：

- Gemfile.lock **没有** 发生 **改变**
- 也就是说 **没有** 下载 最新的 commit 代码

### 2. git repo 添加新的 tag 时, 执行 bundle update xxx

第一次 bundle update haha

```
 ~/Desktop/test   master  bundle update haha
Fetching https://gitee.com/garywade/haha.git
Fetching gem metadata from https://rubygems.org/
Resolving dependencies...
Using bundler 2.0.1
Using haha 0.1.0 from https://gitee.com/garywade/haha.git (at v0.1.0@99afda4)
Bundler attempted to update haha but its version stayed the same
Bundle updated!
```

第二次 bundle update haha：

```
 ~/Desktop/test   master  bundle update haha
Fetching https://gitee.com/garywade/haha.git
Fetching gem metadata from https://rubygems.org/
Resolving dependencies...
Using bundler 2.0.1
Using haha 0.1.0 from https://gitee.com/garywade/haha.git (at v0.1.0@99afda4)
Bundler attempted to update haha but its version stayed the same
Bundle updated!
```

结论：

- Gemfile.lock 仍然 **没有** 发生 **改变**
- 仍然 **没有** 下载 最新的 commit 代码

### 3. 结论：==稳定==

- 使用 **tag** 的方式，来添加 gem 依赖，比较 **稳定**

- 只要不 **手动** 更新 gem 依赖的 tag , 不管什么情况下, 都 **不会更新** gem 版本
  - 1) gem git repo 添加了 **新的 commit**
  - 2) gem git repo 添加了 **新的 tag**
  - 3) **rm Gemfile.lock**



## 2. gem 'haha', git: '..', branch: '..'

### 1. 修改 Gemfile 使用 ==branch== 方式依赖

```ruby
gem 'haha', git: 'https://gitee.com/garywade/haha.git', branch: 'master'`
```

### 2. 第一次 bundle update haha

```
 ~/Desktop/test   master  bundle update haha
Fetching https://gitee.com/garywade/haha.git
Fetching gem metadata from https://rubygems.org/
Resolving dependencies...
Using bundler 2.0.1
Using haha 0.1.0 from https://gitee.com/garywade/haha.git (at master@9420ee5)
Bundler attempted to update haha but its version stayed the same
Bundle updated!
```

- 1) 已经 **重新** 从 **https://gitee.com/garywade/haha.git** 下载 **最新的 commit** 代码了
- 2) **Gemfile.lock** 已经 **发生改变**
- 3) 使用 **commit** 指向 gem git repo 最新的代码

### 3. haha 没有提交情况下, 第二次 bundle update haha

```
 ~/Desktop/test   master  bundle update haha
Fetching https://gitee.com/garywade/haha.git
Fetching gem metadata from https://rubygems.org/
Resolving dependencies...
Using bundler 2.0.1
Using haha 0.1.0 from https://gitee.com/garywade/haha.git (at master@9420ee5)
Bundler attempted to update haha but its version stayed the same
Bundle updated!
```

- 1) 仍然会从 **https://gitee.com/garywade/haha.git** 下载 **最新的 commit** 代码
- 2) 但是 git repo **没有** 提交代码
- 3) 所以还是使用 **Gemfile.lock** 中 **记录的 commit**

### 4. haha 有新提交情况下, 第三次 bundle update haha

**haha** git repo 产生的最新提交 : **553e063886fbaafd4346f88cb2d98c11d4b9af2c**

```
 ~/Desktop/test   master  bundle update haha
Fetching https://gitee.com/garywade/haha.git
Fetching gem metadata from https://rubygems.org/
Resolving dependencies...
Using bundler 2.0.1
Using haha 0.1.0 from https://gitee.com/garywade/haha.git (at master@553e063)
Bundler attempted to update haha but its version stayed the same
Bundle updated!
```

- 1) 继续经从 **https://gitee.com/garywade/haha.git** 拉取了 **master@553e063** 最新代码了
- 2) Gemfile.lock **发生改变**

### 5. 结论: 灵活, 但欠稳定

- 1) 只要 **不主动** 执行 **bundle update xxx** 情况下, 不管怎么执行 **bundle install** , 都只会根据 **Gemfile.lock** 记录的 **commit** 下载代码

- 2) 但是一旦当 **gem git repo** 中对应的 **branch** 有新的 commit 提交之后, 以下情况会自动拉取 **最新** 的代码
  - 1) 直接 **bundle update xxx** 
  - 2) 先 **rm Gemfile.lock** , 然后再 **bundle install** 



## 3. 直接 bundle update

就是相当于对 **所有** 依赖的 **gem** 都执行一遍上面的逻辑.

