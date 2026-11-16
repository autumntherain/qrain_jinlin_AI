# qrain_jinlin_AI工程 · 体积与许可控制手册

> 适用项目：GitHub `autumntherain` · AlexNet 复现与改良（后续扩展为 6 篇论文合集）
> 目标：源码 + 文档 + 演示图，**git 仓库 ≤ 10 MB**，clone 下来 10 秒内可用；许可证为 Apache 2.0。

---

## 一、体积预算表

| 内容 | 预算 | 存放位置 |
|---|---|---|
| 源码（Python / C++ / 脚本） | < 300 KB | git |
| 论文 docx（中英各 6 篇） | ≈ 600 KB | git |
| 预训练权重 | 0（仓库内） | GitHub Release |
| 数据集 | 0（仓库内） | 下载脚本 / 网盘 |

**结论**：把权重和数据集挡在仓库外，剩下的东西天然只有几 MB。体积问题九成是这两样东西造成的。

---

## 二、图片压缩：三档策略

| 档位 | 用途 | 短边 | 质量 | 格式 |
|---|---|---|---|---|
| `web` | README、网页预览 | 336px | 75 | 无透明通道转 WebP |
| `thumb` | 列表缩略图 | 224px | 72 | 转 WebP |

> ⚠️ 插图说明：本项目论文文档不含原版图表，请参阅：
[论文中无插图通告.md](./docs/论文中无插图通告.md)

### 2.1 结论：仓库里一张图都不放

三条理由，按重要性排：

1. **版权零风险** —— 论文原图（AlexNet 属 NeurIPS 论文集、Nature 版 DQN 属 Springer Nature、arXiv 各篇为 arXiv 非独占分发许可）都不属于可公开再分发内容。一张不进来，问题直接消失，无需逐个判断许可。
2. **不用自己画** —— 省掉整套架构图重绘（AlexNet 双 GPU 分支、Transformer 多头结构都很费时）。
3. **体积归零** —— 原预算里 3 MB 的图片额度直接不存在。

那 README 会不会太干？不会。用**文字 + 表格 + 代码块**描述结构；确需视觉对照时，给**指向原文的链接**（如 arXiv 页面），而不是把图本身放进来。

### 2.2 配套声明文件 `ASSETS_POLICY.md`

放在仓库根目录，内容如下：

```text
ASSETS POLICY

This repository intentionally contains NO image assets.

Rationale:
1. Figures from the original papers (NeurIPS, Springer Nature, arXiv) are not
   redistributable under our license. To avoid any copyright ambiguity, no
   figures are included in this repository.
2. This project distributes textual documentation and source code only.
If you need to view the original figures, please refer to the official paper
pages listed in README.md.

Copyright 2026 autumntherain
Licensed under the Apache License, Version 2.0.
```

README 里对应加一行：

```markdown
> 本仓库不含任何图片素材，原因与依据见 [ASSETS_POLICY.md](./ASSETS_POLICY.md)。
> 论文原图请查阅 README 中给出的原文链接。
```

命令：

```bash
# 1) 先体检，看清单和总体积（不改任何文件）
python tools/compress_assets.py .

# 2) 确认后执行，覆盖前自动备份
python tools/compress_assets.py . --apply --backup

# 3) 网页资源单独跑 web 档
python tools/compress_assets.py docs --apply --profile web

# 4) 只处理大于 20KB 的图，其余不动
python tools/compress_assets.py . --apply --min-kb 20
```

实测参考（2400×1800 彩色配图）：

- `doc` 档：721 KB → **85.5 KB（省 88.2%）**
- `web` 档：715 KB → **27.4 KB（省 96.2%）**

安全设计：默认 dry-run；带透明通道的 PNG 不会被转 JPEG（避免黑底）；压缩后反而变大的自动跳过；小于 80 KB 的默认不动。

---

## 三、体积守卫（CI / pre-commit）

```bash
# 默认：总量上限 10MB、单文件上限 2MB
python tools/check_repo_size.py .

# 给个别文件开口子（例如必须提交的演示权重）
python tools/check_repo_size.py . --max-file 2 --allow "*.onnx:30"
```

超限时**退出码为 1**（实测：超限=1，通过=0），可直接挂进 GitHub Actions：

```yaml
- name: Size guard
  run: python tools/check_repo_size.py . --max-total 10 --max-file 2
```

---

## 四、三个最容易踩的坑

1. **删了文件体积不减**
   git 历史仍保留旧 blob。真要大清理得用 `git filter-repo` 重写历史，或干脆新建仓库推干净提交。

2. **docx 里嵌入 WebP 打不开**
   Word 只认 JPEG / PNG / BMP / GIF。文档用图必须走 `doc` 档。

