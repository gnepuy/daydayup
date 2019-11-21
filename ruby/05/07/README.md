[TOC]



## 1. bundle 环境默认 ==继承给子进程==

- App1/Gemfile 中指定依赖 fastlane 版本为 2.135.1
- App2/Gemfile 中指定依赖 fastlane 版本为 2.136.0
- App1/main.rb 调用 App2/main.rb
- 那么 App2/main.rb 最终使用的 fastlane 确是 **2.135.1** , 而不是自己在 Gemfile 指定的 2.136.0



## 2. bundler Shelling Out 脱离 父进程 bundle 环境

https://bundler.io/man/bundle-exec.1.html#Shelling-out



## 3. Bundler.with_clean_env

- Any Ruby code that opens a **subshell** (like system, backticks, or %x{}) will **automatically** use the **current Bundler** environment

- If you need to **shell out** to a Ruby command that is **not part of** your current bundle

- use the **with_clean_env method** with a block

- Any subshells created inside the block will be given the environment present before Bundler was activated.

- For example, Homebrew commands run Ruby, but don't work inside a bundle:

```ruby
Bundler.with_clean_env do
  `brew install wget`
end
```



## 4. run in different bundle

- Using **with_clean_env** is also necessary if you are **shelling out** to a **different bundle**.

- Any Bundler commands run in a **subshell** will **inherit** the **current Gemfile**

- so commands that need to run in the context of a **different bundle** also need to use **with_clean_env**.

```ruby
Bundler.with_clean_env do
  Dir.chdir "/other/bundler/project" do
    `bundle exec ./script` # 在其他目录下的 bundle 环境下, 执行 bundle exec ...
  end
end
```



## 5. Bundler.clean_system

```ruby
Bundler.clean_system('brew install wget')
```



## 6. Bundler.clean_exec

```ruby
Bundler.clean_exec('brew install wget')
```



