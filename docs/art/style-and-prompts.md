# ソダテルモン 画風・プロンプト仕様 v1

この文書は生成入力を再現可能にするための仕様であり、生成実施記録ではない。個別の名前/IDは管理用で、画像上へ描かない。

## 1. 共通画風

- 親しみやすさと生物らしさを両立した、手描き風の2Dゲーム用ファンタジー生物
- 太さに変化がある整理された濃色輪郭、最大3段階程度の面で立体を示すセル塗り、控えめな素材表現
- 背景/台座/落ち影/発光halo/文字/装飾枠なし。1体の全身、余白込みで切れずに見える
- briefの生息地・岩・水面・行動背景は、性質や身体の姿勢を理解するための文脈だけに使う。それらの環境・物体を画像に描かない
- 光源は左上、柔らかな明部。ギラギラした3D renderや写真調へ振らない
- 斜め前の3/4 view、軽い見下ろし、既定の顔の向きは画面右。個別poseに正面/上面などの視点が明示される場合はそちらを優先し、通常/pixelで一致させる。個体の左右非対称特徴は物理的左右を保持する
- 人物・ヒト型、流血、実在の紋章、宗教記号、文字やロゴ、武器を持つ演出は追加しない
- 衣服や道具で個性を補わず、カタログ内の身体構造と素材で差を出す
- 目0個/無顔指定の個体に目や顔を追加しない。性質は表情の代わりに姿勢や動勢で示す

## 2. 個体入力の展開規則

`catalog.json` の各レコードから以下を欠落なく取り出す。

- `id`, `name_ja`, `family`, `family_label_ja`
- `silhouette`（部位数も含めて全て不変）
- `material_motif`
- `palette_hex` の3色（順に主色・副色・accent）
- `signature_features` の3点
- `pose`, `temperament`, `do_not_add`

`signature_features` は必須条件であり「できれば付ける小物」ではない。画面に隠れる特徴があるなら、部位を消さず向き/poseの範囲内で見えるようにする。体の部位数とsignatureが矛盾した場合は生成前にbriefを修正し、黙って片方を無視しない。

下記テンプレート中の `{FIELD}` は同名フィールドの全文へ、配列は `; ` 区切りで展開する。promptにはテンプレート文字のまま残さない。日本語の個体briefを翻訳する必要はない。翻訳した場合も数と特徴を変えない。後続作業では、使用したテンプレートversion、展開済み全文、入力参照hashをmetaへ記録する。

## 3. 通常画像 共通テンプレート `default-v1`

```text
Use case: stylized-concept
Asset type: one original creature portrait sprite for a 2D creature-raising game
Primary request: Create exactly one full-body creature following the complete independent brief below. This is a new design; do not base it on any existing character.
Management only, do not render text: {id} / {name_ja} / {family_label_ja} ({family})
Body and exact anatomy: {silhouette}
Materials and biological motif: {material_motif}
Primary, secondary, accent colors respectively: {palette_hex}
Three mandatory visual identifiers: {signature_features}
Pose: {pose}
Temperament to convey through expression or posture: {temperament}. If the brief specifies zero eyes or no face, do not invent eyes or facial features; express temperament through posture only.
Style: cohesive hand-drawn 2D fantasy game illustration, clear dark colored contour, restrained three-step cel shading, soft upper-left lighting, appealing natural creature proportions, clean readable form.
Composition: isolated single creature, full body; by default use a slight top-down three-quarter view facing toward image-right. If the individual pose explicitly requires a front or top view, that explicit pose takes priority. Preserve the specified pose and physical left/right features. Centered in a square canvas, no cropping. Keep transparent padding on all sides. Intended final canvas is 1024 by 1024 pixels, with the longest creature extent about 768–896 pixels and at least 64 pixels of clear padding.
Background: genuine transparent alpha, no painted checkerboard, no backdrop, no ground plane, no cast shadow, no diffuse glow. Habitat, rocks, water surfaces and action scenery mentioned in the brief describe only body posture or temperament; do not render those objects or environments.
Constraints: preserve exact anatomy, all three identifiers and the position of the palette colors; no costume or held tool unless explicitly part of the brief.
Individual forbidden changes: {do_not_add}
Avoid: extra creatures, humans, text, letters, labels, logos, watermarks, decorative frame, gradients bleeding into the transparent area, photorealism, 3D render, imitation of an existing character.
```

