[TOC]

## 1. 只能在 ==Fastfile== 中使用 ==fastlane_require()==

- 1) If you're using a **third party gem** in your **Fastfile**

- 2) it is **recommended** to use **fastlane_require** instead of **require**

- 3) **fastlane_require** will: (好处)
  - 1) **Verify** the **gem** is **installed**
  - 2) **Show** installation instructions **if not installed**
  - 3) **Require** the gem (**like require** does)

比如如果需要在你的 Fastfile 中使用 **hike** 这个 gem , 就这样写:

```ruby
fastlane_require 'hike'

lane :release do
  # Do stuff with hike
end
```



## 2. 但是 ==action/plugin== 中, 仍然使用 ==require()==

```c
module Fastlane
  module Actions
    module SharedValues
      GITLAB_MR_ADD_NOTE_CUSTOM_VALUE = :GITLAB_MR_ADD_NOTE_CUSTOM_VALUE
    end

    # fastlane_require 'gitlab' #=> 错误
    require 'gitlab' #=> 正确

    class GitlabMrAddNoteAction < Action
      ......
    end
  end
end
```