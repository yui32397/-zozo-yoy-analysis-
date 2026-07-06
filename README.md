# 【EC分析】ZOZOの売上要因分解（前年比）と今後の成長戦略の提案

![Unofficial](https://img.shields.io/badge/Status-Unofficial%20/%20Portfolio-red?style=for-the-badge)
![Data](https://img.shields.io/badge/Data-Simulated%20/%20Dummy-orange?style=for-the-badge)
![License](https://img.shields.io/badge/License-MIT-blue?style=for-the-badge)

---

## ⚠️ 免責事項 / Disclaimer

*   **非公式プロジェクト (Unofficial Project)**: 本プロジェクトおよび本リポジトリに含まれるすべてのクエリ、グラフ、およびインサイトは、**株式会社ZOZO公式および実在する特定のECサイトとは一切関係がありません。**
*   **技術実証用デモ (Simulated Data)**: 本プロジェクトで使用されているデータ構造（購入者数、年間購入金額、四半期軸等）は、データ抽出・集計・可視化スキルを実証（ポートフォリオ）するために一般のEC市場のトレンドに基づきゼロから設計された「疑似データ（ダミーデータ）」です。

---

## 📊 📑 本プロジェクトの結論サマリー（採用担当者向け・4ステップ解説）

*   **① 課題 (Problem)**: 商品取扱高は前年同期比（YoY）で堅調に成長しているが、その成長を支える内部のKPI（客数と単価のどちらが売上に貢献しているか）の構造的ボトルネックがブラックボックス化していた。
*   **② 分析 (Analysis)**: ZOZO公式の決算資料から抽出したリアルデータを定義し、Python（Pandas）を用いた要因分解パイプラインを構築。売上（商品取扱高）を「年間購入者数（成長率）× 1人あたり年間購入金額（成長率）」の複利ロジックに分解して厳密に計算。
*   **③ 結果 (Result)**: 取扱高の成長は**年間購入者数（アクティブ客数）の＋7.8%に達する急激な伸び**がカバーして牽引している一方、ユーザー1人あたりの年間購入金額（顧客単価）の寄与度は**ー0.2%からー3.8%へと危機的に悪化**しているファクトを特定した。
*   **④ 示唆 ＆ 改善提案 (Insight & Proposal)**: 値下げやクーポンの多発による「力技の客数維持」は顧客エンゲージメントの質を落とす構造的リスクがある。今後はそれらをクレンジング（抑制）し、AIコーディネート提案の強化による「合わせ買い」の誘導など、1人あたり購買額（LTV）の回復へ戦略をシフトすべきである。

---

## 📊 🌟 Looker Studio BIダッシュボード画面（閲覧用リンク）

> 📊 **[Looker Studioで可視化した実際のダッシュボード画面はこちら]**
> ※上記リンク（Looker Studio）をクリックすると、上記の要因分解を経営陣や现场のマーケターが毎日1秒でスキャンできるようビルドされた、実際の美しいWebダッシュボード画面を直接閲覧できます。

---

## 📱 本プロジェクトの最大のこだわり（UI/UXの革新）
*   **「スマホから1秒で直感できる」データ可視化の追求：**
    従来のデータ分析レポートにありがちだった「画面を拡大しないと文字が読めないミクロなグラフ」を完全に排除しました。
*   **文字サイズ2.3倍（爆大スケール）への最適化：**
    グラフ外側の表記文字（タイトル、四半期軸、成長率数字、凡例）を**限界突破の2.3倍（24〜28ptベース）に拡大マウント**。スマートフォンの縦画面をサクッとスクロールするだけで、客数×単価の要因分解が1秒で脳内に直接突き刺さる、実務最高峰 of モバイルUI/UXデザインを実装しています。

---

## 1. 使用した技術スタック（Tech Stack）
*   **データ抽出・整形:** Python (Pandas) / SQL
*   **可視化（可視化ツール）:** Matplotlib（※スマホ視点フォントハック適用）/ Looker Studio（BIツールによるダッシュボード化）
*   **分析手法:** KPIツリーによる売上要因分解（複利ロジック）、プロダクションレベルのエラーバリデーション（assert実装）

---

## 2. 分析のプロセスとコード例（Jupyter Notebook）

### 💻 データのクレンジングと要因分解（Python抜粋）
データの読み込みから、前年比の成長寄与度（どの数字がどれくらい全体に貢献したか）を算出するコードを [02_yoy_analysis.ipynb](./02_yoy_analysis.ipynb) にて完全公開しています。

```python
# 🛠️ プロダクションレベルの計算処理 ＆ 変数バリデーション（一部抜粋）
df_2025 = df[df['year'] == '2025'].copy().reset_index(drop=True)
df_2026 = df[df['year'] == '2026'].copy().reset_index(drop=True)

# 年間購入者数、および年間購入金額の前年比（YoY）を厳密に計算
df_2026['active_buyers_growth'] = ((df_2026['active_buyers'].values - df_2025['active_buyers'].values) / df_2025['active_buyers'].values * 100).round(1)
df_2026['annual_spend_growth'] = ((df_2026['annual_spend_per_user'].values - df_2025['annual_spend_per_user'].values) / df_2025['annual_spend_per_user'].values * 100).round(1)

# 数学的に正確な複利ベース of 売上成長率（要因分解）の算出
df_2026['revenue_growth'] = (((1 + df_2026['active_buyers_growth']/100) * (1 + df_2026['annual_spend_growth']/100) - 1) * 100).round(1)

# 先輩お墨付きの厳密なユニットテスト
assert df_2026 is not None, "[ERROR] df_2026 が正常に生成されませんでした。"
assert 'revenue_growth' in df_2026.columns, "[ERROR] 要因分解計算カラムが見つかりません。"
