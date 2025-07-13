# 《我是如何使用Claude code的》

数周以来，我完全沉浸在 Claude Code 中

过去我是将代码编辑框放最大，cursor的AI agent用于辅助编程。

而现在，我会把claude code对话终端放到最大，只有在我需要审查代码时，才会打开编辑器去查看。

以下是关于如何使用 Claude Code 以及我的一些使用技巧分享：



## 一、使用 VS Code 插件

首先，安装 Claude Code 插件。它适用于 VS Code、Cursor，可能也适用于 Windsurf。

当然，它并没有什么特别惊艳的作用。它本质上只是一个启动器。

唯一我觉得好的地方是，它可以让你可以在 IDE 的不同窗格中同时运行多个会话，只要它们处理的是代码库的不同部分。




我仍然会用 Cursor 的 Command+K 补全和 Tab 补全。但是AI侧边栏？我只在 Claude 出现问题时才会碰它。



## 二、终端界面的小技巧

我们可以通过@ 标记文件，使用斜杠命令，并精确选择要包含的上下文：




Tips： 经常使用 /clear 命令。

不需要的历史记录会占用你的 tokens，也不要用 /compact 压缩总结旧对话。直接清空它，然后继续。

方向箭 ⬆️ 可以让你浏览上一句的提问。当你需要重新发类似的命令时，会很方便。



## 三、权限系统的小设置


这是 Claude Code 最烦人的地方：它对每件事都要请求权限。

有一个解决方案：



```bash
claude --dangerously-skip-permissions
```
通过这条命令启动claude，会自动同意所有的权限请求！

你可以把它想象成 Cursor 以前的 YOLO 模式。



## 四、GitHub 集成


一个比较酷的斜杠命令是 /install-github-app。运行后，Claude 会自动审查你的 PR。

随着你 AI 写的代码增多，你的 PR 数量会增加。

Claude 可以帮你发现遗漏的 Bug。人喜欢挑剔变量名，但是 Claude 能发现实际的逻辑错误和安全问题。




这个工具的最初问题是它非常冗长，默认的prompt会评论各种细微、不重要的东西，并针对每个 PR 写一大篇论文。

我们真正最关心的是 Bug 和潜在的漏洞。所以我们明确告诉它这些，并且要求它简洁。

Claude 会添加一个 claude-code-review.yml 文件。我用的是下面这个：

```yaml
# claude-code-review.yml
direct_prompt:|  
Please review this pull request and look for bugs and security 
issues. Only report on bugs and potential vulnerabilities you 
find. Be concise.
```


## 五、操作小技巧


由于它是终端界面，所以有一些不方便的地方：

Shift+Enter 默认情况下不能换行。 告诉 Claude 使用 /terminal-setup 设置你的终端，然后就可以换行了。

拖动文件 通常会像 Cursor 或 VS Code 中一样在新标签页中打开它们。在拖动时按住 Shift 键，以便在 Claude 中正确引用它们。

从剪贴板粘贴图片 不能使用 Command+V。请使用 Control+V。

停止 Claude 不是 Control+C。使用 Esc键才可以中断 Claude。

跳到之前的消息： 按两次 Esc 键会显示所有之前消息的列表，你可以跳回到其中任何一条。

如果你喜欢 Vim 模式，它也有。我个人觉得不好用。



## 六、queuing system非常方便


我离不开的一个功能：消息队列。你可以在执行任务的任何时候，输入新的任务，Claude 会智能地处理它们。

以前的AI coding工具，大多数是需要你排好任务一个个输入。Claude支持你一次性布置好所有任务，并且不断地添加新的任务。




Claude 非常聪明，知道什么时候应该真正运行这些内容。



## 七、hooks系统极大地增加可玩性
通过设置hook，我们可以在以下阶段执行我们的自定义脚本：

PostToolUse：在工具成功完成后立即运行。

Notification：当Claude Code发送通知时运行。

Stop：当主Claude Code代理完成响应时运行。

SubagentStop：当Claude Code子代理（Task工具调用）完成响应时运行。

在我公众号之前的文章中，我们就通过设置notification和stop事件，实现了Claude工作状态实时通知的功能。



## 八、创建自定义斜杠命令


你还可以非常轻松地添加自定义斜杠命令。

要添加命令，只需创建一个 .claude/commands 文件夹，将命令名称作为文件，扩展名为 .md。

你只需用自然语言编写这些内容，并且可以使用 $ARGUMENTS 字符串将参数放入提示中。

例如，如果我想输出一个测试，我可以创建 .claude/commands/test.md：

```markdown
# .claude/hooks/test.md

Please create comprehensive tests for: $ARGUMENTS

Test requirements:

- Use Jest and React Testing Library
- Place tests in __tests__ directory
- Mock Firebase/Firestore dependencies
- Test all major functionality
- Include edge cases and error scenarios
- Test MobX observable state changes
- Verify computed values update correctly
- Test user interactions
- Ensure proper cleanup in afterEach
- Aim for high code coverage
```
然后，运行 /test MyButton cc会将 MyButton 作为参数，完全按照你test.md中定义的那些要求，进行操作。



你甚至可以在commands下创建子文件夹，来分类组织你的斜杠命令：

比如，你有一个名为 builder 的文件夹，里面放了一个 plugin.md 文件。

那么你就可以通过 /builder/plugin 来调用这个命令。




## 九、记忆系统

Claude 可以快速记住你告诉它的事。你只要在 # 号后面输入你让它记得事情就可以了。

比如"以后做新功能都用 MUI 组件"，它就会自动把这句话存到最相关的文件里。

CLAUDE.md 文件可以是多份的，可以在项目级别拥有一个，也可以在子目录中拥有一个。

它会查看所有这些文件，并在相关时优先考虑最具体、嵌套最深的文件。