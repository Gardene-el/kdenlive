# Kdenlive 项目文档

本目录包含了 Kdenlive 项目的详细技术文档，帮助开发者理解项目架构和实现细节。

## 文档列表

### 1. [项目架构报告](./项目架构报告.md)

全面介绍 Kdenlive 的整体架构设计，包括：

- **项目概述**：技术栈、版本信息
- **整体架构**：分层设计、MVC 模式
- **核心组件详解**：
  - Core 核心管理器
  - MLT 集成
  - Timeline 时间线系统
  - Bin 素材管理器
  - Effects & Transitions 效果和转场
  - Project Management 项目管理
  - Rendering 渲染系统
  - Monitor System 监视器系统
- **项目文件结构**
- **设计模式与最佳实践**
- **扩展性设计**
- **测试与质量保证**

**适合阅读对象**：
- 新加入的开发者，需要了解整体架构
- 想要为 Kdenlive 贡献代码的开发者
- 研究视频编辑软件架构的技术人员

### 2. [CLI工具与文件格式报告](./CLI工具与文件格式报告.md)

深入讲解 Kdenlive 的命令行工具、工程文件格式、轨道和片段框架，包括：

- **CLI 命令行工具**：
  - `kdenlive_render` 渲染工具详解
  - 主程序 CLI 接口
  - 其他辅助工具
- **工程文件格式详解**：
  - 文件格式代系演进（Generation 1-5）
  - XML 结构详解
  - Kdenlive 自定义属性
  - 特殊组件（字幕、混合、效果、转场）
- **轨道（Track）框架**：
  - TrackModel 架构
  - 双 Playlist 结构
  - 轨道操作机制
  - 特殊轨道
- **片段（Clip）框架**：
  - ClipModel 架构
  - 片段与 Bin 的关系
  - 片段操作详解
  - 片段分组、吸附、效果栈
- **数据流与交互机制**
- **性能优化技术**

**适合阅读对象**：
- 需要了解项目文件格式的开发者
- 想要开发 CLI 工具或批处理脚本的用户
- 研究时间线实现细节的开发者
- 需要理解轨道和片段工作机制的贡献者

## 其他资源

### 英文开发文档

项目根目录下的 `dev-docs/` 文件夹包含英文开发文档：

- `architecture.md` - 架构概述
- `fileformat.md` - 文件格式详细说明
- `mlt-intro.md` - MLT 框架介绍
- `build.md` - 构建说明
- `coding.md` - 编码规范
- `contributing.md` - 贡献指南
- `packaging.md` - 打包说明

### GitHub Copilot 指令

`.github/copilot-instructions.md` 文件为 GitHub Copilot 提供项目特定的上下文和指导，帮助 AI 助手更好地理解 Kdenlive 代码库并提供更准确的建议。

**包含内容**：
- 项目概述和技术栈
- 架构和关键组件
- 编码规范和最佳实践
- 常见开发任务指南
- 重要注意事项
- 资源链接

### 在线资源

- **官方网站**：https://kdenlive.org
- **用户文档**：https://docs.kdenlive.org
- **开发仓库**：https://invent.kde.org/multimedia/kdenlive
- **MLT 文档**：https://www.mltframework.org/docs/
- **Qt 文档**：https://doc.qt.io/qt-6/
- **KDE API**：https://api.kde.org/frameworks/

## 贡献

如果您发现文档中的错误或希望改进文档，欢迎：

1. 在 [KDE GitLab](https://invent.kde.org/multimedia/kdenlive/-/issues) 提交 Issue
2. 提交 Merge Request 修正或改进文档
3. 在 Matrix 频道 `#kdenlive-dev:kde.org` 讨论

## 文档维护

这些文档基于 Kdenlive 25.11.70 版本的代码分析生成，会随着项目发展不断更新。

---

*最后更新：2024-10-24*
