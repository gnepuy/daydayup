[TOC]



## 1. 找到个简单的 ==unzip== plugin

### 1. github 地址

https://github.com/maxoly/fastlane-plugin-unzip

### 2. fastlane-plugin-unzip 目录结构

```
 ~/Desktop/fastlane-plugin-unzip   master  tree
.
├── Gemfile
├── LICENSE
├── README.md
├── Rakefile
├── circle.yml
├── fastlane
│   ├── Fastfile
│   └── Pluginfile
├── fastlane-plugin-unzip.gemspec
├── lib
│   └── fastlane
│       └── plugin
│           ├── unzip
│           │   ├── actions
│           │   │   └── unzip_action.rb
│           │   ├── helper
│           │   │   └── unzip_helper.rb
│           │   └── version.rb
│           └── unzip.rb
└── spec
    ├── example_action.rb.zip
    ├── spec_helper.rb
    └── unzip_action_spec.rb

8 directories, 15 files
```

### 3. Gemfile

```ruby
source 'https://rubygems.org'

gemspec

# 这句比较重要, 导入 fastlane plugin 文件
plugins_path = File.join(File.dirname(__FILE__), 'fastlane', 'Pluginfile')
eval(File.read(plugins_path), binding) if File.exist?(plugins_path)
```

除了下面这2句，几乎和一个 **ruby gem 开发目录** 没什么不同.

### 4. fastlane-plugin-unzip.gemspec

```ruby
# coding: utf-8
lib = File.expand_path("../lib", __FILE__)
$LOAD_PATH.unshift(lib) unless $LOAD_PATH.include?(lib)
require 'fastlane/plugin/unzip/version'

Gem::Specification.new do |spec|
  spec.name          = 'fastlane-plugin-unzip'
  spec.version       = Fastlane::Unzip::VERSION
  spec.author        = %q{Massimo Oliviero}
  spec.email         = %q{massimo.oliviero@gmail.com}

  spec.summary       = %q{Extract compressed files in a ZIP}
  spec.homepage      = "https://github.com/maxoly/fastlane-plugin-unzip"
  spec.license       = "MIT"

  spec.files         = Dir["lib/**/*"] + %w(README.md LICENSE)
  spec.test_files    = spec.files.grep(%r{^(test|spec|features)/})
  spec.require_paths = ['lib']

  # spec.add_dependency 'your-dependency', '~> 1.0.0'

  spec.add_development_dependency 'pry'
  spec.add_development_dependency 'bundler'
  spec.add_development_dependency 'rspec'
  spec.add_development_dependency 'rake'
  spec.add_development_dependency 'rubocop'
  spec.add_development_dependency 'fastlane', '>= 1.96.0'
end
```

同样与一个 **ruby gem 开发目录** 没什么不同.

### 5. plugin 核心内容: fastlane-plugin-unzip/lib/fastlane/plugin 目录

```
 ~/Desktop/fastlane-plugin-unzip/lib/fastlane/plugin   master  tree
.
├── unzip
│   ├── actions
│   │   └── unzip_action.rb
│   ├── helper
│   │   └── unzip_helper.rb
│   └── version.rb
└── unzip.rb

3 directories, 4 files
```

### 6. fastlane-plugin-unzip/lib/fastlane/plugin/unzip.rb

```ruby
require 'fastlane/plugin/unzip/version'

module Fastlane
  module Unzip
    # Return all .rb files inside the "actions" and "helper" directory
    def self.all_classes
      Dir[File.expand_path('**/{actions,helper}/*.rb', File.dirname(__FILE__))]
    end
  end
end

# By default we want to import all available actions and helpers
# A plugin can contain any number of actions and plugins
Fastlane::Unzip.all_classes.each do |current|
  require current
end
```

如上这个 rb 文件作用，将 `fastlane-plugin-unzip/lib/fastlane/plugin/` 目录下的 **两个子目录中** 所有的 rb 文件 **require 加载到内存**

- 1) fastlane-plugin-unzip/lib/fastlane/plugin/**actions**/*.rb
- 2) fastlane-plugin-unzip/lib/fastlane/plugin/**helper**/*.rb

### 7. fastlane-plugin-unzip/lib/fastlane/plugin/actions/unzip_action.rb

