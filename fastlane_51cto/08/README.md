[TOC]



## 1. Calling other actions (action1 调用其他 action2)

- Some **built-in** utility actions, such as **sh**, may be **used in custom actions** (e.g., in plugins). 

- It's **not** generally a good idea to call a **complex action** from **another action** In particular

- If you're **calling** one plugin action from **another plugin action**, you **should probably refactor** your **plugin helper** to be more easily called from all actions in the plugin.

- **Avoid** wrapping **complex** built-in actions like **deliver** and **gym**.

- There **can be issues** with one plugin **depending** on **another plugin**.

- Certain **simple** built-in utility actions may be used with other_action in your action, 
  - such as: **other_action.git_add**,  **other_action.git_commit**.
  - Think **twice** before **calling** **an action** from another action. 

- There is often a **better solution**.


## 2. 总结来说

- 1) fastlane **不建议** 你的 action/plugin **依赖其他的** action/plugin

- 2) action/plugin **只可以** 依赖 **公共的 helper** 工具类代码, 而 **不能** 是 **action/plugin**

- 3) 如果出现 **依赖 action/plugin** 的关系，那么就应该考虑 **重构** 你的 action/plugin
