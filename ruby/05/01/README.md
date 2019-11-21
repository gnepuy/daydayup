[TOC]



## 1. 通过 gem install 安装的 ruby 软件包

```
╰─○ ruby -v
ruby 2.6.0p0 (2018-12-25 revision 66547) [x86_64-darwin18]
```

```
╰─○ gem install colored
Successfully installed colored-1.2
Parsing documentation for colored-1.2
Done installing documentation for colored after 0 seconds
1 gem installed
```

查看 `/Users/xiongzenghui/.rvm/gems/ruby-2.6.0/gems/colored`

```
╰─○ tree /Users/xiongzenghui/.rvm/gems/ruby-2.6.0/gems/colored-1.2
/Users/xiongzenghui/.rvm/gems/ruby-2.6.0/gems/colored-1.2
├── LICENSE
├── README
├── Rakefile
├── lib
│   └── colored.rb
└── test
    └── colored_test.rb

2 directories, 5 files
```

是一个非常简单的 ruby 软件包。

最核心的就是 **lib 目录** , 这个目录下存放的是一个 ruby 软件包最重要的执行逻辑。



## 2. colored 源码

https://github.com/defunkt/colored

```
git clone https://github.com/defunkt/colored.git
cd colored
```

```
╰─± tree
.
├── LICENSE
├── README
├── Rakefile
├── colored.gemspec
├── lib
│   └── colored.rb
└── test
    └── colored_test.rb

2 directories, 6 files
```

可以看到与通过 **gem install colored** 安装到 gems 目录中的包结构是一样的。



## 3. 通过 bundler 创建一个我们自己的 ruby 软件包 (一个 gem)

```
╰─○ bundler gem Wocaonima
Creating gem 'Wocaonima'...
MIT License enabled in config
Code of conduct enabled in config
      create  Wocaonima/Gemfile
      create  Wocaonima/lib/Wocaonima.rb
      create  Wocaonima/lib/Wocaonima/version.rb
      create  Wocaonima/Wocaonima.gemspec
      create  Wocaonima/Rakefile
      create  Wocaonima/README.md
      create  Wocaonima/bin/console
      create  Wocaonima/bin/setup
      create  Wocaonima/.gitignore
      create  Wocaonima/.travis.yml
      create  Wocaonima/.rspec
      create  Wocaonima/spec/spec_helper.rb
      create  Wocaonima/spec/Wocaonima_spec.rb
      create  Wocaonima/LICENSE.txt
      create  Wocaonima/CODE_OF_CONDUCT.md
Initializing git repo in /Users/xiongzenghui/Desktop/Wocaonima
Gem 'Wocaonima' was successfully created. For more information on making a RubyGem visit https://bundler.io/guides/creating_gem.html
```

查看创建出来的 Wocaonima 这个 ruby 软件包的结构

```
╭─xiongzenghui at xiongzenghui的MacBook Pro in ~/Desktop using ‹ruby-2.6.0›
╰─○ cd Wocaonima
╭─xiongzenghui at xiongzenghui的MacBook Pro in ~/Desktop/Wocaonima on master✘✘✘ using ‹ruby-2.6.0›
╰─± tree
.
├── CODE_OF_CONDUCT.md
├── Gemfile
├── LICENSE.txt
├── README.md
├── Rakefile
├── Wocaonima.gemspec
├── bin
│   ├── console
│   └── setup
├── lib
│   ├── Wocaonima
│   │   └── version.rb
│   └── Wocaonima.rb
└── spec
    ├── Wocaonima_spec.rb
    └── spec_helper.rb

4 directories, 12 files
```

- 基本上结构与上面的 colored 差不多
- 只不过多一些文件、以及 bin 目录、spec 目录
- **结论**：一个 ruby 软件包，就是由这一些文件的打包

