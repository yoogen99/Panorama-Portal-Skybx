# Panorama-Portal-Skybx
360度パノラマ画像を使う、中に入ったらスカイボックスが変わるシェーダー

VRChatでの仕様を想定した、オブジェクトの中に入ったときだけ 360度パノラマへ景色を切り替えるシェーダーです。

横:縦が **2:1** の equirectangular（緯度経度）パノラマ画像を 1 枚割り当てて使えます。

https://booth.pm/ja/items/2121677 コレの360度パノラマ画像を使えるようにしたバージョンのイメージ
## 特長

- 360 度パノラマ画像 1 枚で動作
- カメラがオブジェクト内にいる間だけ景色を置き換え
- VR の Single-Pass Stereo / Instancing を考慮した左右目対応
- 方向・明るさ・上下反転をマテリアル上で調整可能
- 反復処理を使わない解析的なレイと直方体の交差判定

## 仕組み

Cube などのポータル用オブジェクトにこのシェーダーを設定し、カメラがその内側へ入ると、通常の表面の代わりにスカイボックスのような空間を描画します。

画面の各ピクセルについて、カメラから見ている方向へレイを伸ばし、仮想的な大きい直方体との交点を求めます。その交点の方向を、360 度パノラマ画像の UV 座標へ変換して色を取得します。これにより、2:1 のパノラマ画像 1 枚が空全体を覆うように表示されます。

交点は反復型のレイマーチングではなく、レイと直方体の交差を数式で直接求めています。そのため反復回数の設定が不要で、比較的軽量です。VR では左右それぞれの目のカメラ位置を使うため、立体視でも自然に表示されます。

## 動作環境

- VRChat プロジェクト
- Unity Built-in Render Pipeline
- Shader Model 3.0 以上

URP / HDRP には対応していません。

## 最短セットアップ

Unity の標準 Cube を使う場合は、次の設定で始められます。

| 項目 | 推奨値 |
| --- | --- |
| Cube の Scale | 用途に合わせて変更可 |
| Portal Bounds | `(0.5, 0.5, 0.5)` |
| Virtual Sky Half Size | `50` |
| Render From Outside | Off |
| Panorama Texture Wrap Mode | U: Repeat / V: Clamp |

Cube をワールド内に置き、プレイヤーが内部に入るとパノラマが描画されます。Cube の Scale を変えても、標準 Cube なら `Portal Bounds` は変更不要です。

## マテリアル設定

| 項目 | 内容 |
| --- | --- |
| 360 Panorama | 横:縦 2:1 の equirectangular パノラマ画像です。 |
| Virtual Sky Half Size | 仮想的な空の半サイズです。値を大きくすると、移動時の見え方の変化が穏やかになります。 |
| Panorama Y Rotation | パノラマの水平方向を回転します。正面の位置合わせに使用します。 |
| Exposure | 明るさの倍率です。 |
| Flip Panorama Vertically | 上下が逆に表示される画像だけ有効にします。 |
| Render From Outside | 有効にすると、外側からオブジェクトを見たときもパノラマを描画します。通常は Off のまま使います。 |
| Portal Bounds | 入室判定に使うローカル座標の半サイズです。標準 Cube は `(0.5, 0.5, 0.5)` です。 |

## 注意事項

- 入室判定は軸に平行な直方体です。標準 Cube 以外のメッシュでは、`Portal Bounds` をそのメッシュのローカル寸法に合わせてください。
- ポータル用メッシュは、プレイヤーの視界を閉じる形状にしてください。内部にいる間、メッシュの奥にあるワールドの景色はこのシェーダーで覆われます。
- パノラマの向きが合わない場合は、まず `Panorama Y Rotation` を調整してください。上下だけが逆の場合は `Flip Panorama Vertically` を有効にします。

## フォルダ構成

```text
PanoramaPortalSky/
├─ PanoramaPortalSky.shader
├─ PanoramaPortalSky.mat
└─ README.md
```