```ruby
module Fastlane
  module Actions
    class UnzipAction < Action
      def self.run(params)
        require 'shellwords'

        begin
          escaped_file = params[:file].shellescape
          UI.important "🎁  Unzipping file #{escaped_file}..."

          # Base command
          command = "unzip -o #{escaped_file}"

          # Destination
          if params[:destination_path]
            escaped_destination = params[:destination_path].shellescape
            command << " -d #{escaped_destination}"
          end

          # Password
          if params[:password]
            escaped_password = params[:password].shellescape
            command << " -P #{escaped_password}"
          end

          Fastlane::Actions.sh(command, log: false)
          UI.success "Unzip finished ✅"
        rescue => ex
          UI.user_error!("Error unzipping file: #{ex}")
        end
      end

      #####################################################
      # @!group Documentation
      #####################################################

      def self.description
        "Extract compressed files in a ZIP"
      end

      def self.details
        [
          "unzip will extract files from a ZIP archive.",
          "The default behavior is to extract into the current directory all files from the specified ZIP archive."
        ].join("\n")
      end

      def self.available_options
        [
          FastlaneCore::ConfigItem.new(key: :file,
                                       env_name: "FL_UNZIP_FILE",
                                       description: "The path of the ZIP archive",
                                       verify_block: proc do |value|
                                         UI.user_error!("Couldn't find file at path '#{File.expand_path(value)}'") unless File.exist?(value)
                                       end),
          FastlaneCore::ConfigItem.new(key: :destination_path,
                                       env_name: "FL_UNZIP_DESTINATION_PATH",
                                       description: "An optional directory to which to extract files",
                                       optional: true),
          FastlaneCore::ConfigItem.new(key: :password,
                                       env_name: "FL_UNZIP_PASSWORD",
                                       description: "The password to decrypt encrypted zipfile",
                                       optional: true)
        ]
      end

      def self.return_value
      end

      def self.authors
        ["maxoly"]
      end

      def self.is_supported?(platform)
        true
      end
    end
  end
end
```

- 这个文件与前面文章中实现一个 **fastlane action** 基本一样的.

- 也就是说 fastlane 只是把 一个单独的 **action.rb** 文件, 打包成为了一个 **plugin(ruby gem)**

### 8. fastlane-plugin-unzip/lib/fastlane/plugin/helper/unzip_helper.rb

```ruby
module Fastlane
  module Helper
    class UnzipHelper
      # class methods that you define here become available in your action
      # as `Helper::UnzipHelper.your_method`
      #
      def self.show_message
        UI.message("Hello from the unzip plugin helper!")
      end
    end
  end
end
```

`fastlane-plugin-unzip/lib/fastlane/plugin/helper/` 目录下，专门存放各种为 action 服务的 **工具类 XxxHelper**.



## 2. plugin 对比下一个 rubygem 应用 开发目录

### 1. bundle gem haha 创建一个名字为 haha 的 gem 应用

```
 ~/Desktop  bundle gem haha
Creating gem 'haha'...
Do you want to generate tests with your gem?
Type 'rspec' or 'minitest' to generate those test files now and in the future. rspec/minitest/(none): rspec
Do you want to license your code permissively under the MIT license?
This means that any other developer or company will be legally allowed to use your code for free as long as they admit you created it. You can read more about the MIT license at https://choosealicense.com/licenses/mit. y/(n): y
MIT License enabled in config
Do you want to include a code of conduct in gems you generate?
Codes of conduct can increase contributions to your project by contributors who prefer collaborative, safe spaces. You can read more about the code of conduct at contributor-covenant.org. Having a code of conduct means agreeing to the responsibility of enforcing it, so be sure that you are prepared to do that. Be sure that your email address is specified as a contact in the generated code of conduct so that people know who to contact in case of a violation. For suggestions about how to enforce codes of conduct, see https://bit.ly/coc-enforcement. y/(n): y
Code of conduct enabled in config
      create  haha/Gemfile
      create  haha/lib/haha.rb
      create  haha/lib/haha/version.rb
      create  haha/haha.gemspec
      create  haha/Rakefile
      create  haha/README.md
      create  haha/bin/console
      create  haha/bin/setup
      create  haha/.gitignore
      create  haha/.travis.yml
      create  haha/.rspec
      create  haha/spec/spec_helper.rb
      create  haha/spec/haha_spec.rb
      create  haha/LICENSE.txt
      create  haha/CODE_OF_CONDUCT.md
Initializing git repo in /Users/xiongzenghui/Desktop/haha
Gem 'haha' was successfully created. For more information on making a RubyGem visit https://bundler.io/guides/creating_gem.html
```

### 2. gem 应用的 目录结构

```
 ~/Desktop  cd haha
 ~/Desktop/haha   master ✚  tree
.
├── CODE_OF_CONDUCT.md
├── Gemfile
├── LICENSE.txt
├── README.md
├── Rakefile
├── bin
│   ├── console
│   └── setup
├── haha.gemspec
├── lib
│   ├── haha
│   │   └── version.rb
│   └── haha.rb
└── spec
    ├── haha_spec.rb
    └── spec_helper.rb

4 directories, 12 files
```

- 1) Gemfile

- 2) haha.gemspec

- 3) lib/
  - 1) haha.rb
  - 2) haha/*.rb

### 3. 总结 plugin

- 1) 开发一个 **plugin** 就是开发一个 **ruby gem 应用**

- 2) 如果你已经有一个 **action** , 仅仅只需要套一个 **ruby gem** 应用的 **壳子** 就可以变成 **plugin**

