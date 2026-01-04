我将先建立一个简短的待办清单来跟踪分析工作，然后抓取仓库页面内容以核对实际支持的 WiFi 固件/驱动配置符号，从而判断当前 workflow 中 WIFI_DRIVER 选项是否存在冗余或无效。接下来会给出结论与精简建议。

已创建 4 个待办事项

正在使用 "提取网页"

已提取 https://github.com/Yuzhii0718/immortalwrt-mt798x-hanwckf

正在使用 "提取网页"

已提取 https://github.com/Yuzhii0718/immortalwrt-mt798x-hanwckf/blob/openwrt-21.02/defconfig/mt7981-ax3000.config

正在使用 "提取网页"

已提取 https://github.com/Yuzhii0718/immortalwrt-mt798x-hanwckf/raw/refs/heads/openwrt-21.02/defconfig/mt7981-ax3000.config

正在使用 "提取网页"

已提取 https://raw.githubusercontent.com/Yuzhii0718/immortalwrt-mt798x-hanwckf/refs/heads/openwrt-21.02/defconfig/mt7981-ax3000.config

正在使用 "提取网页"

已提取 https://github.com/Yuzhii0718/immortalwrt-mt798x-hanwckf/search?q=CONFIG_MTK_MT_WIFI_MT7981_20240823

正在使用 "提取网页"

已提取 https://github.com/search?q=repo:Yuzhii0718/immortalwrt-mt798x-hanwckf+CONFIG_MTK_MT_WIFI_MT7981_20240823&type=code

正在使用 "提取网页"

已提取 https://github.com/Yuzhii0718/immortalwrt-mt798x-hanwckf/tree/openwrt-21.02/package

正在使用 "提取网页"

已提取 https://github.com/Yuzhii0718/immortalwrt-mt798x-hanwckf/tree/openwrt-21.02/package/mtk

正在使用 "提取网页"

已提取 https://github.com/Yuzhii0718/immortalwrt-mt798x-hanwckf/tree/openwrt-21.02/package/mtk/drivers

正在使用 "提取网页"

已提取 https://github.com/Yuzhii0718/immortalwrt-mt798x-hanwckf/tree/openwrt-21.02/package/mtk/drivers/mt_wifi

正在使用 "提取网页"

已提取 https://github.com/Yuzhii0718/immortalwrt-mt798x-hanwckf/blob/openwrt-21.02/package/mtk/drivers/mt_wifi/config.in

正在使用 "提取网页"

已提取 https://github.com/Yuzhii0718/immortalwrt-mt798x-hanwckf/raw/refs/heads/openwrt-21.02/package/mtk/drivers/mt_wifi/config.in

正在使用 "提取网页"

已提取 https://raw.githubusercontent.com/Yuzhii0718/immortalwrt-mt798x-hanwckf/refs/heads/openwrt-21.02/package/mtk/drivers/mt_wifi/config.in

正在启动: *比对 workflow 选项与仓库支持项* (3/4)

已完成: *比对 workflow 选项与仓库支持项* (3/4)

下面是我将如何判断这些 WiFi 驱动选项是否“多余”的简要路线：我已抓取并核对了当前使用的源码仓库配置，确认哪些固件日期和驱动版本在源码中真实存在与受支持，并据此评估 workflow 中的 WIFI_DRIVER 列表是否包含过时或无效项，同时指出导致这些选项“名义可选但实际不生效”的脚本问题。

## 结论与核对结果

- 源码仓库位置：`https://github.com/Yuzhii0718/immortalwrt-mt798x-hanwckf`（分支 `openwrt-21.02`）
- 已核对关键文件：
  - `defconfig/mt7981-ax3000.config`（原始内容显示）
    - 存在 `CONFIG_MTK_MT_WIFI_DRIVER_VERSION_7673=y`
    - 存在 `CONFIG_MTK_MT_WIFI_MT7981_DEFAULT_FIRMWARE=y`
  - `package/mtk/drivers/mt_wifi/config.in`（原始内容显示）
    - 驱动版本 choice 仅包含：
      - `CONFIG_MTK_MT_WIFI_DRIVER_VERSION_7661`（7.6.6.1）
      - `CONFIG_MTK_MT_WIFI_DRIVER_VERSION_7673`（7.6.7.3）
    - MT7981 固件 choice 明确支持：
      - `DEFAULT_FIRMWARE`
      - `20230306`, `20230330`, `20230411`, `20230717`, `20231024`, `20240823`

