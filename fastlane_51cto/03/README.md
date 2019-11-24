[TOC]



## 1. fastlane Interacting with the user 所有方式

### 1. 打印 信息

| method         | description                                             |
| -------------- | ------------------------------------------------------- |
| error          | Displays an error message in red                        |
| important      | Displays a warning or other important message in yellow |
| success        | Displays a success message in green                     |
| message        | Displays a message in the default output color          |
| deprecated     | Displays a deprecation message in bold blue             |
| command        | Displays a command being executed in cyan               |
| command_output | Displays command output in magenta                      |
| verbose        | Displays verbose output in the default output color     |
| header         | Displays a message in a box for emphasis                |

### 2. 终止 执行

| method              | description                                                  |
| ------------------- | ------------------------------------------------------------ |
| crash!              | Report a catastrophic error                                  |
| user_error!         | Rescue an exception in your action and report a nice message to the user |
| shell_error!        | Report failure of a shell command                            |
| build_failure!      | Report a build failure                                       |
| test_failure!       | Report a test failure                                        |
| abort_with_message! | Report a failure condition that prevents continuing          |

### 3. 获取 用户输入

| method       | description                                      |
| ------------ | ------------------------------------------------ |
| interactive? | Indicates whether interactive input is possible  |
| input        | Prompt the user for string input                 |
| confirm      | Ask the user a binary question                   |
| select       | Prompt the user to select from a list of options |
| password     | Prompt the user for a password (masks output)    |




## 2. UI 提供的 message/success/error/important 颜色输出

### 1. Fastfile

```ruby
lane :lan1 do
  UI.message "Neutral message (usually white)"
  UI.success "Successfully finished processing (usually green)"
  UI.error "Wahaha, what's going on here! (usually red)"
  UI.important "Make sure to use Windows (usually yellow)"
end
```

### 2. fastlane lane 输出

![](03.png)



## 3. UI 提供的 input/confirm/password 输入框

### 1. Fastfile

```ruby
lane :lan2 do
  UI.header "Inputs" # a big box

  name = UI.input("What's your name? ")
  if UI.confirm("Are you '#{name}'?")
    UI.success "Oh yeah"
  else
    UI.error "Wups, invalid"
  end

  UI.password("Your password please: ") # password inputs are hidden
end
```

### 2. fastlane lane 输出

![](04.gif)



## 4. UI 提供的 select 列表选择项

### 1. Fastfile

```ruby
lane :lan3 do
  #### A "Dropdown" for the user
  project = UI.select(
    "Select your project: ", 
    ["Test Project", "Test Workspace"]
  )

  UI.success("selected '#{project}'")
end
```

### 2. fastlane lane 输出

![](05.gif)



## 5. UI 提供的 Helper

### 1. Fastfile

```ruby
lane :lan5 do
  #### or if you just want to receive a simple value use this only if the command doesn't take long
  diff = Helper.backticks("git diff")
  UI.success("diff = #{diff}")
end
```

### 2. fastlane lane 输出

```
 ~/collect_xxx/toolbox   master ●  bundle exec fastlane lan5
[✔] 🚀
[00:49:37]: It seems like you wanted to load some plugins, however they couldn't be loaded
[00:49:37]: Please follow the troubleshooting guide: https://docs.fastlane.tools/plugins/plugins-troubleshooting/
[00:49:37]: Driving the lane 'lan5' 🚀
[00:49:37]: $ git diff
[00:49:37]: ▸ diff --git a/Gemfile b/Gemfile
[00:49:37]: ▸ index 24bb7d8..2d9eb1c 100644
[00:49:37]: ▸ --- a/Gemfile
[00:49:37]: ▸ +++ b/Gemfile
.................
.................
```



## 6. UI 提供的 user_error!

### 1. Fastfile

```ruby
lane :lan6 do
  #### fastlane "crash" because of a user error everything that is caused by the user and is not unexpected
  UI.user_error!("You don't have a project in the current directory")
end
```

### 2. fastlane lane 输出

![](06.png)



## 7. UI 提供的 crash!

### 1. Fastfile

```ruby
lane :lan7 do
  #### an actual crash when something unexpected happened
  UI.crash!("Network timeout")
end
```

### 2. fastlane lane 输出

![](07.gif)



## 8. UI 提供的 deprecated

### 1. Fastfile

```ruby
lane :lan8 do
  #### a deprecation message
  UI.deprecated("The '--key' parameter is deprecated")
end
```

### 2. fastlane lane 输出

![](08.png)

