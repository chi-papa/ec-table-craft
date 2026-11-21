# ec-table-craft
ECモール商品ページ向けのノーコードHTMLビルダー。単一HTML・外部依存ゼロ、Shift_JISバイト数計算・タグ制限チェック・ローカル画像プレビュー対応。
# RMS Page Studio

楽天市場RMS（PC用商品説明文 / PC用販売説明文 / スマホ用商品説明文）に対応した、単一HTMLファイル・外部依存ゼロ（CDN不使用）のノーコード商品ページビルダーです。

ブラウザで `.html` ファイルを開くだけで動作し、HTMLやCSSの知識がなくても、RMSの厳しいタグ制限・文字コード（Shift_JIS）・バイト数上限（10,240 Byte）を守った安全な商品ページHTMLを作成できます。

---

## 主な機能

* **3エリア独立編集**
  * `PC用商品説明文` / `PC用販売説明文` / `スマホ用商品説明文` をタブで切り替え、それぞれのRMS仕様に合わせたHTMLを自動生成します。
  * スマホ用エリアでは、RMSで許可されたタグ（`p`, `br`, `font`, `b`, `u`, `center`, `hr`, `a`, `img`, `table`, `tr`, `td`, `th`）と `%` 指定のみを出力し、CSSや禁止属性を一切出しません。
* **ローカル画像プレビュー ＋ R-Cabinet URL自動生成**
  * プロジェクト単位で「R-CabinetベースURL」を設定可能。
  * 作業中は手元のローカル画像をドラッグ＆ドロップして即座にプレビュー表示し、HTML出力時には `ベースURL + ファイル名` を自動結合します。
  * **安全策:** 日本語・大文字・記号を含むファイル名の警告、パーツごとの「URL直接入力」への個別切替、プレビュー画面での「ローカル画像 / 本番URL確認」切替を搭載しています。
* **RMSルール事前チェック（7項目）**
  * Shift_JIS換算バイト数（改行CRLF=2Byte換算・10,240 Byte上限）、禁止タグ、禁止属性、SP用画像枚数（最大20枚）、SP用固定px幅、リンク属性（PC用 `target="_top"`）・URL安全性、Shift_JIS非対応文字（絵文字・環境依存文字）を自動検証します。
* **データ保存・管理**
  * **自動保存:** 編集停止から0.5秒後にブラウザ内（IndexedDB / LocalStorageフォールバック）へ自動保存されます。
  * **JSON書き出し／読み込み:** プロジェクト単位で `.json` ファイルとしてPCにバックアップ・共有できます。
* **UIテーマ・カラー切替**
  * ライトモード／ダークモードの切替および6色のアクセントカラー変更に対応（設定はブラウザに保存）。

---

## 使い方

1. `rms-page-studio.html` をダブルクリックしてモダンブラウザ（Chrome / Edge 等）で開きます。
2. 必要に応じてヘッダーのフォルダアイコンから「R-CabinetベースURL」を設定します。
3. 左サイドバーからパーツをクリック、または中央キャンバスへドラッグ＆ドロップしてページを組み立てます。
4. キャンバス上のパーツをクリックし、右サイドバーでテキスト・色・画像・テーブルセルなどを編集します。
5. 上部の **「プレビュー」** でPC/スマホ表示や本番URLのリンク切れを確認し、**「RMSチェック」** でエラーがないか検証します。
6. **「HTML」** ボタンから生成されたコードをコピーし、RMSの管理画面へ貼り付けます。

### キーボードショートカット

| ショートカット | 動作 |
|---|---|
| `Ctrl + Z` / `Cmd + Z` | 元に戻す（Undo / 最大50段階） |
| `Ctrl + Shift + Z` / `Ctrl + Y` | やり直す（Redo） |
| `Ctrl + S` / `Cmd + S` | 手動保存 |
| `Ctrl + D` / `Cmd + D` | 選択中のパーツを複製 |
| `Delete` / `Backspace` | 選択中のパーツを削除 |
| `Esc` | モーダルを閉じる / パーツの選択解除 |

---

## 拡張ガイド（コードの追加・変更箇所）

本ツールは単一HTMLファイル内で役割ごとにモジュール（IIFE）を分離しています。機能を拡張する際は、以下の該当セクションにコードを追加してください。

### 1. 新しいパーツを追加したい場合
以下の **6箇所** に同じ `type` 名（例: `badge_box`）の定義を追加します。

1. **`PART_CATALOG`（左サイドバーのパーツ一覧）**
   * パーツのラベル、SVGアイコン、`spAllowed`（スマホ用エリアで許可するか）を定義します。
2. **`PartFactory.create()`（パーツの初期データ）**
   * `switch (type)` に新しい `case` を追加し、デフォルトのプロパティオブジェクトを返します。
3. **`HtmlGeneratorPC.renderPart()`（PC用HTML出力）**
   * PC用の `table` ベースHTML生成ロジックを追加します。
4. **`HtmlGeneratorSP.renderPart()`（スマホ用HTML出力）**
   * スマホ用の許可タグのみを使ったHTML生成ロジックを追加します（SP非対応なら空文字を返す）。
5. **`AppUI.renderBlockVisual()`（中央キャンバスの見た目）**
   * 編集画面上でのWYSIWYGプレビュー用HTMLを返す処理を追加します。
6. **`AppUI.renderProperties()`（右サイドバーの設定UI）**
   * `else if (part.type === '...')` を追加し、入力フォームの描画とイベント登録を行います。
   * ※テキスト入力欄を追加した場合、`attachInputUndoSnapshots()` を呼ぶことで自動的に `Ctrl+Z` 履歴へ対応します。

### 2. RMSチェック項目を追加・変更したい場合
* **`RmsValidator.validate()`**
  * 検証ルール（禁止タグ `PC_FORBIDDEN_TAGS` / `SP_ALLOWED_TAGS`、禁止属性、URLチェック等）を編集・追加します。
  * `checks.push({ id, title, status, detail, partIssues })` の形式で項目を増やすと、モーダルUIに自動反映されます。

### 3. テーマカラー（アクセント色）を増やしたい場合
* **`ThemeManager` 内の `ACCENTS` 配列**
  * `{ id, label, color, hover, soft, border }` のオブジェクトを1行追加するだけで、ヘッダーのテーマ設定ポップアップに自動表示されます。

### 4. ショートカットやヘッダー機能を追加したい場合
* **`AppUI.bindEvents()`**
  * キーボードショートカットは `document.addEventListener('keydown', ...)` 内に追加します。
  * プロジェクトメニューの項目はHTMLの `#projectDropdownMenu` にボタンを追加し、`AppUI.handleProjectAction(action)` で分岐処理を書きます。

### 5. 保存データの項目を増やしたい場合
* **`DataStore.createBlankProject()`** に初期プロパティを追加し、**`DataStore.loadProject()`** で古いJSONを読み込んだ際のデフォルト値補完（マイグレーション処理）を記述します。

---

## 動作環境・セキュリティ

* **推奨ブラウザ:** Google Chrome / Microsoft Edge（最新版）
* **通信仕様:** 外部APIやCDNへの通信は一切行いません（サンプル画像挿入時の `placehold.co` およびユーザーが指定した画像URLの読み込みを除く）。データはすべて端末内のブラウザストレージに保存されます。

---

## ライセンス

This project is licensed under the **MIT License**.

```text
MIT License

Copyright (c) 2026 chi-papa

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
