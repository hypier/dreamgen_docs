# 迁移
## 从旧故事 UI 迁移到新故事 UI

本指南将向您展示如何将故事从“旧 UI”迁移到“新 UI”。您可以在[此专门指南](/stories)中了解有关新故事创作 UI 的更多信息。

假设您有一个旧故事，如下所示：

```
<setting>
Love story between a werewolf and a vampire.

Characters:

Name: Bella
Age: 19
Bella is a vampire who is in love with Cedric.

Name: Cedric
Age: 21
Cedric is a werewolf who is in love with Bella.
</setting>

<instruction>
Bella and Cedric meet in a coffee shop.
</instruction>

It was a rainy day when Bella and Cedric met in a coffee shop...

```

您可以像这样将其迁移到新 UI：

## 1\. 创建新故事

前往新的 [Stories 页面](https://dreamgen.com/app/stories/v2)，然后点击顶部的“从空白开始”按钮。

## 2\. 设置故事

一旦您创建了一个新故事，您将进入故事场景编辑器。在这里，您可以复制您的情节和角色。
根据上述示例，它应该看起来像这样：

![Migrated Story](https://dreamgen.com/_next/static/media/migrated-story.d53f59e0.webp)

现在只需点击“保存”，然后点击“开始写作”。

## 3\. 复制故事内容

现在您可以将旧故事中的故事内容复制到新故事中。前往您的旧故事，然后在右下角点击“复制文本”。然后将内容粘贴到新故事中。

## 4\. 继续写作

现在你的旧故事已经迁移完成，你可以在新的用户界面中继续写作。确保阅读 [新的故事写作指南](/stories) 以了解所有新功能。

以下是亮点：

- 更好的模型：从 "opus-v1-sm"、"opus-v1-lg" 或 "opus-v1-xl" 中选择一个模型。新的模型是更有能力的写作者，能够遵循更复杂的指令。
- 更好的用户界面：不再需要调整简单的文本 <setting> 和 <instruction> 块，你将获得一个侧边栏，在这里你可以编辑情节和角色（以及地点、物品等），还有一个按钮可以添加指令。
- 禁用指令的能力：许多用户不希望模型生成指令——你现在也可以在侧边栏的模型选项卡中禁用这一功能。只需在底部选择“不要生成指令”。