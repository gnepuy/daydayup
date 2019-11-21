[TOC]



## 1. fastlane action 文档: 明确说明有 2种 类型的 action

![https://docs.fastlane.tools/create-action/](01.png)

- 1) **Local** actions

- 2) **remote** actions in **fastlane main repo**




## 2. 但是 fastlane main repo 已经大致不接受 submit action

![](02.png)



## 3. 我们自己的 action 必须放在 ==fastlane/actions/== 目录下

这是我之前写的一个 fastlane app demo :

```
 ~/collect_xxx/toolbox   master ●  tree fastlane
fastlane
├── Appfile
├── Fastfile
├── Pluginfile
├── README.md
├── actions
│   ├── git_archive_install.rb
│   ├── jenkins_mr_build.rb
│   └── my_action.rb
├── dayly_job
│   └── dayly_Fastfile.rb
├── module_job
│   └── module_Fastfile.rb
└── mr_job
    └── mr_Fastfile.rb

4 directories, 10 files
```

- **actions 目录** 下存在三个 action
  - git_archive_install
  - jenkins_mr_build
  - my_action



## 4. 如果 ==重用我们自己 action== , 就必须将 action 文件 ==拷贝== 到其他的地方

- 但如果有 **其他的 fastlane app** 也要使用这几个 action

- 那么只能将 **这个工程中的 三个 action 文件** 原模原样 **拷贝** 另外的 fastlane app `actions/ 目录` 下

显然是比较麻烦的，而 fastlane **plugin** 正是用来解决这个问题。