工具の透明背景オプションがある場合はtrueにする。`1024 by 1024` は完成目標であり、工具APIの未確認引数を作る指示ではない。実際の出力寸法は必ず検査する。

## 4. ドット画像 共通テンプレート `pixel-v1`

先に合格通常絵を実pixelで閲覧し、その画像を入力参照として添付する。画像名だけで参照を代用しない。通常絵のhashを固定してから生成する。

```text
Use case: style-transfer
Asset type: one authentic pixel-art creature sprite, a paired rendition of the approved original illustration
Input image 1: the approved default illustration for {id}; it is the identity reference and must remain the same creature.
Primary request: Redraw this exact creature as carefully constructed pixel art. Change only the rendering medium. Do not invent a replacement design and do not merely downsample or apply a pixelation filter to the input.
Management only, do not render text: {id} / {name_ja} / {family_label_ja} ({family})
Locked body and anatomy: {silhouette}
Locked material cues: {material_motif}
Locked palette roles, primary/secondary/accent: {palette_hex}
All three identifiers must remain recognizable: {signature_features}
Locked pose: {pose}
Locked temperament/expression: {temperament}. Preserve eyeless or faceless designs without adding eyes or a face; for those designs use posture to convey temperament.
Rendering: an intentional 128 by 128 logical pixel grid, readable pixel clusters, controlled staircase contours, consistent 1–2 logical pixel outline, limited palette no more than 32 visible colors, hard-edged opaque pixels and transparent background pixels only. Preserve exact counts of eyes, limbs, wings, tails and other declared structures. Simplify surface texture, never delete identity features or invent anatomy.
Composition: match the reference's facing, pose, relative color placement and body proportions. Full body, centered, longest creature extent about 96–112 logical pixels, at least 8 logical pixels of transparent padding on every side. No cropped body parts.
Background: genuine transparent alpha, no drawn checkerboard, no backdrop, no shadow, no glow spilling out of the silhouette. Do not render any habitat, rock, water surface or action scenery mentioned in the brief; depict only the creature and its posture.
Individual forbidden changes: {do_not_add}
Avoid: antialiasing, blur, smooth gradients, semitransparent pixels, inconsistent pixel sizes, painterly miniatures, pixelation overlays, extra characters, text, letters, logos, watermarks, decorative frame, redesign.
```

生成工具が高解像度で返す場合でも、128の論理グリッドと整数倍率を明示して再生成する。整数gridとして成立しない候補を「pixelらしいから」で採用しない。仕上げ処理の条件は [制作計画](production-plan.md#工具の出力と完成契約を分ける) に従う。

## 5. 同一個体を保つ照合順

1. 輪郭: 高さ/幅、頭胴比、体節、空洞、左右非対称の方向
2. 部位数: 目、肢、翼、尾、触角、突起など記載の数
3. signature_featuresの3点: 形、位置、向き
4. 色: 主色/副色/accentの相対配置。shade追加は許すが全体配色を交換しない
5. pose/表情: 同じ重心・向き・意味。顔を持つ個体の小さな顔は簡略化しても性質を逆転させない。無顔/目0個の個体に顔を新設しない
6. material: 小さな質感は間引いてよい。gelの透過などはpixel版でopaqueな色clusterへ翻訳し、身体が消えないようにする

## 6. 再生成テンプレート

候補画像を参照し、合格部分を保持したまま具体的に修正する。

```text
Revise only this failed condition: {specific_failure}.
Keep the approved silhouette, body proportions, exact anatomy counts, palette placement, all unaffected identifiers, facing and pose unchanged.
Required correction: {targeted_correction}.
Still enforce the complete original brief and the {variant}-v1 rendering contract.
```

画像全体を任意に作り直すのではなく、検査で判明した不合格を1つずつ修正する。ただし既存キャラクターへの明確な類似やbriefの根本矛盾は、個体デザイン自体を改訂して新versionとして扱う。

## 7. 代表画の扱い

Stage Aが合格したら代表10体を画風照合用に使える。ただし後続の個体への参照では「線・塗りの画風のみ」と明示し、先行個体の顔・輪郭・装飾を複製しない。pixel版のidentity referenceは常に当該個体の合格通常絵であり、別の個体のpixel絵で代用しない。
