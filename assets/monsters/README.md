# ソダテルモン モンスター素材

**現在は制作計画のみです。画像の生成・品質検査はまだ行っていません。**

- [`catalog.json`](catalog.json): 独立して設計した215種の候補と全予定パス
- [`manifest.json`](manifest.json): 215対・430画像枠の進捗。予定と実ファイルを区別
- [`production-plan.md`](../../docs/art/production-plan.md): 後続Codexの実行順、品質基準、保存・中断・再開
- [`style-and-prompts.md`](../../docs/art/style-and-prompts.md): 通常/pixelの共通生成prompt

## 保存階層

```text
assets/monsters/
  README.md
  catalog.json
  manifest.json
  gel/       # 粘体 20
  scaled/    # 鱗体 25
  fanged/    # 牙体 25
  winged/    # 翼体 20
  micro/     # 微体 20
  flora/     # 植体 20
  arcane/    # 魔体 25
  decay/     # 腐体 20
  mineral/   # 硬体 25
  enigma/    # 謎体 15
```

各系統の下に `sm-<family>-<3桁番号>` ディレクトリを作る。例:

```text
assets/monsters/gel/sm-gel-001/
  default.png  # 合格通常絵 1024×1024 alpha PNG
  pixel.png    # 同じ個体のドット絵 128×128 alpha PNG
  meta.json    # 来歴・不変条件・hash・検査・保存読戻し
```

Gitは空ディレクトリを保存しないので、計画段階では215個の空folderや `.gitkeep` を置かない。後続Codexはカタログの `paths.default` / `paths.pixel` / `paths.meta` の親ディレクトリを `mkdir -p` 相当で作成する。パスを任意に推測せず、`assets/monsters/<family>/<id>/` と一致することを検査してから作る。絶対パス・`..`・シンボリックリンクを通る外部への書出しは受け付けない。

ファイル名は固定、IDは新規独立採番。番号は制作上の識別子であり、外部作品の個体・図鑑番号との対応はない。画像を別種へ割り当て直して欠番を埋めない。

## カタログschema v1

rootの `families` は系統ラベル・予定種数、`monsters` が215件。各個体:

- `id`, `family`, `family_label_ja`, `name_ja`: 独立ID、系統、名前候補
- `design_version`: brief改訂番号
- `silhouette`, `material_motif`, `palette_hex`, `signature_features`, `pose`, `temperament`, `do_not_add`: 画像生成入力と不変条件
- `paths`: 通常/pixel/metaの予定パス
- `prompt_templates`: `default-v1` と `pixel-v1`。style-and-prompts.mdのテンプレートへその個体の全fieldを展開
- `design_status`: 計画上の候補。完成画像の承認状態ではない

`palette_hex` は主色・副色・accentの順で3色。塗りのために輪郭色/影/明部を追加できるが、この配色の関係を置換しない。名前候補に商標確認済みの意味はない。

## manifest schema v1

rootに `families`, `expected_species`, `expected_assets`, `summary`, `entries` を持つ。`entries` はカタログIDと1対1。各variantは `planned_path` と `actual_file` を分離し、実画像が得られるまで `actual_file: null` を維持する。

生成日時・保存先・hash・byte数・寸法・色数・ツール名・参照hashは、実行した事実に基づいて記録する。作業日時の推測やダミーhashは使わない。`attempts` は不合格も含めて追記し、実際の展開済みpromptと加工履歴を辿れるようにする。

`reference_default_sha256` はpixel版が参照した通常絵のhash。通常絵が変更されたら、その値を新hashで機械的に上書きするのではなく、pixelを再生成する。

各 `pair_qa` に判定結果、確認者、確認日時、特徴照合の記録を残す。現在はすべて `not_run` である。

## 将来生成するmeta.jsonの契約

この例はschema説明であり、現在の実画像の存在を示さない。

```json
{
  "schema_version": 1,
  "id": "sm-gel-001",
  "design_version": 1,
  "catalog_path": "assets/monsters/catalog.json",
  "provenance": {
    "description": "独自デザイン計画に基づくAI生成",
    "generated": false,
    "tool": null,
    "generated_at": null,
    "prompt_template_versions": {"default": "default-v1", "pixel": "pixel-v1"},
    "expanded_prompts": {"default": null, "pixel": null},
    "reference_default_sha256": null,
    "processing_log": []
  },
  "assets": {
    "default": {"file": null, "sha256": null, "bytes": null, "width": null, "height": null, "mode": null},
    "pixel": {"file": null, "sha256": null, "bytes": null, "width": null, "height": null, "mode": null}
  },
  "qa": {"default": "not_run", "pixel": "not_run", "pair": "not_run", "checked_at": null, "notes": []},
  "readback": {"local": "not_run", "repository": "not_run", "commit_sha": null}
}
```

生成後のmetaは、参照するcatalogの不変条件とversion、2画像の実測値、prompt全文を辿れること。外部の元キャラクター名・作品名・資料URL・対応表を追加しない。制作内容を説明する独自briefと実作業の来歴を残す。

同一commit内に自分自身のcommit SHAを埋め込む循環は作らない。repository readback結果は後続の記録commitで直前の対象commit SHAを示すか、PR内の検証結果へ記載する。画像のhashは画像内容のSHA-256でありGit blob SHAとは別物。

## 保存と公開

既存のroot READMEや他ファイルはこの計画の追加で変更しない。新規作業branch・Draft PRを使い、main直接push/mergeをしない。画像公開時も公開対象の範囲を確認し、採用済み画像と必要なメタデータだけを追加する。

来歴表示は、実際に行った独自デザイン/AI生成/加工の範囲に合わせる。AI生成であることを隠したり、未実施の権利確認を実施済みとしたりしない。生成される画像の品質・権利状態はこの計画書だけでは保証しない。
