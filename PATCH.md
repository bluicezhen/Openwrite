# 本地补丁说明

本目录是 `bluicezhen/Openwrite` 的 fork，用于本地运行修复版 Studio。

| 项 | 值 |
|---|---|
| 上游仓库 | https://github.com/LiPu-jpg/Openwrite（分支 `native-core`） |
| 对应 issue | https://github.com/LiPu-jpg/Openwrite/issues/162 |
| 分支 | `fix/162-add-after-container` |
| 基线 commit | `d75093d` |

## 改了什么

`tools/outline_tree.py` 中 `add_after` 在容器节点（volume / act / section）上的行为：

1. `expected_kind` 由 `target["kind"]` 改为 `target.get("child_kind")`——在卷上新增同级应得到 `act`，而不是 `volume`；
2. 插入点由「子树末尾」改为「下一个同级标题之前」，退回空行，得到真正的兄弟位置；
3. 章节级 `add_after` 行为保持不变。

新增回归：
- `tests/test_outline_tree.py::test_add_after_inserts_container_sibling_inside_its_parent`
- `tests/test_windows_data_contracts.py::test_container_add_after_keeps_crlf_without_mixing_line_endings`

## 验证

- 旧源码（`d75093d`）：新增 2 条用例 failed
- 本补丁：`tests/test_outline_tree.py` + `tests/test_windows_data_contracts.py` 共 37 passed
- `ruff check` 改动文件：0 新增（既有 7 条历史告警未处理）

## 回滚

`/home/dsh/workspace/openwrite-studio/run-studio.sh` 的 `PYTHONPATH` 已临时指向本目录。
回滚：把该行改回 `${BASE}/core`，`systemctl --user restart openwrite-studio`，然后删除本目录。
