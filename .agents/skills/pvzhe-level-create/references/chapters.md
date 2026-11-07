# 章节与关卡组织

用于新建章节、同一Mod目录中追加关卡或调整章节结构；模式入口与页面由 [mode-create](../../pvzhe-mode-create/SKILL.md) 负责。发行字段及验证边界见 [levels](../../../../docs/wiki/0.30.0/levels.md)。

0.31的ChoosePacketLimit与MaxSun按[当前差异](../../../../docs/wiki/updates/0.31.0.md#选卡阳光与多格外壳)使用：选卡上限不等于解锁槽数，阳光请求值不等于触顶后的实际增量。旧配置默认仍可沿用，不能为“兼容新版”强行给每个关卡加限制。

0.32把槽数解析收进宿主方法`TowerDefenseLevelSeedBankConfig.ResolveSlotCount(unlockedSlots, mode, limit)`与`TowerDefenseManager.GetPacketSlotNum(config)`；需要读实际槽数时优先复用它们，不自己重算“已解锁与上限取较小值”。差异见[卡槽与选卡](../../../../docs/wiki/updates/0.32.0.md#公开方法的新增可选参数)。

1. 先确认已有项目身份和目录归属。为自己的目录追加章节时保留原有chapterList/levelList、稳定saveKey与顺序；不为一个新章节重建Mod身份，也不覆盖原版或其他Mod目录。确需修改他人目录须另查覆盖契约及进度影响。
2. 新目录使用独立作者键与Content.Identity的运行键；每个关卡独立Name/saveKey，各难度指向相应配置。章节显示名不作为全局唯一身份。自己的导航需要稳定章节ID时另保存明确映射，不以显示文字或易变数组下标当永久键。
3. 按LevelCatalogConfig.chapterList→LevelChapterConfig.levelList→LevelChooseConfig构造。每个可选章节/关卡提供有效图卡；宿主层级固定，不支持把另一个章节对象填进关卡列表。需要更多层级时用自己的页面组织，叶子仍解析到合法关卡身份。
4. 新手可默认全部可进入，除非需求明确要求进度门槛。locked/openKey/preOpen、难度和NextLevel各有不同作用，不能凭字段名假定能完成整套章节解锁或自动续关。要求解锁/续关时另查实际行为并验证跨进程进度，不复制原版saveKey。
5. 在Initialize中一次提供包含全部自有章节的目录，检查ProvideLevelCatalog返回值和清单category=Level；不在每次打开页面时重复注册。JSON资源允许名单列出每个自有文件。

验收至少核对两个不同目标的身份、章节顺序/名称/图片、所选关卡及其实际配置，并检查返回路径。多章节存在不等于每章每关已启动；直接入口与目录入口分别报告。更改已有内容时检查旧saveKey仍能解析，不擅自迁移/删除玩家进度。
