# 設定の一覧

OTL のすべての設定をまとめます。計算の意味は [仕組み](mechanism.md) を参照してください。

## 設定ファイルの書き方

設定ファイルは Java のプロパティファイルと同じ形式です。`python3 -m otl -f <ファイル>` で指定し、省略すると [`config/starconfig.properties`](../config/starconfig.properties) を読みます。

```properties
# '#' または '!' で始まる行はコメント
dome-radius = 6500
star.enlarge-constellations = Orion, Canis Major
```

- `キー = 値` または `キー : 値` の形で 1 行に 1 つ書きます。
- 寸法の単位はすべて mm です。
- `yes` / `no` の設定は `y`・`true`・`n`・`false` も使えます。
- 書かなかった設定や読み取れない値は、下の表の **省略時** の値になります。省略時の値は同梱の設定ファイルの値と違うものがあるので注意してください。

## 投影機の寸法

| 設定 | 内容 | 省略時 | 同梱 |
|---|---|---:|---:|
| `dome-radius` | ドームの半径 | `5000` | `6500` |
| `projector-distance` | ドームの中心から投影機（北天用・南天用それぞれ）の中心までの距離 | `300` | `355` |
| `plate-distance` | 投影機の中心から原板の中心までの距離 | `200` | `400` |
| `focal-length` | レンズの焦点距離 | `50` | `50` |
| `scale` | 拡大して描く倍率。寸法をすべて `scale` 倍して計算する | `1` | `1` |

`scale` は、原板を大きく描いて写真で縮小する場合に使います。`dome-radius`・`projector-distance`・`plate-distance`・`focal-length` が `scale` 倍になり、原板の星の位置も `scale` 倍に広がります。`star-radius` は倍率をかけないので、描く大きさ（縮小前）の値で指定します。

## ユニットの配置

| 設定 | 内容 | 省略時 | 同梱 |
|---|---|---:|---:|
| `unit-12` | 正十二面体の面の方向（12 か所）にユニットを置く | `yes` | `yes` |
| `unit-20` | 正十二面体の頂点の方向（20 か所）にユニットを置く | `no` | `yes` |
| `unit-30` | 正十二面体の辺の方向（30 か所）にユニットを置く | `yes` | `no` |
| `unit-60` | 正十二面体の対称性をもつ 60 か所にユニットを置く | `yes` | `no` |
| `unit-extra` | 赤道付近にユニットを追加する（下の注意を参照） | `yes` | `no` |
| `unit.custom` | `yes` でユニットの向きを個別に指定する（上の 5 つは無視） | `no` | — |
| `unit-position-file` | ユニットの配置図の画像ファイル名。書かなければ出力しない | — | — |

