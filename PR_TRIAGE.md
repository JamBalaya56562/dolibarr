# Dolibarr 停滞プルリクエスト Triage レポート

- **対象リポジトリ:** [Dolibarr/dolibarr](https://github.com/Dolibarr/dolibarr)（upstream）
- **調査日:** 2026-07-14
- **最終更新:** 2026-07-16（調査日〜本日の差分を再取得。掲載した停滞PR 170件のうちこの期間にClose/mergeされたものは **0件** のため、内容の削除・追加なし。同期間にClose/mergeされたPR26件はいずれも直近作成の活発PRで本レポート対象外）
- **調査対象:** Open な PR のうち、**6ヶ月以上更新が止まっている停滞PR = 170件**（`updated < 2026-01-14`）
- **最終更新の分布:** 2024-04（約27ヶ月前）〜 2026-01（約6ヶ月前）
- **参考:** Open PR 全体は **477件**。うち直近6ヶ月に更新のある活発なPR（307件）は本レポートの対象外
- **手法:** GitHub API で停滞PRを取得し、メンテナのPRステータスラベル・停滞期間・最終コメント者・base branch・変更規模・本文を手がかりに、5チャンクに分割して並列分類

> **分類の主軸はメンテナのPRステータスラベル**です（`PR to fix - See feedback`, `Conflict or CI error`, `Discussion`, `postponed` 等）。マージ可否(mergeable)はGitHubの遅延計算で多くが`UNKNOWN`、形式的レビュー(reviewDecision)もほぼ未使用のため、補助的にのみ利用しました。

---

## 1. エグゼクティブサマリー

停滞PR **170件** を「停滞の根本原因(Category)」と「推奨アクション(Action)」の2軸で分類しました。

### 停滞の根本原因(Category)別

| カテゴリ | 件数 | 割合 | 意味 |
|---|---:|---:|---|
| **Needs-Discussion**（設計・方針が未合意） | 67 | 39% | 「Discussion」ラベル等。メンテナの設計判断が先 |
| **Needs-Rebase/CI**（コンフリクト/CI失敗） | 32 | 19% | rebase・CI修正が必要 |
| **Ready-ish**（ほぼ取り込める） | 30 | 18% | 小さく非論争的。メンテナの再確認/マージ待ち |
| **Author-Unresponsive**（作者が無応答） | 22 | 13% | レビュー指摘/情報要求に作者が反応せず |
| **Postponed**（意図的に保留/凍結ブランチ狙い） | 12 | 7% | メンテナが延期、またはEOL/凍結ブランチが対象 |
| **Needs-Restructure**（分割・再構成が必要） | 3 | 2% | 複数の関心事が混在／大きすぎる |
| **Superseded**（他実装で解決済み/重複） | 2 | 1% | 新しいPRや別実装に置き換わった |
| **Reimplement**（新規で作り直すべき） | 2 | 1% | アイデアは良いがPRが放置・陳腐化 |

### 推奨アクション(Action)別

| アクション | 件数 | 割合 |
|---|---:|---:|
| **MAINTAINER_REVIEW**（メンテナのレビュー/判断） | 105 | 62% |
| **REBASE_CI**（rebase/CI修正） | 32 | 19% |
| **PING_AUTHOR**（作者へ催促、無反応ならClose） | 25 | 15% |
| **CLOSE**（Close推奨） | 3 | 2% |
| **SPLIT**（分割依頼） | 3 | 2% |
| **REIMPLEMENT**（作り直し） | 2 | 1% |

### 主要な所見

1. **最大のボトルネックは「作者」ではなく「メンテナのレビュー容量」。** 停滞の原因として最多は `Needs-Discussion`（67件）で、アクションの62%（105件）が**メンテナの判断/レビュー待ち**。作者が無応答なのは22件（13%）に過ぎない。つまり「作者が放置して停滞」よりも「**メンテナが判断を保留したまま滞留**」しているケースが圧倒的に多い。
2. **`Discussion` ラベルの滞留が構造的問題。** 68件のPRが「Discussion」ラベルのまま何ヶ月も放置。多くは**1〜2行の小さなバグ修正**が「設計判断待ち」で塩漬けになっている（例: #33685 extrafield SQL回帰, #34366 空product_typeのSQLエラー）。判断を下すだけで大量に消化できる。
3. **すぐ取り込める "Ready-ish" が30件。** 作者が指摘対応済み、または非論争的な小修正で、メンテナが再確認してマージするだけ。**最も費用対効果の高い消化対象**。
4. **明確なClose候補は少数（実質3件＋条件付き数件）。** EOL/凍結ブランチ狙い（#32661→14.0, #31347→19.0, #30481→14.0）や重複（#36551←#36609に置換）。「停滞＝Close」ではなく、多くは**復活可能な資産**。
5. **rebase/CI待ちが32件。** 中身は妥当だがコンフリクト放置。作者かボランティアがrebaseすれば前進する。一部は超大規模diff（#32667 は194ファイル変更）で恒久的にコンフリクトするため、作り直しの方が早い。
6. **作り直し推奨(Reimplement)は2件。** #30199（倉庫quarantine機能、2年停滞）, #30327（梱包ページWIP放置）。アイデアは有用だが現PRは救済困難。

---

## 2. 🔴 Close 推奨（他実装で解決済み・EOL/凍結ブランチ・放置）

| PR | 停滞 | 内容 | Close理由 |
|---|---|---|---|
| [#36551](https://github.com/Dolibarr/dolibarr/pull/36551) | 7.2mo | opensurveysondages API整理 | より新しく小さい **#36609 に置換**（重複）|
| [#32661](https://github.com/Dolibarr/dolibarr/pull/32661) | 17.9mo | select_company AJAXイベント修正のbackport | **凍結/EOLブランチ 14.0** 向け。受付終了 |
| [#23614](https://github.com/Dolibarr/dolibarr/pull/23614) | 11.3mo | propal/invoiceの製品ラベル更新 | 2023年の3行パッチ、関与ゼロ。既に対処済みの可能性 |

### 条件付きClose（作者への催促に無反応ならClose）

| PR | 停滞 | 内容 | メモ |
|---|---|---|---|
| [#29434](https://github.com/Dolibarr/dolibarr/pull/29434) | 26.7mo | Iyedben patch 2 | 26ヶ月放置・空テンプレ・31ファイル/2328行の巨大diff |
| [#31347](https://github.com/Dolibarr/dolibarr/pull/31347) | 21.0mo | massactionメールの連絡先置換キー | 対象が**EOLの19.0**。developへの再提出を依頼 |
| [#30481](https://github.com/Dolibarr/dolibarr/pull/30481) | 6.4mo | 仮想在庫定数の修正 | **凍結14.0**狙い。対応ブランチへの再ターゲット依頼 |
| [#32667](https://github.com/Dolibarr/dolibarr/pull/32667) | 11.3mo | type引数追加 | 194ファイル変更で恒久コンフリクト。作り直し推奨 |

---

## 3. 🔁 作り直し(Reimplement)推奨

アイデアは有用だが、現PRは陳腐化・放置・未対応フィードバックが多く、新規PRで作り直す方が早い。

| PR | 停滞 | 内容 | 理由 |
|---|---|---|---|
| [#30199](https://github.com/Dolibarr/dolibarr/pull/30199) | 7.0mo | `STOCK_USE_WAREHOUSE_USAGE`（在庫quarantine/非計上倉庫） | 有用な機能だが2年停滞・14件の未対応フィードバック |
| [#30327](https://github.com/Dolibarr/dolibarr/pull/30327) | 11.3mo | 梱包(packing.php)ページ | 空チェックリストのWIP放置。アイデアを新規PRで再実装 |
| [#32667](https://github.com/Dolibarr/dolibarr/pull/32667) | 11.3mo | 全体へのtype引数付与 | 194ファイルの機械的変更で恒久コンフリクト。スクリプトで再生成が現実的 |

---

## 4. 🟢 すぐ再マージできる候補（Ready-ish / MAINTAINER_REVIEW）

作者が指摘対応済み、または非論争的な小修正。メンテナが再確認してマージするだけで消化できる**最優先の費用対効果**枠。

| PR | 停滞 | 内容 |
|---|---|---|
| [#33749](https://github.com/Dolibarr/dolibarr/pull/33749) | 11.3mo | type-9行を受入数量から除外（明確な再現手順つきバグ修正） |
| [#33321](https://github.com/Dolibarr/dolibarr/pull/33321) | 11.5mo | 状況請求書の進捗修正（作者がフィードバック対応済、#33315を置換） |
| [#34588](https://github.com/Dolibarr/dolibarr/pull/34588) | 11.3mo | 作成時にorigin元からプロジェクトを設定（作者対応済） |
| [#33947](https://github.com/Dolibarr/dolibarr/pull/33947) | 11.3mo | result.phpの月次/期間合計の計算修正（デバッグログ除去のみ必要） |
| [#30000](https://github.com/Dolibarr/dolibarr/pull/30000) | 11.3mo | ref_clientをinterventionへ複製（小・対応済） |
| [#30369](https://github.com/Dolibarr/dolibarr/pull/30369) | 10.9mo | 出荷でロット/シリアル別に部品選択（作者が追加情報提供済） |
| [#36466](https://github.com/Dolibarr/dolibarr/pull/36466) | 7.5mo | MO原価にワークステーション費用を反映（的を絞ったバグ修正） |
| [#36183](https://github.com/Dolibarr/dolibarr/pull/36183) | 7.4mo | payments/CloseBillパラメータのフック（作者対応済） |
| [#36533](https://github.com/Dolibarr/dolibarr/pull/36533) | 7.3mo | product lotクラス属性のリファクタ（小） |
| [#36548](https://github.com/Dolibarr/dolibarr/pull/36548) | 7.2mo | 前払い（月次揃え）請求モード |
| [#36513](https://github.com/Dolibarr/dolibarr/pull/36513) | 7.1mo | 小数点以下2桁に制限（独ロケールのバグ修正） |
| [#36786](https://github.com/Dolibarr/dolibarr/pull/36786) | 6.3mo | batchlot管理の権限セット |
| [#36553](https://github.com/Dolibarr/dolibarr/pull/36553) | 6.2mo | マージン丸め/ゼロ除算の修正（v18） |
| [#35558](https://github.com/Dolibarr/dolibarr/pull/35558) | 6.2mo | `RELOAD_PAGE_ON_CUSTOMER_CHANGE` 修正（v18 backport） |
| [#32945](https://github.com/Dolibarr/dolibarr/pull/32945) | 11.3mo | コード統一＋multicompany（コア開発者hregis） |
| [#32483](https://github.com/Dolibarr/dolibarr/pull/32483) | 11.3mo | フック呼び出しにviewコードを渡す（3行） |

> ほか Ready-ish: #31292, #31267, #33806, #32038, #31875, #31621, #32662, #34089, #34187, #36415, #32514, #35714, #36602, #33606。

---

## 5. 🟠 rebase / CI 修正待ち（Needs-Rebase/CI）

中身は妥当だが、コンフリクトまたはCI失敗が未解決。作者かボランティアがrebaseすれば前進。放置が長いものは作者へ催促。

| PR | 停滞 | 内容 |
|---|---|---|
| [#35325](https://github.com/Dolibarr/dolibarr/pull/35325) | 3.5mo | プロジェクトのマージ機能（本群で最も新しい・要rebase後レビュー） |
| [#36700](https://github.com/Dolibarr/dolibarr/pull/36700) | 6.7mo | getUserProjects API関数 |
| [#36609](https://github.com/Dolibarr/dolibarr/pull/36609) | 7.1mo | opensurveysondages API整理（#36551の新版） |
| [#36001](https://github.com/Dolibarr/dolibarr/pull/36001) | 8.4mo | 複数経費精算の支払い（#30920/#24884の再提出） |
| [#36110](https://github.com/Dolibarr/dolibarr/pull/36110) | 7.7mo | 会計ファイルエクスポートのVAT詳細（CONFLICTING確定） |
| [#32682](https://github.com/Dolibarr/dolibarr/pull/32682) | 8.7mo | MRP数量の誤計算修正 |
| [#33186](https://github.com/Dolibarr/dolibarr/pull/33186) | 11.3mo | 出荷をキャンセルせず部分在庫戻し（CONFLICTING） |
| [#30407](https://github.com/Dolibarr/dolibarr/pull/30407) | 11.3mo | 受注の署名機能 |
| [#28947](https://github.com/Dolibarr/dolibarr/pull/28947) | 6.0mo | 請求書ボックスの累積グラフ（2年停滞・小） |

> ほか Rebase/CI待ち: #30744, #32644, #32485, #32481, #32480, #32484, #33069, #30559(＋Discussion), #29669, #34918, #34620, #34566, #33559, #30418, #35198, #31645, #29441, #31775, #35729, #36247, #36252, #34037, #35235。

---

## 6. 🟡 作者が無応答（Author-Unresponsive）— 催促→無反応ならClose

レビュー指摘・情報要求に作者が反応していないもの。テンプレ催促し、猶予期間内に反応がなければClose。中身が有用なものは第7章の復活候補と重複。

| PR | 停滞 | 内容 |
|---|---|---|
| [#31644](https://github.com/Dolibarr/dolibarr/pull/31644) | 11.3mo | リソースの二重予約防止バグ修正（有用・復活価値大） |
| [#32918](https://github.com/Dolibarr/dolibarr/pull/32918) | 11.3mo | interventionのキャンセルステータス |
| [#32215](https://github.com/Dolibarr/dolibarr/pull/32215) | 11.3mo | stock-at-dateのソート/フィルタ |
| [#31705](https://github.com/Dolibarr/dolibarr/pull/31705) | 11.3mo | cash control用extrafields（eldyが変更要求） |
| [#32614](https://github.com/Dolibarr/dolibarr/pull/32614) | 11.3mo | 型トグルのJSをCSSに置換（hregisが変更要求） |
| [#31903](https://github.com/Dolibarr/dolibarr/pull/31903) | 9.6mo | APIの`DATE(field)`フィルタ |
| [#30622](https://github.com/Dolibarr/dolibarr/pull/30622) | 11.3mo | 頭金つきデフォルト支払条件（要情報） |
| [#30608](https://github.com/Dolibarr/dolibarr/pull/30608) | 11.3mo | selectForFormsListWhereのgetEntity修正 |
| [#31049](https://github.com/Dolibarr/dolibarr/pull/31049) | 11.3mo | 顧客の担当者でフィルタ |
| [#29387](https://github.com/Dolibarr/dolibarr/pull/29387) | 11.3mo | 外国顧客の会計番号 |

> ほか Author-Unresponsive: #30267, #30194, #26566, #28184, #31012, #33747, #35991, #35544, #34242, #36476, #34360(凍結ブランチ)。

---

## 7. 🔷 復活の価値がある（メンテナ判断待ちのバグ修正）

「Discussion」ラベル等で塩漬けだが、**実バグの修正**で、メンテナが方針を下すだけで前進するもの。判断を優先的に。

| PR | 停滞 | 内容 |
|---|---|---|
| [#33685](https://github.com/Dolibarr/dolibarr/pull/33685) | 11.5mo | extrafield検索でSQLエラーを起こす回帰の小修正 |
| [#34366](https://github.com/Dolibarr/dolibarr/pull/34366) | 11.3mo | 空product_typeで発生する重大SQLエラーのガード |
| [#32209](https://github.com/Dolibarr/dolibarr/pull/32209) | 11.5mo | 多通貨の値引き/NaNバグ2件の修正 |
| [#32217](https://github.com/Dolibarr/dolibarr/pull/32217) | 11.5mo | 状況請求書のトリガー欠落（extrafieldアクション有効化） |
| [#34286](https://github.com/Dolibarr/dolibarr/pull/34286) | 11.3mo | 辞書から最高VAT率を採用（デフォルト選択ロジック） |
| [#26852](https://github.com/Dolibarr/dolibarr/pull/26852) | 11.3mo | 仕入注文の不承認時の在庫二重計上バグ修正 |
| [#33066](https://github.com/Dolibarr/dolibarr/pull/33066) | 11.3mo | `dol_htmlentitiesbr()` のメモリ枯渇クラッシュ |
| [#33191](https://github.com/Dolibarr/dolibarr/pull/33191) | 8.9mo | 支払済請求書の残額0修正（コメント20件・コア開発者hregis） |
| [#34360](https://github.com/Dolibarr/dolibarr/pull/34360) | 9.4mo | 仕入価格計算＋警告ログの修正（凍結18.0→developへ再ターゲット） |
| [#33747](https://github.com/Dolibarr/dolibarr/pull/33747) | 9.4mo | 請求書一覧のrest列修正（mergeable） |
| [#34105](https://github.com/Dolibarr/dolibarr/pull/34105) | 11.3mo | actioncommのemail列を新テーブルへ（要設計・大きめ） |
| [#34372](https://github.com/Dolibarr/dolibarr/pull/34372) | 7.5mo | classmapオートローディング（インフラ改善・要設計判断） |
| [#30559](https://github.com/Dolibarr/dolibarr/pull/30559) | 11.3mo | `/contract/expired` RESTエンドポイント |

---

## 8. 🧩 分割・再構成が必要（Needs-Restructure / SPLIT）

複数の関心事が混在、または大きすぎるため、そのままではレビュー不能。分割を依頼。

| PR | 停滞 | 内容 |
|---|---|---|
| [#33769](https://github.com/Dolibarr/dolibarr/pull/33769) | 11.3mo | 定期請求書の連絡先（「構造とコード変更の混在」ラベル、#33764の再提出） |
| [#30943](https://github.com/Dolibarr/dolibarr/pull/30943) | 11.3mo | チケットの事前計算所要時間（11ファイルに混在） |
| [#36245](https://github.com/Dolibarr/dolibarr/pull/36245) | 6.2mo | v22からのbackport（「複数PRに分割」ラベル、無関係な修正が束） |

---

## 9. ⏸️ 保留(Postponed)

メンテナが意図的に延期したもの。方針（再開 or 却下）の再判断が必要。

代表例: #31386（テスト型修正）, #31924（メールモデルのメソッド追加・大リファクタ）, #34236（仕入請求のcreateFromフック）, #34374（非int hook返却の廃止）, #36453（massactionフック）, #36602（CRONのpathデフォルト）, #36634（プロジェクト会社表の予算列）, #34283（c_country.sqlの電話コード）。

---

## 10. 推奨される次のアクション（PR運用）

1. **`Discussion` ラベル滞留の一斉判断（最優先）:** 停滞の最大要因。特に**1〜2行の小バグ修正**（#33685, #34366, #32217 等）は判断コストが低く、まとめてGo/No-Goを出せば大量消化できる。
2. **Ready-ish 30件の再確認マージ（第4章）:** 作者対応済み・非論争的。最も費用対効果が高い。#33947 のようにデバッグログ除去のみで済むものも。
3. **rebase/CI 32件の解消（第5章）:** 作者へrebase依頼、または good-first-contribution としてボランティアに割り当て。恒久コンフリクトの巨大PR（#32667）は作り直しへ。
4. **作者無応答 22件の催促→自動Close（第6章）:** テンプレ催促＋`Bug or PR need more information`。猶予後に無反応ならClose。中身が有用なもの（#31644 等）は第7章の復活/再実装へ回す。
5. **Close候補の整理（第2章）:** EOL/凍結ブランチ狙い・重複を即Close。トラッカーのノイズ削減。
6. **復活価値のあるバグ修正の優先化（第7章）:** SQLエラー・メモリクラッシュ・在庫/会計の実バグ修正が「Discussion」で塩漬け。品質に直結するため優先レビュー。
7. **凍結ブランチ狙いPRの再ターゲット依頼:** #34360, #31347, #30481, #30481 等は develop/対応ブランチへ付け替え依頼。
8. **恒常運用:** `PR waiting more user feedbacks` 系にstaleボットを適用し、無反応PRの自動Closeで滞留を防止。

---

## 付録A: 分類基準

**Category（停滞の根本原因）:** Superseded（他実装で解決/重複）, Author-Unresponsive（作者無応答）, Needs-Rebase/CI（コンフリクト/CI）, Needs-Discussion（設計未合意）, Postponed（保留/凍結ブランチ）, Needs-Restructure（分割要）, Ready-ish（ほぼ取込可）, Reimplement（作り直し）

**Action:** CLOSE, PING_AUTHOR（催促、無反応ならClose）, REBASE_CI, MAINTAINER_REVIEW（メンテナのレビュー/判断/マージ）, REIMPLEMENT, SPLIT

## 付録B: 全170件の分類表

停滞（更新停止）が古い順に5バッチで掲載。`停滞(mo)` は最終更新からの経過月数。

### バッチ1（#29434〜#30367）

| PR | Title (short) | 停滞(mo) | Category | Action | Reason |
|---|---|---|---|---|---|
| #29434 | Iyedben patch 2 | 26.7 | Author-Unresponsive | PING_AUTHOR | Maintainer feedback, author silent 26mo; huge empty-body diff. |
| #31347 | Contact substitution keys in massaction emails | 21.0 | Postponed | PING_AUTHOR | Targets EOL 19.0; resubmit small fix against develop. |
| #32661 | Backport select_company AJAX event fix | 17.9 | Postponed | CLOSE | Backport to frozen/EOL 14.0; not accepted. |
| #33685 | Quote value in extra field SQL where | 11.5 | Needs-Discussion | MAINTAINER_REVIEW | Tiny fix for extrafield SQL regression; needs decision. |
| #32217 | Missing trigger on situation invoice | 11.5 | Needs-Discussion | MAINTAINER_REVIEW | One-line trigger fix; awaits agreement. |
| #32209 | Bugfix multicurrency discount/NaN | 11.5 | Needs-Discussion | MAINTAINER_REVIEW | Fixes two multicurrency bugs; needs call. |
| #31706 | Fix unit output (use_short_label) | 11.5 | Needs-Discussion | MAINTAINER_REVIEW | Label-consistency fix; approach unresolved. |
| #32869 | Constant to avoid price error on TTC base | 11.5 | Needs-Discussion | MAINTAINER_REVIEW | 19-comment contentious thread; needs decision. |
| #33321 | Situation invoices progress fix | 11.5 | Ready-ish | MAINTAINER_REVIEW | Replaced #33315, feedback addressed; re-check. |
| #34667 | restrictedArea ajaxtooltip multi-entity | 11.3 | Needs-Discussion | MAINTAINER_REVIEW | One-line guard; author unsure, needs review. |
| #34620 | Conf for product photo order by position | 11.3 | Needs-Rebase/CI | REBASE_CI | Conflict/CI label; needs rebase. |
| #34643 | Property 'donthavesocparent' on CommonObject | 11.3 | Needs-Discussion | MAINTAINER_REVIEW | Multi-entity edge case; approach questioned. |
| #34566 | Autogen societe barcode | 11.3 | Needs-Rebase/CI | REBASE_CI | Conflict/CI; fix for #33610 needs rebase. |
| #34588 | Set project from origin on creation | 11.3 | Ready-ish | MAINTAINER_REVIEW | Author addressed feedback; re-review. |
| #33947 | Fix monthly/period totals in result.php | 11.3 | Ready-ish | MAINTAINER_REVIEW | Calc fix; remove debug logs then merge. |
| #33749 | Exclude type-9 lines from reception qty | 11.3 | Ready-ish | MAINTAINER_REVIEW | Bug fix with clear repro; ready. |
| #34380 | Accept tiny & big int with size | 11.3 | Needs-Discussion | MAINTAINER_REVIEW | Type-parsing change, no description. |
| #34366 | Product_type empty line SQL error | 11.3 | Needs-Discussion | MAINTAINER_REVIEW | Guards empty product_type SQL error. |
| #34236 | Hook 'createFrom' for supplier invoice | 11.3 | Postponed | MAINTAINER_REVIEW | Deliberately postponed hook. |
| #34242 | Function to style total amount | 11.3 | Author-Unresponsive | PING_AUTHOR | Feedback requested, empty body, inactive. |
| #34131 | Update line minimum price in propale | 11.3 | Needs-Discussion | MAINTAINER_REVIEW | Small price_base_type fix; needs decision. |
| #34105 | Move actioncomm email field to new table | 11.3 | Needs-Discussion | MAINTAINER_REVIEW | 14-comment schema change; design open. |
| #34318 | Use arrays in default values | 11.3 | Needs-Discussion | MAINTAINER_REVIEW | Behavioral change; discussion open. |
| #34374 | Stop supporting non-int hook returns | 11.3 | Postponed | MAINTAINER_REVIEW | Deprecation refactor postponed. |
| #34388 | Update num payment | 11.3 | Ready-ish | MAINTAINER_REVIEW | Thin description; verify scope. |
| #33559 | More controls on supplier invoices | 11.3 | Needs-Rebase/CI | REBASE_CI | Conflict/CI; date-control needs rebase. |
| #33514 | Empty accountancy code if perentity shared | 11.3 | Needs-Discussion | MAINTAINER_REVIEW | Conditional reset; needs decision. |
| #33520 | ACCOUNTANCY_SELL_JOURNAL precision conf | 11.3 | Needs-Discussion | MAINTAINER_REVIEW | New global for precision; unresolved. |
| #34839 | Add precautionary verification | 11.3 | Needs-Discussion | MAINTAINER_REVIEW | Guards missing multicurrency amounts. |
| #34155 | Add invoice pie chart | 11.3 | Needs-Discussion | MAINTAINER_REVIEW | 150-line new feature; design open. |
| #34222 | Fix menu icon (png + font) | 11.3 | Needs-Discussion | MAINTAINER_REVIEW | 7-comment thread; needs resolution. |
| #34286 | Use highest VAT rate from dictionary | 11.3 | Needs-Discussion | MAINTAINER_REVIEW | Changes default VAT logic; pending. |
| #30418 | Update card.php (ticket trigger) | 11.3 | Needs-Rebase/CI | REBASE_CI | Conflict/CI; vague old proposal. |
| #30367 | Increase precision in total calculations | 11.3 | Needs-Discussion | MAINTAINER_REVIEW | Precision change; sensitive, needs call. |

### バッチ2（#29966〜#32785）

| PR | Title (short) | 停滞(mo) | Category | Action | Reason |
|---|---|---|---|---|---|
| #29966 | Fix width td desc contract card | 11.3 | Needs-Discussion | MAINTAINER_REVIEW | Cosmetic CSS tweak flagged Discussion. |
| #31292 | Insert overwrite translation | 11.3 | Ready-ish | MAINTAINER_REVIEW | Unlabeled, never reviewed. |
| #31267 | Use ContratLigne in Contracts API | 11.3 | Ready-ish | MAINTAINER_REVIEW | API refactor, no review yet. |
| #30744 | includetimespent in tasks API | 11.3 | Needs-Rebase/CI | REBASE_CI | Conflict/CI; needs rebase. |
| #31386 | Fix type on onNotSuccessfulTest | 11.3 | Postponed | MAINTAINER_REVIEW | Test-type fix labeled postponed. |
| #31534 | Update list.php php8.2 type | 11.3 | Needs-Discussion | MAINTAINER_REVIEW | One-line php8.2 fix, Discussion. |
| #33806 | STATUS_CLOSED on intervention graph | 11.3 | Ready-ish | MAINTAINER_REVIEW | frederic34 last commented; re-check. |
| #33769 | Contacts on recurring invoices | 11.3 | Needs-Restructure | SPLIT | Mixing structure+code; re-try of #33764. |
| #33768 | Add CCI in Soc Rib | 11.3 | Needs-Discussion | MAINTAINER_REVIEW | Discussion; bank field not agreed. |
| #33794 | Recurring invoice services date | 11.3 | Needs-Discussion | MAINTAINER_REVIEW | Date logic change; Discussion. |
| #32038 | Category feature on fichinter 3/3 | 11.3 | Ready-ish | MAINTAINER_REVIEW | Series final; frederic34 engaged. |
| #31644 | Busy resources double-booking fix | 11.3 | Author-Unresponsive | PING_AUTHOR | See-feedback; valuable bug fix. |
| #32215 | Stock-at-date sortable/filterable | 11.3 | Author-Unresponsive | PING_AUTHOR | eldy asked more feedback; silent. |
| #31875 | Fix dependent select extrafields | 11.3 | Ready-ish | MAINTAINER_REVIEW | Devcamp fix (#30801), never reviewed. |
| #31903 | API filter on DATE(field) | 9.6 | Author-Unresponsive | PING_AUTHOR | See-feedback; reviewer last, silent. |
| #31924 | Add methods for mail models | 11.3 | Postponed | MAINTAINER_REVIEW | Large refactor postponed. |
| #31705 | Extrafields for cash control | 11.3 | Author-Unresponsive | PING_AUTHOR | eldy requested changes; unresponsive. |
| #31841 | Serialize array attributes | 11.3 | Needs-Discussion | MAINTAINER_REVIEW | Data-persistence fix; approach debated. |
| #31862 | Fix #31025 | 11.3 | Author-Unresponsive | PING_AUTHOR | See-feedback, empty body, silent. |
| #31979 | NEW constants EMAIL | 11.3 | Needs-Discussion | MAINTAINER_REVIEW | Bundles unrelated NEW items. |
| #31621 | Prefer given field props in showInputField | 11.3 | Ready-ish | MAINTAINER_REVIEW | Focused refactor, unreviewed. |
| #32662 | Global search on services list | 11.3 | Ready-ish | MAINTAINER_REVIEW | Self-contained feature, no review. |
| #32667 | add type param and return function | 11.3 | Needs-Rebase/CI | REBASE_CI | 194-file mass change; perpetual conflict. |
| #32761 | Link contract and fichinter | 11.3 | Needs-Discussion | MAINTAINER_REVIEW | New cross-object link; needs design. |
| #32945 | Uniformize code + multicompany | 11.3 | Ready-ish | MAINTAINER_REVIEW | QUAL cleanup by hregis, unreviewed. |
| #32918 | Cancel status on interventions | 11.3 | Author-Unresponsive | PING_AUTHOR | See-feedback; useful feature, silent. |
| #32644 | Show only open projects in short list | 11.3 | Needs-Rebase/CI | REBASE_CI | Conflict/CI; needs rebase. |
| #32485 | Hook for pay.php actions | 11.3 | Needs-Rebase/CI | REBASE_CI | Small hook; needs rebase. |
| #32481 | Hook for credit-note creation params | 11.3 | Needs-Rebase/CI | REBASE_CI | Small hook; needs rebase. |
| #32480 | Hook for Takepos lines | 11.3 | Needs-Rebase/CI | REBASE_CI | Large diff; needs rebase. |
| #32483 | Add view code on hook invocation | 11.3 | Ready-ish | MAINTAINER_REVIEW | 3-line hook fix; quick check. |
| #32484 | Hook for payment actions | 11.3 | Needs-Rebase/CI | REBASE_CI | Small hook; needs rebase. |
| #32614 | Replace JS with CSS for type toggles | 11.3 | Author-Unresponsive | PING_AUTHOR | hregis requested changes; silent. |
| #32785 | WEBSITE_PHP_ALLOW_DYNAMIC_FUNCTIONS passby | 11.3 | Needs-Discussion | MAINTAINER_REVIEW | Security-sensitive; eldy pushed back. |

### バッチ3（#32514〜#30369）

| PR | Title (short) | 停滞(mo) | Category | Action | Reason |
|---|---|---|---|---|---|
| #32514 | Bookcal working-hours + entities check | 11.3 | Ready-ish | MAINTAINER_REVIEW | New feature, never reviewed. |
| #33066 | Memory exhaustion in dol_htmlentitiesbr() | 11.3 | Needs-Discussion | MAINTAINER_REVIEW | Raising memory_limit contentious. |
| #33069 | Delete logo from disk | 11.3 | Needs-Rebase/CI | REBASE_CI | Conflict/CI; unresolved. |
| #33186 | Return stock partial without cancel shipment | 11.3 | Needs-Rebase/CI | REBASE_CI | CONFLICTING; needs rebase. |
| #33306 | Show misc payments/loans in accounting result | 11.3 | Needs-Discussion | MAINTAINER_REVIEW | Discussion; needs design decision. |
| #33269 | Email subject encoding fix | 11.3 | Needs-Discussion | MAINTAINER_REVIEW | One-line fix; awaits call. |
| #30608 | selectForFormsListWhere getEntity fix | 11.3 | Author-Unresponsive | PING_AUTHOR | hregis asked feedback; silent. |
| #30622 | Default payment term with deposit | 11.3 | Author-Unresponsive | PING_AUTHOR | Need-more-info; eldy asked, silent. |
| #30559 | /contract/expired REST endpoint | 11.3 | Needs-Discussion | MAINTAINER_REVIEW | Discussion+conflict; needs design+rebase. |
| #30955 | New form/table helpers in html.form.class | 11.3 | Needs-Discussion | MAINTAINER_REVIEW | Large API refactor; approach open. |
| #31049 | Filter by customer sellers | 11.3 | Author-Unresponsive | PING_AUTHOR | See-feedback; frederic34 commented. |
| #29557 | Fix search-all hidden input retained | 11.3 | Needs-Discussion | MAINTAINER_REVIEW | Author asks "delete code?"; needs ruling. |
| #29387 | Accounting number for foreign customers | 11.3 | Author-Unresponsive | PING_AUTHOR | See-feedback, no follow-up. |
| #30143 | Prioritize object contact in mail-to | 11.3 | Needs-Discussion | MAINTAINER_REVIEW | Code vs user bug debated. |
| #30327 | WIP packing.php page | 11.3 | Reimplement | REIMPLEMENT | Abandoned WIP; salvage fresh. |
| #30267 | Add company type on filter | 11.3 | Author-Unresponsive | PING_AUTHOR | See-feedback, no response. |
| #30194 | Fix create-invoice-after-payment error | 11.3 | Author-Unresponsive | PING_AUTHOR | See-feedback; never returned. |
| #30407 | Sign functionality on orders | 11.3 | Needs-Rebase/CI | REBASE_CI | Conflict/CI; author absent. |
| #29669 | Constant to not truncate IBAN in PDF | 11.3 | Needs-Rebase/CI | REBASE_CI | Conflict/CI; no activity. |
| #30036 | Factoring fetchObjectByElement | 11.3 | Needs-Discussion | MAINTAINER_REVIEW | QUAL refactor; needs buy-in. |
| #30000 | Duplicate ref_client to intervention | 11.3 | Ready-ish | MAINTAINER_REVIEW | Tiny, feedback addressed. |
| #26566 | Sort/separate linked objects | 11.3 | Author-Unresponsive | PING_AUTHOR | 2023 see-feedback; author silent. |
| #26886 | ModuleBuilder loop on $arrayfields | 11.3 | Needs-Discussion | MAINTAINER_REVIEW | WIP refactor w/ conflicts; design. |
| #26852 | Fix stock on supplier order disapproval | 11.3 | Needs-Discussion | MAINTAINER_REVIEW | 11-comment debate unresolved; real bug. |
| #23614 | Product label update on propal/invoice | 11.3 | Superseded | CLOSE | 2023 3-line, zero engagement. |
| #27590 | Remove WAREHOUSE_ASK_WAREHOUSE test | 11.3 | Needs-Discussion | MAINTAINER_REVIEW | One-line behavior change; Discussion. |
| #28184 | Add dolistore_id on module | 11.3 | Author-Unresponsive | PING_AUTHOR | eldy asked changes; no response. |
| #30943 | Pre-calculated ticket duration | 11.3 | Needs-Restructure | SPLIT | Mixing structure+code across 11 files. |
| #31012 | objectline_create hooks for extrafields | 11.3 | Author-Unresponsive | PING_AUTHOR | See-feedback; no follow-up. |
| #34804 | Sender mail setting for holiday | 11.2 | Needs-Discussion | MAINTAINER_REVIEW | Depends on #34802; needs decision. |
| #34918 | Update pdf_blochet (invoice list) | 11.1 | Needs-Rebase/CI | REBASE_CI | Conflict/CI; no activity. |
| #34674 | ModuleBuilder non-direct CommonObject | 11.1 | Needs-Discussion | MAINTAINER_REVIEW | Reasonable fix flagged Discussion. |
| #34826 | Avoid duplicate-entry on empty unique field | 11.1 | Needs-Discussion | MAINTAINER_REVIEW | Small fix; needs validation. |
| #30369 | Select parts via lot/serial in shipment | 10.9 | Ready-ish | MAINTAINER_REVIEW | Author provided detail; ball w/ maintainer. |

### バッチ4（#35198〜#35544）

| PR | Title (short) | 停滞(mo) | Category | Action | Reason |
|---|---|---|---|---|---|
| #35198 | Auto-gen SN/Lot in MO & Reception | 10.4 | Needs-Rebase/CI | REBASE_CI | Conflict/CI; needs rebase. |
| #35233 | Untitled #35232 | 10.3 | Needs-Discussion | MAINTAINER_REVIEW | 1992-line no-description diff; unclear. |
| #35220 | Revert Order↔Invoice-billed status link | 10.3 | Needs-Discussion | MAINTAINER_REVIEW | Targets released 19.0; not agreed. |
| #30596 | Introduce planned time on element | 10.1 | Needs-Discussion | MAINTAINER_REVIEW | 1087-line feature, ~2yr, no engagement. |
| #31645 | Add ECM on document download | 9.6 | Needs-Rebase/CI | REBASE_CI | Conflict/CI; rebase required. |
| #33606 | Unset fk_user_cloture on reopen | 9.4 | Ready-ish | MAINTAINER_REVIEW | 1-line clean fix; merge decision. |
| #34825 | Fix product/service image sort order | 9.4 | Needs-Discussion | MAINTAINER_REVIEW | 2-line fix, Discussion on v18. |
| #34360 | Fix supplier price calc + warnings | 9.4 | Postponed | PING_AUTHOR | Refacto into frozen 18.0; retarget develop. |
| #33747 | Fix rest column on invoice list | 9.4 | Author-Unresponsive | PING_AUTHOR | Mergeable but see-feedback; silent. |
| #29441 | Fix 18.0 member API | 9.1 | Needs-Rebase/CI | REBASE_CI | Conflict/CI; needs rebase. |
| #34283 | Add phone code to c_country.sql | 9.0 | Postponed | MAINTAINER_REVIEW | Labeled postponed; reactivate/close. |
| #35744 | Fix credit-note sign for gift cards | 8.9 | Needs-Discussion | MAINTAINER_REVIEW | total_ttc vs total_ht under discussion. |
| #33191 | Remain-to-pay 0 for paid invoice | 8.9 | Needs-Discussion | MAINTAINER_REVIEW | 20 comments, contentious; needs decision. |
| #34187 | Bill from contract: enabled lines only | 8.8 | Ready-ish | MAINTAINER_REVIEW | Small feature; awaits review. |
| #34037 | API to list/enable/disable modules | 8.8 | Needs-Rebase/CI | REBASE_CI | Conflict/CI; rebase needed. |
| #32682 | Fix bad MRP qty computation | 8.7 | Needs-Rebase/CI | REBASE_CI | Conflict/CI; needs rebase. |
| #34089 | Add afterapicall hook | 8.7 | Ready-ish | MAINTAINER_REVIEW | Small hook; needs review. |
| #34908 | Separate submit/delete perms in GED | 8.5 | Needs-Discussion | MAINTAINER_REVIEW | 6-comment; permission design open. |
| #35991 | Add bank-card banner URL display | 8.5 | Author-Unresponsive | PING_AUTHOR | Need-more-info; author silent. |
| #31775 | Hook for credit-note creation params | 8.4 | Needs-Rebase/CI | REBASE_CI | Conflict/CI; needs rebase. |
| #36004 | Fix NewOnlineSign response format | 8.4 | Needs-Discussion | MAINTAINER_REVIEW | includes vs trim under discussion. |
| #36001 | Payments on several expense reports | 8.4 | Needs-Rebase/CI | REBASE_CI | Resubmit of #30920/#24884; rebase. |
| #35729 | API methods for shipment contacts | 8.3 | Needs-Rebase/CI | REBASE_CI | Conflict/CI; rebase required. |
| #35902 | Yearly/monthly totals on invoice templates | 8.1 | Needs-Discussion | MAINTAINER_REVIEW | Discussion; needs maintainer input. |
| #35030 | CATEGORY_SHOW_MENU / HIDE_EDIT settings | 8.1 | Needs-Discussion | MAINTAINER_REVIEW | Two settings vs one debated. |
| #36247 | Add expensereport classes | 8.0 | Needs-Rebase/CI | REBASE_CI | Conflict/CI; large diff, rebase. |
| #36165 | Perm to modify HRM/salary info | 7.9 | Needs-Discussion | MAINTAINER_REVIEW | Discussion; perm model review. |
| #36252 | API project filtering by user perms | 7.9 | Needs-Rebase/CI | REBASE_CI | Conflict/CI; rebase required. |
| #36304 | User-rights checks for holiday API | 7.8 | Needs-Discussion | MAINTAINER_REVIEW | Author unsure; needs input. |
| #36110 | VAT details on accounting-file export | 7.7 | Needs-Rebase/CI | REBASE_CI | CONFLICTING; needs rebase. |
| #36249 | Kanban view in WebPortal | 7.6 | Needs-Discussion | MAINTAINER_REVIEW | 623-line new feature; scope decision. |
| #36415 | API: elements linked to contact/user | 7.6 | Ready-ish | MAINTAINER_REVIEW | Additive API; needs review/merge. |
| #35325 | Merging projects | 3.5 | Needs-Rebase/CI | REBASE_CI | Conflict/CI; least stale. |
| #35544 | PROPALE_KEEP_CLOSE_NOTE param | 7.6 | Author-Unresponsive | PING_AUTHOR | See-feedback; changes not addressed. |

### バッチ5（#36435〜#28947）

| PR | Title (short) | 停滞(mo) | Category | Action | Reason |
|---|---|---|---|---|---|
| #36435 | generateInputFieldPassword max length | 7.6 | Needs-Discussion | MAINTAINER_REVIEW | One-line fix flagged Discussion. |
| #34372 | classmap autoloading | 7.5 | Needs-Discussion | MAINTAINER_REVIEW | Architectural change; approach open. |
| #36453 | Add mass-action hooks | 7.5 | Postponed | MAINTAINER_REVIEW | Postponed; revive/drop decision. |
| #36454 | Fix missing entity field + more bugs | 7.5 | Needs-Discussion | MAINTAINER_REVIEW | Discussion + conflict vs 22.0; bundles fixes. |
| #36466 | Fix workstation cost in MO cost | 7.5 | Ready-ish | MAINTAINER_REVIEW | Targeted bugfix; needs merge. |
| #36448 | Comment before first sending line in invoice | 7.5 | Needs-Discussion | MAINTAINER_REVIEW | Discussion; awaits decision. |
| #36476 | Holiday negative balance calculation | 7.4 | Author-Unresponsive | PING_AUTHOR | See-feedback; never responded. |
| #36183 | Hook for payments/CloseBill params | 7.4 | Ready-ish | MAINTAINER_REVIEW | Author replied; maintainer's court. |
| #35714 | module_part for controller override | 7.4 | Ready-ish | MAINTAINER_REVIEW | Author engaged, no blocking label. |
| #36478 | Fix default value on propal card | 7.3 | Needs-Discussion | MAINTAINER_REVIEW | Discussion; eldy last commented. |
| #36506 | Const to hide product not for sale/purchase | 7.3 | Needs-Discussion | MAINTAINER_REVIEW | Discussion; constant approach. |
| #36533 | Refactor product lot class attributes | 7.3 | Ready-ish | MAINTAINER_REVIEW | Tiny unreviewed refactor. |
| #36548 | Prepaid (Aligned month) invoice mode | 7.2 | Ready-ish | MAINTAINER_REVIEW | Self-contained feature; awaits review. |
| #35235 | Massaction increase/decrease customer price | 7.2 | Needs-Rebase/CI | REBASE_CI | Conflict/CI; FHenry last commented. |
| #36551 | Clean API opensurveysondages | 7.2 | Superseded | CLOSE | Duplicated by newer #36609. |
| #36609 | Clean API opensurveysondages (resubmit) | 7.1 | Needs-Rebase/CI | REBASE_CI | Newer OpenSurvey API PR; conflict/CI. |
| #36602 | Better default $pathtoscript in CRON | 7.1 | Postponed | MAINTAINER_REVIEW | Postponed; revive/drop. |
| #36513 | Cap 2 digits after decimal separator | 7.1 | Ready-ish | MAINTAINER_REVIEW | German-locale bugfix; validate/merge. |
| #30199 | STOCK_USE_WAREHOUSE_USAGE quarantine | 7.0 | Reimplement | REIMPLEMENT | Valuable but 2yr stale, unaddressed feedback. |
| #36634 | Budget column in project company table | 7.0 | Postponed | MAINTAINER_REVIEW | Postponed; needs decision. |
| #36615 | Ticket display section in admin page | 6.9 | Needs-Discussion | MAINTAINER_REVIEW | Discussion; exposing constants. |
| #36700 | getUserProjects API function | 6.7 | Needs-Rebase/CI | REBASE_CI | Conflict/CI; no discussion. |
| #30481 | Fix virtual stock const usage | 6.4 | Postponed | PING_AUTHOR | Targets frozen 14.0; retarget. |
| #36502 | DST tolerance in holiday date validation | 6.3 | Needs-Discussion | MAINTAINER_REVIEW | Discussion; awaits design call. |
| #36732 | Select default customer type | 6.3 | Needs-Discussion | MAINTAINER_REVIEW | Discussion; needs decision. |
| #36570 | Link member to third party (breaking) | 6.3 | Needs-Discussion | MAINTAINER_REVIEW | Breaking API + conflict; needs call. |
| #36796 | Skill list on user skill tabs | 6.3 | Needs-Discussion | MAINTAINER_REVIEW | Discussion; awaits design. |
| #36786 | Permissions for batchlot management | 6.3 | Ready-ish | MAINTAINER_REVIEW | New perm set, no blocking label. |
| #36245 | Fixes backported from v22 | 6.2 | Needs-Restructure | SPLIT | "Split into several PRs"; unrelated fixes. |
| #35558 | Fix RELOAD_PAGE_ON_CUSTOMER_CHANGE | 6.2 | Ready-ish | MAINTAINER_REVIEW | v18 backport bugfix; awaits review. |
| #36716 | Hide BAN payment on Sponge template | 6.2 | Needs-Discussion | MAINTAINER_REVIEW | Discussion; option needs sign-off. |
| #36553 | Fix margin rounding / division by zero | 6.2 | Ready-ish | MAINTAINER_REVIEW | v18 bugfix; awaits review. |
| #36492 | Warning errors on opensurvey | 6.1 | Needs-Discussion | MAINTAINER_REVIEW | Discussion, no description. |
| #28947 | Cumuled graph on invoice box | 6.0 | Needs-Rebase/CI | REBASE_CI | 2yr stale, conflicting; small. |

> 生データは `prs_full.json`（ラベル・停滞期間・最終コメント者・base branch・変更規模・本文抜粋）として保持。
</content>
