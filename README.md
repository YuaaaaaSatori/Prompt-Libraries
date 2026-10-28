# Prompt-Libraries

面向图像生成(NovelAI / Stable Diffusion / Illustrious 等)的**提示词资料库**:整理过的资料索引、中文提示词库、ShuaiHui 条目与 Danbooru 标签中英对照。
所有数据以 JSONL.gz 分片发布,并附一键重建 SQLite(含 FTS5 全文索引)的脚本。

> **许可:许可待定**(由仓库所有者决定,见 `LICENSES.md`)。第三方来源保留各自许可,见 `NOTICE.md`。
> **成人向内容仅限成年人**:`data/nsfw/` 含成人向内容,请自行确认所在地法律并只在成年人范围内使用。

## 目录
```
data/
  sources.jsonl.gz            来源登记(来源名/链接/许可/版本)
  general/                    一般向
    categories / subcategories      大类、子分类
    collected_refs                  整理的资料(摘要、短提示词、来源链接)
    prompts_n714                    n714 中文提示词库(MIT)
    entries_shuaihui                ShuaiHui 仓库条目(许可未知, 仅供参考)
    tags_ffdkj_NN / tags_jio7_NN    Danbooru 标签(中英对照 / 分类)
  nsfw/                       成人向(同样的文件划分; 类别与标签只含成人向部分)
scripts/build_db.py           从 JSONL.gz 重建 SQLite + FTS5
docs/
  subcategories.md            子分类说明
  collected_refs.md           资料索引(一般向,按大类/子分类)
  collected_refs_nsfw.md      资料索引(成人向)
  MANIFEST.md                 每个数据文件的大小与 sha256
LICENSES.md  NOTICE.md
```
每个数据文件 < 50MB,整个 `data/` 约 9MB。

## 重建数据库
只需 Python 3 标准库(SQLite ≥ 3.34,支持 FTS5 trigram):
```
python3 scripts/build_db.py                      # 全部 -> build/prompt_libraries.db
python3 scripts/build_db.py --only general       # 仅一般向
python3 scripts/build_db.py --out /tmp/x.db --no-tags-fts
```
所有表带 `nsfw` 列(0=一般向目录,1=成人向目录);视图 `v_refs` 附带子分类名与大类名。
FTS5 trigram 分词要求查询词**至少 3 个字符**(如 `光影氛` 可查,`光影` 不行);更短的词请用 `LIKE`。
```sql
select id,topic,source_url from collected_refs_fts join collected_refs r on r.id=collected_refs_fts.rowid where collected_refs_fts match 'lighting';
```

## 数据说明
- **collected_refs**:整理者对公开教程/文章/仓库/实测的摘要与短提示词短语,含 `subcat`(主子分类)、`subcat_secondary`、`models`、`source_url`、`license_label`。
- **prompts_n714**:n714 提示词库的条目与说明,MIT。
- **entries_shuaihui**:ShuaiHui 仓库的条目;**许可未知,仅供参考,原作者要求即下架**。
- **tags_***:Danbooru 标签表,一行一个 (标签, 来源);同一标签若在任一来源被判为成人向,则只出现在 `nsfw/`。
- 字段为 UTF-8 JSON,一行一条记录。

## 范围与过滤
已剔除:含未成年指向、非自愿、偷拍、乱伦、人兽、真人相关的内容,以及规则复查命中的条目;过滤清单不公开。
