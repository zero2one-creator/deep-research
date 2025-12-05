# **深度研究报告：GitHub Spec Kit 与 Fission AI OpenSpec 在 iOS 规范驱动开发（SDD）中的架构对比与选型指南**

## **1\. 执行摘要**

在生成式人工智能（Generative AI）重塑软件工程的浪潮中，移动应用开发正经历从“手写代码”向“意图编排”的范式转移。传统的对话式 AI 编程（即所谓的“直觉式编码”或 "Vibe Coding"）在面对复杂的 iOS 工程时，往往因上下文丢失、架构漂移和 Xcode 项目结构的脆弱性而难以为继。为了解决这一痛点，**规范驱动开发（Spec-Driven Development, SDD）** 应运而生，旨在通过可执行的文档将人类意图锁定为确定性的工程输出。

本报告针对用户计划开发的 iOS 应用程序，对当前 SDD 领域的两个主流框架——**GitHub Spec Kit** 与 **Fission AI OpenSpec**——进行了详尽的对比研究。分析显示，两者虽然殊途同归，旨在解决 AI 编程的随机性问题，但在设计哲学、工作流机制及对 iOS 生态的适配性上存在本质差异。

**GitHub Spec Kit** 采用严格的结构化瀑布流模式，强调“宪法”（Constitution）与“计划”（Plan）的分离，适合对架构规范性要求极高、从零开始（Greenfield）的 iOS 项目 1。相比之下，**OpenSpec** 采用轻量级的事务性变更模式，强调“提案”（Proposal）与“增量”（Delta）的快速迭代，在处理遗留代码（Brownfield）及追求开发速度的独立开发者场景中表现卓越 3。

对于 iOS 开发者而言，选择的核心在于权衡**架构控制力**与**迭代速率**。本报告建议，若用户追求构建企业级、多人协作且架构严谨的 iOS 应用，Spec Kit 是更优选择；若用户倾向于独立开发、快速验证想法或在现有项目中引入 AI 辅助，OpenSpec 则能提供更流畅的开发者体验。

## ---

**2\. 范式转移：规范驱动开发（SDD）在 iOS 工程中的理论基础**

### **2.1 “直觉式编码”的危机与 iOS 的特殊性**

在 AI 辅助编程的早期阶段，开发者主要依赖 IDE（如 Xcode 配合 Copilot）中的对话窗口进行交互。这种模式被称为“直觉式编码”（Vibe Coding），即开发者不断调整提示词（Prompt），直到生成的代码“看起来感觉是对的” 6。

然而，在 iOS 开发语境下，这种模式面临严峻挑战：

1. **上下文窗口的耗尽（Context Exhaustion）：** 随着对话深入，LLM 逐渐遗忘初始的架构约束。例如，在 SwiftUI 项目中，AI 可能在后期突然引入 UIKit 组件，破坏了声明式 UI 的一致性 3。  
2. **Xcode 项目文件的脆弱性：** iOS 工程依赖复杂的 .xcodeproj 和 project.pbxproj 文件管理引用。非结构化的 AI 修改极易导致项目文件损坏，无法编译。  
3. **强类型系统的约束：** Swift 是一门强类型语言，对可选值解包（Optional Unwrapping）、内存管理（ARC）有严格要求。缺乏全局规范的 AI 容易生成导致运行时崩溃的代码（如强制解包 \!）9。

SDD 通过引入“可执行规范”解决了上述问题。它将需求文档从“死文档”转变为“活的真理源”（Source of Truth），AI 必须依据规范生成代码，而非仅仅响应聊天历史。

### **2.2 SDD 的核心价值主张**

无论是 Spec Kit 还是 OpenSpec，其核心价值均建立在以下三个支柱之上：

* **确定性（Determinism）：** 相同的规范应产生可预期的代码结构，消除 AI 的随机幻觉 3。  
* **可追溯性（Traceability）：** 每一行代码的变更都能追溯到特定的需求文档或提案，而非散落在聊天记录中 10。  
* **意图锁定（Intent Locking）：** 在编写代码之前，人类与 AI 先就“做什么”达成一致，从而减少返工 11。

## ---

**3\. GitHub Spec Kit：严谨的立法者与规划师**

### **3.1 核心设计哲学：宪法与分层治理**

