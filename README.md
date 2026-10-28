# Prompt-Libraries

面向图像生成(NovelAI / Stable Diffusion / Illustrious 等)的**提示词资料库(一般向)**:整理过的资料索引、中文提示词库、ShuaiHui 条目与 Danbooru 标签中英对照。
所有数据以 JSONL.gz 分片发布,并附一键重建 SQLite(含 FTS5 全文索引)的脚本。

> **许可:许可待定**(由仓库所有者决定,见 `LICENSES.md`)。第三方来源保留各自许可,见 `NOTICE.md`。
> 成人向内容在姊妹仓库 [Prompt-Libraries-NSFW](https://github.com/YuaaaaaSatori/Prompt-Libraries-NSFW)(仅限成年人)。

## 目录
```
data/
  sources.jsonl.gz            来源登记(来源名/链接/许可/版本)
  general/
    categories / subcategories      大类、子分类
    collected_refs                  整理的资料(摘要、短提示词、来源链接)
    prompts_n714                    n714 中文提示词库(MIT)
    entries_shuaihui                ShuaiHui 仓库条目(许可未知, 仅供参考)
    tags_ffdkj_NN / tags_jio7_NN    Danbooru 标签(中英对照 / 分类)
    tags_school_benign              校园类标签(良性部分, 见下方"范围与过滤")
    prompts_n714_school_benign      n714 校园类词条(良性部分)
scripts/build_db.py           从 JSONL.gz 重建 SQLite + FTS5
docs/
  subcategories.md            子分类说明
  collected_refs.md           资料索引(按大类/子分类)
  MANIFEST.md                 每个数据文件的大小与 sha256
LICENSES.md  NOTICE.md
```
每个数据文件 < 50MB。

## 重建数据库
只需 Python 3 标准库(SQLite ≥ 3.34,支持 FTS5 trigram):
```
python3 scripts/build_db.py                      # -> build/prompt_libraries.db
python3 scripts/build_db.py --out /tmp/x.db --no-tags-fts
```
视图 `v_refs` 附带子分类名与大类名。
FTS5 trigram 分词要求查询词**至少 3 个字符**(如 `光影氛` 可查,`光影` 不行);更短的词请用 `LIKE`。
```sql
select id,topic,source_url from collected_refs_fts join collected_refs r on r.id=collected_refs_fts.rowid where collected_refs_fts match 'lighting';
```

## 数据说明
- **collected_refs**:整理者对公开教程/文章/仓库/实测的摘要与短提示词短语,含 `subcat`(主子分类)、`subcat_secondary`、`models`、`source_url`、`license_label`。
- **prompts_n714**:n714 提示词库的条目与说明,MIT。
- **entries_shuaihui**:ShuaiHui 仓库的条目;**许可未知,仅供参考,原作者要求即下架**。
- **tags_***:Danbooru 标签表,一行一个 (标签, 来源)。
- 字段为 UTF-8 JSON,一行一条记录。

## 范围与过滤
收录前已用规则脚本逐条复查,命中者已剔除;过滤清单不公开。

**校园类词条(诚实说明)**:`tags_school_benign`(标签)与 `prompts_n714_school_benign`(n714 词条)是此前因"校园类"标记而暂缓公开、
后经二次筛选放出的校园/学校/校服类词条。规则(修订版):不含性相关词、擦边/内衣类词、其他类标记的,放出;其中含学段词(高中/中学/初中)、
书包(randoseru)、洛丽塔时装、little idol 的,只要不含性/擦边/年龄指向词也放出。仍剔除:shota/loli 类低龄角色词、儿童/幼儿/未成年/小学/幼儿园等年龄指向词、
性相关词、擦边词、命中其他类标记的。所以这部分是"校园类里不含性与低龄指向的那一部分",并非全部校园类词条;被剔除的不公开、不列清单。
成人向的校园类条目不在本仓库发布。
