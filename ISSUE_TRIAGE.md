# Dolibarr Upstream Issue Triage レポート

- **対象リポジトリ:** [Dolibarr/dolibarr](https://github.com/Dolibarr/dolibarr)（upstream）
- **調査日:** 2026-07-14
- **最終更新:** 2026-07-17（調査日〜本日の差分を反映。Close済みIssueを本レポートから除外し、新規Open Issueを追加。2026-07-17に #37826 のClose（PR #39191 マージ）を追加除外）
- **調査対象:** Open な **Issue のみ**（Pull Request は除外）を調査日時点で **全598件** 精査（全数調査）。うち **13件がその後Close** されたため除外し、**新規Open 2件を追加**、**現在 587件**
- **対象期間:** 2018-04-17 〜 2026-07-15（全期間）
- **手法:** GitHub API で全 Open Issue を取得し、12チャンクに分割して並列に本文・ラベル・コメント数を精査・分類。2026-07-16に調査日以降のClose/新規Issueを再取得し差分反映

> 分類は各 Issue のタイトル・本文・ラベル・コメント数をもとに行いました。実装の可否や重大度は本文記載の再現手順・根本原因の精度から推定したものです。最終判断は各 Issue のフルスレッド確認を推奨します。
>
> **注:** 本レポートは当初「直近200件」を対象に作成し、その後 **残り398件（〜2018年の古いIssue）を追加分類**して全598件をカバーしました。第2〜7章の優先度リストは全598件から抽出した代表例です。全件の分類明細は付録B（12バッチ）にあります。

> **2026-07-16 差分反映（調査日 2026-07-14 → 本日）:** 本レポートは「今後の実装の参考にするため、**現在Openな Issue のみ**」を掲載する方針です。調査日以降にClose済みとなった以下 **13件** を全章・付録から除外しました。
> - Close済み（除外）: #38963（FCKeditorアクセス制御）, #38949（TakePOS権限）, #38957 / #38958（仕入多通貨）, #38401（TakePOS Cash Control）, #38860（LDAP bind PHP8.3）, #39113（CSVヘッダHTML）, #34969（escpos画像）, #34682（PgSQL string_agg）, #34720（銀行振込重複）, #37661（price_base_type NULL）, #32407（辞書未翻訳）, #37826（load_tva() country_code／PR #39191）
> - 新規Open（追加）: #39190（Time Spent請求で単価が誤り／Bug）, #39187（クレジットノート作成時の税・修正の課題／Feature request）
>
> ※ Close分の多くは対応PRのマージによるものです（例: #38963→PR#39160, #38860→PR#39167, #39113→PR#39151）。停滞PRレポート(PR_TRIAGE.md)側は本期間にClose/mergeされた対象PRはありませんでした。
>
> **2026-07-16 追加クリーンアップ:** 現在Openな全Issue(590件)と全数照合し、**主エントリはすべてOpen**であることを確認（新規Close無し）。加えて、Open Issueの説明内に相互参照として残っていた **Close済みIssue 5件**（#13010, #15831, #18079, #24377, #35660 — いずれも2021〜2026-06過去にClose）への言及を除去しました（ホスト側の #32115・#32379・#20670・#23186・#36841 はOpenのため存置）。特に #36841 は重複相手のv22ブロッカーが修正済みのため「現行版で再現するか要確認」と注記。

---

## 1. エグゼクティブサマリー（全598件）

Open Issue **全598件** を「種別(Type)」と「対応方針(Action)」の2軸で分類しました（数値は概算）。

### 対応方針(Action)別の分布

| 対応方針 | 件数 | 割合 | 意味 |
|---|---:|---:|---|
| **NEEDS_IMPL**（実装/修正が必要） | ~332 | 55% | 根本原因や仕様が明確で、コード変更に着手できる |
| **NEEDS_DISCUSSION**（要議論/設計） | ~182 | 30% | 大きめの機能・データモデル変更で、メンテナの方針決定が先 |
| **NEEDS_INFO**（追加情報待ち） | ~47 | 8% | 再現手順・バージョン・ログ不足。報告者への返信が必要 |
| **CLOSE_BY_ANSWER**（回答でClose可能） | ~19 | 3% | 質問・告知・仕様範囲外。返信すればClose可能 |

### 種別(Type)別の分布

| 種別 | 件数 | 割合 |
|---|---:|---:|
| Bug-Actionable（着手可能なバグ） | ~219 | 36% |
| Feature-Small（小規模な機能要望） | ~191 | 31% |
| Feature-Large（大規模な機能要望） | ~100 | 16% |
| Bug-NeedsInfo（情報不足のバグ） | ~44 | 7% |
| Question/Support（質問・サポート） | ~13 | 2% |
| Invalid/Stale（無効・陳腐化） | ~8 | 1% |
| Bug-CantRepro（再現困難） | ~4 | 1% |
| Duplicate?（重複の疑い） | ~3 | 1% |

> **バグ vs 機能要望:** Bug系 合計 約271件（45%）、Feature系 合計 約291件（49%）。特に古いIssue（398件）では機能要望が約半数を占め、**未実装の機能要望が長期滞留**しているのがバックログの主因。

### 主要な所見（全598件を踏まえて）

1. **バグ報告の質が高い。** バグ系のうち約8割が根本原因やファイル/行番号まで特定済みで、すぐ着手できる状態（NEEDS_IMPL）。情報不足で差し戻すべきものは1割程度と少ない。
2. **セキュリティ Issue が複数放置されている（最重要）。** SQLインジェクション、IDOR、アクセス制御の無効化、古いCKEditor同梱の既知脆弱性など、調査日時点で **12件** を確認（うち #38963・#38949 の2件はその後修正Close、**現在Openは10件**）。**最優先での対応が必要**（第2章）。
3. **会計・税・在庫のデータ破損/計算誤り系が非常に多い。** VATレポート、多通貨（金額のint切り捨て多発）、状況請求書、BOM/MRP、TakePOS 現金管理、棚卸など、金額・在庫の正確性に関わるバグが全期間を通じて目立つ（第3章）。
4. **規制・コンプライアンス対応の期限付き案件が集中。** e-Invoicing（FacturX/ZUGFeRD #30078 はコメント53件）、スペインVeriFactu(2027)、ベルギーVAT(2026)、SEPA構造化アドレス(2026/11)、フランスNAF(2027)など、**法定期限のある大型案件**が複数（第6章）。
5. **「回答してClose」できる純粋な質問は少ない（約19件）。** トラッカーは主にバグ/機能要望に使われ、使い方の質問はフォーラムに流れている模様。Close の主戦場は「情報不足の差し戻し」と「小さな修正のマージ」「陳腐化Issueの整理」。
6. **真に陳腐化したIssueは意外に少ない。** 2018〜2023年の古いIssueも多くが更新され続けており、明確にClose可能な陳腐化候補は約7件のみ（第7章）。古い＝閉じてよい、ではない点に注意。
7. **報告の偏り。** 直近200件中 **57件が単一の報告者(JonBendtsen)** によるイベント運営(Event Organization)モジュール関連の機能要望。まとめて設計レビュー（RFC化）する価値あり。
8. **一次トリアージの遅延。** 全体で無ラベル約50件、コメントゼロ（未着手）が約190件。一次対応（ラベル付け＋一次返信）が追いついていない。

---

## 2. 🔴 最優先: セキュリティ Issue（即対応推奨）

これらは情報漏洩・不正操作に直結します。公開トラッカー上で詳細が晒されているため、**速やかな修正とリリース**を推奨します。

| # | 内容 | 深刻度 | 対応 |
|---|---|---|---|
| [#38768](https://github.com/Dolibarr/dolibarr/issues/38768) | 4つのREST APIエンドポイントで `sqlfilters` 経由のSQLインジェクション（UNIONでDB全読み出し可能）。sink特定済み | 致命的 | NEEDS_IMPL |
| [#38947](https://github.com/Dolibarr/dolibarr/issues/38947) | WebPortal `viewimage` にIDOR。他社のドキュメントが取得可能なデータ漏洩 | 致命的 | NEEDS_IMPL |
| [#37842](https://github.com/Dolibarr/dolibarr/issues/37842) | `productlot_note.php` のアクセス制御チェックが全てコメントアウト | 高 | NEEDS_IMPL |
| [#37935](https://github.com/Dolibarr/dolibarr/issues/37935) | ログインエンドポイントがGETを受理し、認証情報がURL/Referer/ログに漏洩 | 高 | NEEDS_IMPL |
| [#37881](https://github.com/Dolibarr/dolibarr/issues/37881) | Propalリストの権限判定不備。他ユーザーの提案書が見える権限リーク | 中〜高 | NEEDS_IMPL |
| [#38859](https://github.com/Dolibarr/dolibarr/issues/38859) | グループ説明フィールドに保存型HTMLインジェクション（サニタイズ漏れ） | 中 | NEEDS_IMPL |
| [#32359](https://github.com/Dolibarr/dolibarr/issues/32359) | 同梱の CKEditor 4.22 に既知の脆弱性（EOL）。更新/緩和が必要 | 中〜高 | NEEDS_IMPL |
| [#34659](https://github.com/Dolibarr/dolibarr/issues/34659) | ユーザー作成時のAPIキー生成が弱く、重複キー（unique制約違反）が発生 | 中 | NEEDS_IMPL |
| [#34323](https://github.com/Dolibarr/dolibarr/issues/34323) | `RESTRICTHTML_ONLY_VALID_HTML` 有効時にDOM/preg_replace処理が壊れる（サニタイズ経路） | 中 | NEEDS_IMPL |
| [#34223](https://github.com/Dolibarr/dolibarr/issues/34223) | extrafield可視性で `$_SERVER` がブロックされ、正規利用が破綻（要ポリシー判断） | 低〜中 | NEEDS_DISCUSSION |

> 補足1: `Priority 1 - Security` ラベルが付与されているのは全598件中わずか2件。上記は多くが未ラベルのため、**セキュリティラベルの付与漏れ**の可能性が高い。まず全件に `Priority 1 - Security` を付与し、非公開修正→まとめてリリースを推奨。
>
> 補足2: セキュリティ機能の要望（未実装）: WebPortal 2FA(#38224)、マジックリンク(#38108)、Passkey/WebAuthn(#35458)、リリースのPGP署名(#20673)、自己登録フォームのCaptcha(#25047)。これらは第6章の機能要望として別途扱い。

---

## 3. 🟠 重要な実データ系バグ（データ損失/会計・税・在庫の不正確）

セキュリティ以外で、金額・在庫・監査証跡など「壊れると業務影響が大きい」バグ。着手可能(NEEDS_IMPL)です。

### データ損失・破壊

| # | 内容 |
|---|---|
| [#37381](https://github.com/Dolibarr/dolibarr/issues/37381) | 出荷キャンセル時に `Expedition::cancel` が `deleteObjectLinked` を呼び、オブジェクト間リンクを消してしまう |
| [#38598](https://github.com/Dolibarr/dolibarr/issues/38598) | `{ttt}` マスク採番で連番判定がグローバルになり、請求書が恒久的に削除不能に |
| [#38672](https://github.com/Dolibarr/dolibarr/issues/38672) | 署名済み・注文にリンク済みの見積を削除でき、参照番号(ref)が再利用される |
| [#37790](https://github.com/Dolibarr/dolibarr/issues/37790) | v23アップグレードでマイグレーション欠落し、過去のCash Control残高が0表示に |

### 会計・税・多通貨の計算誤り

| # | 内容 |
|---|---|
| [#38061](https://github.com/Dolibarr/dolibarr/issues/38061) | クレジットノートの支払いがVATレポートから欠落し、申告値が不正確 || [#37519](https://github.com/Dolibarr/dolibarr/issues/37519) | 定期請求で仕入価格の小数が `(int)` キャストで切り捨て（データ破損） |
| [#38853](https://github.com/Dolibarr/dolibarr/issues/38853) / [#38849](https://github.com/Dolibarr/dolibarr/issues/38849) / [#38848](https://github.com/Dolibarr/dolibarr/issues/38848) | 状況請求書(situation)の保証金(warranty)まわりの合計・支払額が誤る |
| [#37398](https://github.com/Dolibarr/dolibarr/issues/37398) / [#37438](https://github.com/Dolibarr/dolibarr/issues/37438) / [#37566](https://github.com/Dolibarr/dolibarr/issues/37566) | 頭金が値引き扱いされる／状況請求のPU計算・PDF値が不正 |
| [#39190](https://github.com/Dolibarr/dolibarr/issues/39190) 🆕 | プロジェクトTime Spentからの請求生成で単価(UP Net)が誤り、ユーザー平均時間単価が使われず工数が多重請求される（v23.0.3、Bug） |

### 在庫・製造(MRP)

| # | 内容 |
|---|---|
| [#38378](https://github.com/Dolibarr/dolibarr/issues/38378) | 製造指図(MO)のPMP平均原価計算が `empty()` 判定の誤りで静かに壊れる |
| [#38352](https://github.com/Dolibarr/dolibarr/issues/38352) | 出荷削除APIがキット子品目の在庫戻しを忘れ、在庫がずれる |
| [#38177](https://github.com/Dolibarr/dolibarr/issues/38177) | `Mo::processBOM()` が凍結BOM行の数量を1固定にし、消費数量が誤る |
| [#38010](https://github.com/Dolibarr/dolibarr/issues/38010) | 製品キットの仮想在庫が表示されない |

### GDPR・規制・監査証跡

| # | 内容 | 対応 |
|---|---|---|
| [#38787](https://github.com/Dolibarr/dolibarr/issues/38787) / [#38785](https://github.com/Dolibarr/dolibarr/issues/38785) | datapolicy の匿名化条件が誤りで、GDPR匿名化が事実上一度も実行されない | NEEDS_IMPL |
| [#38581](https://github.com/Dolibarr/dolibarr/issues/38581) / [#38582](https://github.com/Dolibarr/dolibarr/issues/38582) | API close/クローン操作でイベント（監査ログ）が残らない | NEEDS_IMPL |
| [#38641](https://github.com/Dolibarr/dolibarr/issues/38641) | フランスの非EU向けVAT免除文言が誤り（法的根拠の記載ミス） | NEEDS_IMPL |
| [#38509](https://github.com/Dolibarr/dolibarr/issues/38509) | ドイツ語(オーストリア)の日付フォーマットが23.0.2以降で米国式に（ロケール回帰） | NEEDS_IMPL |
| [#38850](https://github.com/Dolibarr/dolibarr/issues/38850) | SEPA構造化アドレスが2026年11月から必須。XML生成の対応が必要（期限あり） | NEEDS_DISCUSSION |
| [#38728](https://github.com/Dolibarr/dolibarr/issues/38728) | フランスNAFコード2027年1月改定への辞書対応 | NEEDS_DISCUSSION |

### 導入・アップグレードを阻害するバグ（環境立ち上げ不能）

| # | 内容 |
|---|---|
| [#38634](https://github.com/Dolibarr/dolibarr/issues/38634) | `llx_categorie_supplier_proposal` テーブル欠落で仕入提案作成が致命的エラー |
| [#37437](https://github.com/Dolibarr/dolibarr/issues/37437) | 23.0新規インストールで `llx_categorie_propal` が無く、提案作成でDBエラー |
| [#37411](https://github.com/Dolibarr/dolibarr/issues/37411) | MySQL 5.7でインデックスにTEXT列があり `linktoref` のインストールがSQL 1170で失敗 |
| [#37508](https://github.com/Dolibarr/dolibarr/issues/37508) | `mysqliDoli` クラス未検出の致命的エラーでTLS DBアップグレードが不能 |
| [#38059](https://github.com/Dolibarr/dolibarr/issues/38059) | 会計モジュール有効化時にテーブルが作成されない |
| [#34094](https://github.com/Dolibarr/dolibarr/issues/34094) | `llx_categorie_extrafields` テーブル欠落でカテゴリ表示が致命的エラー |
| [#32802](https://github.com/Dolibarr/dolibarr/issues/32802) | 同梱の mike42/escpos が古くPHP 8.xでTypeError（レシート印刷不能） |

### 【古いIssueより】データ損失・会計・在庫の重要バグ

全期間の精査で見つかった、業務影響の大きいバグ（いずれもNEEDS_IMPL）。多通貨の金額int切り捨てが複数モジュールで頻発している点に注意。

| # | 内容 |
|---|---|
| [#30673](https://github.com/Dolibarr/dolibarr/issues/30673) | 請求書一覧からの多通貨支払いで保存金額が誤る（金額データ破損・最優先） |
| [#32605](https://github.com/Dolibarr/dolibarr/issues/32605) / [#31606](https://github.com/Dolibarr/dolibarr/issues/31606) | 仕入/多通貨支払い額が `(int)` キャストで小数切り捨て（金額損失、行番号特定済み） |
| [#33192](https://github.com/Dolibarr/dolibarr/issues/33192) | 棚卸クローズ時に理論数量が実数量で上書きされる（監査整合性の損失） |
| [#33180](https://github.com/Dolibarr/dolibarr/issues/33180) | 「非平衡取引」で会計転記が失敗し、年度末クローズがブロックされる |
| [#35207](https://github.com/Dolibarr/dolibarr/issues/35207) | ページネーション>20件で棚卸数量が保存されず失われる（データ損失） |
| [#35786](https://github.com/Dolibarr/dolibarr/issues/35786) | 状況請求書のクレジットノート条件で比較演算子が逆（`>`→`<`）。金額条件が誤る |
| [#35253](https://github.com/Dolibarr/dolibarr/issues/35253) / [#20876](https://github.com/Dolibarr/dolibarr/issues/20876) | 仕入請求で第2税(Tax2)が適用されない/欠落（税額の誤り、21.0回帰） |
| [#35658](https://github.com/Dolibarr/dolibarr/issues/35658) | 先頭ゼロの勘定科目コードがグループ集計から除外される（会計レポート誤り） |
| [#35382](https://github.com/Dolibarr/dolibarr/issues/35382) | 明細数によって地方税(localtax1)合計が変動する丸め誤差 |
| [#36899](https://github.com/Dolibarr/dolibarr/issues/36899) | TTCで0%VATの値引きが課税ベースを壊し、実効VAT率が過大に |
| [#36840](https://github.com/Dolibarr/dolibarr/issues/36840) / [#36841](https://github.com/Dolibarr/dolibarr/issues/36841) | 絶対額値引きで税額が0になる／請求書が検証不能に（22.0.2回帰、ブロッカー） |
| [#37074](https://github.com/Dolibarr/dolibarr/issues/37074) / [#36784](https://github.com/Dolibarr/dolibarr/issues/36784) | マイナス明細が符号反転／絶対値引きに化ける（金額の正確性、同根の可能性） |
| [#36838](https://github.com/Dolibarr/dolibarr/issues/36838) | mysqldumpに重複FK制約が出力され、DBリストアが破綻（復旧不能） |
| [#23557](https://github.com/Dolibarr/dolibarr/issues/23557) | IDNドメイン名の非ASCII文字がエラーなく破損保存（データ破損） |
| [#23201](https://github.com/Dolibarr/dolibarr/issues/23201) | 給与が会計に計上されず支払のみ記帳される（会計整合性の欠落） |
| [#33281](https://github.com/Dolibarr/dolibarr/issues/33281) | 病欠(sick leave)が誤って有給休暇日数から差し引かれる（HR計算の誤り） |
| [#32069](https://github.com/Dolibarr/dolibarr/issues/32069) | 定期請求cronが検証前のPROV請求（PDFなし）をメール送信（順序バグ） |
| [#30771](https://github.com/Dolibarr/dolibarr/issues/30771) | 未払リマインダcronの `nbdays` 判定が逆で、誤ったタイミングで顧客に督促 |
| [#31043](https://github.com/Dolibarr/dolibarr/issues/31043) / [#31233](https://github.com/Dolibarr/dolibarr/issues/31233) | ODT生成が19+で500エラー／`<` `>` を含むと文書が破損切り詰め |

---

## 4. 🟢 回答すればCloseできる Issue（Quick Wins）

### 4-1. 返信でClose可能（純粋な質問・告知・範囲外）

| # | 内容 | 推奨アクション |
|---|---|---|
| [#37720](https://github.com/Dolibarr/dolibarr/issues/37720) | `robot@domain.com`（チケット送信元アドレス）の変更方法が分からない | 設定場所（メール設定）を案内してClose |
| [#37610](https://github.com/Dolibarr/dolibarr/issues/37610) | Object Link 拡張フィールドのフィルタ構文が分からない | 構文をドキュメント案内してClose |
| [#37328](https://github.com/Dolibarr/dolibarr/issues/37328) | 添付前に「+」クリックが必要になったUI変更について | 仕様変更である旨を説明してClose |
| [#37689](https://github.com/Dolibarr/dolibarr/issues/37689) | 非公式のコミュニティHelm chartの告知 | 感謝＋非公式である旨で告知系Close |
| [#38331](https://github.com/Dolibarr/dolibarr/issues/38331) | EU AI Act / GDPR に関する注意喚起投稿 | Discussionへ誘導しClose |
| [#38356](https://github.com/Dolibarr/dolibarr/issues/38356) | AI MCPサーバ/チャットアシスタントのトラッキングスレッド | Discussion化 or トラッキング用に整理 |
| [#37843](https://github.com/Dolibarr/dolibarr/issues/37843) | Transifexに `fr_NC` 言語を追加してほしい | Transifex管理者作業＋返信（コード不要） |
| [#36985](https://github.com/Dolibarr/dolibarr/issues/36985) | ユーザーが独自APIスクリプトを共有（バグ報告ではない） | 感謝してClose |
| [#36645](https://github.com/Dolibarr/dolibarr/issues/36645) | 手動インストールでディレクトリ一覧が見える | Webサーバ設定(.htaccess/docroot)を案内してClose |
| [#36722](https://github.com/Dolibarr/dolibarr/issues/36722) | 複数注文から特定明細を選んで請求したい | 既存の部分請求機能を案内してClose |
| [#32738](https://github.com/Dolibarr/dolibarr/issues/32738) | Coolifyで新規インストール後にログインできない | 認証情報リセット手順を案内してClose |
| [#34998](https://github.com/Dolibarr/dolibarr/issues/34998) | インドGST/HSN設定の質問 | 設定方法を案内してClose |
| [#36516](https://github.com/Dolibarr/dolibarr/issues/36516) | SEPA CRDT/DBITフィルタの仕様質問 | 仕様を回答してClose |
| [#36446](https://github.com/Dolibarr/dolibarr/issues/36446) | 会員の誕生日メール自動送信の方法 | スケジュールジョブ設定を案内（フォーラム誘導） |
| [#32920](https://github.com/Dolibarr/dolibarr/issues/32920) | Europe/Kyivタイムゾーンで致命的エラー | OS側tzdata/ICU更新を案内（Dolibarr側の欠陥ではない） |
| [#33282](https://github.com/Dolibarr/dolibarr/issues/33282) | composer.lockを同梱してほしい | パッケージング方針を回答（likely won't-fix） |

### 4-2. 「マージすればほぼClose」の小さな修正（good-first-issue候補）

根本原因が特定済みで、数行〜小規模の修正で済むもの。着手すれば即Close可能です。

| # | 内容 |
|---|---|
| [#38926](https://github.com/Dolibarr/dolibarr/issues/38926) | 請求書一覧に「拡張フィールド一括編集」マスアクションを追加（注文と同様に配線） |
| [#38804](https://github.com/Dolibarr/dolibarr/issues/38804) | v23で選択行のハイライトが消えた回帰。`Form::showCheckAddButtons()` でJSがコメントアウト |
| [#38928](https://github.com/Dolibarr/dolibarr/issues/38928) | 拡張フィールドselectキーの非英数字が `aZ09` GETPOSTフィルタで削られる。`array:alphanohtml` へ |
| [#38930](https://github.com/Dolibarr/dolibarr/issues/38930) | `MAIN_ODT_AS_PDF_DEL_SOURCE=1` でODT元ファイルが消えない。`unlink()` をdownload前へ移動 |
| [#38867](https://github.com/Dolibarr/dolibarr/issues/38867) | `$dolibarr_allow_localurl_for_webhooks` をモジュールUIから設定可能に |
| [#39082](https://github.com/Dolibarr/dolibarr/issues/39082) | 経費精算承認メールを無効化するグローバル定数を追加 |
| [#35608](https://github.com/Dolibarr/dolibarr/issues/35608) | フォーム項目に `spellcheck="false"`（メンテナがgood-first-issue指定済み） |
| [#35072](https://github.com/Dolibarr/dolibarr/issues/35072) | `card_group.php` の `fistname` タイプミス（1行修正） |
| [#36133](https://github.com/Dolibarr/dolibarr/issues/36133) | Contributingドキュメントの文法/スペル修正（初PR向け） |
| [#36365](https://github.com/Dolibarr/dolibarr/issues/36365) | ドキュメントプレビューでXML対応（`dolIsAllowedForPreview` 拡張、1行） |
| [#35961](https://github.com/Dolibarr/dolibarr/issues/35961) | TakePOSフックのタイプミス（`$res==1` のネスト誤り）でボタン差替不可 |
| [#34653](https://github.com/Dolibarr/dolibarr/issues/34653) | 2エンティティでメニュー位置が同一（`$conf->entity` vs `$this->entity`、trivial） |
| [#32397](https://github.com/Dolibarr/dolibarr/issues/32397) | 辞書/イベントタイトルの未翻訳（小さな翻訳修正） |
| [#31375](https://github.com/Dolibarr/dolibarr/issues/31375) / [#29405](https://github.com/Dolibarr/dolibarr/issues/29405) | ドロップダウン/スキルの並び順修正 |

---

## 5. 🟡 追加情報待ち（NEEDS_INFO）— テンプレ返信で差し戻し

再現手順・バージョン・ログが不足しているもの。一定期間反応がなければ `Works for me / Can't reproduce` でClose候補。

| # | 内容 |
|---|---|
| [#39154](https://github.com/Dolibarr/dolibarr/issues/39154) | Stripe決済で「検証失敗」エラーが出るが決済自体は成功。ログ要 |
| [#39069](https://github.com/Dolibarr/dolibarr/issues/39069) | WebPortalエイリアスURLでメニュークリック時にセッション喪失（DNS/設定依存の可能性） |
| [#39045](https://github.com/Dolibarr/dolibarr/issues/39045) | VAT支払いの会計仕訳が1ヶ月ずれる（根本原因未確定、`need more info` ラベル済み） |
| [#38725](https://github.com/Dolibarr/dolibarr/issues/38725) | マイグレーション「実バージョン更新」でDB接続失敗（環境依存の可能性、conf要） |
| [#38389](https://github.com/Dolibarr/dolibarr/issues/38389) | 仕入請求のPDF結合が500エラー（エラーログ要） |
| [#38376](https://github.com/Dolibarr/dolibarr/issues/38376) | Windows上の製品PDFで文字化け（スクショ添付失敗・再現手順要） |
| [#38568](https://github.com/Dolibarr/dolibarr/issues/38568) | 参加者へのextrafield objectlinkが機能しない（バージョン/再現要） |
| [#38190](https://github.com/Dolibarr/dolibarr/issues/38190) | Ganttで親タスクがサブタスクにより勝手に展開（再現手順要） |
| [#37738](https://github.com/Dolibarr/dolibarr/issues/37738) | 「v20で安定、v21で壊れる(PT)」テンプレ空欄で情報ゼロ |
| [#37671](https://github.com/Dolibarr/dolibarr/issues/37671) | 発注書で税込単価が0表示（手順/DB情報要） |
| [#37618](https://github.com/Dolibarr/dolibarr/issues/37618) | 2回目の状況請求書で明細順が崩れる（バージョン/再現要） |
| [#37541](https://github.com/Dolibarr/dolibarr/issues/37541) | Belugaプロジェクトレポートで要素欠落（バージョン/設定要） |
| [#37537](https://github.com/Dolibarr/dolibarr/issues/37537) | インストール画面でWeb/DBフィールドが非活性（旧環境/ブラウザ依存の疑い） |
| [#37458](https://github.com/Dolibarr/dolibarr/issues/37458) | EU OSS VATルールが部分的にしか効かない（具体ケース要） |
| [#37368](https://github.com/Dolibarr/dolibarr/issues/37368) | PHP8.2でAPIのoptionalフィールド（失敗リクエスト具体例要） |
| [#37353](https://github.com/Dolibarr/dolibarr/issues/37353) | タグ付き連絡先がエクスポートできない（手順/エラー要） |
| [#37305](https://github.com/Dolibarr/dolibarr/issues/37305) | 分解(disassembly)が製造(MRP)のように振る舞う（再現/期待値要） |
| [#36428](https://github.com/Dolibarr/dolibarr/issues/36428) / [#36299](https://github.com/Dolibarr/dolibarr/issues/36299) / [#36288](https://github.com/Dolibarr/dolibarr/issues/36288) | テンプレ空欄・記述ほぼ無し（内容要求） |
| [#34870](https://github.com/Dolibarr/dolibarr/issues/34870) | スペイン語の小数/桁区切りパース誤り（大量の価格破損の訴え、再現手順要・高深刻度） |
| [#32510](https://github.com/Dolibarr/dolibarr/issues/32510) / [#32243](https://github.com/Dolibarr/dolibarr/issues/32243) | Stripe決済が登録されない／CSVダウンロードが無反応（ログ要） |
| [#34463](https://github.com/Dolibarr/dolibarr/issues/34463) | OpenAIキーでAIプロンプト無応答（Mistralは動作、レスポンス本文要） |
| [#33826](https://github.com/Dolibarr/dolibarr/issues/33826) | 出荷クローズで白画面（バージョン/ログ/再現要） |

> このほか、テンプレ空欄・タイトルのみの薄い報告が全体で20件以上。既存の `Bug or PR need more information` ＋ `Issue Stale (automatic label)` ボットと組み合わせ、無反応は自動Close運用を推奨。

---

## 5b. 🗑️ 陳腐化(Stale)クリーンアップ候補 — Close推奨

古いIssueを精査した結果、**明確に陳腐化してCloseすべきものは意外に少数**（全期間で約7件）でした。2018〜2023年のものでも多くは更新が続いており「古い＝閉じてよい」ではありません。以下は返信のうえClose推奨の候補です。

| # | 作成年 | 内容 | 理由 |
|---|---|---|---|
| [#14818](https://github.com/Dolibarr/dolibarr/issues/14818) | 2020 | ODTテンプレート対応のトラッキング表 | 起票者アカウント消失・内容陳腐化 |
| [#14295](https://github.com/Dolibarr/dolibarr/issues/14295) | 2020 | REST/UIのテスト作成（曖昧なgood-first-issue） | 2023以降進展なし・テスト基盤は既に刷新 |
| [#8612](https://github.com/Dolibarr/dolibarr/issues/8612) | 2018 | GDPR機能（包括タグ） | DataPolicyモジュールが実装済み。Close or 再スコープ |
| [#23186](https://github.com/Dolibarr/dolibarr/issues/23186) | 2021 | CardDAV/CalDAVのコア統合 | 外部モジュールで対応済み（コア統合は要議論） |
| [#30748](https://github.com/Dolibarr/dolibarr/issues/30748) | — | GitHub Discussions有効化 | プロジェクト運営提案（コード不要）。返信でClose |
| [#26421](https://github.com/Dolibarr/dolibarr/issues/26421) | 2023 | Ansible自動化 | 曖昧・既存APIで代替可・traction無し |
| [#32025](https://github.com/Dolibarr/dolibarr/issues/32025) | — | develop一時版のマイグレーションSQLエラー | 現行developでは修正済みの可能性大 |

> 補足: 自動 `Issue Stale` ラベルが付いた #34728, #34721, #34716, #25299, #25297 などは中身は**まだ有効な要望**。安易にCloseせず、un-staleを検討。

---

## 6. 📅 規制・コンプライアンス対応（期限付きの大型案件）

法定期限があり、計画的に着手すべき大型案件。多くが `NEEDS_DISCUSSION`（設計・ロードマップ化）です。

| # | 期限/地域 | 内容 |
|---|---|---|
| [#30078](https://github.com/Dolibarr/dolibarr/issues/30078) | EU（進行中） | FacturX/XRechnung/ZUGFeRD e-invoicing実装（**コメント53件**の最重要スレッド） |
| [#39120](https://github.com/Dolibarr/dolibarr/issues/39120) | EU | EN 16931 B2G e-invoicing のコアフィールド対応 |
| [#36628](https://github.com/Dolibarr/dolibarr/issues/36628) | スペイン 2027 | VeriFactuモジュール（Discussionラベル済・進行中） |
| [#36818](https://github.com/Dolibarr/dolibarr/issues/36818) | ベルギー 2026 | VATを明細ごとでなく合計で丸めるモード |
| [#38850](https://github.com/Dolibarr/dolibarr/issues/38850) | SEPA 2026/11 | 構造化アドレスが必須化。XML生成の改修 |
| [#38728](https://github.com/Dolibarr/dolibarr/issues/38728) | フランス 2027/01 | NAFコード改定への辞書対応 |
| [#31652](https://github.com/Dolibarr/dolibarr/issues/31652) | EU | ViDA/e-reporting向けデジタル出荷伝票 |
| [#35124](https://github.com/Dolibarr/dolibarr/issues/35124) | ISO 20022 | 構造化アドレス（番地分離）フィールド |

---

## 7. 機能要望(Feature Requests) — 全体で約291件（バックログの約半分）

機能要望は全598件の約半数（Feature-Small ~191、Feature-Large ~100）を占め、特に古いIssueに滞留しています。

### 7-1. 小規模・着手可能（Feature-Small / NEEDS_IMPL）— 代表例

すぐ実装できる well-scoped な要望。good-first-issue やコントリビュータ向けに割り当てやすい。

| # | 内容 |
|---|---|
| [#39187](https://github.com/Dolibarr/dolibarr/issues/39187) 🆕 | クレジットノート作成で税が0固定になる／誤ったクレジットノートを修正できない。税ごとに明細を分ける提案（スペインRecargo de Equivalenciaを含む、要議論） |
| [#38989](https://github.com/Dolibarr/dolibarr/issues/38989) | メール送信フォームでオブジェクトのリンク済みファイルを添付するチェックボックス |
| [#38044](https://github.com/Dolibarr/dolibarr/issues/38044) | 約100万行テーブルで前後ナビのクエリが遅い→scanlist無効時にナビ矢印を無効化 |
| [#38002](https://github.com/Dolibarr/dolibarr/issues/38002) | REST APIに請求書/支払いのDELETEが無い（UIには存在、ライフサイクル欠落） |
| [#37396](https://github.com/Dolibarr/dolibarr/issues/37396) | 受注で販売価格を隠す権限の追加 |
| [#23230](https://github.com/Dolibarr/dolibarr/issues/23230) | タスクを別プロジェクトへ移動（**コメント17件**の人気要望） |
| [#29244](https://github.com/Dolibarr/dolibarr/issues/29244) / [#28914](https://github.com/Dolibarr/dolibarr/issues/28914) | 連絡先/会員のマージ機能（thirdpartyのマージを踏襲） |
| [#31210](https://github.com/Dolibarr/dolibarr/issues/31210) | 選択した会員への一括メール送信（注文一覧と同様のマスアクション） |
| [#27300](https://github.com/Dolibarr/dolibarr/issues/27300) | Dolibarr送信メールをIMAP「送信済み」フォルダに保存 |
| [#31699](https://github.com/Dolibarr/dolibarr/issues/31699) | ODTテンプレートにSwiss QRタグ対応 |

> テーマ別に集中: **CSVインポートの項目追加**（#31416, #31304, #26971, #21437, #35113 等）、**拡張フィールド強化**（#34906 色選択, #38242, #35476 等）、**経費精算**（#38652, #38635, #38630, #38629, #38273 等）。テーマ単位でまとめて実装すると効率的。

### 7-2. 大規模・要議論（Feature-Large / NEEDS_DISCUSSION）— 約100件

メンテナの方針決定・設計合意が必要。テーマ別にまとまっており、**テーマ単位でのRFC化**を推奨。

- **イベント運営(Event Organization)モジュール強化**（最大クラスタ、多くが単一報告者由来）: 参加者の入替/クローン/連絡先化/タスク化/文書生成/一括メール — #38127, #37954, #38241, #38250, #37749, #38183, #37729, #37948 ほか多数
- **タスク/プロジェクト管理の刷新**: ツリー表示・テンプレート・色分け・メタタスク・未割当・複数タスク一括編集 — #38191, #38192, #38187, #38188, #38200, #31150
- **会計/価格の高度化**: 損益・貸借・キャッシュフロー帳票(#31760), 期間別価格(#32380/#30202), 多通貨価格(#32379), 前受金/前払金(#30438), 予算機能(#38829)
- **生産(MRP)/在庫**: MRPロードマップ(#19661, コメント24件), ワークステーション計画(#36721/#36224), 倉庫ロケーション概念(#31883), 返品/交換機能(#16491/#34012), 複数ロット受入(#23957)
- **アーキテクチャ級**: UUIDv7/ULID採用(#37891), TCPDF→tc-lib-pdf移行(#38639), Composer導入(#35108), Symfonyコンポーネント化(#34767), CommonListクラス(#21268), ドキュメントのRead the Docs移行(#38554), JS Contextフレームワーク(#37993/#37991/#37990)
- **認証/セキュリティ機能**: WebPortal 2FA(#38224), マジックリンク(#38108), Passkey(#35458), API impersonation(#37652), MS OAuth2(#37137), バックアップ暗号化(#32085), リリースPGP署名(#20673)
- **統合/連携**: 決済ゲートウェイ(Paystack/Flutterwave #36130), CalDAV/CardDAV(#23186/#29664), メールテンプレートAPI(#26550)

---

## 8. 重複・無効

| # | 種別 | メモ |
|---|---|---|
| [#38661](https://github.com/Dolibarr/dolibarr/issues/38661) | Duplicate? | #38662 と同一著者・同テーマ（ダッシュボード/レポートの別バリアント） |
| [#37605](https://github.com/Dolibarr/dolibarr/issues/37605) | Duplicate? | #37415 / #37583 と同じ `email_sent_counter` 回帰。まとめて対応 |
| [#38331](https://github.com/Dolibarr/dolibarr/issues/38331) | Invalid/Stale | 注意喚起投稿。Discussionへ |
| [#38356](https://github.com/Dolibarr/dolibarr/issues/38356) | Invalid/Stale | トラッキングスレッド |
| [#37689](https://github.com/Dolibarr/dolibarr/issues/37689) | Invalid/Stale | 非公式chart告知 |
| [#28914](https://github.com/Dolibarr/dolibarr/issues/28914) | Duplicate? | #29244（連絡先マージ）と同パターンの会員マージ。同時対応 |
| [#23186](https://github.com/Dolibarr/dolibarr/issues/23186) | Feature-Large | 外部にCalDAV/CardDAVモジュールが存在（コア統合は未実装） |

> **関連クラスタ（要統合）:**
> - `email_sent_counter` / `isEditable` の請求書再編集回帰 → #37415・#37583・#37605（コメント多数）を1つのマスターIssueに集約推奨。
> - 絶対額値引きの請求書検証ブロック → #36840・#36841（関連する22.0系ブロッカーは修正済みのため、現行版で再現するか要確認）。
> - マイナス明細の符号反転 → #37074・#36784 は同根の可能性。
> - 多通貨金額のint切り捨て → #32605・#31606・#30673 は共通の根本原因の可能性。

---

## 9. 推奨される次のアクション（トリアージ運用）

1. **セキュリティ最優先（第2章の現在Open 10件）:** 全件に `Priority 1 - Security` を付与し、非公開修正→まとめてセキュリティリリース。公開トラッカーで詳細が晒されている点に留意。同梱ライブラリの更新（CKEditor #32359, escpos #32802）も含む。
2. **規制・期限案件のロードマップ化（第6章）:** e-Invoicing(#30078)、VeriFactu(2027)、ベルギーVAT(2026)、SEPA(2026/11)、NAF(2027)は法定期限があるため、期限から逆算して着手計画を立てる。
3. **回帰(Regression)バグの優先対応:** v19〜v23アップグレードに伴う回帰が多数（#38804, #38402, #38509, #38909, #37790, #37415, #34075, #33578, #33167, #31209, #31043 等）。直近リリースの品質に直結。
4. **データ損失・会計誤りバグの優先対応（第3章）:** 多通貨のint切り捨て、棚卸数量喪失、税額誤り、年度末クローズ不能など、業務影響が大きい。
5. **導入阻害バグの即修正:** 新規インストール/アップグレードを止めるバグ（#38634, #37437, #37411, #37508, #34094 等）は新規ユーザー獲得に直接影響。
6. **Quick Wins の消化（第4章）:** 質問系（約17件）は即返信Close、trivialな小修正（約15件）はgood-first-issueとして募集し Open Issue 数を圧縮。
7. **陳腐化Issueの整理（第5b章）:** 明確な陳腐化候補7件をClose。ただし古い＝Closeではない点に注意し、個別確認する。
8. **NEEDS_INFO の自動差し戻し（第5章、約47件）:** テンプレ返信＋`Bug or PR need more information` ラベル。既存の `Issue Stale (automatic label)` ボットと組み合わせ、無反応は自動Close。
9. **Feature-Large のRFC化（第7-2章）:** 特にイベント運営モジュール（単一報告者由来の大量要望）は、個別Issueを1本のロードマップIssueに束ねて設計レビュー。
10. **一次トリアージの底上げ:** 全体でコメントゼロ（未着手）が約190件。ラベル付け＋一次返信のボランティア割当を検討。

---

## 付録A: 分類基準

**種別(Type):**
- `Question/Support` 使い方・設定の質問（コード欠陥ではない）
- `Bug-Actionable` 再現手順・情報が揃った着手可能なバグ
- `Bug-NeedsInfo` 問題報告だが再現手順/バージョン/ログ不足
- `Bug-CantRepro` 環境依存・曖昧で再現困難
- `Feature-Small` 小規模でスコープ明確な機能追加
- `Feature-Large` 設計・議論が必要な大規模機能
- `Duplicate?` 既存の頻出要望と重複の疑い
- `Invalid/Stale` スパム・話題外・告知・陳腐化

**対応方針(Action):**
- `CLOSE_BY_ANSWER` 返信のみで解決（質問・回避策あり・won't-fix）
- `NEEDS_INFO` 着手前に報告者へ再現情報を要求
- `NEEDS_IMPL` 実コードの変更/修正が必要
- `NEEDS_DISCUSSION` 着手前にメンテナの方針/設計決定が必要

## 付録B: 全件の分類表（調査日 598件 → Close 13件除外・新規 2件追加で現在 約587件）

作成日の新しい順（50件区切り、12バッチ）で全件を掲載。各行の「Reason」列に分類根拠を記載。バッチ1〜4が直近200件、バッチ5〜12が残り398件（〜2018年）。

### バッチ1（#39190〜#38658）

| # | Title (short) | Type | Action | Reason |
|---|---|---|---|---|
| #39190 🆕 | Time Spent invoice uses wrong unit price (not user avg rate) | Bug-Actionable | NEEDS_IMPL | 2026-07-15 新規。Duration billed multiple times; UP Net wrong on invoice-from-time-spent. |
| #39187 🆕 | Credit note: tax not applied / can't fix a mistaken credit note | Feature-Small | NEEDS_DISCUSSION | 2026-07-15 新規。Credit-note-with-remaining-unpaid defaults tax to 0; Spain Recargo edge case. |
| #39154 | Stripe payment verification failed error | Bug-NeedsInfo | NEEDS_INFO | Intermittent Stripe error but payments actually succeed; needs logs/repro. |
| #39147 | Expense report file not auto-generated after validation | Bug-Actionable | NEEDS_IMPL | Doc should auto-generate on validation but requires manual click. |
| #39146 | Multicompany: payment account not saved across entities | Bug-Actionable | NEEDS_DISCUSSION | fk_account not entity-aware; fix needs design on sharing. |
| #39143 | Fix computed extrafields (MAIN_STORE_COMPUTED_EXTRAFIELDS) | Bug-Actionable | NEEDS_IMPL | Root cause + fix for computed extrafield storage/display. |
| #39120 | Native EN 16931 B2G e-invoicing core fields | Feature-Large | NEEDS_DISCUSSION | Broad new data model; needs design and buy-in. |
| #39082 | Option to disable expense-report-approved email | Feature-Small | NEEDS_IMPL | Add a global constant to skip hardcoded notification. |
| #39078 | Member child tags don't inherit from parent tag | Bug-Actionable | NEEDS_IMPL | Parent-tag filter omits members with child tags. |
| #39069 | Web portal alias URL loses session on menu click | Bug-NeedsInfo | NEEDS_INFO | Config/DNS-alias-specific; needs setup clarification. |
| #39047 | Supplier invoice template: multicurrency price = 0 | Bug-Actionable | NEEDS_IMPL | Foreign unit price not forwarded to addline() in card-rec.php. |
| #39046 | CashControl daily period saved as next day (UTC- TZ) | Bug-Actionable | NEEDS_IMPL | tzuserrel vs gmt mix; repro + code location. |
| #39045 | Wrong accountancy entries for VAT payments (1-month offset) | Bug-NeedsInfo | NEEDS_INFO | Offset symptom lacks confirmed root cause. |
| #39035 | Ticket timeline message disappears on "Read more" | Bug-Actionable | NEEDS_IMPL | Unclosed div from dolGetFirstLineOfText() truncation. |
| #39020 | Duplicate MO when BOM has sub-BOM | Bug-Actionable | NEEDS_IMPL | Child MO duplicates parent instead of sub-assembly MO. |
| #39017 | Broken ru_RU/uk_UK translations | Bug-Actionable | NEEDS_IMPL | Concrete mistranslations + dropped placeholders. |
| #39016 | Partial delivery/invoicing for services | Feature-Large | NEEDS_DISCUSSION | Fundamental change to service delivery/invoicing. |
| #38989 | Attach existing linked documents to email | Feature-Small | NEEDS_IMPL | Checkboxes to attach object's linked files. |
| #38982 | Resend any email sent to a thirdparty/contact | Feature-Large | NEEDS_DISCUSSION | Reusing past emails as templates; needs design. |
| #38978 | Event tab default view option (agenda vs messaging) | Feature-Small | NEEDS_IMPL | Small opt-in setting, ideally per-user. |
| #38967 | Currency rates need 6+ significant decimals | Bug-Actionable | NEEDS_DISCUSSION | Touches price2num/constants broadly; needs decision. |
| #38961 | More filters in product referer lists | Feature-Small | NEEDS_IMPL | Add company/invoice-type filters and hooks. |
| #38947 | IDOR in webportal viewimage controller | Bug-Actionable | NEEDS_IMPL | Security: portal serves any company's documents. |
| #38930 | ODT source not deleted (MAIN_ODT_AS_PDF_DEL_SOURCE=1) | Bug-Actionable | NEEDS_IMPL | unlink() after a throwing block; move before download. |
| #38928 | Extrafield select keys with non-alnum chars stripped | Bug-Actionable | NEEDS_IMPL | aZ09 GETPOST filter; change to array:alphanohtml. |
| #38926 | Invoice list missing "Modify extrafields" mass action | Feature-Small | NEEDS_IMPL | Wire edit_extrafields into facture/list.php. |
| #38919 | PDF loses line breaks with < or > chars | Bug-Actionable | NEEDS_IMPL | &lt;/&gt; treated as markup, breaking line breaks. |
| #38909 | V22 delivery date not recorded on supplier order receipt | Bug-Actionable | NEEDS_IMPL | Regression vs v21: stores current date instead. |
| #38867 | Make webhook localurl flag configurable in module UI | Feature-Small | NEEDS_IMPL | Expose $dolibarr_allow_localurl_for_webhooks. |
| #38859 | Stored HTML injection in group description field | Bug-Actionable | NEEDS_IMPL | Security: plain-text field not sanitized. |
| #38853 | Retained warranty not correctly managed (totals wrong) | Bug-Actionable | NEEDS_IMPL | Situation-invoice warranty totals wrong; repro given. |
| #38850 | SEPA structured addresses mandatory from Nov 2026 | Feature-Large | NEEDS_DISCUSSION | Regulatory change; needs design + deadline planning. |
| #38849 | Situation invoice: "To be paid" wrong on last situation | Bug-Actionable | NEEDS_IMPL | Warranty wrongly subtracted from to-be-paid. |
| #38848 | Warranty selection button on final situation no effect | Bug-Actionable | NEEDS_IMPL | Selector ignored; warranty always applied. |
| #38829 | Detailed budget functionality incl. liquidity | Feature-Large | NEEDS_DISCUSSION | Large new budgeting module; needs design. |
| #38804 | Selected rows no longer highlighted in v23 lists | Bug-Actionable | NEEDS_IMPL | Regression; highlight JS commented out. |
| #38787 | datapolicy: customer+supplier excluded from policies | Bug-Actionable | NEEDS_IMPL | fournisseur=0 conditions misroute; anonymization fails. |
| #38785 | datapolicy: NOT EXISTS(llx_facture) over-excludes | Bug-Actionable | NEEDS_IMPL | Replace absolute NOT EXISTS with recency check. |
| #38768 | SQL injection via sqlfilters in 4 REST API endpoints | Bug-Actionable | NEEDS_IMPL | Security: UNION exfiltration; sinks pinpointed. |
| #38728 | French NAF codification change Jan 2027 | Feature-Large | NEEDS_DISCUSSION | Regulatory dictionary change; needs planning. |
| #38725 | Migration "update actual version" fails to connect DB | Bug-NeedsInfo | NEEDS_INFO | Env/config-specific; needs logs/conf. |
| #38686 | Point-of-tax delivery date should be mandatory when used | Feature-Small | NEEDS_IMPL | Require delivery date on validate when constant set. |
| #38672 | Signed propal linked to order can be deleted | Bug-Actionable | NEEDS_IMPL | Data-integrity: deletion → ref reuse. |
| #38662 | Project stats focus on conference/booth attendees | Feature-Small | NEEDS_DISCUSSION | Niche reporting; vague scope. |
| #38661 | Project dashboard focus on conference/booth attendees | Duplicate? | NEEDS_DISCUSSION | Overlaps #38662. |
| #38658 | Pre-select email recipient by contact role | Feature-Small | NEEDS_IMPL | Opt-in constant mirroring postal-address role mapping. |

### バッチ2（#38652〜#38184）

| # | Title (short) | Type | Action | Reason |
|---|---|---|---|---|
| #38652 | Project expense report lines by status | Feature-Small | NEEDS_IMPL | UI grouping of expense lines by status. |
| #38651 | ODT-to-PDF fails on Windows/IIS (quoting/URI) | Bug-Actionable | NEEDS_IMPL | Clear root cause + fix for command quoting. |
| #38641 | Wrong VAT exemption mention (FR non-EU) | Bug-Actionable | NEEDS_IMPL | Root cause in pdf_crabe; compliance issue. |
| #38639 | TCPDF deprecated, migrate to tc-lib-pdf | Feature-Large | NEEDS_DISCUSSION | Library migration blocked by PHP version support. |
| #38635 | Expense reports title/summary line | Feature-Small | NEEDS_IMPL | Add a title field for lists/emails. |
| #38634 | Missing dbr_categorie_supplier_proposal table | Bug-Actionable | NEEDS_IMPL | Fatal error creating supplier proposal; missing table. |
| #38633 | Calendar line to pick dates | Feature-Small | NEEDS_DISCUSSION | UI widget idea needs UX design. |
| #38631 | Approve individual expense report lines | Feature-Large | NEEDS_DISCUSSION | Per-line approval workflow needs design. |
| #38630 | Allowed projects list per expense report | Feature-Small | NEEDS_IMPL | Scope project selection per report. |
| #38629 | Expense line date default to report period | Feature-Small | NEEDS_IMPL | Prefill next line date; small UX fix. |
| #38627 | Duplicate ref in llx_paiement (race condition) | Bug-NeedsInfo | NEEDS_DISCUSSION | Data-integrity concern; needs locking decision. |
| #38615 | API exportdata: no way to fetch file | Bug-Actionable | NEEDS_IMPL | Export file inaccessible to client. |
| #38598 | {ttt} mask invoices undeletable | Bug-Actionable | NEEDS_IMPL | Numbering continuity global not per-{ttt}. |
| #38582 | Cloning creates no event entry | Bug-Actionable | NEEDS_IMPL | Clone leaves no audit trace. |
| #38581 | API close proposal leaves no event trace | Bug-Actionable | NEEDS_IMPL | Status change not logged; audit gap. |
| #38568 | Extrafield objectlink to attendee broken | Bug-NeedsInfo | NEEDS_INFO | No version/env; unclear repro. |
| #38554 | Migrate docs to Read the Docs | Feature-Large | NEEDS_DISCUSSION | Docs pipeline overhaul; needs buy-in. |
| #38553 | Event Organization per-session registration | Feature-Large | NEEDS_DISCUSSION | Larger workflow/UI feature; needs design. |
| #38528 | Point of tax editable on validated invoices | Bug-Actionable | NEEDS_IMPL | Field should lock once invoice validated. |
| #38509 | Date format broken for German (Austria) | Bug-Actionable | NEEDS_IMPL | Locale regression since 23.0.2. |
| #38496 | Add Myanmar font support | Feature-Small | NEEDS_IMPL | Add Pyidaungsu font for TCPDF. |
| #38468 | Variants inherit categories/tags from parent | Feature-Small | NEEDS_IMPL | Inheritance for product variants. |
| #38424 | Massaction assign user/contact to tasks | Feature-Small | NEEDS_IMPL | Mass-assign action on project tasks. |
| #38421 | time.php search loses task id | Bug-Actionable | NEEDS_IMPL | Filter drops task scope; line numbers given. |
| #38402 | Missing REOPEN button in POS Cash Control | Bug-Actionable | NEEDS_IMPL | Regression from v19; lost functionality. |
| #38389 | Fusion PDF fails for supplier bills | Bug-NeedsInfo | NEEDS_INFO | No logs for 500 error. |
| #38378 | MO PMP averaging silently broken | Bug-Actionable | NEEDS_IMPL | Silent data corruption in empty() check. |
| #38376 | Garbled words in product PDF on Windows | Bug-NeedsInfo | NEEDS_INFO | Screenshot upload failed; no repro. |
| #38356 | AI MCP Server and chat assistant | Invalid/Stale | NEEDS_DISCUSSION | Tracking thread, not a defect. |
| #38352 | Shipment delete API forgets kit re-stock | Bug-Actionable | NEEDS_IMPL | Stock errors on API shipment deletion. |
| #38331 | EU AI Act and GDPR | Invalid/Stale | CLOSE_BY_ANSWER | Awareness post, not actionable. |
| #38328 | Design: roles for actioncomm resources | Feature-Large | NEEDS_DISCUSSION | Data-model design proposal. |
| #38286 | Massaction member subscription label | Feature-Small | NEEDS_IMPL | Add label text to mass subscription. |
| #38273 | Single date for service invoice line | Feature-Small | NEEDS_DISCUSSION | Needs design choice on approach. |
| #38252 | Volunteer shift & capacity management | Feature-Large | NEEDS_DISCUSSION | Large proposal spanning multiple tables. |
| #38250 | fk_contact on conferenceorboothattendees | Feature-Large | NEEDS_DISCUSSION | Schema change tied to event-org design. |
| #38245 | Send mail on tasks like tickets | Feature-Large | NEEDS_DISCUSSION | Multi-part feature; needs design. |
| #38242 | $fields arrayofkeyval support | Feature-Small | NEEDS_IMPL | Reuses extrafields parsing in showInputField(). |
| #38241 | Attendees as task contact | Feature-Large | NEEDS_DISCUSSION | Schema + API changes; needs design. |
| #38224 | WebPortal two-factor login | Feature-Large | NEEDS_DISCUSSION | Security feature needing design. |
| #38200 | Colours on objects (tasks/thirdparty/…) | Feature-Large | NEEDS_DISCUSSION | Broad cross-module UI feature. |
| #38192 | Task tree view for events | Feature-Large | NEEDS_DISCUSSION | New tree/status visualization. |
| #38191 | Task templates applied to project | Feature-Large | NEEDS_DISCUSSION | Templating subsystem for task trees. |
| #38190 | Gantt parent silently expanded by subtask | Bug-NeedsInfo | NEEDS_INFO | No version/repro; screenshots only. |
| #38189 | Warn on subtask outside parent dates | Feature-Small | NEEDS_IMPL | Add validation/warning. |
| #38188 | Meta/container tasks with colour | Feature-Large | NEEDS_DISCUSSION | New task type + colouring. |
| #38187 | Tasks assignable to Nobody/Unassigned | Feature-Large | NEEDS_DISCUSSION | Assignment model change. |
| #38186 | List/Gantt of subtasks from a task | Feature-Small | NEEDS_IMPL | Add subtasks tab/list view. |
| #38184 | Gantt zoom via scroll/pinch | Feature-Small | NEEDS_IMPL | Add zoom interaction to Gantt. |

### バッチ3（#38183〜#37729）

| # | Title (short) | Type | Action | Reason |
|---|---|---|---|---|
| #38183 | Documents list for Conference/Booth Attendee | Feature-Large | NEEDS_DISCUSSION | Broad multi-document generation feature. |
| #38177 | Mo::processBOM() hardcodes qty=1 for frozen BOM lines | Bug-Actionable | NEEDS_IMPL | File/line + root cause for MRP defect. |
| #38169 | Billing contact SIREN/SIRET missing on invoice | Bug-Actionable | NEEDS_IMPL | Missing contact data on invoices. |
| #38166 | API call for replacing event attendee | Feature-Small | NEEDS_IMPL | Small scoped API endpoint. |
| #38127 | Event Attendee replacement with another attendee | Feature-Large | NEEDS_DISCUSSION | New status/schema/UI flow. |
| #38119 | Project photo gallery on main page | Feature-Large | NEEDS_DISCUSSION | New UI section + multi-image storage. |
| #38108 | WebPortal passwordless magic link login | Feature-Large | NEEDS_DISCUSSION | Security-sensitive auth flow. |
| #38106 | Tag combo: show only non-assigned companies | Feature-Small | NEEDS_IMPL | Small filter enhancement. |
| #38099 | REST PUT thirdparties ignores price_level | Bug-Actionable | NEEDS_IMPL | Field silently dropped by API. |
| #38092 | Loan module: non-monthly / more schedules | Feature-Large | NEEDS_DISCUSSION | New frequencies/amortization types. |
| #38088 | Complimentary order lines missing in list view | Bug-Actionable | NEEDS_IMPL | Extrafield display defect. |
| #38080 | Allow #supplier_min_price# in vendor pricing | Feature-Small | NEEDS_IMPL | Dynamic-price variable extension. |
| #38067 | Membership renewal amount ignored by HelloAsso | Bug-Actionable | NEEDS_IMPL | Chosen amount not passed to gateway. |
| #38061 | Credit-note payments missing from VAT reports | Bug-Actionable | NEEDS_IMPL | Incorrect VAT statements; data-correctness. |
| #38059 | Accounting tables not created on module enable | Bug-Actionable | NEEDS_IMPL | Missing table creation on enable. |
| #38051 | Hook in num_open_day() for per-user adjustments | Feature-Small | NEEDS_IMPL | Extension point; PR planned. |
| #38044 | Slow prev/next queries on ~1M-row tables | Feature-Small | NEEDS_IMPL | Disable nav arrows when scanlist disabled. |
| #38018 | Thirdparty type should be mandatory again | Bug-Actionable | NEEDS_IMPL | Regression from v18; suggested fix. |
| #38010 | Virtual stock of product kits not displayed | Bug-Actionable | NEEDS_IMPL | Missing kit virtual-stock display. |
| #38009 | TakePos cannot paginate subcategories | Bug-Actionable | NEEDS_IMPL | Root cause in MoreProducts(). |
| #38002 | REST DELETE invoices/payments missing | Feature-Small | NEEDS_IMPL | API gap; UI action exists. |
| #37993 | JS Context Tools: object / CommonObject | Feature-Large | NEEDS_DISCUSSION | Large JS framework component. |
| #37991 | JS Context: unsaved-changes exit alert | Feature-Large | NEEDS_DISCUSSION | Centralized change-tracking tool. |
| #37990 | JS Context tools for WYSIWYG | Feature-Large | NEEDS_DISCUSSION | New shared JS API to design. |
| #37985 | Record clone link in llx_element_element | Feature-Small | NEEDS_DISCUSSION | relationtype/schema approach needs decision. |
| #37980 | Inventory module SQL error (undefined te.rowid) | Bug-Actionable | NEEDS_IMPL | Reproducible SQL error after upgrade. |
| #37954 | Clone/transfer EventAttendees to another event | Feature-Large | NEEDS_DISCUSSION | Cross-event cloning; needs design. |
| #37949 | Attendee list include sub-event attendees | Feature-Small | NEEDS_DISCUSSION | Vague; needs scope clarification. |
| #37948 | Mass mailing include subproject attendees | Feature-Small | NEEDS_IMPL | Small scoped option. |
| #37935 | Login endpoint accepts GET, leaks credentials | Bug-Actionable | NEEDS_IMPL | Security: credentials in URL/Referer. |
| #37924 | Vendor prices not recalculated on currency change | Bug-Actionable | NEEDS_IMPL | Stale currency conversion. |
| #37900 | Add "title" field to all sales/purchase docs | Feature-Large | NEEDS_DISCUSSION | Schema change across many tables. |
| #37892 | User card tabs: tickets, tasks, related items | Feature-Small | NEEDS_IMPL | Adding existing-pattern tabs. |
| #37891 | UUID v7 / ULID instead of IDs | Feature-Large | NEEDS_DISCUSSION | Sweeping architectural change. |
| #37881 | Permission checks wrong on propal list | Bug-Actionable | NEEDS_IMPL | Permission leakage of others' proposals. |
| #37848 | Hooks in getOnlinePaymentUrl()/newpayment.php | Feature-Small | NEEDS_IMPL | Hook insertion; code provided. |
| #37843 | Add fr_NC language to Transifex | Question/Support | CLOSE_BY_ANSWER | Transifex admin action, not code. |
| #37842 | productlot_note.php: all security checks commented out | Bug-Actionable | NEEDS_IMPL | Security: unenforced permission. || #37811 | Scheduled job to detect unused/duplicate contacts | Feature-Large | NEEDS_DISCUSSION | Open-ended dedup feature. |
| #37810 | Scheduled job to detect mergeable records | Feature-Large | NEEDS_DISCUSSION | Fuzzy-match merge detection. |
| #37800 | Ampersand breaks line feeds with RESTRICTHTML | Bug-Actionable | NEEDS_IMPL | Double-encoding for PDF line-break loss. |
| #37790 | TakePOS cash control shows 0.00 after v23 upgrade | Bug-Actionable | NEEDS_IMPL | Missing migration for new fields. |
| #37782 | Cloning product does not copy supplier prices | Bug-Actionable | NEEDS_IMPL | Supplier prices skipped despite checkbox. |
| #37750 | Members should be able to create tickets | Feature-Large | NEEDS_DISCUSSION | New member↔ticket linkage. |
| #37749 | Attendee list: contact and attendee-type columns | Feature-Large | NEEDS_DISCUSSION | Attendee-contact linkage + typing. |
| #37748 | Link events with products/services | Feature-Large | NEEDS_DISCUSSION | Bidirectional product links. |
| #37747 | Mass-email all customers/suppliers of a product | Feature-Large | NEEDS_DISCUSSION | Related-items → mass-mailing workflow. |
| #37738 | API stable in v20, broken in v21 (PT) | Bug-NeedsInfo | NEEDS_INFO | Empty template; no description/repro. |
| #37729 | Mass email: easy add event attendees by status | Feature-Small | NEEDS_IMPL | Scoped UI addition with mockups. |

### バッチ4（#37722〜#37137）

| # | Title (short) | Type | Action | Reason |
|---|---|---|---|---|
| #37722 | Wrong CGI VAT exemption code | Feature-Small | NEEDS_IMPL | Configurable/removable VAT exemption text. |
| #37720 | Can't change robot@domain.com | Question/Support | CLOSE_BY_ANSWER | Point to ticket email config. |
| #37709 | Pick existing invoice for member contribution | Feature-Small | NEEDS_IMPL | Link existing invoice/line to contribution. |
| #37701 | Tickets tab on event attendee card | Feature-Large | NEEDS_DISCUSSION | New tab + cross-module ticket flow. |
| #37696 | Attendee list menu + project column | Feature-Small | NEEDS_IMPL | Small list/menu enhancement. |
| #37694 | Order thirdparty invoices first in list | Feature-Small | NEEDS_IMPL | Reorder invoice dropdown. |
| #37689 | Community Helm chart available | Invalid/Stale | CLOSE_BY_ANSWER | Third-party chart announcement. |
| #37687 | Related products relationships | Feature-Large | NEEDS_DISCUSSION | New product-relationship subsystem. |
| #37686 | Add/subtract attendee to mass mailing | Feature-Large | NEEDS_DISCUSSION | Mass-action + mailing integration. |
| #37671 | Unit Price inc.tax shows 0 in PO | Bug-NeedsInfo | NEEDS_INFO | No steps/DB info. |
| #37655 | Searchable/editable complementary attributes | Feature-Small | NEEDS_IMPL | Sortable/searchable/editable attribute list. |
| #37652 | Button to log into API explorer as user | Feature-Large | NEEDS_DISCUSSION | Impersonation raises permission/security questions. |
| #37618 | Bad line order on 2nd situation invoice | Bug-NeedsInfo | NEEDS_INFO | Needs version/repro detail. |
| #37610 | Object Link extra field filter syntax | Question/Support | CLOSE_BY_ANSWER | Documentation/syntax help. |
| #37605 | Re-editing invoice blocked / counter leak | Duplicate? | NEEDS_DISCUSSION | Same regression as #37415/#37583. |
| #37583 | Constant to edit invoices already emailed | Feature-Small | NEEDS_IMPL | Config constant relaxing email_sent_counter. |
| #37569 | Resources time available/booked | Feature-Large | NEEDS_DISCUSSION | New booking/scheduling model. |
| #37568 | Public ticket form per project link | Feature-Large | NEEDS_DISCUSSION | New public-form + hashing/config. |
| #37566 | Bad progression on situation s3+ in PDF | Bug-Actionable | NEEDS_IMPL | Wrong PDF values across templates. |
| #37541 | Beluga project report missing elements | Bug-NeedsInfo | NEEDS_INFO | Lacks version/config. |
| #37540 | DST distorts leave day count | Bug-Actionable | NEEDS_IMPL | num_public_holiday breaks across summer-time. |
| #37537 | Install: webserver/DB fields disabled | Bug-CantRepro | NEEDS_INFO | Browser/env-specific (old stack). |
| #37519 | buying_price decimal truncated (rec. invoice) | Bug-Actionable | NEEDS_IMPL | (int) cast truncates decimals. |
| #37517 | EmailCollector confirmation not sent | Bug-Actionable | NEEDS_IMPL | Key mismatch + missing origin_email fallback. |
| #37508 | mysqliDoli class not found on upgrade | Bug-Actionable | NEEDS_IMPL | Fatal error; blocks TLS DB upgrades. |
| #37458 | EU OSS VAT rules only partly working | Bug-NeedsInfo | NEEDS_INFO | Needs concrete cases/config. |
| #37438 | Situation PU TTC not calc when progress 0 | Bug-Actionable | NEEDS_IMPL | Reproducible calc bug with screenshots. |
| #37437 | llx_categorie_propal missing on 23.0 install | Bug-Actionable | NEEDS_IMPL | Table absent from install scripts. |
| #37435 | Public ticket category empty after upgrade | Bug-Actionable | NEEDS_IMPL | Migration doesn't move old tag categories. |
| #37420 | Machine-readable compat info | Feature-Small | NEEDS_DISCUSSION | Release-tooling change; needs buy-in on format. |
| #37415 | Cannot edit non-last standard invoice | Bug-Actionable | NEEDS_IMPL | isEditable wrongly runs isErasable tests. |
| #37411 | SQL 1170 linktoref key length on install | Bug-Actionable | NEEDS_IMPL | Install fails on MySQL 5.7 (TEXT in index). |
| #37400 | Comment field on massaction payment | Feature-Small | NEEDS_IMPL | Add comment field to payment confirm form. |
| #37398 | Down payment shown as discount | Bug-Actionable | NEEDS_IMPL | Mixes down-payment with discounts. |
| #37396 | Permission to hide prices on orders | Feature-Small | NEEDS_IMPL | New permission to hide sales price. |
| #37381 | Cancelling shipping removes links | Bug-Actionable | NEEDS_IMPL | Expedition::cancel calls deleteObjectLinked. |
| #37368 | PHP8.2 API optional fields | Bug-NeedsInfo | NEEDS_INFO | Needs failing request/error. |
| #37353 | Can't export contacts with tags | Bug-NeedsInfo | NEEDS_INFO | No steps/error. |
| #37328 | Linked files require '+' click before upload | Question/Support | CLOSE_BY_ANSWER | Explain upload UI change. |
| #37305 | Disassembly behaves like manufacturing (MRP) | Bug-NeedsInfo | NEEDS_INFO | No repro steps/expected-vs-actual. |
| #37303 | Default https for societe URL | Feature-Small | NEEDS_IMPL | Default/normalize client URLs to https. |
| #37253 | Replenish theoretical stock hook missing | Feature-Small | NEEDS_IMPL | Add hook to adjust replenish-page stock. |
| #37251 | Extrafield sellist categorie USF filter | Bug-Actionable | NEEDS_IMPL | Broken filter syntax for categorie sellist. |
| #37240 | Project emails not logged in agenda | Bug-Actionable | NEEDS_IMPL | Emails from project create no agenda event. |
| #37204 | Add existing members/contacts as attendees | Feature-Large | NEEDS_DISCUSSION | Cross-module linkage; needs design. |
| #37193 | Keep file modification date on upload | Feature-Small | NEEDS_IMPL | Conf-gated preserve of file dates. |
| #37148 | Service uses product ref template | Bug-Actionable | NEEDS_IMPL | New service uses product numbering mask. |
| #37146 | Improve pre-commit CI on MacOS | Feature-Small | NEEDS_IMPL | Add lighter lang-check variant. |
| #37137 | EmailCollector MS OAuth2 client credentials | Feature-Large | NEEDS_DISCUSSION | New app-only auth flow; security review. |

### バッチ5（#37109〜#36628）

| # | Title (short) | Type | Action | Reason |
|---|---|---|---|---|
| #37109 | Header width on small screens (thirdparties) | Bug-Actionable | NEEDS_IMPL | Responsive layout defect with screenshot. |
| #37092 | Adding files needs too many clicks | Feature-Small | NEEDS_DISCUSSION | UX regression complaint; needs decision. |
| #37090 | Set tags/categories without opening product card | Feature-Small | NEEDS_DISCUSSION | Inline category editing enhancement. |
| #37074 | Discount with negative lines creates positive line | Bug-Actionable | NEEDS_IMPL | Sign-flip bug on discounts; financial. |
| #37067 | Deposit slip filter as dropdown | Feature-Small | NEEDS_IMPL | Small ergonomics fix on reconciliation list. |
| #37027 | Force password change on first login | Feature-Small | NEEDS_DISCUSSION | Security enhancement; needs decision. |
| #37026 | Export uses base chart of accounts, not custom | Bug-Actionable | NEEDS_IMPL | Export pulls parent chart instead of active. |
| #36985 | Shared custom API script for buying price | Question/Support | CLOSE_BY_ANSWER | Sharing a workaround, not a defect. |
| #36949 | API contract creation fails for non-admin | Bug-Actionable | NEEDS_IMPL | Regression since 22.0.2; prior PR incomplete. |
| #36924 | Indian language translation strings | Feature-Small | NEEDS_IMPL | Translation additions; route via Transifex. |
| #36922 | Total balance per currency on bank list | Feature-Small | NEEDS_IMPL | Multicurrency total reporting. |
| #36916 | Situation invoice mode 2 can't create credit note | Bug-Actionable | NEEDS_IMPL | Progression validation blocker. |
| #36912 | Add Project column to invoice binding list | Feature-Small | NEEDS_IMPL | Extra join for project in binding view. |
| #36899 | TTC 0% VAT discount breaks tax base | Bug-Actionable | NEEDS_IMPL | Inflates effective VAT rate; compliance. |
| #36885 | Easier demo-data installation | Feature-Small | NEEDS_IMPL | Onboarding improvement to init demo. |
| #36861 | Optional order ref in mass invoice billing | Feature-Small | NEEDS_IMPL | Exclude order refs from notes/lines. |
| #36841 | Can't validate invoice with absolute discount | Bug-Actionable | NEEDS_IMPL | Blocker since 22.0.2; a related v22 blocker was already fixed — verify if still reproducible. |
| #36840 | Tax amount zero with absolute discount | Bug-Actionable | NEEDS_IMPL | VAT wrongly reduced by discount. |
| #36838 | Invalid FK constraint restoring mysql dump | Bug-Actionable | NEEDS_IMPL | Duplicated FK breaks DB restore. |
| #36837 | Download-PDF mass action (zip) | Feature-Small | NEEDS_DISCUSSION | Core-vs-module decision. |
| #36831 | Extra fields on task timespent | Feature-Large | NEEDS_DISCUSSION | UI+API design for clock-in/out. |
| #36829 | Final invoice from proforma in one click | Feature-Small | NEEDS_DISCUSSION | Workflow shortcut; needs decision. |
| #36828 | Copy purchase cost to supplier PO from order | Feature-Small | NEEDS_IMPL | Carry purchase cost for margin. |
| #36826 | Conditional prospect/customer/supplier numbering | Feature-Small | NEEDS_DISCUSSION | Numbering-by-checkbox; needs agreement. |
| #36818 | VAT rounding by total, not per line | Feature-Large | NEEDS_DISCUSSION | Belgian 2026 rule; global impact. |
| #36784 | Negative service in order becomes discount | Bug-Actionable | NEEDS_IMPL | Related to #37074. |
| #36768 | Sound + barcode scanner for TakePOS | Feature-Small | NEEDS_IMPL | POS UX enhancement with toggle. |
| #36762 | Mail notifications for supplier invoices | Feature-Small | NEEDS_IMPL | Extend triggers to supplier invoices. |
| #36760 | Supplier invoice email attachments missing | Bug-Actionable | NEEDS_IMPL | "Attach linked docs" broken for supplier. |
| #36750 | Installer doesn't check php-ctype | Bug-Actionable | NEEDS_IMPL | Missing ctype → post-login fatal. |
| #36749 | Installer doesn't check php-dom | Bug-Actionable | NEEDS_IMPL | Missing DOMDocument → white login page. |
| #36724 | Customer PO ref, country, HS code on invoice | Feature-Small | NEEDS_DISCUSSION | Export-invoice columns; needs scoping. |
| #36723 | Delivery note from multiple partial sales orders | Feature-Large | NEEDS_DISCUSSION | Consolidated shipment workflow. |
| #36722 | Pick lines from multiple orders to invoice | Question/Support | CLOSE_BY_ANSWER | Partial invoicing already exists. |
| #36721 | Workstation planning timeline (MO) | Feature-Large | NEEDS_DISCUSSION | Substantial MRP scheduling UI. |
| #36713 | Attachments on public recruitment form | Feature-Small | NEEDS_IMPL | Add file upload to public form. |
| #36685 | Bad MVC architecture in web portal | Bug-Actionable | NEEDS_DISCUSSION | Refactor concern; needs buy-in. |
| #36665 | Prevent invoice PDF regeneration when closed | Feature-Small | NEEDS_DISCUSSION | Inalterability; opened for discussion. |
| #36662 | Discount on tax-incl. product changes unit price | Bug-Actionable | NEEDS_IMPL | Unit price shifts on discount for TTC. |
| #36651 | Export accountancy comma joins ledger/sub-ledger | Bug-NeedsInfo | NEEDS_INFO | No version/repro/sample. |
| #36645 | Directory listing visible in manual install | Question/Support | CLOSE_BY_ANSWER | Webserver config question. |
| #36641 | Separate line number and ref field | Feature-Small | NEEDS_DISCUSSION | ISO-compliance field split. |
| #36640 | Generate delivery note from invoice | Feature-Small | NEEDS_IMPL | Priceless delivery note from invoice. |
| #36639 | Work order from product auto-fills BOM | Feature-Small | NEEDS_IMPL | Pick product, auto-populate BOM. |
| #36638 | PDF module for salaries | Feature-Small | NEEDS_IMPL | Contributor offers PDF scaffold. |
| #36637 | Customs/import VAT with no base amount | Feature-Small | NEEDS_DISCUSSION | VAT-without-base handling. |
| #36636 | Manufacturing order from sales order | Feature-Small | NEEDS_DISCUSSION | Workflow shortcut; needs decision. |
| #36630 | Client acceptance notification not received (PT) | Bug-NeedsInfo | NEEDS_INFO | Empty template; no repro. |
| #36628 | VeriFactu module for Spain compliance (2027) | Feature-Large | NEEDS_DISCUSSION | Major compliance feature; Discussion. |

### バッチ6（#36626〜#35852）

| # | Title (short) | Type | Action | Reason |
|---|---|---|---|---|
| #36626 | User auto-marked "external" when linked to thirdparty | Feature-Small | NEEDS_IMPL | Allow overriding automatic external-user flag. |
| #36614 | PDF viewer fails in Chrome Android | Bug-Actionable | NEEDS_IMPL | Browser limitation; PDF.js viewer fix. |
| #36613 | Customer settings not updated on document clone | Bug-Actionable | NEEDS_DISCUSSION | Snapshot vs refresh is a design decision. |
| #36587 | Add LTS acronym to long-life versions | Feature-Small | NEEDS_DISCUSSION | Release-policy labeling. |
| #36562 | LDAP module does not set LDAP_FIELD_LOGIN | Bug-Actionable | NEEDS_IMPL | Script error; missing config property. |
| #36560 | Easier removal of contact from a project | Feature-Small | NEEDS_IMPL | Unassign tasks when removing contact. |
| #36532 | VCF export only supports VCard 4.0 | Feature-Small | NEEDS_IMPL | Add optional legacy v3.0 export. |
| #36528 | Linked Objects: full tree/context option | Feature-Large | NEEDS_DISCUSSION | Merging an external module. |
| #36516 | Logic behind payment types filter (CRDT/DBIT) | Question/Support | CLOSE_BY_ANSWER | SEPA filter comprehension question. |
| #36512 | Pricing rule for total cost on a BOM | Feature-Small | NEEDS_DISCUSSION | Contributor waiting on maintainer opinion. |
| #36489 | Reception: negative qty unsupported | Bug-Actionable | NEEDS_IMPL | qty<=0 skip blocks stock movement. |
| #36446 | Auto birthday email to members | Question/Support | CLOSE_BY_ANSWER | How-to; answerable. |
| #36441 | Expense report adds extra <br> newlines | Bug-Actionable | NEEDS_IMPL | V20→V22 regression with screenshots. |
| #36432 | Holidays: negative balance not blocked | Bug-Actionable | NEEDS_IMPL | Check ignores requested days. |
| #36428 | Offer: adding text changes gross to net | Bug-NeedsInfo | NEEDS_INFO | Almost no detail; needs repro. |
| #36421 | Calculated URL extrafields broken since v20 | Bug-Actionable | NEEDS_IMPL | Reproducible regression with minimal repro. |
| #36418 | Bad control checking worked-hour insertion | Bug-Actionable | NEEDS_IMPL | timestamp vs date compare; one-line fix. |
| #36416 | Contact Function field as configurable dropdown | Feature-Small | NEEDS_DISCUSSION | Dictionary-backed enhancement. |
| #36388 | Intervention signature on wrong PDF page | Bug-NeedsInfo | NEEDS_INFO | Screenshot only, no repro. |
| #36365 | Add XML preview support in documents | Feature-Small | NEEDS_IMPL | Extend dolIsAllowedForPreview; good first. |
| #36355 | Multi-invoice single payment errors on transfer | Bug-Actionable | NEEDS_IMPL | "Already recorded" on grouped payment. |
| #36324 | Automate tax payment import with employee | Feature-Small | NEEDS_IMPL | Add/link employee field on tax payments. |
| #36314 | Duplicate/convert invoice as propal | Feature-Small | NEEDS_DISCUSSION | Clone-across-object; needs design. |
| #36299 | Bug: Envoi email impossible | Bug-NeedsInfo | NEEDS_INFO | Empty template; needs info. |
| #36297 | VAT by difference causes rounding errors | Bug-Actionable | NEEDS_DISCUSSION | Core VAT calc; sensitive, needs decision. |
| #36288 | Bug: impossible de valider le reglement | Bug-NeedsInfo | NEEDS_INFO | Empty template; needs repro. |
| #36286 | TakePOS shows wrong (non-customer) prices | Bug-Actionable | NEEDS_IMPL | PRODUIT_CUSTOMER_PRICES repro; 18.0→dev. |
| #36280 | Minimum contribution for "Any amount" | Feature-Small | NEEDS_IMPL | Add min amount + comment with validation. |
| #36277 | Dictionary lists need search/filter boxes | Feature-Small | NEEDS_IMPL | UX consistency filters. |
| #36239 | Incorrect manufacturing order cost (THM) | Bug-NeedsInfo | NEEDS_INFO | Old v20; needs current-version repro. |
| #36226 | Add records limit in user-card agenda | Feature-Small | NEEDS_IMPL | Paginate events; from core dev. |
| #36224 | MRP line tracker (workstation scheduling) | Feature-Large | NEEDS_DISCUSSION | Substantial scheduling feature. |
| #36211 | Separator field can't collapse | Bug-NeedsInfo | NEEDS_INFO | One-line report; needs repro. |
| #36186 | BOM PDF export | Feature-Small | NEEDS_IMPL | Add PDF template for BOM. |
| #36181 | Member list: date search on end-date | Feature-Small | NEEDS_IMPL | List-filter enhancement; good first. |
| #36155 | Multi-currency: wrong "unit price with VAT" | Bug-Actionable | NEEDS_IMPL | Label/value error on currency column. |
| #36133 | Grammar/spelling fix in Contributing docs | Feature-Small | NEEDS_IMPL | Trivial docs fix; first-PR quick-win. |
| #36130 | Integrate Paystack/Flutterwave gateways | Feature-Large | NEEDS_DISCUSSION | Large payment integration. |
| #36091 | Stock calc on order validate (no serial) | Feature-Small | NEEDS_DISCUSSION | Conditional stock-calc option. |
| #36032 | Desync fields between thirdparty and member | Feature-Small | NEEDS_IMPL | Option to stop address/email sync. |
| #36023 | Edit first-term interest in loan schedule | Feature-Small | NEEDS_IMPL | Editable first-period interest. |
| #35987 | Resources: add status field | Feature-Small | NEEDS_DISCUSSION | Floats a module rework; needs scope. |
| #35985 | TakePOS: create/consume customer discounts | Feature-Large | NEEDS_DISCUSSION | Framed as discussion. |
| #35984 | User-editable default notes on documents | Feature-Small | NEEDS_IMPL | Template default notes. |
| #35961 | TakePOS hook AddAction typo | Bug-Actionable | NEEDS_IMPL | Nested $res==1 typo; clear fix. |
| #35916 | TakePOS stock/batch qty mismatch | Bug-Actionable | NEEDS_IMPL | find() returns wrong batch row; data-integrity. |
| #35901 | JSON get/post/edit for extrafield config | Feature-Small | NEEDS_DISCUSSION | Admin-UX; needs design. |
| #35889 | Add __AUTHOR_EMAIL__ substitution | Feature-Small | NEEDS_IMPL | More notification vars; partly good first. |
| #35860 | Rights to view Prospects | Feature-Small | NEEDS_IMPL | Permission set mirroring thirdparty. |
| #35852 | Mass mailing target by extrafield value | Feature-Small | NEEDS_DISCUSSION | Targeting enhancement; needs UI design. |

### バッチ7（#35847〜#35162）

| # | Title (short) | Type | Action | Reason |
|---|---|---|---|---|
| #35847 | Mass mailing staging area | Feature-Large | NEEDS_DISCUSSION | Staging/shadow-list design. |
| #35845 | Subtract recipients from mass mailing | Feature-Small | NEEDS_IMPL | Add/subtract recipient toggle. |
| #35825 | BOM show max producible qty | Feature-Small | NEEDS_IMPL | min(floor(stock/qty)) column. |
| #35818 | Webhook body too long (30MB) | Bug-Actionable | NEEDS_IMPL | Strip db/linkedObjects from payload. |
| #35811 | Situation: can't remove from cycle after credit | Bug-Actionable | NEEDS_IMPL | Remove-from-cycle button stays disabled. |
| #35802 | Recast dictionary management | Feature-Large | NEEDS_DISCUSSION | Refactor into Dictionary parent class. |
| #35786 | Situation credit-note condition wrong | Bug-Actionable | NEEDS_IMPL | Operator > should be <; pinpointed. |
| #35772 | Public expense report link | Feature-Large | NEEDS_DISCUSSION | Public/hashed flow; security design. |
| #35768 | Separate permission for mailing recipients | Feature-Small | NEEDS_IMPL | Add recipient permission (GUI+API). |
| #35761 | Stock decrement on order/invoice w/ batch | Feature-Large | NEEDS_DISCUSSION | Changes batch stock workflow. |
| #35757 | Income tax module | Feature-Large | NEEDS_DISCUSSION | Broad new tax feature; vague. |
| #35730 | Mass mailing visibility like projects | Feature-Large | NEEDS_DISCUSSION | Roles/visibility; overlaps #35768. |
| #35713 | Bank accounts with different currencies | Feature-Large | NEEDS_DISCUSSION | Multicurrency misc payments; large. |
| #35698 | Hookmanager: two modules, one visible | Bug-Actionable | NEEDS_IMPL | Hook return-code masking on payment. |
| #35666 | UI shows legal/deadname | Feature-Small | NEEDS_DISCUSSION | Preferred-name field + hide in header. |
| #35658 | Groups skip account numbers leading zero | Bug-Actionable | NEEDS_IMPL | Leading-zero codes ignored in totals. |
| #35649 | Credit note HT can be positive (gift card) | Bug-CantRepro | NEEDS_DISCUSSION | Works-for-me; edge VAT case. |
| #35637 | Mass action add thirdparty to mailing | Feature-Small | NEEDS_IMPL | From propal/order/invoice lists. |
| #35608 | spellcheck="false" on form fields | Feature-Small | NEEDS_IMPL | Good-first-issue; simple attribute. |
| #35605 | POS additional notes don't work | Bug-Actionable | NEEDS_IMPL | Restaurant note button not enabled. |
| #35584 | Project overview amounts stay 0.00 | Bug-Actionable | NEEDS_IMPL | BOM/MO costs not in project totals. |
| #35575 | Virtual stock wrong if received > ordered | Bug-Actionable | NEEDS_IMPL | Over-receipt not reflected. |
| #35574 | Notify payer on failed card payment | Feature-Small | NEEDS_IMPL | Business notification on Stripe failure. |
| #35560 | Rate-including-tax for Tax2/Tax3 | Feature-Small | NEEDS_IMPL | Extend inclusive-rate to secondary taxes. |
| #35525 | Edit email recipients in ticket | Feature-Small | NEEDS_IMPL | Edit recipient list in ticket dialog. |
| #35476 | $ID$ not substituted in extrafield SQL | Bug-Actionable | NEEDS_IMPL | $ID$ left literal in cross-table filter. |
| #35472 | PO shipping = destination warehouse address | Feature-Small | NEEDS_DISCUSSION | Needs approach decision. |
| #35458 | Passkey support | Feature-Large | NEEDS_DISCUSSION | WebAuthn/passkey; large security. |
| #35448 | Octopus template last column confusing | Bug-Actionable | NEEDS_DISCUSSION | Column semantics; configurable columns. |
| #35434 | "Invoiced" filter on supplier orders broken | Bug-Actionable | NEEDS_IMPL | Filter has no effect; regression. |
| #35390 | Vendor invoice template: no project select | Bug-Actionable | NEEDS_IMPL | Project list empty on convert. |
| #35385 | Track time on tickets | Feature-Large | NEEDS_DISCUSSION | Data-model decision. |
| #35382 | Wrong invoice local tax total sometimes | Bug-Actionable | NEEDS_IMPL | localtax1 varies by line count; rounding. |
| #35374 | Show public holidays in Agenda | Feature-Small | NEEDS_IMPL | Render dictionary holidays in agenda. |
| #35348 | Double-check holiday day logic | Bug-NeedsInfo | NEEDS_INFO | Vague; needs concrete case. |
| #35300 | Website model import SQL KO (PostgreSQL) | Bug-NeedsInfo | NEEDS_INFO | Postgres-specific; needs logs. |
| #35281 | Ticket widget/stats triple with multicompany | Bug-Actionable | NEEDS_IMPL | Duplicated ticket under multicompany. |
| #35275 | Webhook TICKET_SENTBYMAIL not firing | Bug-Actionable | NEEDS_IMPL | Silent in prod; trigger not dispatched. |
| #35270 | otherCurlOptions in getURLContent | Feature-Small | NEEDS_IMPL | Pass custom cURL options (SSL). |
| #35265 | Add more event notifications | Feature-Small | NEEDS_IMPL | Missing supplier/project events. |
| #35253 | Second tax ignored on supplier invoice | Bug-Actionable | NEEDS_IMPL | Regression since 21.0.0. |
| #35238 | Conditional/non-empty extrafield display | Feature-Large | NEEDS_DISCUSSION | Hook granularity limits; needs design. |
| #35232 | Delivery-note PDF mode for PO | Feature-Small | NEEDS_IMPL | New PDF template variant. |
| #35230 | Numeric check in mass stock move qty | Bug-NeedsInfo | NEEDS_INFO | Title only, no repro. |
| #35216 | Read permission ignored in mail recipients | Bug-NeedsInfo | NEEDS_INFO | No repro; related to #35768. |
| #35207 | Inventory qty lost with pagination >20 | Bug-Actionable | NEEDS_IMPL | Quantities not persisted; good repro. |
| #35197 | Auto-generate SN/Lot in MO/Reception | Feature-Small | NEEDS_DISCUSSION | Mislabeled bug; needs decision. |
| #35184 | PGP signature in sent emails | Feature-Large | NEEDS_DISCUSSION | OpenPGP signing; substantial. |
| #35169 | Custom-group formula total should sum months | Feature-Small | NEEDS_IMPL | Sum monthly formula for annual total. |
| #35162 | Releve module rejects empty statements | Feature-Small | NEEDS_IMPL | Allow saving empty bank statements. |

### バッチ8（#35157〜#34700）

| # | Title (short) | Type | Action | Reason |
|---|---|---|---|---|
| #35157 | SPDX file headers | Feature-Large | NEEDS_DISCUSSION | Project-wide licensing policy. |
| #35137 | Events on project not displayed | Bug-NeedsInfo | NEEDS_INFO | v22 bug; needs repro clarity. |
| #35124 | Structured ISO 20022 address fields | Feature-Large | NEEDS_DISCUSSION | Split street/number; wide impact. |
| #35113 | Add description to Supplier Price Import | Feature-Small | NEEDS_IMPL | Import-tool field addition. |
| #35108 | Use Composer for dependencies | Feature-Large | NEEDS_DISCUSSION | Architectural debate; wiki-rejected. |
| #35107 | OAuth2 email token not refreshed | Bug-Actionable | NEEDS_IMPL | Token-refresh failure; recurring. |
| #35088 | Sales Order "No price/qty for vendor" | Bug-Actionable | NEEDS_IMPL | Regression; null idprodfournprice. |
| #35080 | Doliwamp build broken after path move | Bug-Actionable | NEEDS_IMPL | /build/→/dev/build/ breaks packaging. |
| #35073 | box_actions_future code_compta field | Bug-Actionable | NEEDS_IMPL | Field-name mismatch; proposed fix. |
| #35072 | Typo fistname in card_group.php | Bug-Actionable | NEEDS_IMPL | Confirmed typo; one-line fix. |
| #35060 | Extrafield keypairs | Feature-Large | NEEDS_DISCUSSION | New complex extrafield type. |
| #35059 | V22 multicurrency PO second-currency price 0 | Bug-NeedsInfo | NEEDS_INFO | No version details/screenshots. |
| #35057 | Module Builder testable out of box | Feature-Small | NEEDS_DISCUSSION | Needs test-scaffolding decision. |
| #35051 | Separate self vs subordinate rights | Feature-Small | NEEDS_IMPL | Permission granularity expense/leave. |
| #35037 | TakePOS payment confirmation screen | Feature-Small | NEEDS_IMPL | POS payment validation safeguard. |
| #34998 | India GST/HSN config help | Question/Support | CLOSE_BY_ANSWER | Config usage question. |
| #34947 | PDF top margin breaks layout | Bug-Actionable | NEEDS_IMPL | Only header moves with top margin. |
| #34937 | PJ reference disappears on PR/SI | Bug-NeedsInfo | NEEDS_INFO | Vague; needs details. |
| #34906 | Color selector in extrafield type | Feature-Small | NEEDS_IMPL | Reuse tag color selector. |
| #34905 | Thunderbird can't update agenda (CalDAV) | Bug-NeedsInfo | NEEDS_INFO | No logs/version. |
| #34904 | No link to Dolibarr in Thunderbird event | Bug-NeedsInfo | NEEDS_INFO | Regression since v20; lacks specifics. |
| #34903 | ODT substitution vars for Receptions | Feature-Small | NEEDS_IMPL | Parity with Shipments ODT. |
| #34901 | Admin page for CERTIFICATE_CRT confs | Feature-Small | NEEDS_IMPL | Admin UI for PDF signing certs. |
| #34890 | Filters on customer payments report | Feature-Small | NEEDS_IMPL | Added filters on rapport.php. |
| #34887 | Left menu stuck / home button double index | Bug-NeedsInfo | NEEDS_INFO | Empty template; best cleanup close. |
| #34870 | Spanish decimal/thousands parsing error | Bug-NeedsInfo | NEEDS_INFO | Price-parsing; template unfilled. |
| #34868 | ODT upload for Receptions fails | Bug-Actionable | NEEDS_IMPL | "Field file required" on Receptions. |
| #34867 | Add Date Sent field on objects | Feature-Small | NEEDS_IMPL | Column + display + auto-fill. |
| #34851 | VAT change alters HT instead of TTC | Bug-Actionable | NEEDS_DISCUSSION | Expected behavior debatable. |
| #34845 | Donator name missing in bank journal | Bug-Actionable | NEEDS_IMPL | Reporting gap in donation journal. |
| #34842 | Can't edit product lines in invoices | Bug-Actionable | NEEDS_IMPL | v21/v22 regression; buttons missing. |
| #34820 | Bold/Italic in PDF from HTML fields | Feature-Small | NEEDS_IMPL | HTML-style rendering in PDF. |
| #34811 | tinyint instead of text for enabled column | Feature-Small | NEEDS_DISCUSSION | Schema optimization; core table. |
| #34793 | Import update mode fails for third parties | Bug-Actionable | NEEDS_IMPL | Reports success but writes nothing. |
| #34773 | Missing subscription fields in notifications | Question/Support | NEEDS_INFO | Setup confusion; config help. |
| #34767 | Refactor core with Symfony components | Feature-Large | NEEDS_DISCUSSION | Sweeping architecture proposal. |
| #34766 | Enhance Contracts module | Feature-Large | NEEDS_DISCUSSION | Multi-feature overhaul. |
| #34765 | Emails from ticket for external users | Feature-Small | NEEDS_DISCUSSION | Author wants design agreement. |
| #34761 | Russian letters broken after GETPOST | Bug-Actionable | NEEDS_IMPL | UTF-8 corruption in sendmails (Postgres). |
| #34758 | Members mass action for debit notes | Feature-Small | NEEDS_IMPL | Mass-billing on member list. |
| #34754 | Bulk payments for vendor invoices | Feature-Small | NEEDS_IMPL | Parity with customer bulk payment. |
| #34753 | Calculated fields broken/undocumented | Question/Support | NEEDS_DISCUSSION | Doc gap + behavior change. |
| #34747 | Enable REST update reception lines | Feature-Small | NEEDS_IMPL | Commented-out putLine to finish. |
| #34728 | Reply-to + remove X-Remoteaddr | Feature-Small | NEEDS_IMPL | Email header options; stale-labeled. |
| #34721 | Inefficient builddoc recursive scan | Feature-Small | NEEDS_IMPL | Perf optimization; stale-labeled. |
| #34716 | Hardcoded theme_vars.inc.php include | Feature-Small | NEEDS_IMPL | Hook for module themes; stale-labeled. |
| #34705 | Multi-currency vendor payment rounds down | Bug-Actionable | NEEDS_IMPL | Rounds non-base-currency amounts. |
| #34700 | LINEORDER_DELETE trigger not fired | Bug-Actionable | NEEDS_IMPL | Missing call_trigger; proposed fix. |

### バッチ9（#34699〜#32802）

| # | Title (short) | Type | Action | Reason |
|---|---|---|---|---|
| #34699 | Buffer/margin on multicurrency rates | Feature-Small | NEEDS_DISCUSSION | Config field; needs buy-in. |
| #34668 | Tags column in lists | Feature-Small | NEEDS_DISCUSSION | PR offered; wants core agreement. |
| #34659 | Duplicate API key on user creation | Bug-Actionable | NEEDS_IMPL | Unique-constraint from weak key gen. |
| #34653 | Menu same position on two entities | Bug-Actionable | NEEDS_IMPL | $conf->entity vs $this->entity; trivial. |
| #34641 | Template replace by name not path | Bug-Actionable | NEEDS_DISCUSSION | Design flaw; needs architectural call. |
| #34631 | Extrafield "link" to category breaks list | Bug-NeedsInfo | NEEDS_INFO | Bug or misconfig unclear. |
| #34604 | Slow variant lookup on sales orders | Bug-Actionable | NEEDS_IMPL | Perf; proposed SQL optimization. |
| #34592 | mo_production cost price "0.00000" | Bug-Actionable | NEEDS_IMPL | empty() skips PMP fallback; patch. |
| #34554 | cabyprodserv report header params | Bug-Actionable | NEEDS_IMPL | Params reinitialized; cause identified. |
| #34542 | {mycompany_logo} not replaced in ODS | Bug-Actionable | NEEDS_IMPL | Works ODT not ODS; templates attached. |
| #34463 | No response on AI prompt (ChatGPT key) | Bug-NeedsInfo | NEEDS_INFO | OpenAI hangs; needs response body. |
| #34413 | Various payment in foreign currency error | Bug-Actionable | NEEDS_IMPL | Error when base != payment currency. |
| #34399 | Default payment terms per customer | Feature-Small | NEEDS_DISCUSSION | Placement/design decision. |
| #34323 | RESTRICTHTML_ONLY_VALID_HTML preg_replace | Bug-Actionable | NEEDS_IMPL | DOM saveHTML breaks preg_replace. |
| #34289 | Journal assets date filter (elseif?) | Bug-Actionable | NEEDS_IMPL | Likely missing elseif in date logic. |
| #34223 | $_SERVER blocked in extrafield visibility | Bug-Actionable | NEEDS_DISCUSSION | Security restriction breaks usage. |
| #34191 | Undefined array key "References" collector | Bug-Actionable | NEEDS_IMPL | PHP warning on email-collector save. |
| #34180 | Bank account boxes too narrow | Feature-Small | NEEDS_IMPL | UI width fix. |
| #34146 | Candidature API "EMail required" | Bug-Actionable | NEEDS_IMPL | Field mapping rejects EMail. |
| #34140 | Error importing plan comptable | Bug-Actionable | NEEDS_IMPL | Import expects date in example field. |
| #34094 | Crash: llx_categorie_extrafields missing | Bug-Actionable | NEEDS_IMPL | Fatal viewing categories; missing table. |
| #34075 | onlinesign header shown twice | Bug-Actionable | NEEDS_IMPL | v20→v21 regression; 11 comments. |
| #34051 | Hooks debugger highlight tooling | Feature-Large | NEEDS_DISCUSSION | Debugbar/hook-tracing tooling. |
| #34012 | Missing return/exchange feature | Feature-Large | NEEDS_DISCUSSION | Return module exists; product discussion. |
| #33974 | OpenID Connect 500 after auth (EntraID) | Bug-Actionable | NEEDS_IMPL | Null $error then 500; repro given. |
| #33959 | Generic object linking via REST API | Feature-Small | NEEDS_DISCUSSION | Endpoint design decision. |
| #33826 | Blank screen closing shipment | Bug-NeedsInfo | NEEDS_INFO | One vague line, no logs. |
| #33816 | Japanese prefectures gibberish | Bug-Actionable | NEEDS_IMPL | Dictionary in wrong encoding. |
| #33804 | dispatch.php double batch values | Bug-Actionable | NEEDS_IMPL | Duplicate lines with batch; patch. |
| #33733 | Tax details in Sponge invoice (India) | Feature-Small | NEEDS_DISCUSSION | Conditional tax/line printing. |
| #33686 | Expensereport PDF fails on PNG logo | Bug-Actionable | NEEDS_IMPL | Memory exhaustion; PNG handling. |
| #33650 | Mandatory public message on ticket close | Feature-Small | NEEDS_DISCUSSION | Behavior/UX decision. |
| #33642 | User from corporation member no name | Feature-Small | NEEDS_IMPL | Fill user name from company field. |
| #33578 | Product description always printed | Bug-Actionable | NEEDS_IMPL | v19→v20 regression; deleted desc shows. |
| #33282 | Ship composer.lock in sources | Feature-Small | NEEDS_DISCUSSION | Packaging policy; likely won't-fix. |
| #33281 | Sick leave reduces holiday count | Bug-Actionable | NEEDS_IMPL | Sickdays deducted from holidays. |
| #33273 | API /documents for interventions/tickets | Feature-Small | NEEDS_IMPL | Add modulepart to document API. |
| #33236 | API subscription doesn't validate member | Bug-Actionable | NEEDS_IMPL | UI validates, API leaves pending. |
| #33192 | Inventory expected qty overwritten | Bug-Actionable | NEEDS_IMPL | Expected set to real qty on close. |
| #33180 | Non-balanced transaction blocks close | Bug-Actionable | NEEDS_IMPL | Blocks year-end close. |
| #33167 | Wrong signature position on interv. PDF | Bug-Actionable | NEEDS_IMPL | v21 regression; code diff shown. |
| #32995 | Wrong Total HT when entering TTC | Bug-Actionable | NEEDS_IMPL | Line HT miscomputed from TTC. |
| #32953 | Separate interventions in soleil template | Feature-Small | NEEDS_IMPL | PDF spacing/separator. |
| #32924 | Element counts on Proposals/Orders/Invoices | Feature-Small | NEEDS_DISCUSSION | UI badge counts; needs agreement. |
| #32920 | Install fatal: Europe/Kyiv timezone | Bug-CantRepro | CLOSE_BY_ANSWER | Outdated Windows tzdata/ICU. |
| #32887 | Grouped payment of social/fiscal taxes | Feature-Small | NEEDS_DISCUSSION | Batch pay; workflow design. |
| #32814 | Box-size database for shipments | Feature-Large | NEEDS_DISCUSSION | Shipment-packaging subsystem. |
| #32804 | Per-project notification email | Feature-Small | NEEDS_DISCUSSION | Notification routing; needs decision. |
| #32802 | escpos TypeError on PHP 8.x | Bug-Actionable | NEEDS_IMPL | Bundled escpos too old; upgrade to v4.0. |

### バッチ10（#32799〜#31447）

| # | Title (short) | Type | Action | Reason |
|---|---|---|---|---|
| #32799 | Default notification template per project | Feature-Small | NEEDS_DISCUSSION | Template hierarchy design. |
| #32768 | createFromOrder drops client ref | Bug-Actionable | NEEDS_IMPL | ref_client not copied to invoice. |
| #32751 | Umlauts not stored in URL/email fields | Bug-Actionable | NEEDS_IMPL | Data-truncation despite "feature" label. |
| #32738 | Can't login on new Coolify install | Question/Support | CLOSE_BY_ANSWER | Deployment/credential question. |
| #32735 | Generic FTP/SFTP connector classes | Feature-Large | NEEDS_DISCUSSION | Broad refactor. |
| #32679 | Multi-currency invoice totals mix currencies | Bug-Actionable | NEEDS_IMPL | Currency-mixing on totals. |
| #32625 | Variant prices zero on subsequent variants | Bug-Actionable | NEEDS_IMPL | % price only applied to first variant. |
| #32605 | Supplier payment cast to int (float lost) | Bug-Actionable | NEEDS_IMPL | Data-loss; drops decimals. |
| #32553 | Auto-send invoice from template | Feature-Small | NEEDS_DISCUSSION | Auto-send/validate flow design. |
| #32524 | Encoding of non-ASCII at fresh install | Bug-NeedsInfo | NEEDS_INFO | Env/charset; needs isolation. |
| #32510 | Stripe payment done but not registered | Bug-NeedsInfo | NEEDS_INFO | Needs logs to trace webhook. |
| #32500 | Invoice ref not updated despite payment | Bug-CantRepro | NEEDS_INFO | One-off; not reproducible. |
| #32476 | Invoice-count stat shown as decimal | Bug-Actionable | NEEDS_IMPL | Axis-formatting; count as integer. |
| #32460 | Module Builder enhancements | Feature-Large | NEEDS_DISCUSSION | Multi-point overhaul. |
| #32458 | Fatal on Asia/Calcutta timezone | Bug-Actionable | NEEDS_IMPL | Handle deprecated/alias tz names. |
| #32428 | VAT reset to 0% on product select | Bug-Actionable | NEEDS_IMPL | Loss of default VAT on product line. |
| #32406 | "dict" undefined-key warning on import | Bug-Actionable | NEEDS_IMPL | PHP warning on thirdparty import. |
| #32397 | Event title not translated (thirdparty) | Bug-Actionable | NEEDS_IMPL | Missing-translation fix. |
| #32380 | Product prices with start/end date | Feature-Large | NEEDS_DISCUSSION | Pricing/history feature. |
| #32379 | Product prices in different currencies | Feature-Large | NEEDS_DISCUSSION | Big pricing feature. |
| #32359 | CKEditor 4 bundled vulnerabilities | Bug-Actionable | NEEDS_IMPL | Outdated CKEditor 4.22; security. |
| #32344 | Can't link intervention to invoice via API | Bug-Actionable | NEEDS_IMPL | 200 but creates no link. |
| #32332 | Online sign URL for proposals in API | Feature-Small | NEEDS_IMPL | Expose signature URL via REST. |
| #32250 | Keep module activation data on disable | Feature-Small | NEEDS_DISCUSSION | Persistence decision. |
| #32243 | Emailing contacts CSV download does nothing | Bug-NeedsInfo | NEEDS_INFO | 200 but no file; needs repro. |
| #32115 | Invoices should keep historical thirdparty name | Feature-Large | NEEDS_DISCUSSION | Historization. |
| #32098 | Inconsistent object names across API/URL | Bug-Actionable | NEEDS_DISCUSSION | Naming; needs mapping decision. |
| #32085 | Automatic encryption of backups | Feature-Large | NEEDS_DISCUSSION | Security feature; needs design. |
| #32071 | New hook in admin/mails.php | Feature-Small | NEEDS_IMPL | Hook addition for module authors. |
| #32069 | Recurring-invoice mail sent before validation | Bug-Actionable | NEEDS_IMPL | Cron mails PROV invoice, no PDF. |
| #32025 | Migration SQL error fk_department_buyer | Invalid/Stale | CLOSE_BY_ANSWER | Old develop-snapshot; likely fixed. |
| #31981 | Support HEIF image previews | Feature-Small | NEEDS_DISCUSSION | Depends on server libs. |
| #31980 | Export module: more project fields | Feature-Small | NEEDS_IMPL | Add project category/contacts. |
| #31971 | Loan end date optional + phases | Feature-Small | NEEDS_DISCUSSION | Loan-model change. |
| #31883 | Add warehouse location concept | Feature-Large | NEEDS_DISCUSSION | New object; logistics feature. |
| #31760 | Balance sheet / P&L / cash flow reports | Feature-Large | NEEDS_DISCUSSION | Major accounting reporting. |
| #31751 | Project profit include subprojects | Feature-Small | NEEDS_DISCUSSION | Aggregation UI; needs design. |
| #31749 | Indent subprojects in project list | Feature-Small | NEEDS_IMPL | List-display improvement. |
| #31744 | Print outstanding-bill statement | Feature-Small | NEEDS_DISCUSSION | Unpaid-invoices printout design. |
| #31723 | Filter purchase orders by "invoiced" | Feature-Small | NEEDS_IMPL | Filter on project overview. |
| #31699 | Swiss QR in ODT templates | Feature-Small | NEEDS_IMPL | Add {swiss_qr} to ODT engine. |
| #31652 | Digital shipment sheet (ViDA/e-reporting) | Feature-Large | NEEDS_DISCUSSION | EU e-reporting feature. |
| #31608 | Persist supplier-invoice list filter | Feature-Small | NEEDS_IMPL | Retain filter; small UX fix. |
| #31606 | Multi-currency payment value cast to int | Bug-Actionable | NEEDS_IMPL | Data-loss; decimals dropped. |
| #31531 | Some edits don't update modification date | Bug-Actionable | NEEDS_IMPL | Several fields skip tms update. |
| #31495 | Buying price not cloned with service | Bug-Actionable | NEEDS_IMPL | Clone omits supplier/buying price. |
| #31484 | Translate CSV import template fields | Feature-Small | NEEDS_IMPL | Translate import column labels. |
| #31447 | Products hidden when only Service module on | Bug-Actionable | NEEDS_IMPL | Menu should show Products if Services on. |

### バッチ11（#31416〜#25830）

| # | Title (short) | Type | Action | Reason |
|---|---|---|---|---|
| #31416 | Mobile phone field missing in Tiers CSV import | Feature-Small | NEEDS_IMPL | Expose mobile field to import mapping. |
| #31375 | Sort Effectif (workforce) filter | Feature-Small | NEEDS_IMPL | Ordering fix on dropdown. |
| #31362 | Long TVA note breaks layout on small screen | Bug-Actionable | NEEDS_IMPL | CSS/wrapping fix. |
| #31307 | Graceful handling when DNS resolution fails | Feature-Small | NEEDS_IMPL | Wrap dns_get_record; exact fix given. |
| #31304 | Label field missing in supplier invoice import | Feature-Small | NEEDS_IMPL | Add libellé to import template. |
| #31290 | Import files show only within their category | Feature-Small | NEEDS_DISCUSSION | Tag/filter uploaded files; needs design. |
| #31233 | ODT truncated when text contains < or > | Bug-Actionable | NEEDS_IMPL | XML-escaping defect; corrupts docs. |
| #31210 | Mass-send email to selected members | Feature-Small | NEEDS_IMPL | Add mass-email to member list. |
| #31209 | ProfID 3 & 4 missing in Italian on V20 | Bug-Actionable | NEEDS_IMPL | Regression pinpointed to commit. |
| #31150 | Multi-task (mass) editing in projects | Feature-Large | NEEDS_DISCUSSION | Mass-edit task fields; UX design. |
| #31043 | ODT no longer generated after 19+ (500) | Bug-Actionable | NEEDS_IMPL | Regression; 500/blank page. |
| #30908 | get_default_tva misses EEC VAT threshold | Bug-Actionable | NEEDS_DISCUSSION | EU distance-selling logic; needs decision. |
| #30874 | Password confirmation (second) field | Feature-Small | NEEDS_IMPL | UX safeguard on password change. |
| #30816 | Allow entering reversed exchange rate | Feature-Small | NEEDS_DISCUSSION | Touches all multicurrency calcs. |
| #30789 | MO cost not added to project overview | Bug-Actionable | NEEDS_IMPL | Costing/reporting gap. |
| #30771 | CRON unpaid-invoice reminder: nbdays inverted | Bug-Actionable | NEEDS_IMPL | Reminders at wrong time; code given. |
| #30748 | Activate GitHub Discussions | Invalid/Stale | CLOSE_BY_ANSWER | Meta request, not code. |
| #30676 | API: get product by lot number | Feature-Small | NEEDS_IMPL | Lot-number lookup endpoint. |
| #30673 | Multicurrency payment from invoice list wrong amount | Bug-Actionable | NEEDS_IMPL | Wrong stored payment amount; data corruption. |
| #30612 | Grant project access to a whole group | Feature-Small | NEEDS_IMPL | Group permission enhancement. |
| #30611 | Project graph: sales vs time | Feature-Small | NEEDS_IMPL | Reporting chart. |
| #30438 | Advance payment linked to propal/order | Feature-Large | NEEDS_DISCUSSION | Big accounting feature; 16 comments. |
| #30400 | Sales order omits default tax for variants | Bug-Actionable | NEEDS_IMPL | Inconsistency vs non-variant. |
| #30352 | House number as separate address field | Feature-Large | NEEDS_DISCUSSION | Core schema change to address. |
| #30202 | Schedule future price-campaign changes | Feature-Large | NEEDS_DISCUSSION | Time-bounded pricing engine. |
| #30175 | Vinci PDF broken for MO with many lines | Bug-Actionable | NEEDS_IMPL | Overlapping text/empty pages. |
| #30078 | Implement FacturX/XRechnung/ZUGFeRD | Feature-Large | NEEDS_DISCUSSION | EU-mandated standard; 53 comments. |
| #29992 | One SKU with multiple names/codes (ASIN) | Feature-Large | NEEDS_DISCUSSION | Multi-listing model; needs scoping. |
| #29664 | CalDAV reminder/push-notification sync | Feature-Large | NEEDS_DISCUSSION | Reminder field + CalDAV sync. |
| #29405 | Sort skills alphabetically / by sequence | Feature-Small | NEEDS_IMPL | Ordering option to skills lists. |
| #29404 | Empty pages in Crabe evaluation PDF | Bug-Actionable | NEEDS_IMPL | Pagination defect; PDF attached. |
| #29244 | Contact merging functionality | Feature-Small | NEEDS_IMPL | Mirror thirdparty-merge for contacts. |
| #29046 | Filter lists by proposal/order contact | Feature-Small | NEEDS_IMPL | Add contact filter column. |
| #28914 | Member merging functionality | Duplicate? | NEEDS_IMPL | Same pattern as #29244. |
| #28248 | Wrong default vendor on price request | Bug-Actionable | NEEDS_IMPL | Defaults vendor to customer thirdparty. |
| #28223 | Default notifications for new thirdparties | Feature-Small | NEEDS_IMPL | Baseline notification config. |
| #27300 | Save sent mails to IMAP "Sent" folder | Feature-Small | NEEDS_IMPL | IMAP append config for SMTP. |
| #27259 | Add note de débours to customer invoices (FR) | Feature-Small | NEEDS_DISCUSSION | Vague FR accounting; needs scoping. |
| #27130 | Virtual stock wrong on partial reception | Bug-Actionable | NEEDS_IMPL | Miscalc with virtual-stock constant. |
| #27103 | Superadmin suspension mode via conf | Feature-Large | NEEDS_DISCUSSION | Privilege model change. |
| #26971 | Civility missing from user import fields | Feature-Small | NEEDS_IMPL | Add civility column to user import. |
| #26970 | Import/export field name mismatch (Departement) | Bug-Actionable | NEEDS_IMPL | i18n label inconsistency. |
| #26864 | Linking 2nd member to thirdparty no warning | Bug-Actionable | NEEDS_IMPL | Silent overwrite; 11 comments. |
| #26731 | DIRECTDOWNLOAD_URL_PROPOSAL empty in notify | Bug-Actionable | NEEDS_IMPL | Token empty on validation notification. |
| #26550 | Version control / API for email templates | Feature-Large | NEEDS_DISCUSSION | Broad API + integration. |
| #26513 | Receive variants from parent product | Feature-Small | NEEDS_IMPL | Reception UX to pick variants. |
| #26421 | Ansible automation of Dolibarr | Invalid/Stale | CLOSE_BY_ANSWER | Vague 2023 wish; covered by API. |
| #26230 | Resource module improvements | Feature-Large | NEEDS_DISCUSSION | New fields/relations; needs qualification. |
| #25935 | Richer public membership registration form | Feature-Small | NEEDS_IMPL | Config links/info to public form. |
| #25830 | Duplicate line-delete triggers (MOLINE) | Bug-Actionable | NEEDS_DISCUSSION | Trigger naming inconsistency. |

### バッチ12（#25829〜#8612、最古参）

| # | Title (short) | Type | Action | Reason |
|---|---|---|---|---|
| #25829 | Auto-send template invoice by email | Feature-Small | NEEDS_IMPL | Still relevant (updated 2026). |
| #25622 | Multicurrency fails on local currency | Bug-NeedsInfo | NEEDS_INFO | Old v17; needs current repro. |
| #25299 | Fixed discounts on payment page | Feature-Small | NEEDS_DISCUSSION | Credit-note-at-payment; stale-labeled. |
| #25297 | Reorder columns in lists | Feature-Large | NEEDS_DISCUSSION | Drag-drop column order; stale-labeled. |
| #25047 | Captcha on project self-registration | Feature-Small | NEEDS_IMPL | Security add; mirrors ticket captcha. |
| #24619 | Delivery note for warehouse transfers | Feature-Small | NEEDS_DISCUSSION | Legal need; 11 comments. |
| #24304 | Attach linked files to expense report PDF | Feature-Small | NEEDS_IMPL | Append justificatifs into PDF. |
| #24271 | ECM trashbin/versioning | Feature-Large | NEEDS_DISCUSSION | File-lifecycle rework of GED/ECM. |
| #23999 | Hook for intervention lines | Feature-Small | NEEDS_IMPL | alterLineInter hook; PR offered. |
| #23957 | Multiple batch/serial per product at reception | Feature-Large | NEEDS_DISCUSSION | Traceability data-model change. |
| #23870 | Add Purchase Order from sales order | Feature-Small | NEEDS_DISCUSSION | Linked-PO workflow; design needed. |
| #23821 | Separate Project vs Opportunity | Feature-Large | NEEDS_DISCUSSION | Large-analysis-labeled redesign. |
| #23557 | IDN domain names stripped | Bug-Actionable | NEEDS_IMPL | Non-ASCII silently mangled; data corruption. |
| #23528 | Extrafields for employee-position (HRM) | Feature-Small | NEEDS_IMPL | Consistent with extrafields pattern. |
| #23386 | Filter reset after deleting accounting entry | Bug-Actionable | NEEDS_IMPL | Filter lost after trashbin delete. |
| #23230 | Move task to another project | Feature-Small | NEEDS_IMPL | Well-scoped; 17 comments. |
| #23229 | Link supplier invoice lines to projects | Feature-Large | NEEDS_DISCUSSION | Per-line allocation; data-model. |
| #23201 | Link salaries to accounting | Bug-Actionable | NEEDS_IMPL | Salary not booked; data-integrity. |
| #23186 | Integrate CardDAV/CalDAV into core | Feature-Large | NEEDS_DISCUSSION | External CalDAV/CardDAV module exists. |
| #23095 | Auto-split discount when over invoice amount | Feature-Small | NEEDS_IMPL | Clear popup behavior. |
| #23028 | Margin module wrong on negative values | Bug-Actionable | NEEDS_IMPL | Miscalc with negative prices. |
| #22944 | Default/excluded fields in import tool | Feature-Small | NEEDS_IMPL | Set defaults, skip required fields. |
| #22283 | Toggle targetwithdetails in propal/facture | Feature-Small | NEEDS_IMPL | Setup checkbox for PDF mode. |
| #21437 | Civility missing in user import | Bug-Actionable | NEEDS_IMPL | DB column exists, not in import. |
| #21269 | Replace "copy address from thirdparty" | Feature-Small | NEEDS_DISCUSSION | Address handling; affects PDFs. |
| #21268 | CommonList class for all lists | Feature-Large | NEEDS_DISCUSSION | Architectural refactor of lists. |
| #20876 | Tax 2 missing via replenishment | Bug-Actionable | NEEDS_IMPL | Tax-2 calc bug in supplier invoice. |
| #20744 | Show lot numbers on invoice lines | Feature-Small | NEEDS_IMPL | Surface batch numbers onto invoices. |
| #20682 | Setting: default file preview vs download | Feature-Small | NEEDS_IMPL | UX preference toggle. |
| #20673 | Cryptographically sign releases (PGP) | Feature-Small | NEEDS_DISCUSSION | Supply-chain security; process decision. |
| #20670 | Graphic progress bar for projects | Feature-Small | NEEDS_IMPL | UI enhancement. |
| #20641 | Hierarchical notes/journal | Feature-Large | NEEDS_DISCUSSION | Cross-object journaling. |
| #20549 | Status field on project task card | Feature-Small | NEEDS_IMPL | Concrete implementation steps. |
| #20117 | Assign spent time directly to project | Feature-Small | NEEDS_DISCUSSION | Touches time-tracking model. |
| #20085 | Protect module extrafields from edit/delete | Feature-Small | NEEDS_IMPL | Data-loss prevention. |
| #20014 | Generate template invoices in advance | Feature-Small | NEEDS_IMPL | Scheduling enhancement. |
| #19752 | Convert deposit with overpayment | Feature-Small | NEEDS_DISCUSSION | Niche accounting edge case. |
| #19661 | MRP/GPAO evolution | Feature-Large | NEEDS_DISCUSSION | Production-module roadmap; 24 comments. |
| #19407 | Can't change PRE/VIR payment codes | Bug-Actionable | NEEDS_IMPL | Hardcoded codes break withdrawals. |
| #19310 | Show links in attached-files section | Feature-Small | NEEDS_IMPL | Display associated links on cards. |
| #19003 | Credit note partial refund + deduction | Feature-Small | NEEDS_IMPL | Accounting flexibility. |
| #18951 | Extend ALLOW_COMMENT to more areas | Feature-Small | NEEDS_IMPL | Reuse project/task comment feature. |
| #18378 | Dedicated "option" flag for lines | Feature-Small | NEEDS_IMPL | Line-option feature; concrete steps. |
| #16491 | Return/exchange function | Feature-Large | NEEDS_DISCUSSION | New return workflow; needs design. |
| #14818 | ODT template support overview | Invalid/Stale | CLOSE_BY_ANSWER | 2020 tracking table; obsolete. |
| #14295 | Tests for REST API / web UI | Invalid/Stale | CLOSE_BY_ANSWER | 2020 vague; no traction since 2023. |
| #13645 | Notes icon missing on some list cards | Feature-Small | NEEDS_IMPL | UI-consistency gap; updated 2025. |
| #8612 | GDPR features | Feature-Large | NEEDS_DISCUSSION | 2018 strategic; DataPolicy now exists. |

> 全件の生データ（タイトル・本文抜粋・ラベル・コメント数）は調査用JSON（`issues200.json` / `issues_rest.json`）として保持。特定Issueの詳細追跡が必要な場合に参照可能です。
</content>
</invoke>