GitHub Spec Kit 由 GitHub Next 团队推出，其设计哲学深受大型软件工程实践的影响。它将开发过程视为一个**分层治理**的体系，强调“做什么”（Spec）与“怎么做”（Plan）的严格分离 1。在 iOS 开发中，这种分离尤为关键，因为它允许开发者在 AI 动工之前，先审查其技术选型（例如：选择 Core Data 还是 SwiftData）。

Spec Kit 的工作流包含四个明确的阶段，构成了其独特的生命周期：

1. **制定宪法（Constitution）：** 确立项目的基本法则。  
2. **定义规范（Specify）：** 描述用户需求和业务逻辑。  
3. **制定计划（Plan）：** 将需求转化为技术实施方案。  
4. **执行任务（Implement）：** 将计划分解为原子任务并生成代码 2。

### **3.2 深度解析：Spec Kit 的 iOS 工作流机制**

#### **3.2.1 宪法层（The Constitution）：iOS 架构的守护神**

/speckit.constitution 命令是 Spec Kit 在 iOS 开发中最具杀伤力的功能。它允许开发者定义一份不可违背的技术纲领。对于 iOS 项目，这意味着可以通过自然语言强制执行 Swift 最佳实践 2。

iOS 宪法示例分析：  
在实际应用中，一个针对 SwiftUI 项目的宪法可能包含以下条款：

* *架构约束：* “所有视图（View）必须遵循 MVVM 模式，逻辑必须封装在遵循 ObservableObject 的 ViewModel 中。”  
* *代码风格：* “禁止在生产代码中使用 print()，必须使用统一的 Logger。”  
* *安全性：* “严禁使用强制解包（Force Unwrap \!），必须使用 if let 或 guard let 处理可选值。”  
* *UI 规范：* “所有颜色必须引用 ColorAsset，禁止硬编码 HEX 值。”

这种机制直接解决了 AI “乱写代码”的问题。在生成代码前，Spec Kit 会根据宪法审查计划（Plan），如果 AI 试图在 SwiftUI 中混用 Storyboard，宪法检查机制会直接驳回该计划 12。

#### **3.2.2 规划层（The Plan）：预演 Xcode 工程变更**

/speckit.plan 是 Spec Kit 的战略中心。在这个阶段，AI 会读取 spec.md（需求）和 constitution.md（规则），然后生成一份详细的技术实施文档 plan.md。

对于 iOS 开发者，这一步至关重要，因为它允许人类介入审查极其敏感的 Xcode 工程变更：

* **依赖管理审查：** 计划会列出需要引入的 Swift Package。开发者可以在此阶段阻止 AI 引入不必要的第三方库。  
* **文件结构审查：** 计划会列出将要创建的 .swift 文件路径。这有助于保持项目目录的整洁，防止 AI 将所有文件都堆在根目录下 9。

#### **3.2.3 任务层（The Tasks）与实现**

/speckit.tasks 将宏大的计划拆解为一系列微小的、可测试的步骤（Tasks）。在 iOS 开发中，这意味着 AI 不会试图一次性重写整个 AppDelegate，而是会分步进行：

1. 创建 Model 层数据结构。  
2. 编写 ViewModel 逻辑。  
3. 创建 SwiftUI 视图。  
4. 将视图连接到 ViewModel。

这种原子化的任务分解极大地提高了代码生成的成功率，减少了因单次输出过长导致的截断错误 2。

### **3.3 生态系统与技术栈：Python/UV 的门槛**

Spec Kit 的一个显著特征（或缺陷，取决于开发者的背景）是其依赖 Python 生态。它主要通过 uv（一个高性能 Python 包管理器）安装和运行 specify-cli 1。  
对于习惯于 Node.js (npm/yarn) 或 Ruby (CocoaPods/Fastlane) 的 iOS 开发者来说，引入 Python 工具链可能带来额外的环境配置负担。尽管 uv 旨在简化这一过程，但这仍是一个不可忽视的异构技术栈引入 5。

## ---

**4\. Fission AI OpenSpec：敏捷的交易员与实干家**

### **4.1 核心设计哲学：变更即一切**

与 Spec Kit 的宏大叙事不同，Fission AI 推出的 OpenSpec 奉行极简主义和实用主义。其设计哲学认为，软件开发本质上是一系列离散的“状态变更”（State Changes）。因此，OpenSpec 不维护庞大的全局文档，而是聚焦于\*\*“变更文件夹”（Change Folder）\*\*的概念 3。

