[TOC]



## 1. json 读取为 object

```ruby
require 'pp'
require 'json'

h = JSON.parse(File.read('./haha.json'))
pp h.class
pp h

obj = JSON.parse(File.read('./haha.json'), object_class: OpenStruct)
pp obj.class
pp obj

pp obj.name
pp obj.age
pp obj.sex
pp obj.favors
```

```
╰─± ruby main.rb
Hash
{"name"=>"xiong", "age"=>99, "sex"=>true, "favors"=>["111", "222"]}
OpenStruct
#<OpenStruct name="xiong", age=99, sex=true, favors=["111", "222"]>
"xiong"
99
true
["111", "222"]
```


