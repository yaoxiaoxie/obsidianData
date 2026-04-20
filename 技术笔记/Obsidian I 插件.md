#### 一、图片附件管理插件配置（Custom Attachment Location）

**目标**：实现附件按笔记独立存放，文件名自动规范，引用路径使用相对地址。

| 配置项 | 设定值 | 操作路径 / 说明 |
| :--- | :--- | :--- |
| **新附件位置** | `./assets/${noteFileName}` | 插件设置页 → 此项填入。附件将存放于当前笔记同级 `assets/笔记名/` 文件夹内。 |
| **生成的附件文件名** | `file-${date:YYYYMMDDHHmmssSSS}` | 粘贴图片时自动以 `file-` 加精确到毫秒的时间戳重命名，杜绝重名。 |
| **Markdown URL 格式** | `assets/${noteFileName}/${generatedAttachmentFileName}` | 插入图片时生成的链接格式，确保相对路径正确。 |
| **附件重命名格式** | `全部` | 对所有粘贴附件应用重命名规则。 |
| **是否重命名附件文件夹** | ✅ **勾选** | 允许插件根据 `${noteFileName}` 变量自动创建和重命名文件夹。 |
| **是否重命名附件文件** | ✅ **勾选** | 允许插件自动重命名附件文件。 |

> **配置效果**：在 `我的笔记.md` 中粘贴图片 `screenshot.png`，图片将被保存为 `assets/我的笔记/file-20260120143025123.png`，并在笔记中生成 `![](assets/我的笔记/file-20260120143025123.png)` 引用。

---

#### 二、Obsidian 系统设置：文件与链接规范

**目标**：确保笔记间链接和图片引用均采用标准相对路径，提升库的可移植性和跨编辑器兼容性。

| 设置项 | 推荐配置 | 操作步骤 |
| :--- | :--- | :--- |
| **内部链接类型** | `基于当前笔记的相对路径` | `设置` → `文件与链接` → 下拉选择此项。 |
| **使用 Wiki 链接** | ❌ **取消勾选** | 取消后链接格式为 `[文本](路径.md)`，而非 `[[文本]]`。 |

> **注意**：取消 Wiki 链接后，原有的双链 `[[笔记名]]` 不会自动转换，需手动修改或使用插件批量处理。

---

#### 三、导出增强插件配置（Enhancing Export）

**目标**：配置 Pandoc 引擎路径，使 Obsidian 能通过 Pandoc 导出为 Word、PDF、EPUB 等高级格式。

##### 1. 获取 Pandoc 可执行文件
- 访问 [Pandoc GitHub Releases](https://github.com/jgm/pandoc/releases) 页面。
- 下载对应系统的 **zip 压缩包**（Windows 用户选择 `pandoc-*-windows-x86_64.zip`）。
- 将压缩包解压，找到其中的 `pandoc.exe` 文件。

##### 2. 放置 Pandoc 并记录路径
- 将 `pandoc.exe` 移动到一个固定的、不会随意变动的目录，例如：`D:\Tools\pandoc\pandoc.exe`。
- **复制该文件的完整绝对路径**（包含文件名）。

##### 3. 在 Obsidian 中填写路径
- 打开 Obsidian **设置** → 第三方插件 → **Enhancing Export**。
- 找到 **Pandoc 路径** 设置项。
- 将复制的完整路径粘贴进去，例如 `D:\Tools\pandoc\pandoc.exe`。

##### 4. 验证配置
- 在任意笔记中右键，选择 `Enhancing Export` → `Export to...`，若能正常调出格式选项窗口，则配置成功。

---

### 附录：配置速查表

| 插件/设置 | 关键参数 | 值 |
| :--- | :--- | :--- |
| Custom Attachment Location | 新附件位置 | `./assets/${noteFileName}` |
| Custom Attachment Location | 附件文件名模板 | `file-${date:YYYYMMDDHHmmssSSS}` |
| Obsidian 系统 | 内部链接类型 | 基于当前笔记的相对路径 |
| Obsidian 系统 | 使用 Wiki 链接 | 取消勾选 |
| Enhancing Export | Pandoc 路径 | `D:\Tools\pandoc\pandoc.exe`（示例，替换为实际路径） |