OpenSpec 将工作流压缩为三个核心指令，极其符合“提议-执行-归档”的事务性心智模型：

1. **提案（Proposal）：** /openspec:proposal —— 定义变更意图。  
2. **应用（Apply）：** /openspec:apply —— 执行变更。  
3. **归档（Archive）：** /openspec:archive —— 固化知识 16。

### **4.2 深度解析：OpenSpec 的 iOS 工作流机制**

#### **4.2.1 提案系统（The Proposal）：轻量级的 RFC**

在 OpenSpec 中，当你需要开发一个新功能（如“添加深色模式支持”），你不需要更新全局规格说明书，而是创建一个独立的变更请求。  
系统会生成一个包含 proposal.md 和 tasks.md 的文件夹。最关键的创新在于 “Spec Deltas”（规范增量）。OpenSpec 鼓励开发者只定义“什么改变了”，而不是“系统是什么” 3。  
**iOS 场景下的优势：**

* **Token 效率：** 在处理大型 iOS 项目（如数千个 Swift 文件）时，Spec Kit 可能会因为加载完整的 spec.md 和 constitution.md 而消耗大量 Token。OpenSpec 仅加载“增量”信息，极大降低了成本并提高了响应速度 16。  
* **专注度：** AI 仅关注当前变更，减少了因上下文过载导致的幻觉。

#### **4.2.2 直接执行（Apply）：速度至上**

OpenSpec 省略了独立的“Plan”阶段，直接从提案跳转到代码实现。对于经验丰富的 iOS 开发者来说，这是一个巨大的效率提升。

* *场景：* “为 ProfileView 添加一个登出按钮。”  
* *Spec Kit 流程：* Specify \-\> Plan (分析影响) \-\> Tasks \-\> Implement。  
* *OpenSpec 流程：* Proposal \-\> Apply。

研究表明，在执行相同任务时，OpenSpec 的产出速度显著快于 Spec Kit，生成的文档行数仅为后者的 1/3（约 250 行 vs 800 行），且生成的代码质量并未因此下降，甚至在处理 CSS（类比 iOS 的 UI 布局细节）时表现出对细节的关注 5。

#### **4.2.3 归档（Archive）与知识累积**

当变更被接受后，OpenSpec 将其归档，并更新项目根目录下的 AGENTS.md 或全局规范。这意味着项目知识是随着变更逐渐累积的，而不是预先定义的。这种方式非常适合\*\*遗留代码（Brownfield）\*\*的改造，开发者可以在修改现有 iOS 代码库的过程中，逐步建立起项目的规范文档，而无需在开始前进行大规模的逆向工程 6。

### **4.3 生态系统：Node.js 的亲和力**

OpenSpec 基于 TypeScript 编写，通过 npm 安装。这对于现代前端开发者和许多全栈开发者来说是原生环境。即使是纯 iOS 开发者，通常也安装了 Node.js 环境（用于某些 CLI 工具），因此其安装和集成阻力相对较小 3。

## ---

**5\. 深度对比：iOS 开发语境下的异同与权衡**

为了给您的 iOS App 开发提供精准建议，本节将通过多维度的表格和深度分析，对比两个框架在 iOS 特定场景下的表现。

### **5.1 功能特性与架构对比表**

| 比较维度 | GitHub Spec Kit | Fission AI OpenSpec |
| :---- | :---- | :---- |
| **核心隐喻** | 立法与规划（Constitution & Plan） | 事务性变更（Proposal & Delta） |
| **工作流阶段** | 4 阶段：Specify $\\to$ Plan $\\to$ Tasks $\\to$ Implement | 3 阶段：Proposal $\\to$ Apply $\\to$ Archive |
| **架构控制力** | **极高**。通过 Constitution 强制执行 Swift 模式。 | **中等**。依赖 AI 对现有代码的理解，缺乏强制拦截层。 |
| **上下文管理** | 全局加载，Token 消耗大，上下文全面。 | 增量加载（Delta），Token 效率高，专注度高。 |
| **分支管理** | **自动化**。为每个功能自动创建 Git 分支。 | **手动**。需要开发者自己管理 Git 分支策略。 |
| **文档量级** | 繁重（Heavyweight）。产出大量 Markdown 文件。 | 轻量（Lightweight）。仅产出必要文件。 |
| **生态基础** | Python / uv | Node.js / TypeScript / npm |
| **适用场景** | 从零开始（Greenfield）、大型团队、复杂架构。 | 遗留改造（Brownfield）、独立开发、快速迭代。 |
| **IDE 集成** | 依赖 Copilot Chat 或 CLI。 | 与 Cursor / Composer 深度集成（Slash Commands）。 |