3. **数据集「先提交再删」**
   一次提交几百 MB 就把仓库报废（GitHub 单文件 100 MB 硬限制）。**`git init` 之前先放 `.gitignore`**。

---

## 五、许可证与版权（Apache 2.0）

### 5.1 选型结论

本项目采用 **Apache License 2.0**，理由：

- 宽松：任何人可商用、修改、闭源分发；
- **自带专利授权**：贡献者自动授出相关专利许可，若使用者对项目发起专利诉讼，其许可立即失效；
- 要求标注修改点，利于后续「复现 + 改良」的可追溯；
- 企业接受度高，比 MIT 多一层保护。

### 5.2 必须齐备的三件套

| 文件 | 内容 | 要不要手改 |
|---|---|---|
| `LICENSE` | Apache 2.0 英文全文 | **不要改**，GitHub 新建仓库时自动生成即为标准 |
| `NOTICE` | 项目归属与来源说明 | 需要写，见下 |
| 源码头注释 | 版权行 + SPDX 标识 | 每个源码文件顶部加 |

> 关键澄清：`LICENSE` 全文末尾那段 `Copyright [yyyy] [name of copyright owner]` 是 **Apache 官方给的示例模板**，说明「你该怎样在源码头写声明」。放在 `LICENSE` 里是标准文本的一部分，**保持原样**；真正要替换成 2026 和 `autumntherain` 的地方，是**你自己源码文件的头部**。

### 5.3 NOTICE 文件

```text
AI Paper Reproduction Project
Copyright 2026 autumntherain

This project is an open-source reproduction and improvement implementation
of classic deep learning papers, including AlexNet, Transformer, BERT, Adam,
DQN, and RLVR.

Portions of this work reference original research papers and public
open-source implementations; their respective licenses are retained
where applicable.

Licensed under the Apache License, Version 2.0.
```

### 5.4 源码文件头模板

Python：

```python
# Copyright 2026 autumntherain
# SPDX-License-Identifier: Apache-2.0
```

C++：

```cpp
// Copyright 2026 autumntherain
// SPDX-License-Identifier: Apache-2.0
```

Shell / 配置：

```bash
# Copyright 2026 autumntherain
# SPDX-License-Identifier: Apache-2.0
```

> `SPDX-License-Identifier` 一行就够，机器（GitHub、扫描工具）能直接识别，不必贴全文。

### 5.5 分发时你要履行的四条义务

1. 给接收者一份 `LICENSE` 副本；
2. 改过的文件要**显著标注「已修改」**（改良版尤其注意）；
3. 保留源码中所有版权、专利、商标、归属声明；
4. 若仓库含 `NOTICE`，衍生作品也要带上其中归属声明。

### 5.6 与第三方代码的兼容性

| 引入的代码 | 能否并入 Apache 2.0 仓库 |
|---|---|
| MIT / BSD / ISC | ✅ 可以，保留原版权声明 |
| Apache 2.0 | ✅ 可以 |
| LGPL | ⚠️ 谨慎，动态链接可，静态需评估 |
| GPL / AGPL | ❌ 不兼容，不要合并进主仓库 |

### 5.7 文档与图片可以另用许可

代码采用 Apache 2.0；论文译文、阅读笔记文档如果需要，可单独声明 CC‑BY 4.0，写在`docs/README.md`，**不要将 CC0 应用于源代码**，写在 `docs/README.md` 里即可区分。**别把 CC0 套在代码上**，法律边界不如 Apache 清晰。

---

## 六、发布形态建议

- **仓库**：源码 + 文档（无任何图片资源，≤10MB）
- **Release**：`weights.zip`、`demo_data.zip`，附 SHA256
- **运行时**：`python scripts/download_assets.py` 自动拉 Release 资源并校验哈希
- 对外呈现为「轻仓库 + 按需下载」：clone 快，贡献者不会被大文件劝退

---

## 七、上线前检查清单

- [ ] `.gitignore` 已写好，`data/`、`*.pth`、`runs/` 已挡住
- [ ] `.gitattributes` 已提交，图片标 binary
- [ ] `ASSETS_POLICY.md` 已放置仓库根目录
- [ ] `docs/论文中无插图通告.md` 已就位，README添加引用提示
- [ ] `python tools/check_repo_size.py .` 返回 0
- [ ] `LICENSE` 为 GitHub 生成的 Apache 2.0 全文，未改动
- [ ] `NOTICE` 已写入年份 2026 与 `autumntherain`
- [ ] 所有源码文件顶部有 `Copyright 2026 autumntherain` + SPDX 行
- [ ] 改良版文件已在文件头或 README 标注「modified from original」
- [ ] README 末尾有 License 段落，指向 `LICENSE` 与 `NOTICE`

## License

This project is licensed under the Apache License 2.0.
See the [LICENSE](./LICENSE) and [NOTICE](./NOTICE) files for full license information.
