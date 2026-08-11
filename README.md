# Manuscript DOCX Formatting Skill

[English](README_EN.md)

这是一个独立的 Codex / Agent Skill，用于把科学和生物医学 DOCX 草稿转换成节制、自然、接近人工编辑的投稿格式。它只处理排版，不自动改动科研结论、引文或文章结构。

[![Validate skill](https://github.com/Jameslxr/manuscript-docx-formatting/actions/workflows/validate.yml/badge.svg)](https://github.com/Jameslxr/manuscript-docx-formatting/actions/workflows/validate.yml)
![Version](https://img.shields.io/badge/version-v1.2.0-2563eb)
[![License: MIT](https://img.shields.io/badge/license-MIT-2ea44f)](LICENSE)

## 核心功能

- 将正文段前/段后间距设为 `0 pt`，并清除 Word 隐藏自动段距。
- 在相邻正文段落之间插入且仅插入一个真实空白段落，不用 paragraph spacing 伪造空行。
- 强制显式行距；没有官方要求时默认双倍行距。
- 将同一个已解析行距应用到作者、单位、通讯信息、Keywords、全部标题/
  小标题、正文和 CRediT/声明区；这些角色默认与正文同为 12 pt。
- `Keywords:` 和行内声明标签仅标签加粗；Keywords 后、每个新 section/
  subsection/声明区前恰好一个真实空段落，标题与首段正文之间不放空段落。
- `References/Bibliography` 后的条目保持独立非正文角色，不在参考文献条目之间插入正文空行。
- 使用 Word 原生连续行号和动态 `PAGE` 页码。
- 将标题、作者、单位和通讯信息调整为节制的左对齐 manuscript 首页；精确期刊模板可以覆盖。
- 在 Title 与 Authors 之间强制且审计一个真实 Enter 空段落；只有官方期刊/模板依据才可切换为紧接。
- 将 CRediT 设为紧凑语义区块，检查官方 14 种角色词汇，禁止在相邻作者条目之间插入空段落。
- 支持当前期刊、文章类型和投稿阶段的官方格式覆盖。
- 分别检查结构、首页、语义纵向节奏、内容保留、期刊规则和逐页渲染；只有全部通过才返回 `FORMAT_RELEASE_PASS`。

## 安装

最简单的方式是把下面这句话交给 Codex：

```text
使用 $skill-installer 安装这个 Skill：
https://github.com/Jameslxr/manuscript-docx-formatting/tree/main/manuscript-docx-formatting
```

也可以在终端安装：

```bash
python3 ~/.codex/skills/.system/skill-installer/scripts/install-skill-from-github.py \
  --repo Jameslxr/manuscript-docx-formatting \
  --path manuscript-docx-formatting
```

手动安装到 Claude Code 或其他兼容 Agent Skills 的环境：

```bash
git clone https://github.com/Jameslxr/manuscript-docx-formatting.git
cp -R manuscript-docx-formatting/manuscript-docx-formatting \
  "$HOME/.claude/skills/manuscript-docx-formatting"
python3 -m pip install -r manuscript-docx-formatting/requirements.txt
```

## 调用示例

```text
使用 $manuscript-docx-formatting，将这个 DOCX 调整为自然的投稿格式。
保留全部文字、引文、图表和顺序；输出新 DOCX 及全部格式审计结果。
```

期刊个性化格式：

```text
使用 $manuscript-docx-formatting。
目标期刊：[exact journal]
文章类型：[article type]
投稿阶段：[initial / revision / final]
请查阅当前官方要求后排版，并列出来源、执行和验证结果。
```

## 工作边界

Skill 会保留原文件并生成新文件。它不进行科学审稿、证据核查、语言改写或引用真实性审计；这些任务应使用 `manuscript-review-revision`。`FORMAT_RELEASE_PASS` 只证明已解析的 DOCX 排版契约通过，不代表科学结论正确、期刊已接收或投稿已成功。

## 验证

```bash
python3 -m pip install -r requirements.txt
python3 -m py_compile manuscript-docx-formatting/scripts/*.py
python3 -m unittest discover -s manuscript-docx-formatting/tests -v
```

当前回归套件包含 23 个测试，覆盖真实 Title–Authors 空段落及期刊紧接覆盖、紧凑 CRediT 作者条目、官方角色词汇、作者/单位字号、全局双倍与 1.5 倍行距、Keywords 和 section/subsection 边界，并继续覆盖真实正文空段落、隐藏自动段距、行/页码、对抗性首页、幂等修复、文字保留和 fail-closed 发布。

## License

[MIT](LICENSE)