### **5.2 关键差异点深度剖析**

#### **5.2.1 架构一致性 vs. 开发速度**

* **Spec Kit 的“刹车机制”：** 在 iOS 开发中，架构一致性至关重要。Spec Kit 的 Plan 阶段实际上是一个“刹车”，强迫开发者在 AI 写代码前停下来思考：“这个 ViewModel 的设计真的合理吗？”这种机制虽然降低了开发速度，但极大地减少了“写了再改”的沉没成本，特别是在涉及 Core Data 模型设计或复杂的网络层封装时 10。  
* **OpenSpec 的“油门机制”：** OpenSpec 假设开发者已经想清楚了，或者愿意在代码生成后进行快速调整。它移除了中间的繁文缛节，让 AI 能够像一个熟练的结对程序员一样迅速给出结果。对于 UI 调整、简单的逻辑在 SwiftUI 中的实现，这种速度优势是压倒性的 17。

#### **5.2.2 对 Xcode 项目结构的保护**

iOS 开发的一个痛点是多人协作时的 .xcodeproj 冲突。

* **Spec Kit** 通过自动分支策略（Automatic Branching），强制每个功能在独立分支开发，这在物理上隔离了风险。其 plan.md 也可以作为代码审查（Code Review）的一部分，在合并代码前先审查“变更意图”，这对于保护 Xcode 项目结构非常有价值 5。  
* **OpenSpec** 不自动管理分支，这给予了开发者自由，但也意味着开发者必须有更强的 Git 纪律。如果开发者在主分支直接运行 /openspec:apply 并导致项目文件损坏，回滚成本可能较高 5。

#### **5.2.3 遗留代码（Brownfield）的适配性**

* 如果您的 iOS App 是一个全新的项目（Greenfield），**Spec Kit** 的优势在于它能帮你建立一套“宪法”，从第一行代码开始就确立高标准。  
* 如果您是在一个现有的 iOS 项目中引入 AI，**OpenSpec** 是更好的选择。Spec Kit 往往需要对现有项目进行全面的“索引”或“规范化”才能发挥最大效用，而 OpenSpec 可以立即针对某个具体的小功能（如“修复登录页面的 Bug”）开始工作，无需为整个旧项目补全文档 6。

## ---

**6\. 实战场景模拟：应该选择哪个？**

为了帮助您做出最终决定，我们构建了两个典型的 iOS 开发场景。

### **6.1 场景 A：企业级/复杂架构 iOS 应用（推荐 Spec Kit）**

**项目特征：**

* 您计划开发一个包含复杂本地数据库、实时网络同步、且架构要求严格（如 The Composable Architecture, TCA）的应用。  
* 您可能与后端团队协作，API 变动频繁。  
* 您希望代码具有高度的一致性，即使是 AI 生成的代码也必须符合严格的 Lint 规则。

为什么选择 Spec Kit：  
在这种场景下，错误的架构决策代价高昂。Spec Kit 的\*\*宪法（Constitution）\*\*功能是无可替代的。您可以在宪法中写明：“所有副作用必须封装在 Effect 中，禁止在 Reducer 中直接调用 API。” 这样，AI 生成的每一个功能模块都会严格遵守 TCA 的模式，避免了后期大规模重构的风险。其详尽的文档也为后续接手的开发者提供了完美的交接材料 9。

### **6.2 场景 B：独立开发/快速原型 iOS 应用（推荐 OpenSpec）**

**项目特征：**

* 您是一名独立开发者（Indie Developer）或处于初创团队，追求快速上线（Time-to-Market）。  
* 项目主要使用 SwiftUI，架构相对灵活（标准 MVVM）。  
* 您使用 Cursor 作为主要的 IDE。  
* 您希望 AI 能够快速帮您实现“添加一个设置页面”、“修改字体颜色”等具体任务。