組み合わせとユニット数・画角の関係は [仕組み](mechanism.md#組み合わせと画角) を参照してください。

> [!WARNING]
> `unit-extra = yes` では、追加分のユニットの向きが正しく計算されません（[仕組み](mechanism.md#2-ユニットの配置)）。省略時の値が `yes` なので、`unit-` の設定を書くときは `unit-extra = no` も明記してください。

### ユニットの個別指定

`unit.custom = yes` にすると、`units[0]` から連番でユニットの向きを指定できます。番号が途切れたところで読み込みを終えます。向きは次の 4 通りのいずれかで書きます。

```properties
unit.custom = yes

# 1. 直交座標 (x, y, z) を 1 行で
units[0].xyz = 0 0 1

# 2. 直交座標を個別に
units[1].x = 1
units[1].y = 0
units[1].z = 0

# 3. 赤経 [h] と赤緯 [°] を 1 行で
units[2].alpha_delta = 6 45

# 4. 赤経と赤緯を個別に
units[3].alpha = 18
units[3].delta = -45
units[3].hemisphere = south
```

- 直交座標は z 軸が天の北極、x 軸が赤経 0h の方向です。**長さ 1 の単位ベクトル** で書いてください（長さを正規化せずに計算に使います）。
- 北天用・南天用のどちらに付けるかは、z（赤緯）の符号で決まります。`hemisphere = north`（`n`）/ `south`（`s`）で明示することもできます。

## 星の選別と穴の大きさ

| 設定 | 内容 | 省略時 | 同梱 |
|---|---|---:|---:|
| `star-radius` | 0 等星の穴の半径 | `0.25` | `0.275` |
| `star.maximum` | 最輝星の等級。これ以下の等級の星は次の設定に従う。`-Infinity` で除外しない | `1.5` | `1.5` |
| `star.above-maximum` | 最輝星より明るい星を、最輝星の大きさで描く。`no` なら描かない | `yes` | `no` |
| `star.minimum` | 最微星の等級 | `7.5` | `7.5` |
| `star.under-minimum` | 最微星より暗い星を、確率的に最微星の大きさで描く | `no` | `no` |
| `rc3.enabled` | RC3 の系外銀河を最微星の点の集まりで描く | `no` | `yes` |
| `star.excluding` | 描かない Hipparcos の星の HIP 番号（**スペース区切り**） | — | — |

```properties
# HIP 番号をスペースで区切って並べる
star.excluding = 24608 69673
```

HIP 番号は [SIMBAD](https://simbad.cds.unistra.fr/simbad/) などで星の名前から調べられます。

## 星座線を構成する星の拡大

| 設定 | 内容 | 省略時 | 同梱 |
|---|---|---:|---:|
| `star.enlarge-rate` | 拡大率（`0`〜`2`）。`1` 増えるごとに、4 等星より暗い星の穴が 1 等級分大きくなる。`0` で拡大しない | `0` | `1.0` |
| `star.enlarge-constellations` | 拡大する星座。`IAU88.hlc` の `Name:` の値をカンマ区切りで書く。空欄で全星座 | 全星座 | 15 星座 |

Java 版のキー `star-EnlargeRate`・`star-EnlargeConstellations` も読めます。

同梱の設定で拡大する星座は次の 15 個です。

```properties
star.enlarge-constellations = Cepheus, Cassiopeia, Andromeda, Cetus, Perseus, Pegasus, Lyra, Cygnus, Aquila, Orion, Taurus, Auriga, Canis Minor, Canis Major, Gemini
```

対話形式（`-i`）では、星座名を 1 行に 1 つ書いた UTF-8 のテキストファイル（例: [`config/constellations.txt`](../config/constellations.txt)）のパスを入力します。カンマ区切りや `#` で始まるコメント行も使えます。

## 原板と用紙

| 設定 | 内容 | 省略時 | 同梱 |
|---|---|---:|---:|
| `plate.column` | 用紙の横に並べる原板の数 | `1` | `1` |
| `plate.row` | 用紙の上半分（下半分）に縦に並べる原板の数 | `1` | `1` |
| `plate.frame-radius` | 円形の原板の枠の半径。枠の外に中心がある星は描かない。`0` で自動計算 | `0` | `38.15` |
| `color.invert` | `yes`: 黒地に白い星（エッチング）、`no`: 白地に黒い星（リスフィルム） | `no` | `no` |

用紙への並べ方は [全体の構成](overview.md#用紙への配置) を参照してください。

## 出力

| 設定 | 内容 | 省略時 | 同梱 |
|---|---|---:|---:|
| `output.directory` | 出力先のフォルダ。中に `star_SVG/` などを作る | `output/default` | `output/inverse` |
| `file.prefix` | 恒星原板のファイル名の接頭辞 | `star-` | `star-` |
| `polygon.enabled` | 担当星域ポリゴンも出力する（`--polygons` でも有効になる） | `no` | `no` |
| `polygon.samples` | ポリゴンの頂点数（12 以上）。大きいほど境界がなめらか | `180` | `180` |
| `polygon.file-prefix` | 担当星域のファイル名の接頭辞 | `polygon-` | `polygon-` |

## コマンドラインオプション

| オプション | 内容 |
|---|---|
| `-f <ファイル>` | 設定ファイルを指定する |
| `-PDF` | SVG の代わりに PDF で出力する（`--pdf` も可） |
| `--polygons` | 担当星域ポリゴンも出力する |
| `-i` | 設定ファイルを使わず、質問に答えて設定する（`--interactive` も可） |
| `-h` | ヘルプを表示する（`-help`・`--help` も可） |

旧 `-PS`（PostScript 出力）は廃止しました。`-PDF` を使ってください。

### 対話形式の省略時の値

`-i` で何も入力せずに Enter を押したときの値は、設定ファイルの省略時の値とおおむね同じです。ただしユニットの配置は、`unit-12`・`unit-30`・`unit-extra` が `yes`、`unit-20`・`unit-60` が `no` になります。出力形式は `star_SVG`・`star_pdf`・`polygon_SVG`・`polygon_pdf` から選べ、未入力なら `star_SVG` だけを出力します。
