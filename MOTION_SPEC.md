# 動き方データの書式（FCStd ビューワー 用）

FreeCAD の部品に「どの部分が・どの向きに・どれだけ動くか」を書き込んでおくと、
FCStd ビューワーの「開閉」ボタンで、その動きを再現できます。
ビューワーは部品の種類を知らなくても動かせるので、**新しい動く部品（抽斗・扉・引き戸など）を作っても、ビューワーを直す必要はありません。**

## 書き込む場所

| 項目 | 内容 |
|---|---|
| プロパティ名 | `Motion`（この名前で固定） |
| 型 | `App::PropertyString`（文字列） |
| 置く部品 | 形状（Shape）を持つ部品。`Part::Feature`、`Part::FeaturePython` など種類は問わない |
| 表示 | プロパティ欄では隠してよい（`obj.setEditorMode('Motion', 2)`）。隠しても保存される |
| 中身 | 下の JSON（UTF-8。日本語可） |

## JSON の中身

```json
{
  "version": 1,
  "motions": [
    {
      "name": "引出し",
      "type": "slide",
      "axis": [0, -1, 0],
      "min": 0,
      "max": 450,
      "value": 0,
      "faces": [[0, 18, 0], [18, 36, 0.5], [36, 194, 1]]
    },
    {
      "name": "扉",
      "type": "rotate",
      "axis": [0, 0, -1],
      "origin": [0, 0, 0],
      "min": 0,
      "max": 110,
      "value": 0,
      "faces": [[194, 200, 1]]
    }
  ]
}
```

| キー | 必須 | 内容 |
|---|---|---|
| `version` | ○ | 書式の版。今は `1` |
| `motions` | ○ | 動きの一覧。1 つの部品に複数書ける（例：引出しと扉） |
| `name` | | 表示名（任意） |
| `type` | ○ | `"slide"`（まっすぐ動く）または `"rotate"`（軸のまわりに回る） |
| `axis` | ○ | slide：開く向き。rotate：回転軸の向き（右ねじの向き。**正の角度で開く**ように向きを決める）。大きさは問わない |
| `origin` | rotate のみ | 回転軸が通る点（mm） |
| `min` / `max` | ○ | 動かせる範囲。slide は mm、rotate は度。`min` が「閉」、`max` が「全開」 |
| `value` | ○ | **保存したときの値**。形状ファイルはこの状態で保存されていること |
| `faces` | ○ | 動く面の範囲と倍率の一覧 `[開始, 終了, 倍率]` |

### faces（動く面）

- 面の番号は FreeCAD の `obj.Shape.Faces` の順番（0 から数える）。`[開始, 終了]` は「開始以上・終了未満」
- 倍率：値の変化 × 倍率 だけ動く。`1` = そのまま、`0.5` = 半分（スライドレールの中間部材など）、`0` = 動かない
- どの範囲にも入らない面は動かない
- 1 つの面を 2 つの動きに入れたときは、先に書いた方だけが効く（動きの入れ子＝「動く箱の中で動く引出し」はできない）

### 座標

- `axis` と `origin` は **部品自身の座標**（部品の Placement を掛ける前）で書く
- ビューワーが部品の配置（Placement）と、親（App::Part など）の配置を掛けて向きを直すので、部品を回転・移動して置いても正しく動く

## ビューワーでの動かし方

- 開閉できる部品があると、下のツールバーに「開閉」ボタンが出る
- スライダーは「開き具合」0〜100 %。それぞれの動きを `min + 開き具合 × (max − min)` に合わせる
  （動きが全部 slide で範囲が同じなら mm、全部 rotate で同じなら度も表示）
- 「開く」= 100 %、「閉じる」= 0 %

## FreeCAD 側の書き方の例（Python）

部品を組み立てるときに、面の数を数えておけば範囲が分かる。

```python
import json

faces = []            # [開始, 終了, 倍率]
n = 0                 # ここまでの面の数
shapes = []

def add(shape, factor):
    """部品を足す。factor = その部品が動く倍率（動かないなら 0）"""
    global n
    shapes.append(shape)
    k = len(shape.Faces)
    if factor:
        faces.append([n, n + k, factor])
    n += k

# ... add(固定部材, 0) / add(中間部材, 0.5) / add(引出し, 1) ...

obj.Shape = Part.makeCompound(shapes)        # 面の順番 = add した順番
if 'Motion' not in obj.PropertiesList:
    obj.addProperty('App::PropertyString', 'Motion', 'Base', '動き方（ビューワー用）')
    obj.setEditorMode('Motion', 2)
obj.Motion = json.dumps({'version': 1, 'motions': [
    {'name': '引出し', 'type': 'slide', 'axis': [0, -1, 0], 'min': 0, 'max': 450,
     'value': 今の引出し量, 'faces': faces}]}, ensure_ascii=False)
```

注意：`Part.makeCompound` の面の順番は、入れた形の順番のまま。
ただし `fuse`（結合）や `cut`（切り取り）をすると面の番号が変わるので、動く部分と動かない部分は別の形のまま compound にまとめる。

## 古いファイル（Motion が無いもの）

家具CAD の試作版で作ったワイヤーシェルフII・フレームバスケットII は、`Motion` が無くても
ビューワーが部品の種類（`furniture.wire_shelf` / `furniture.frame_basket`）と形の並び順から動きを作る（互換のため）。
新しく作る部品は、必ず `Motion` を書き込むこと。