为什么选择 OpenSpec：  
在这种场景下，Spec Kit 的繁文缛节会成为负担。为了改一个按钮颜色而生成 4 个文档并创建一个新分支显得过于笨重。OpenSpec 的 /openspec:proposal 允许您用自然语言快速描述变更，然后 /openspec:apply 立即在几秒钟内完成代码修改。其低 Token 消耗也意味着更低的使用成本。它与 Cursor 的深度集成（通过 Slash Commands）能提供丝滑的编码体验，让您保持在“流”（Flow）状态中 5。

## ---

**7\. 结论与建议**

在对比了 **GitHub Spec Kit** 和 **Fission AI OpenSpec** 的实现异同后，针对您“准备开发一个 iOS APP”的需求，结论如下：

**首选推荐：Fission AI OpenSpec**

**理由：**

1. **启动阻力最小：** 对于个人开发者或新项目起步，OpenSpec 的配置（npm install）和心智模型（提议-执行）更加直观，能让您最快进入开发状态。  
2. **更适合现代 AI 工具：** 既然您选择 SDD，很可能也会使用 Cursor 等先进的 AI 编辑器。OpenSpec 对 Cursor 的原生支持（Slash Commands）体验优于 Spec Kit 17。  
3. **灵活度高：** iOS 开发初期往往伴随着频繁的需求变更。OpenSpec 的轻量级迭代模式能更好地适应这种变化，而不会被厚重的文档拖累。  
4. **Token 成本控制：** 在开发初期频繁试错时，OpenSpec 的增量更新策略能为您节省可观的 API 费用。

何时转向 GitHub Spec Kit？  
如果您的项目发展到中期，团队扩展到 3 人以上，或者发现 AI 生成的代码开始出现严重的架构不一致（如混乱的导航逻辑、混合的 UI 模式），此时应考虑引入 Spec Kit，利用其“宪法”功能来强制进行架构治理。  
最终建议：  
建议您从 OpenSpec 开始。在项目根目录创建一个 project.md 文件，在其中模仿 Spec Kit 的宪法，写上您的 iOS 架构原则（如 SwiftUI 最佳实践）。这样您既能享受 OpenSpec 的速度，又能保留 Spec Kit 的部分规范性优势，达到最佳的开发平衡。

#### **引用的著作**

