# **Modern BEM コーディング規約 (Draft)**

本ドキュメントは、コンポーネント指向のUI開発における「Modern BEM」の設計仕様とコーディング規約を定義します。フレームワークの有無は問いません。Vanilla（プレーンな HTML / CSS / JS）でも、コンポーネント単位でマークアップとスタイルを分割する前提であれば有効です。

## **1\. 目的と前提**

> - **コンポーネントとの一致:** CSSのBlockは、UIコンポーネント（またはそれに相当するマークアップ単位）と1対1で対応させます。
> - **可読性と保守性の向上:** クラス名を見るだけで、コンポーネント内の要素（Element）なのか、状態変化（Modifier/State）なのかを明確に判別できるようにします。

## **2\. 命名規則（基本）**

コンポーネントを **Block (ブロック)**、その構成要素を **Element (エレメント)**、見た目や状態の変化を **Modifier (モディファイア)** として定義します。クラス名は kebab-case で統一します。

```css
.user-profile {}
.user-profile__avatar {}
```

```html
<div class="user-profile">
  <img class="user-profile__avatar" src="..." alt="" />
</div>
```

| 要素         | 命名規則                                    | 例                                                   |
| :----------- | :------------------------------------------ | :--------------------------------------------------- |
| **Block**    | コンポーネント名（kebab-case）              | .user-profile                                        |
| **Element**  | Block名にアンダースコア2つ（\_\_）を繋ぐ    | .user-profile\_\_avatar                              |
| **Modifier** | BlockまたはElementにハイフン2つ（--）を繋ぐ | .user-profile--compact .user-profile\_\_btn--primary |

## **3\. 状態（State）の管理**

Modern BEMでは、UIの「状態（State）」と「見た目のバリエーション（Modifier）」を明確に区別します。状態管理には is-\* プレフィックス、または data-\* 属性（データ属性）の使用を推奨します。

> - **Modifier (--):** 静的な見た目のバリエーション（サイズ、色、レイアウトの変更など）。
> - **State (is-\* または data-\*):** 動的な状態（アクティブ、無効化、開閉状態など）。主にJavaScriptからクラスや属性が付与・削除されることを前提とします。

```html
<button class="button button--primary is-loading" data-disabled="true">
  送信
</button>
```

```css
.button { /* ベーススタイル */ }
.button--primary { /* 青色などのテーマ変更 (Modifier) */ }
.button.is-loading { /* ロード中のスピナー表示など (State) */ }
.button[data-disabled="true"] { /* 非活性状態 (State: データ属性) */
  opacity: 0.5;
  pointer-events: none;
}
```

## **4\. コンポーネント設計との境界**

CSSのBlock階層とコンポーネントの階層は一致させる必要があります。BEMの階層が深くなりすぎる場合は、CSSの問題ではなくコンポーネント分割が必要なサインです。

> - **孫要素の禁止:** Block\_\_Element\_\_Element のような深いネストの命名は絶対に行いません。
> - **レイアウトの責務分離:** コンポーネント自身（Blockのルート要素）には、外側の余白（margin）や絶対配置（position）などのコンテキストに依存するスタイルを持たせません。配置は親コンポーネントが担うか、レイアウト用のラッパー要素を使用します。

## **5\. SCSSのディレクトリ・ファイル構成**

Blockとファイル（またはディレクトリ）は1対1で対応させます。Element / Modifier / Stateは別ファイルに分けず、所属するBlockのファイル内に記述します。ファイル名はBlock名と同じkebab-caseとします（例: `.user-profile` ↔ `_user-profile.scss`）。

```text
scss/
├── foundation/          # リセット、トークン、ミックスイン（Blockではない）
│   ├── _reset.scss
│   ├── _tokens.scss
│   └── _mixins.scss
├── layout/              # 配置専用（margin / position などコンテキスト依存）
│   ├── _spacing.scss    # .mt-4 などのユーティリティクラス
│   ├── _stack.scss
│   └── _grid.scss
├── blocks/              # UI Block（コンポーネント）
│   ├── _button.scss
│   ├── _card.scss
│   └── _user-profile.scss
└── main.scss            # @use で集約する入口
```

| ディレクトリ | 役割 |
| :----------- | :--- |
| **foundation/** | リセット、デザイントークン、ミックスインなど、BEMの階層に属さない共通基盤 |
| **layout/** | 外側の余白や配置など、コンテキスト依存のスタイル専用（第4節の責務分離に対応） |
| **blocks/** | UIコンポーネント（Block）。1ファイル = 1 Block |
| **main.scss** | 上記を `@use` で読み込むエントリポイント |

小さく始める場合は `foundation/` + `blocks/` + `main.scss` だけでも構いません。規模に応じて `layout/` を追加します。

> - **Element単位の分割をしない:** `_card__title.scss` のようにElementごとにファイルを切らない。
> - **ページ単位の巨大ファイルに寄せない:** `_home.scss` に複数Blockを同居させない。
> - **子Blockの上書き用ファイルを作らない:** 親から子コンポーネントを直接スタイリングする置き場は設けない（第6節のアンチパターン2と同趣旨）。

### **ユーティリティクラスの配置**

`.mt-4` や `.mt-8` のようなユーティリティクラスは BEM の Block ではないため、`blocks/` には置きません。外側の余白はレイアウトの責務なので、`layout/_spacing.scss` に配置します。

```scss
// layout/_spacing.scss
.mt-4 {
  margin-top: 1rem;
}

.mt-8 {
  margin-top: 2rem;
}
```

| 書き方 | 置き場所 | 例 |
| :----- | :------- | :--- |
| コンポーネント内のレイアウト | Block 内の Element | `.card__action-area` |
| ページ組み立て用の汎用クラス | `layout/`（または `utilities/`） | `.mt-4`, `.stack` |
| UI コンポーネント本体 | `blocks/` | `.card`, `.button` |

`margin` 以外のユーティリティ（`p-4`, `flex`, `text-center` など）も増える場合は、`layout/` ではなく `utilities/` を切り分けても構いません。

```text
scss/
├── utilities/
│   ├── _spacing.scss
│   └── _display.scss
```

まずは Block 内の Element でレイアウトを表現し、汎用化が必要な場合にのみ `layout/`（または `utilities/`）へ置くことを推奨します。

## **6\. アンチパターン（やってはいけない書き方）**

### **❌ アンチパターン 1: 孫要素（Elementのネスト）**

ElementがBlock内のどの要素に属するか（DOMのツリー構造）をクラス名で表現してはいけません。Elementは常にBlockに直接属します。

```css
/* 悪い例（DOM構造をクラス名に反映している） */
.card__list__item { ... }
.card__list__item__text { ... }
```

```css
/* 良い例（Elementは常にBlockに属するフラットな構造） */
.card__list { ... }
.card__item { ... }
.card__text { ... }
```

### **❌ アンチパターン 2: 親から子のコンポーネントを直接スタイリング**

カプセル化を破壊し、予期せぬ副作用を生むため、他のコンポーネントのクラスを親から直接上書きしてはいけません。

```css
/* 悪い例 */
.card .button {
  margin-top: 16px; /* 別のコンポーネント(.button)のスタイルを直接変更している */
}
```

```css
/* 良い例（親コンポーネント内にレイアウト用のクラスを用意し、そこに子を配置する） */
.card__action-area {
  margin-top: 16px;
}
```

```html
<!-- HTML側 -->
<div class="card__action-area"><Button /></div>
```
