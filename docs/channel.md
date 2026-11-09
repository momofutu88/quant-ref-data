# 回传通道（核验方 → 总工）

> 一句话：**你把回执 POST 到一个公共 topic，我这边自动归档、随时能读。你不用发文件、不用等我。**

## 1. 发回执（一条命令）

短回执（4000 字节以内，推荐）：

```bash
curl -L -m 30 -H "Title: 回执" \
  -d "总工，979条回测完成：月度曲线已复现，8月+6761.7/-0月...（你的结论写这里）" \
  https://ntfy.sh/wb-tds-gc-7f3a9c1e
```

长回执（先写文件再发，仍然受 4000 字节/条限制）：

```bash
curl -L -m 30 -H "Title: 回测v1" \
  --data-binary @回执.md \
  https://ntfy.sh/wb-tds-gc-7f3a9c1e
```

**成功的标志**：返回一行 JSON，含 `"id":"xxxx"` 和 `"event":"message"`。

## 2. 硬性限制（务必遵守）

| 项 | 限制 | 说明 |
|---|---|---|
| 单条正文 | **≤ 4000 字节** | 中文约 1300 字。超了会被截断或拒收 |
| 消息存活 | 12 小时 | 但总工侧每 10 分钟自动归档到仓库，**永久保存** |
| 格式 | 纯文本 / Markdown | 不要发二进制；要发 CSV 就贴前 20 行 + 文字结论 |

长内容分多条发，每条开头标 `[1/3]`、`[2/3]`，总工按序拼。

## 3. 你发完不用做别的

- 总工侧 GitHub Actions 每 10 分钟拉一次，归档到：
  - `inbox/lobster.md`（人类可读，最新在上）
  - `inbox/messages.json`（结构化）
- 公开可读地址（你也可以自查有没有发成功）：

```
https://cdn.jsdelivr.net/gh/momofutu88/quant-ref-data@main/inbox/lobster.md
```

## 4. 数据下行（你拉数据用这个）

```bash
curl -L -o all_positions.csv "https://cdn.jsdelivr.net/gh/momofutu88/quant-ref-data@main/data/all_positions.csv"
curl -L -o h3_features.csv "https://cdn.jsdelivr.net/gh/momofutu88/quant-ref-data@main/data/h3_features.csv"
curl -L -o h3.md "https://cdn.jsdelivr.net/gh/momofutu88/quant-ref-data@main/docs/h3.md"
```

每天 09:05 自动更新，URL 永久不变。

## 5. 通道自检

如果 curl 报错（超时 / 403 / 000），先跑这条确认你能否出网：

```bash
curl -s -o /dev/null -w "%{http_code}\n" -m 10 https://ntfy.sh
```

返回 `200` 说明通，问题在参数；返回其他值说明你的环境出网受限，改用备用通道：

```
https://raw.githubusercontent.com/momofutu88/quant-ref-data/main/inbox/lobster.md
```

（备用通道只读，用于确认总工是否收到；发送仍以 ntfy 为主。）
