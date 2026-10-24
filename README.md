# Orb Transform Library (OTL) — Ver. 1.5

OTL は、プラネタリウム投影機用の恒星原板（星の穴を開けるための版下）を SVG / 印刷用 PDF で生成する Python 製ツールです。星表の位置を、ドーム・投影機・レンズの設定に合わせて各原板上の座標へ変換します。星座線を構成する星の拡大や、各原板が担当する星域の出力にも対応しています。

## セットアップ

Python と Git LFS を用意し、このディレクトリで次を実行してください。

```bash
git lfs install
git lfs pull
python3 -m pip install -r requirements.txt
```

カタログファイル `hip_main.dat`、`tyc_main.dat`、`rc3.dat`、`IAU88.hlc` は、スクリプトと同じディレクトリに配置します。`tyc_main.dat` は Git LFS で取得します。

## 使い方

### 対話形式で実行する

```bash
python3 transformer.py
```

表示される質問に従って、投影機の寸法や星の明るさなどを入力します。出力形式は `star_SVG`、`star_pdf`、`polygon_SVG`、`polygon_pdf` から複数選択でき、未入力なら `star_SVG` のみを出力します。

### 設定ファイルで実行する

[starconfig.properties](starconfig.properties) を投影機に合わせて編集し、実行します。

```bash
# 恒星原板を SVG で出力
python3 transformer.py -f starconfig.properties

# PDF で出力
python3 transformer.py -PDF -f starconfig.properties

# 担当星域ポリゴンも出力（-PDF と併用可能）
python3 transformer.py --polygons -f starconfig.properties

# ヘルプを表示
python3 transformer.py -h
```

### 主な設定

寸法の単位は mm です。その他の設定やユニット配置の個別指定は、[設定ファイルのコメント](starconfig.properties)を参照してください。

| 設定 | 内容 |
|---|---|
| `dome-radius` | ドームの半径 |
| `projector-distance` / `plate-distance` | ドーム中心から投影機中心まで / 投影機中心から原板中心までの距離 |
| `focal-length` | レンズの焦点距離 |
| `unit-12` / `unit-20` / `unit-30` / `unit-60` / `unit-extra` | 使用するユニット配置を `yes` / `no` で選択 |
| `star-radius` | 0 等星の穴の半径 |
| `star.maximum` / `star.minimum` | 最輝星 / 最微星の等級 |
| `star.excluding` | 除外する星の HIP 番号 |
| `plate.column` / `plate.row` | 用紙上の原板の配置数（横 / 縦） |
| `plate.frame-radius` | 円形原板の枠半径。枠外に中心がある星穴は出力しない。`0` で自動計算 |
| `color.invert` | `yes`: 黒地に白い星（エッチング）、`no`: 白地に黒い星（リスフィルム） |
| `output.directory` / `file.prefix` | 出力先フォルダ / 恒星原板のファイル名の接頭辞 |

### 暗い恒星・銀河を追加する

次の設定は独立して切り替えられます。どちらも省略時は `no` です。

```properties
# 最微星より暗い恒星を確率的に採用する
star.under-minimum = yes

# RC3 カタログの銀河を点の集合として描く
rc3.enabled = yes
```

いずれも `star.minimum` の等級の点として描画します。同梱の設定ファイルでは暗い恒星は無効、銀河は有効です。旧設定で `star.under-minimum` により銀河も描いていた場合は、`rc3.enabled = yes` を追加してください。

### 星座線を構成する星を拡大する

```properties
star.enlarge-rate = 1.0
star.enlarge-constellations = Orion, Canis Major
```

- `star.enlarge-rate`: `0`〜`2`。`0` で拡大なし、`1.0` で 4 等星相当の穴が 1 等級分大きくなります。明るい星ほど拡大効果を抑えます。
- `star.enlarge-constellations`: `IAU88.hlc` の `Name:` にある星座名をカンマ区切りで指定します。空欄なら全星座が対象です。

対話モードでは、対象の星座名を 1 行に 1 つ記載した UTF-8 テキストファイルのパスを入力します。空欄なら全星座が対象です。

### 担当星域を出力する

`--polygons` を付けるか、設定ファイルで `polygon.enabled = yes` にします。各原板の担当領域を、黒い原板・白い領域で別ファイルに出力します。

`polygon.samples` で境界の頂点数（12 以上、既定値 180）、`polygon.file-prefix` でファイル名の接頭辞を指定できます。

## 出力先・印刷

生成物は `output.directory` 内の次のサブフォルダに保存されます。未指定時は `output`、同梱の設定ファイルでは `output_inverse` です。

| サブフォルダ | 内容 |
|---|---|
| `star_SVG` / `star_pdf` | 恒星原板の SVG / PDF |
| `polygon_SVG` / `polygon_pdf` | 担当星域の SVG / PDF |
| `unit_position` | ユニット配置画像（`unit-position-file` 指定時） |

用紙サイズは A4（210 × 297 mm）です。**印刷時は「用紙に合わせる」を無効にし、100%（実寸）で出力してください。** 原板の左上の枠外には位置確認用の黒い × 印が付きます。旧 `-PS` オプションは使用できません。

## 関連ツール・ライセンス

天の川原板の生成は GingaForegeに移管しました。

本ツールは東京大学地文研究会天文部の Java 版 OTL（Ver. 1.4）をもとにした Python 版です。機能追加・改良を行っており、元の Java 版との完全な動作一致は保証しません。

ライセンスは GNU General Public License v2 (or later) です。詳細は各ソースファイル冒頭のヘッダーを参照してください。

Copyright (C) 2007, 2012-2014, 2026 東京大学地文研究会天文部