据此可以得出：

1. workflow 中所有 “v7.6.7.2-...” 选项在当前仓库已不再对应任何配置符号（源码已升级到 `7.6.7.3`）。因此这类选项是过时/无效的，应当替换为 “v7.6.7.3-...” 或直接移除 7.6.7.2 的选项集合。
2. MT7981 的固件日期在源码中都是真实存在的配置项（包括 `default` 与所有列出的日期值），这些并非冗余。从功能上看，保留各日期是有意义的。
3. 但当前 workflow 的“设置WiFi驱动版本”脚本存在两个致命问题，导致这些选项“理论可选、实际不生效”，这会让人感觉选项“多余”：
   - 驱动版本切换逻辑只将 `7672` 替换为 `7661`，而仓库默认是 `7673`，因此选择 `v7.6.6.1` 时并不会将 `7673` 切换为 `7661`，造成选项无效。
   - 固件切换的 sed 使用了单引号 `'...'` 包住 `${firmware}`，导致变量不展开；同时它只替换 `CONFIG_MTK_MT_WIFI_MT7981_20240823=y` 这一硬编码行，如果 `.config` 当前是 `DEFAULT_FIRMWARE`（仓库里就出现这种情形），替换也找不到目标行，从而不生效。

因此，严格来说，功能层面这些固件日期选项并不冗余，但在现有脚本实现下，很多选项会“不起作用”，从结果上看等于冗余。

## 精简与修正建议

- 删除或更新下列过时选项：
  - 移除全部 “v7.6.7.2-...” 项，改为 “v7.6.7.3-...” 对应当前仓库的驱动版本。
- 保留固件日期选项（它们是有效的配置），但建议在 UI 中标注少量“推荐值”，例如：
  - 7.6.7.3：`default`、`20240823`（默认与最新）
  - 7.6.6.1（如确需兼容旧版）：`20230306`（已有“recommend”标签）、`default`
  - 其他日期可作为“高级选项”保留，便于回退或做兼容测试。
- 修正脚本以保证选项“真正生效”：
  - 驱动版本替换应兼容 7673 当前值，例如先统一禁用所有版本符号，再启用目标版本，或至少将 `7673` 切换到 `7661`：
    - 示例思路：把 `.config` 中 `CONFIG_MTK_MT_WIFI_DRIVER_VERSION_7661` 与 `CONFIG_MTK_MT_WIFI_DRIVER_VERSION_7673` 都置为 `#` 注释/`n`，再根据用户选择写入目标 `=y`。更稳妥的方式是使用 `scripts/config`（OpenWrt 的 `kconfig` 工具）进行启用/禁用，而不是简单 sed。
  - 固件切换不要硬编码源符号行，也不要用单引号阻止变量展开：
    - 用双引号保证 `${firmware}` 被展开。
    - 先关闭所有 `CONFIG_MTK_MT_WIFI_MT7981_*` 条目，再启用目标固件符号；或者使用 `scripts/config` 进行菜单选择设置。

如果你需要，我可以直接在 `HY3000-hanwckf.yml` 中修正“设置WiFi驱动版本”这一步的脚本，让这些选项即刻生效，并同步把 7.6.7.2 的选项更新为 7.6.7.3。

## 轻量待办清单

- [已完成] 抓取仓库与关键文件内容
- [已完成] 确认驱动与固件配置符号的真实支持列表
- [已完成] 比对 workflow 选项，识别过时与失效项
- [已完成] 输出精简与修正建议

需要我继续把脚本修正好并替换掉 7.6.7.2 的选项吗？我可以直接为你改好并测试。