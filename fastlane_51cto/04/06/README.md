[TOC]



## 1. output = Actions.sh("shell command")

### 1. fastlane/actions/my_action.rb

```ruby
module Fastlane
  module Actions
    module SharedValues
      MY_ACTION_CUSTOM_VALUE = :MY_ACTION_CUSTOM_VALUE
    end

    class MyActionAction < Action
      def self.run(params)
        output = Actions.sh("ls -l")
        UI.message("output = #{output}")
      end

      ####################################
      # @!group Documentation
      ####################################

      def self.description
        "A short description with <= 80 characters of what this action does"
      end

      def self.details
        # Optional:
        # this is your chance to provide a more detailed description of this action
        "You can use this action to do cool things..."
      end

      def self.available_options
        [
          FastlaneCore::ConfigItem.new(key: :api_token,
                                       env_name: "FL_MY_ACTION_API_TOKEN", # The name of the environment variable
                                       description: "API Token for MyActionAction", # a short description of this parameter
                                       verify_block: proc do |value|
                                          UI.user_error!("No API token for MyActionAction given, pass using `api_token: 'token'`") unless (value and not value.empty?)
                                          # UI.user_error!("Couldn't find file at path '#{value}'") unless File.exist?(value)
                                       end),
          FastlaneCore::ConfigItem.new(key: :development,
                                       env_name: "FL_MY_ACTION_DEVELOPMENT",
                                       description: "Create a development certificate instead of a distribution one",
                                       is_string: false, # true: verifies the input is a string, false: every kind of value
                                       default_value: false) # the default value if the user didn't provide one
        ]
      end

      def self.output
        [
          ['MY_ACTION_CUSTOM_VALUE', 'A description of what this value contains']
        ]
      end

      def self.return_value
        # If your method provides a return value, you can describe here what it does
      end

      def self.authors
        ["Your GitHub/Twitter Name"]
      end

      def self.is_supported?(platform)
        platform == :ios
      end
    end
  end
end
```

### 2. fastlane run action

![](10.png)



## 2. Actions.sh error_callback

```ruby
Actions.sh("ls xxx", error_callback: lambda { |result|
  UI.error "Wahaha, what's going on here! (usually red)"
})
```



## 3. UI 提供的 FastlaneCore::CommandExecutor.execute

### 1. Fastfile

```ruby
lane :lan4 do
  FastlaneCore::CommandExecutor.execute( command: "ls",
    print_all: true,
    error: proc do |error_output|
      # handle error here
    end)
end
```

### 2. fastlane lane 输出

```
 ~/collect_xxx/toolbox   master ●  bundle exec fastlane lan4
[✔] 🚀
[00:47:35]: It seems like you wanted to load some plugins, however they couldn't be loaded
[00:47:35]: Please follow the troubleshooting guide: https://docs.fastlane.tools/plugins/plugins-troubleshooting/
[00:47:35]: Driving the lane 'lan4' 🚀
[00:47:35]: $ ls
[00:47:35]: ▸ Appfile		Pluginfile	actions		module_job	report.xml
[00:47:35]: ▸ Fastfile	README.md	dayly_job	mr_job
[00:47:35]: fastlane.tools finished successfully 🎉
```



## 4. 其他的几种调用 shell command 方式

### 1. system()

```ruby
system "cat fastlane/Fastfile"
UI.user_error! "Could not execute command" unless $?.exitstatus == 0
```

### 2. Using backticks

```ruby
pod_cmd = `which pod`
UI.important "'pod' command not found" if pod_cmd.empty?
```

### 3. Using the sh()

```ruby
sh "pwd"
```

```ruby
sh "ls", "-la" do |status, result, command|
  unless status.success?
    UI.error "Command #{command} (pid #{status.pid}) failed with status #{status.exitstatus}"
  end
  UI.message "Output is #{result.inspect}"
end
```

```ruby
success = true

sh("pwd", error_callback: ->(result) { success = false })
UI.user_error "Command failed" unless success
```

### 4. Escaping in shell commands

```ruby
`git commit -aqm #{Shellwords.escape commit_message}`
```

```ruby
sh "git", "commit", "-aqm", commit_message
```
