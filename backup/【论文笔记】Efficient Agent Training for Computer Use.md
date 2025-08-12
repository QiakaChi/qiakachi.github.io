官方仓库：[https://github.com/GAIR-NLP/PC-Agent-E](https://github.com/GAIR-NLP/PC-Agent-E)

<details class="lake-collapse"><summary id="u7e8ced35"><span class="ne-text" style="font-size: 16px">【AI整理】论文速读</span></summary><h3 id="AiFDh"><span class="ne-text" style="color: rgb(88, 90, 115); background-color: rgba(255, 255, 255, 0)">方法描述</span></h3><p id="ue1664bdc" class="ne-p"><span class="ne-text" style="color: rgb(88, 90, 115); background-color: rgba(255, 255, 255, 0)">该论文提出了一种名为PC Agent-E的高效训练框架，用于提高计算机使用能力，并具有显著的数据效率。该方法通过</span><strong><span class="ne-text" style="color: rgb(88, 90, 115); background-color: rgba(255, 255, 255, 0)">结合真实的人机交互和多样化的动作决策来生成高质量的行为轨迹数据</span></strong><span class="ne-text" style="color: rgb(88, 90, 115); background-color: rgba(255, 255, 255, 0)">，从而在现实性和多样性方面都具有优势。 具体来说，该方法首先收集了少量的任务轨迹数据，记录屏幕状态观察和人类行为，然后过滤掉错误的步骤和轨迹。接着，利用这些数据重建人类思考过程之前的状态，基于相应的屏幕状态观察和历史步长上下文。然后，将人类轨迹作为环境快照，使用Claude模型合成多样化替代动作决策，最后开发出PC Agent-E模型，该模型是在增强后的轨迹上进行简单端到端训练的最先进的本地代理模型。</span></p><h3 id="oTj7K"><span class="ne-text" style="color: rgb(88, 90, 115); background-color: rgba(255, 255, 255, 0)">方法改进</span></h3><p id="u115c18ca" class="ne-p"><span class="ne-text" style="color: rgb(88, 90, 115); background-color: rgba(255, 255, 255, 0)">该方法改进了之前的任务收集方式，通过</span><strong><span class="ne-text" style="color: rgb(88, 90, 115); background-color: rgba(255, 255, 255, 0)">手动组成小种子集并使用LLMs扩大范围</span></strong><span class="ne-text" style="color: rgb(88, 90, 115); background-color: rgba(255, 255, 255, 0)">来生成任务，使得任务更加丰富。同时，该方法还引入了规则基础的过滤器来去除整个轨迹或单个步骤中的错误或其他不良行为，确保数据质量。此外，该方法还使用了一个迭代的过程来</span><strong><span class="ne-text" style="color: rgb(88, 90, 115); background-color: rgba(255, 255, 255, 0)">重构隐含的思考过程</span></strong><span class="ne-text" style="color: rgb(88, 90, 115); background-color: rgba(255, 255, 255, 0)">，以及一个简单的但有效的方法——Trajectory Boost，以增加轨迹的多样性。</span></p><h3 id="skYw6"><span class="ne-text" style="color: rgb(88, 90, 115); background-color: rgba(255, 255, 255, 0)">解决的问题</span></h3><p id="u72ee1835" class="ne-p"><span class="ne-text" style="color: rgb(88, 90, 115); background-color: rgba(255, 255, 255, 0)">该方法解决了计算机使用代理模型的训练问题，通过生成高质量的行为轨迹数据来提高计算机使用能力，并且具有显著的数据效率。此外，该方法还改善了任务收集方式、数据过滤和轨迹多样性等方面，提高了评估可靠性。</span></p></details>

![](https://cdn.nlark.com/yuque/0/2025/png/39039688/1754475412900-08c2efbf-fa8b-49ce-8cca-d4ec00172c50.png)

# 1. 训练阶段：轨迹数据收集与合成

## 1.1. Trajectory Collection（PC Tracker） 

✅支持 Windows 操作系统


官方仓库：[https://github.com/GAIR-NLP/PC-Agent](https://github.com/GAIR-NLP/PC-Agent)

用户手册：[https://github.com/GAIR-NLP/PC-Agent/blob/main/tracker/README_zh.md](https://github.com/GAIR-NLP/PC-Agent/blob/main/tracker/README_zh.md)

PC Tracker 采集人类与 PC 的交互轨迹。

> <font style="color:rgb(31, 35, 40);">PC Tracker 是一个轻量级工具，用于高效收集大规模真实人机交互轨迹。类似于屏幕录制，PC Tracker在后台无缝运行，自动捕获屏幕截图和键鼠操作。收集到的人机交互轨迹示例如下：</font>
>
> ![](https://github.com/GAIR-NLP/PC-Agent/raw/main/assets/raw_trajectory_example.png)
>

借助PC Tracker，获取了 312 条人类轨迹数据。

## 1.2. Thought Completion

上一步中我们得到了人类的操作轨迹，在这一步我们将把原始的人类轨迹数据转化为有思考的轨迹（trajectory with thoughts）。

PC Agent-E 为 Claude 3.7 Sonnet 提供：任务描述，历史动作（with constructed thought processes）当前动作和相关截图，并结合对应 prompt（见论文附录 B.1 部分），得到关于“人类为什么会进行这一步操作”的推理。

<details class="lake-collapse"><summary id="u82a5d5c3"><span class="ne-text" style="font-size: 16px">【翻译】论文附录 B.1 部分 Thought Completion Prompt</span></summary><h3 id="LXWHF"><span class="ne-text">表格4：思维补全提示</span></h3><h4 id="MSbXi"><span class="ne-text">思维补全提示</span></h4><p id="ucfcc1228" class="ne-p"><span class="ne-text" style="font-size: 16px">你是一个用于在计算机上完成任务的有帮助的算机使用智能体。你的目标是重现你在执行特定操作背后的思维过程。</span></p><p id="u005130d6" class="ne-p"><span class="ne-text" style="font-size: 16px">你将获得以下信息：</span></p><ol class="ne-ol"><li id="u8865b617" data-lake-index-type="0"><span class="ne-text" style="font-size: 16px">你试图完成的任务。</span></li><li id="u3789e20c" data-lake-index-type="0"><span class="ne-text" style="font-size: 16px">你已执行步骤的历史记录（如有，最多 50 条；如果是第一个操作，则无）。</span></li><li id="u943d11b8" data-lake-index-type="0"><span class="ne-text" style="font-size: 16px">你选择执行的特定操作。</span></li><li id="u044d3b0a" data-lake-index-type="0"><span class="ne-text" style="font-size: 16px">你点击的元素名称（如果你点击了某个元素）。如果名称过于笼统模糊，你需要根据截图判断点击什么。</span></li><li id="ufe1eaeef" data-lake-index-type="0"><span class="ne-text" style="font-size: 16px">你决定执行操作时计算机屏幕的截图。</span></li><li id="uebf8dc31" data-lake-index-type="0"><span class="ne-text" style="font-size: 16px">截图上的红色标记指示点击或拖动操作的位置。</span></li></ol><p id="ubbafcff5" class="ne-p"><span class="ne-text" style="background-color: #FBE4E7; font-size: 16px">要梳理你的思维过程，请考虑：</span></p><ol class="ne-ol"><li id="u51b92fcf" data-lake-index-type="0"><span class="ne-text" style="background-color: #FBE4E7; font-size: 16px">你在屏幕上观察到了什么？分析当前截图时，要结合你的任务和之前的操作。</span></li><li id="u68ad207c" data-lake-index-type="0"><span class="ne-text" style="background-color: #FBE4E7; font-size: 16px">（如适用）评估你之前的操作：</span></li></ol><ul class="ne-list-wrap"><ul ne-level="1" class="ne-ul"><li id="u83d6fb77" data-lake-index-type="0"><span class="ne-text" style="background-color: #FBE4E7; font-size: 16px">它是否达到了预期效果？如果没有，找出可能的原因（例如，误点击、元素未激活）。<br /></span><span class="ne-text" style="background-color: #FBE4E7; font-size: 16px"> 无效操作的一些典型示例：</span></li><li id="u75ae0679" data-lake-index-type="0"><span class="ne-text" style="background-color: #FBE4E7; font-size: 16px">在空白处误点击</span></li><li id="u9aa8bcfc" data-lake-index-type="0"><span class="ne-text" style="background-color: #FBE4E7; font-size: 16px">点击一些无效元素（未双击）</span></li><li id="ud1bf86c8" data-lake-index-type="0"><span class="ne-text" style="background-color: #FBE4E7; font-size: 16px">因输入框未激活而无效的打字/按键操作</span></li><li id="ud5984d58" data-lake-index-type="0"><span class="ne-text" style="background-color: #FBE4E7; font-size: 16px">结果是否与你之前的计划一致，还是发生了意外情况？<br /></span><span class="ne-text" style="background-color: #FBE4E7; font-size: 16px"> 无效操作的一些典型示例：</span></li><li id="u2878f1ba" data-lake-index-type="0"><span class="ne-text" style="background-color: #FBE4E7; font-size: 16px">误点击错误元素</span></li><li id="ucef94c93" data-lake-index-type="0"><span class="ne-text" style="background-color: #FBE4E7; font-size: 16px">忘记清除输入栏中已有的文本</span></li></ul></ul><ol start="3" class="ne-ol"><li id="ud598328b" data-lake-index-type="0"><span class="ne-text" style="background-color: #FBE4E7; font-size: 16px">根据你的操作历史，评估你在完成整体任务方面的进度。</span></li><li id="u1531327f" data-lake-index-type="0"><span class="ne-text" style="background-color: #FBE4E7; font-size: 16px">考虑是否因为之前步骤中的尝试失败，你正在摸索如何完成任务。</span></li></ol><p id="u87c91f95" class="ne-p"><span class="ne-text" style="background-color: #FBE4E7; font-size: 16px">将你的思维过程以清晰、自然的第一人称叙述呈现，解释你当时的推理。</span></p><p id="ud9032634" class="ne-p"><span class="ne-text" style="font-size: 16px">重要要求：</span></p><ol class="ne-ol"><li id="uf1e28182" data-lake-index-type="0"><strong><span class="ne-text" style="font-size: 16px">切勿</span></strong><span class="ne-text" style="font-size: 16px">在回复中提及红色标记。这些标记是</span><strong><span class="ne-text" style="font-size: 16px">事后</span></strong><span class="ne-text" style="font-size: 16px">添加的，用于指示你的点击或拖动操作的位置，在你做决定时，它们并不在屏幕上。</span><strong><span class="ne-text" style="font-size: 16px">切勿</span></strong><span class="ne-text" style="font-size: 16px">在回复中提及 “红框”“红圈”“红箭头” 等表述。</span></li><li id="ue51028ba" data-lake-index-type="0"><span class="ne-text" style="font-size: 16px">写作时，要仿佛你在执行操作前实时思考一样。不要包含操作后的评估或事后的感悟。</span></li></ol><p id="u6db3b634" class="ne-p"><span class="ne-text" style="font-size: 16px">你试图完成的任务：{task_description} <br /></span><span class="ne-text" style="font-size: 16px">你的操作历史：{history_str} <br /></span><span class="ne-text" style="font-size: 16px">你选择执行的特定操作：{action}</span></p></details>

## 1.3. Trajectory Boost

![](https://cdn.nlark.com/yuque/0/2025/png/39039688/1754485277774-17d66777-23a5-40bf-9dd1-4ed6e9ef80dc.png)

进行 Boost 的理论依据是同一个任务可能会有多种轨迹（eg. 我在 Win11 打开QQ邮箱，既可以单击QQ图标→单击邮箱图标，也可以打开浏览器→在地址栏输入`https://wx.mail.qq.com/`）。Trajectory Boost 收集这些没有被人类执行的可选方案。

PC Agent-E 为 Claude 3.7 Sonnet 提供：现有的任务轨迹、屏幕截图和历史上下文，并结合对应 prompt（见论文附录 B.2 部分）。

经过 Boost，数据从 312 条增长到 27,000 条左右。

<details class="lake-collapse"><summary id="u85b7ade4"><span class="ne-text" style="font-size: 16px">【翻译】论文附录 B.2 部分 Trajectory Boost Prompt</span></summary><h3 id="uZ0vQ"><span class="ne-text">表格5：轨迹增强提示</span></h3><h4 id="w6Q22"><span class="ne-text">轨迹增强提示</span></h4><p id="ufd3ac01e" class="ne-p"><span class="ne-text" style="font-size: 16px">你是一个 有帮助的助手，可协助用户完成计算机任务，且</span><strong><span class="ne-text" style="font-size: 16px">完全有权限</span></strong><span class="ne-text" style="font-size: 16px">对用户的计算机进行任何操作。操作系统为 Windows 。</span><span class="ne-text" style="background-color: #D9EAFC; font-size: 16px">基于所提供的当前状态，你需要提出完成任务的下一步操作。</span><span class="ne-text" style="font-size: 16px">不要试图一步完成整个任务，要将其拆分为更小的步骤，每一步操作后会得到新状态，以便继续交互。</span></p><p id="uc5304a26" class="ne-p"><strong><span class="ne-text" style="font-size: 16px">重要规则</span></strong><span class="ne-text" style="font-size: 16px">：你必须严格遵守以下规则：</span></p><ol class="ne-ol"><li id="uedef705c" data-lake-index-type="0"><span class="ne-text" style="font-size: 16px">每次回复仅从以下列表中选择</span><strong><span class="ne-text" style="font-size: 16px">一个</span></strong><span class="ne-text" style="font-size: 16px">操作，每一步不要执行多个操作。 </span></li><li id="u89a548cf" data-lake-index-type="0"><span class="ne-text" style="font-size: 16px">严格遵循所选操作的语法格式，不要创建或使用列表外的任何操作。 </span></li><li id="u1e26c57a" data-lake-index-type="0"><span class="ne-text" style="font-size: 16px">任务完成后，输出 “action finish” 。</span></li></ol><h3 id="wr47e"><span class="ne-text">有效操作</span></h3><ol class="ne-ol"><li id="u13ff492e" data-lake-index-type="0"><strong><span class="ne-text" style="font-size: 16px">click (x, y)</span></strong><span class="ne-text" style="font-size: 16px">：点击屏幕上坐标为 (x, y) 位置的元素 </span></li><li id="ue95e9b7b" data-lake-index-type="0"><strong><span class="ne-text" style="font-size: 16px">right click (x, y)</span></strong><span class="ne-text" style="font-size: 16px">：右键点击屏幕上坐标为 (x, y) 位置的元素 </span></li><li id="u5d7dc6b4" data-lake-index-type="0"><strong><span class="ne-text" style="font-size: 16px">double click (x, y)</span></strong><span class="ne-text" style="font-size: 16px">：双击屏幕上坐标为 (x, y) 位置的元素 </span></li><li id="ue1a88210" data-lake-index-type="0"><strong><span class="ne-text" style="font-size: 16px">drag from (x1, y1) to (x2, y2)</span></strong><span class="ne-text" style="font-size: 16px">：将元素从位置 (x1, y1) 拖动到 (x2, y2) </span></li><li id="u90841079" data-lake-index-type="0"><strong><span class="ne-text" style="font-size: 16px">scroll (x)</span></strong><span class="ne-text" style="font-size: 16px">：垂直滚动屏幕，偏移像素为 x 。x 为正值时向上滚动，为负值时向下滚动 </span></li><li id="ufa981b69" data-lake-index-type="0"><strong><span class="ne-text" style="font-size: 16px">press key: key_content</span></strong><span class="ne-text" style="font-size: 16px">：按下键盘上 key_content 对应的按键 </span></li><li id="u9c556b64" data-lake-index-type="0"><strong><span class="ne-text" style="font-size: 16px">hotkey (key1, key2)</span></strong><span class="ne-text" style="font-size: 16px">：按下由 key1 和 key2 组成的快捷键 </span></li><li id="ue594ee4e" data-lake-index-type="0"><strong><span class="ne-text" style="font-size: 16px">hotkey (key1, key2, key3)</span></strong><span class="ne-text" style="font-size: 16px">：按下由 key1、key2 和 key3 组成的快捷键 </span></li><li id="u8acf0c55" data-lake-index-type="0"><strong><span class="ne-text" style="font-size: 16px">type text: text_content</span></strong><span class="ne-text" style="font-size: 16px">：在键盘输入 text_content 内容。注意，输入文字前，需确保文本框或输入字段已激活 / 获得焦点。若文本框未激活，应先点击激活它，然后在单独步骤中使用 “type text” 操作 </span></li><li id="u72d51931" data-lake-index-type="0"><strong><span class="ne-text" style="font-size: 16px">wait</span></strong><span class="ne-text" style="font-size: 16px">：等待一段时间，通常用于等待系统响应、屏幕刷新、广告结束 </span></li><li id="u97399e5b" data-lake-index-type="0"><strong><span class="ne-text" style="font-size: 16px">finish</span></strong><span class="ne-text" style="font-size: 16px">：表明任务已完成 </span></li><li id="u00eb7a0b" data-lake-index-type="0"><strong><span class="ne-text" style="font-size: 16px">fail</span></strong><span class="ne-text" style="font-size: 16px">：表明任务失败，即因提供信息不足，任务无法执行</span></li></ol><p id="u8bbc14e4" class="ne-p"><span class="ne-text" style="font-size: 16px">在决定下一步操作前，需仔细考量屏幕当前状态和操作历史。思维过程应涵盖以下要点：</span></p><ol class="ne-ol"><li id="u03abf1eb" data-lake-index-type="0"><span class="ne-text" style="font-size: 16px">你在屏幕上观察到了什么？分析当前截图时，结合任务和之前的操作。 </span></li><li id="u43ee4610" data-lake-index-type="0"><span class="ne-text" style="font-size: 16px">（如适用）之前的计划和操作是怎样的？分三种情况评估之前的操作：</span></li></ol><ol class="ne-list-wrap"><ol ne-level="1" class="ne-ol"><li id="u69e45906" data-lake-index-type="0"><span class="ne-text" style="font-size: 16px">操作无任何效果。应找出可能原因（如误点击、元素未激活），并在本步骤调整计划。 <br /></span><span class="ne-text" style="font-size: 16px"> 无效操作的典型示例：</span></li></ol></ol><ul class="ne-list-wrap"><ul ne-level="1" class="ne-ul"><li id="ue5461e06" data-lake-index-type="0"><span class="ne-text" style="font-size: 16px">在空白处误点击 </span></li><li id="u201d53b7" data-lake-index-type="0"><span class="ne-text" style="font-size: 16px">未双击打开部分元素，操作无效 </span></li><li id="u2e0476e6" data-lake-index-type="0"><span class="ne-text" style="font-size: 16px">因输入框未激活，打字 / 按键操作无效</span></li></ul></ul><ol class="ne-list-wrap"><ol start="2" ne-level="1" class="ne-ol"><li id="u14fed9f0" data-lake-index-type="0"><span class="ne-text" style="font-size: 16px">操作有效果，但结果与之前计划不一致。应找出可能原因（如误点击、元素未激活），并在本步骤纠正。 <br /></span><span class="ne-text" style="font-size: 16px"> 无效操作的典型示例：</span></li></ol></ol><ul class="ne-list-wrap"><ul ne-level="1" class="ne-ul"><li id="u57a3ff9b" data-lake-index-type="0"><span class="ne-text" style="font-size: 16px">误点击错误元素 </span></li><li id="ua9bbc4c2" data-lake-index-type="0"><span class="ne-text" style="font-size: 16px">忘记清除输入栏中已有的文本</span></li></ul></ul><ol class="ne-list-wrap"><ol start="2" ne-level="1" class="ne-ol"><li id="u189bae3e" data-lake-index-type="0"><span class="ne-text" style="font-size: 16px">操作有效果，且与之前计划一致。应基于当前状态推进到下一步。</span></li></ol></ol><ol start="3" class="ne-ol"><li id="u8981af18" data-lake-index-type="0"><span class="ne-text" style="font-size: 16px">根据操作历史，评估完成整体任务的进度。 </span></li><li id="ud4cc1e86" data-lake-index-type="0"><span class="ne-text" style="font-size: 16px">若之前步骤中有失败尝试，探索完成任务的新方法。</span><strong><span class="ne-text" style="font-size: 16px">切勿重复</span></strong><span class="ne-text" style="font-size: 16px">历史操作。</span></li></ol><h3 id="qi1jq"><span class="ne-text">回复格式</span></h3><p id="u5b4a6d66" class="ne-p"><span class="ne-text" style="font-size: 16px">Your thought process（你的思维过程）</span></p><p id="u7b9ccc33" class="ne-p"><span class="ne-text" style="font-size: 16px">Action: The specific action you choose to take.（操作：你选择执行的具体操作） </span></p><p id="u64f45e78" class="ne-p"><span class="ne-text" style="font-size: 16px">你试图完成的任务：</span><span class="ne-text" style="color: #DF2A3F; font-size: 16px">{task_description} </span><span class="ne-text" style="font-size: 16px"><br /></span><span class="ne-text" style="font-size: 16px">你的操作历史：</span><span class="ne-text" style="color: #DF2A3F; font-size: 16px">{history_str} </span><span class="ne-text" style="font-size: 16px"><br /></span><span class="ne-text" style="font-size: 16px">现有如下</span><span class="ne-text" style="color: #DF2A3F; font-size: 16px">截图</span><span class="ne-text" style="font-size: 16px">。为协助完成任务，你接下来会执行哪一步操作？</span></p></details>

## 1.4. Agent Training

PC Agent-E 为 Claude 3.7 Sonnet 提供：任务描述，历史和截图，并结合对应 prompt（见论文附录 B.3 部分）。输出为 <thought, action>。

![](https://cdn.nlark.com/yuque/0/2025/png/39039688/1754316638463-25e602bd-4bf8-402e-bd53-1b54a257b84e.png)

由仓库中 train/sft.yaml 内容可知，Agent Training 的方式为：

+  SFT（有监督微调），即从预训练的开源模型（Qwen2.5-VL-72B-Instruct）之后继续训练  ；
+   全参数微调（而非LoRA等）；
+ 数据集：Boost 后的轨迹数据；
+ 使用了 LLaMA-Factory 框架进行 Agent 训练（README.md）。

<details class="lake-collapse"><summary id="u3ab48f59"><span class="ne-text" style="font-size: 16px">【翻译】论文附录 B.3 部分 </span><span class="ne-text" style="color: rgb(0,0,0); font-size: 16px">PC Agent-E scaffold Promp</span></summary><h3 id="Dl6v2"><span class="ne-text">表格6：PC Agent - E 脚手架提示</span></h3><h4 id="XrZAX"><span class="ne-text">PC Agent - E 脚手架提示</span></h4><p id="u3c4b7df3" class="ne-p"><span class="ne-text">你是一个有帮助的助手，能够协助用户完成计算机任务，且</span><strong><span class="ne-text">完全有权限</span></strong><span class="ne-text">对用户的计算机进行任何操作。基于所提供的当前状态，</span><span class="ne-text" style="background-color: #FBF5CB">你需要提出完成任务的下一步操作</span><span class="ne-text">。不要试图一步完成整个任务，要把它拆分成更小的步骤，每一步操作后你会获得一个新状态以继续交互。</span></p><p id="u079741c8" class="ne-p"><strong><span class="ne-text">重要提示</span></strong><span class="ne-text">：你必须严格遵守以下规则：</span></p><ol class="ne-ol"><li id="u9aa511e3" data-lake-index-type="0"><span class="ne-text">每次回复仅从以下列表中选择</span><strong><span class="ne-text">一个</span></strong><span class="ne-text">操作，每一步不要执行多个操作。</span></li><li id="u572af8ff" data-lake-index-type="0"><span class="ne-text">严格遵循所选操作的语法格式，不要创建或使用列表之外的任何操作。</span></li><li id="u807b2e52" data-lake-index-type="0"><span class="ne-text">一旦任务完成，输出 “finish” 。</span></li></ol><h3 id="n5ilL"><span class="ne-text">有效操作</span></h3><ol class="ne-ol"><li id="ub25e3442" data-lake-index-type="0"><strong><span class="ne-text">click (x, y)</span></strong><span class="ne-text">：点击屏幕上坐标为 (x, y) 位置的元素 </span></li><li id="u6a8f54ed" data-lake-index-type="0"><strong><span class="ne-text">right click (x, y)</span></strong><span class="ne-text">：右键点击屏幕上坐标为 (x, y) 位置的元素 </span></li><li id="udd435aa3" data-lake-index-type="0"><strong><span class="ne-text">double click (x, y)</span></strong><span class="ne-text">：双击屏幕上坐标为 (x, y) 位置的元素 </span></li><li id="u86a32297" data-lake-index-type="0"><strong><span class="ne-text">drag from (x1, y1) to (x2, y2)</span></strong><span class="ne-text">：将元素从位置 (x1, y1) 拖动到 (x2, y2) </span></li><li id="u50ec1ce7" data-lake-index-type="0"><strong><span class="ne-text">scroll (x)</span></strong><span class="ne-text">：垂直滚动屏幕，偏移像素为 x 。x 为正值时向上滚动，为负值时向下滚动 </span></li><li id="ub20fb38f" data-lake-index-type="0"><strong><span class="ne-text">press key: key_content</span></strong><span class="ne-text">：按下键盘上 key_content 对应的按键 </span></li><li id="u707dfc67" data-lake-index-type="0"><strong><span class="ne-text">hotkey (key1, key2)</span></strong><span class="ne-text">：按下由 key1 和 key2 组成的快捷键 </span></li><li id="u9c07f036" data-lake-index-type="0"><strong><span class="ne-text">hotkey (key1, key2, key3)</span></strong><span class="ne-text">：按下由 key1、key2 和 key3 组成的快捷键 </span></li><li id="ubcbb3fee" data-lake-index-type="0"><strong><span class="ne-text">type text: text_content</span></strong><span class="ne-text">：在键盘输入 text_content 内容 </span></li><li id="u60cc4fdb" data-lake-index-type="0"><strong><span class="ne-text">wait</span></strong><span class="ne-text">：等待一段时间，通常用于等待系统响应、屏幕刷新、广告结束 </span></li><li id="ud93821b0" data-lake-index-type="0"><strong><span class="ne-text">finish</span></strong><span class="ne-text">：表明任务已完成 </span></li><li id="u53657c0e" data-lake-index-type="0"><strong><span class="ne-text">fail</span></strong><span class="ne-text">：表明任务失败，即因提供信息不足，任务无法执行</span></li></ol><h3 id="dh7fF"><span class="ne-text">回复格式</span></h3><p id="u9cdca1d4" class="ne-p"><span class="ne-text" style="color: #DF2A3F">{Your thought process}</span><span class="ne-text">（你的思维过程）</span></p><p id="uaa523f9e" class="ne-p"><span class="ne-text">Action: {The specific action you choose to take}（操作：你选择执行的具体操作） </span></p><p id="u47f94126" class="ne-p"><span class="ne-text">你的任务是：</span><span class="ne-text" style="color: #DF2A3F">{task_description} </span><span class="ne-text"><br /></span><span class="ne-text">为达到当前屏幕状态，你之前的操作和思路历史：</span><span class="ne-text" style="color: #DF2A3F">{history_str} </span><span class="ne-text"><br /></span><span class="ne-text">结合</span><span class="ne-text" style="color: #DF2A3F">截图</span><span class="ne-text">，你接下来会执行哪一步操作来协助完成任务？</span></p></details>
<details class="lake-collapse"><summary id="u083d6018"><span class="ne-text" style="font-size: 16px">【笔记】LLaMA-Factory 框架</span></summary><p id="u93bc5f24" class="ne-p"><span class="ne-text" style="font-size: 16px">官方仓库：</span><a href="https://github.com/hiyouga/LLaMA-Factory" data-href="https://github.com/hiyouga/LLaMA-Factory" target="_blank" class="ne-link"><span class="ne-text" style="font-size: 16px">https://github.com/hiyouga/LLaMA-Factory</span></a></p><p id="u4efb6905" class="ne-p"><span class="ne-text" style="font-size: 16px">LLaMA-Factory 支持多种预训练模型和微调算法，提供了一套完整的工具和接口，使得用户能够轻松地对预训练的模型进行定制化的训练和调整，以适应特定的应用场景。</span></p><p id="u7fe859e3" class="ne-p"><span class="ne-text" style="font-size: 16px"></span></p><p id="u5f7b4f5a" class="ne-p"><span class="ne-text" style="font-size: 16px">类似的还有peft库。</span></p></details>

# 2. 测试阶段：任务集的选取与评估

<details class="lake-collapse"><summary id="ue6dfd558"><span class="ne-text" style="font-size: 16px">【笔记】Benchmark 和 Baseline</span></summary><p id="u946861c6" class="ne-p"><span class="ne-text" style="font-size: 16px">Benchmark 是于用于评测方法表现的任务/平台，是评测环境，相当于“一个考场、一张试卷”；Baseline 是于一组用于比较的已有方法/模型，是对比对象，相当于“其他考生”。 Benchmark 的目的是为了</span><strong><span class="ne-text" style="font-size: 16px">公平评估模型性能</span></strong><span class="ne-text" style="font-size: 16px">，而 Baseline 的目的是为了</span><strong><span class="ne-text" style="font-size: 16px">作为参照，验证新方法的改进效果</span></strong><span class="ne-text" style="font-size: 16px">。  </span></p><div id="xtww4" class="ne-bookmark"><a href="https://zhuanlan.zhihu.com/p/31427279391" target="_blank">https://zhuanlan.zhihu.com/p/31427279391</a></div></details>

## 2.1. Benchmark
论文中使用的 Benchmark 为 WindowsAgentArena V2 和 <font style="color:rgb(54, 54, 54);">OSWorld。</font>

### 2.1.1. WindowsAgentArena
官方仓库：[https://github.com/microsoft/WindowsAgentArena](https://github.com/microsoft/WindowsAgentArena)

> <font style="color:rgb(31, 35, 40);">Windows Agent Arena (WAA) 是一个可扩展的 Windows AI Agent 平台，用于测试和基准化多模态、桌面 AI Agent。WAA 为研究人员和开发人员提供了一个可重复和逼真的 Windows 操作系统环境，用于 AI 研究，其中可以在各种任务中测试 Agent AI 工作流程。</font>
>

<details class="lake-collapse"><summary id="u14cc67ff"><span class="ne-text" style="color: rgb(31, 35, 40); font-size: 16px">配置和安装 WindowsAgentArena</span></summary><p id="u19ca677a" class="ne-p"><span class="ne-text" style="font-size: 16px">✅</span><span class="ne-text" style="color: rgb(31, 35, 40); font-size: 16px">Docker 部署。</span></p><p id="ubd7f5e8d" class="ne-p"><span class="ne-text" style="font-size: 16px">✅</span><span class="ne-text" style="color: rgb(31, 35, 40); font-size: 16px">Python环境：推荐 Python 3.9。</span></p><p id="ube10ea15" class="ne-p"><span class="ne-text" style="font-size: 16px">✅</span><span class="ne-text" style="color: rgba(0, 0, 0, 0.85); font-size: 16px">被评估的虚拟机系统：Windows 11。</span></p></details>

### 2.1.2. WindowsAgentArena V2（使用）

本论文在 WindowsAgentArena 的基础上进行了以下改进，并最终在 V2 进行评估。

![](https://cdn.nlark.com/yuque/0/2025/png/39039688/1754482786077-7e5266fb-1e61-4c2d-a2ae-911c6f283d98.png)

1. 解决了评估依赖性问题。

原始基准测试在任务评估之间缺乏虚拟机状态的重置，可能导致前一任务的更改影响后续任务。

V2 实现了每次评估前恢复虚拟机快照的功能，以确保一致的初始状态，避免任务间的相互干扰。此外还安装了一些原始虚拟机快照中缺失但正确评估所必需的基本软件。

2. 防止不可行攻击（infeasible hacking）。

有些任务由于系统功能已弃用或用户生成的幻觉指令等原因而根本无法实现。这些任务的评估依据是：若在执行过程中输出“FAIL”，即可认定任务成功。这种评估方法使 Agent 在不可行的任务上获得了高分，而无需展示出有意义的能力。

V2 移除了所有不可行任务以提高评估的公平性。

3. 确保虚拟机初始状态的稳定性。

任务初始配置后虚拟机的状态经常会出现网络连接不稳定、软件启动失败或系统延迟等问题。

V2 为了验证初始状态，结合了基于规则（rule-based）和基于LLM的评估。并引入重试机制，允许对初始化错误进行重启尝试，降低了由硬件带来的初始化失败率。

4. 修正评估中的缺陷。

一些评估函数存在漏洞或缺乏稳健性。

V2 识别并纠正了几处评估错误，并在一些复杂任务上依赖人工评估人员，以提高评估的可靠性。

### 2.1.3. OSWorld（使用）</font>

官方仓库：[https://github.com/xlang-ai/OSWorld](https://github.com/xlang-ai/OSWorld)

<font style="color:rgba(0, 0, 0, 0.85);">OSWorld 是一个用于评估多模态智能体在真实计算机环境中执行开放式任务能力的基准测试平台。</font>

<details class="lake-collapse"><summary id="u6927edee"><span class="ne-text">配置和安装 OSWorld</span></summary><p id="ue85d8abe" class="ne-p"><span class="ne-text" style="font-size: 16px">✅</span><span class="ne-text" style="color: rgb(31, 35, 40); font-size: 16px">宿主操作系统：支持 Ubuntu，Windows 或 Mac OS。支持 Docker。</span></p><p id="ubf677516" class="ne-p"><span class="ne-text" style="font-size: 16px">✅</span><span class="ne-text" style="color: rgb(31, 35, 40); font-size: 16px">Python环境：要求 Python&gt;=3.10。</span></p><p id="ue78945e1" class="ne-p"><span class="ne-text" style="font-size: 16px">✅</span><span class="ne-text" style="color: rgb(31, 35, 40); font-size: 16px">虚拟机软件：支持</span><span class="ne-text" style="color: rgba(0, 0, 0, 0.85); font-size: 16px"> VMware Workstation Pro（Windows/Linux）、VMware Fusion（macOS）和 VirtualBox。</span></p><p id="ub9769b77" class="ne-p"><span class="ne-text" style="font-size: 16px">✅</span><span class="ne-text" style="color: rgba(0, 0, 0, 0.85); font-size: 16px">被评估的虚拟机系统：Ubuntu，Windows 或 Mac OS。</span></p></details>

## 2.2. Baseline

 Claude / UI-TARS / Qwen 等。





## 2.3. 实验结果

### 2.3.1. 性能评估（WindowsAgentArena-V2）

1. 结果

![](https://cdn.nlark.com/yuque/0/2025/png/39039688/1754481733140-86b795bd-2bfb-493b-af12-cfa6dd6e802c.png)

2. 结论

通过一组数量极少但质量极高的轨迹数据，即可获得计算机使用的复杂智能体能力。

3. 分析

（1）Real-world task completion，记录了真实的交互过程，并确保任务的成功执行；

（2）多样化的行动决策，由前沿模型生成，在每一步超越人类标注者所选择的单一解决方案，以理性思考生成多种合理的行动方案。

（3）对50条 Qwen 犯错而 PC Agent-E 不犯错的，和Qwen 和 PC Agent-E 都犯错的轨迹进行分析：

+ 这些失败可分为三种类型，这些类型可能在同一任务轨迹甚至单一步骤中同时出现：（1）Knowledge：模型可能缺乏特定的计算机使用知识。（2）Planning：模型可能做出错误的动作决策，例如未能识别并纠正先前的错误操作。（3）Grounding：即使模型的规划正确，执行的动作也可能与其规划过程不一致，主要表现为鼠标点击错误。
+ PC Agent-E 改进主要源于长期规划能力的提升。训练后的 PC Agent-E 生成的思考过程明显更长，并在验证、反思和自我修正方面表现出更强的推理能力。
+ Qwen2.5-VL-72B 和 PC Agent-E 有时也会因 Grounding 和 Knowledge 问题而出现失败。

### 2.3.2. 验证 Trajectory Boost 效果的训练时期的动作增强

在训练过程中改变所使用的合成动作数量，并评估模型性能。

1. 结果

![](https://cdn.nlark.com/yuque/0/2025/png/39039688/1754484130478-153cde7c-b025-4408-ab3f-05323fb4537f.png)

2. 结论

随着动作决策数量的增加，模型性能显著提升。与仅基于人类轨迹训练相比，PC Agent-E 实现了显著更高的增益。

### 2.3.3. 测试时期的动作增强

评估该模型在任务执行过程中允许的最大步数情况下的区别。

1. 结果

![](https://cdn.nlark.com/yuque/0/2025/png/39039688/1754484533831-2642c4d1-6d54-456c-a789-8b294d30c8d3.png)

2. 结论

 PC Agent-E 在测试阶段能更好地利用更长的执行时间（执行动作的步数），表现出更强的适应性和修复能力。  

### 2.3.4. 跨平台的泛化能力（OSWorld）

1. 结果

![](https://cdn.nlark.com/yuque/0/2025/png/39039688/1754484788086-67877083-d20d-4da9-bae0-1ccd4e12d95e.png)

2. 分析

（1）PC Agent-E 具有强大的跨平台泛化能力。

（2）不可行的攻击（infeasible hacking）。较弱的 Qwen2.5-VL-72B 模型在不可行任务上表现出显著更好的性能。表明当前对不可行任务的评估方法并未准确反映计算机使用代理的能力。未来的研究可以着眼于设计更强的不可行任务评估标准，包括评估代理在声明任务为不可能时的理由，以尽量减少利用性行为。