1. Diving Into Spec-Driven Development With GitHub Spec Kit \- Microsoft for Developers, 访问时间为 十二月 5, 2025， [https://developer.microsoft.com/blog/spec-driven-development-spec-kit](https://developer.microsoft.com/blog/spec-driven-development-spec-kit)  
2. github/spec-kit: Toolkit to help you get started with Spec-Driven Development, 访问时间为 十二月 5, 2025， [https://github.com/github/spec-kit](https://github.com/github/spec-kit)  
3. Fission-AI/OpenSpec: Spec-driven development for AI coding assistants. \- GitHub, 访问时间为 十二月 5, 2025， [https://github.com/Fission-AI/OpenSpec](https://github.com/Fission-AI/OpenSpec)  
4. Spec-driven AI coding: Spec-kit, BMAD, Agent OS and Kiro | by Tim Wang \- Medium, 访问时间为 十二月 5, 2025， [https://medium.com/@tim\_wang/spec-kit-bmad-and-agent-os-e8536f6bf8a4](https://medium.com/@tim_wang/spec-kit-bmad-and-agent-os-e8536f6bf8a4)  
5. OpenSpec vs Spec Kit: Choosing the Right AI-Driven Development Workflow for Your Team, 访问时间为 十二月 5, 2025， [https://hashrocket.com/blog/posts/openspec-vs-spec-kit-choosing-the-right-ai-driven-development-workflow-for-your-team](https://hashrocket.com/blog/posts/openspec-vs-spec-kit-choosing-the-right-ai-driven-development-workflow-for-your-team)  
6. GitHub Just Launched Spec Kit — But Fission AI's OpenSpec Fights Back\! \- YouTube, 访问时间为 十二月 5, 2025， [https://www.youtube.com/watch?v=7UMiGPeC0qc](https://www.youtube.com/watch?v=7UMiGPeC0qc)  
7. Spec-Driven Development (SDD) Is the Future of Software Engineering, 访问时间为 十二月 5, 2025， [https://medium.com/@shenli3514/spec-driven-development-sdd-is-the-future-of-software-engineering-85b258cea241](https://medium.com/@shenli3514/spec-driven-development-sdd-is-the-future-of-software-engineering-85b258cea241)  
8. Spec-driven development is underhyped\! Here's how you build better with Cursor\! \- Reddit, 访问时间为 十二月 5, 2025， [https://www.reddit.com/r/cursor/comments/1nomd8t/specdriven\_development\_is\_underhyped\_heres\_how/](https://www.reddit.com/r/cursor/comments/1nomd8t/specdriven_development_is_underhyped_heres_how/)  
9. Spec-Driven Development for iOS: A Practical Spec-kit Guide, 访问时间为 十二月 5, 2025， [https://maddevs.io/writeups/spec-driven-development-in-ios/](https://maddevs.io/writeups/spec-driven-development-in-ios/)  
10. Comprehensive Guide to Spec-Driven Development Kiro, GitHub Spec Kit, and BMAD-METHOD, 访问时间为 十二月 5, 2025， [https://medium.com/@visrow/comprehensive-guide-to-spec-driven-development-kiro-github-spec-kit-and-bmad-method-5d28ff61b9b1](https://medium.com/@visrow/comprehensive-guide-to-spec-driven-development-kiro-github-spec-kit-and-bmad-method-5d28ff61b9b1)  
11. Spec-driven development with AI: Get started with a new open source toolkit, 访问时间为 十二月 5, 2025， [https://github.blog/ai-and-ml/generative-ai/spec-driven-development-with-ai-get-started-with-a-new-open-source-toolkit/](https://github.blog/ai-and-ml/generative-ai/spec-driven-development-with-ai-get-started-with-a-new-open-source-toolkit/)  
12. spec-kit/spec-driven.md at main \- GitHub, 访问时间为 十二月 5, 2025， [https://github.com/github/spec-kit/blob/main/spec-driven.md](https://github.com/github/spec-kit/blob/main/spec-driven.md)  
13. Spec-Kit Rules as part of Default Constitution \#980 \- GitHub, 访问时间为 十二月 5, 2025， [https://github.com/github/spec-kit/discussions/980](https://github.com/github/spec-kit/discussions/980)  
14. Spec-Driven Development Tutorial using GitHub Spec Kit | Scalable Path, 访问时间为 十二月 5, 2025， [https://www.scalablepath.com/machine-learning/spec-driven-development-workflow](https://www.scalablepath.com/machine-learning/spec-driven-development-workflow)  
15. Spec-Driven Development and Context Engineering—A Smarter Approach to AI-Enabled Coding \- TechChannel, 访问时间为 十二月 5, 2025， [https://techchannel.com/artificial-intelligence/sdd-and-context-engineering/](https://techchannel.com/artificial-intelligence/sdd-and-context-engineering/)  
16. How to make AI follow your instructions more for free (OpenSpec) \- DEV Community, 访问时间为 十二月 5, 2025， [https://dev.to/webdeveloperhyper/how-to-make-ai-follow-your-instructions-more-for-free-openspec-2c85](https://dev.to/webdeveloperhyper/how-to-make-ai-follow-your-instructions-more-for-free-openspec-2c85)  
17. OpenSpec \- Lightweight & portable spec driven framework for AI coding assistants\!, 访问时间为 十二月 5, 2025， [https://forum.cursor.com/t/openspec-lightweight-portable-spec-driven-framework-for-ai-coding-assistants/134052](https://forum.cursor.com/t/openspec-lightweight-portable-spec-driven-framework-for-ai-coding-assistants/134052)  
18. I Found the Simplest AI Dev Tool Ever \- YouTube, 访问时间为 十二月 5, 2025， [https://www.youtube.com/watch?v=cQv3ocbsKHY](https://www.youtube.com/watch?v=cQv3ocbsKHY)  
19. Spec-Driven Development: 0 to 1 with Spec-kit & Cursor \- Mad Devs, 访问时间为 十二月 5, 2025， [https://maddevs.io/writeups/project-creation-using-spec-kit-and-cursor/](https://maddevs.io/writeups/project-creation-using-spec-kit-and-cursor/)  
20. How to Use Spec-Driven Development to Explore Legacy Codebases \- EPAM, 访问时间为 十二月 5, 2025， [https://www.epam.com/insights/ai/blogs/using-spec-kit-for-brownfield-codebase](https://www.epam.com/insights/ai/blogs/using-spec-kit-for-brownfield-codebase)