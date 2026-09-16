# citypdp 数据集

本仓库用于发布 GNSS 多场景定位基准数据集，数据量约数十 GB，以 **分卷压缩包** 的形式通过 [Releases](../../releases) 提供下载。

## 下载

在 Releases 页面下载 `dataset-v1` 的 **全部** 分卷文件，例如：

```
dataset.7z.001
dataset.7z.002
dataset.7z.003
...
```

> 必须下载全部分卷，缺任何一个都无法完整解压。

## 解压

1. 把所有分卷放在 **同一个目录** 下（保持文件名不变）
2. 用 [7-Zip](https://www.7-zip.org/) 打开 `dataset.7z.001`（右键 → 7-Zip → 解压到当前文件夹）
3. 7-Zip 会自动识别后续分卷并还原完整数据

## 说明

- 每个分卷 2000 MiB（GitHub 单附件上限 2 GiB）
- 已排除 `.git`、`.venv`、`.downloads`、`__pycache__` 等非数据内容
- 分卷 SHA-256 校验值见 Release 附件 `checksums.sha256.txt`