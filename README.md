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

## **5\. アンチパターン（やってはいけない書き方）**

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
