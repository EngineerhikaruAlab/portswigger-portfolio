<div align="center">

# Authentication

**認証機構の観察・フロー分析・実装不備の検証を通じて、  
Webアプリケーションにおける Authentication の脆弱性を整理したWriteup群**

![Category](https://img.shields.io/badge/category-Authentication-111111?style=for-the-badge)
![Focus](https://img.shields.io/badge/focus-Auth%20Flow%20%7C%20Response%20Diff-111111?style=for-the-badge)
![Tools](https://img.shields.io/badge/tools-Burp%20Suite%20%7C%20DevTools-111111?style=for-the-badge)
![Writeups](https://img.shields.io/badge/writeups-3-111111?style=for-the-badge)

</div>

---

## 概要

このディレクトリでは、**PortSwigger Web Security Academy** の Authentication テーマを通じて学習した  
認証機構の脆弱性に関するWriteupを整理しています。

Authentication は、単に「ログインできる / できない」を扱うだけでなく、  
以下のような複数の観点が絡む領域です。

- ユーザー名や認証状態の扱い
- 認証失敗時のレスポンス統一
- ブルートフォース耐性
- 多要素認証（2FA / MFA）のフロー設計
- パスワードリセット機構の安全性
- サーバ側で保持すべき認証状態の管理

本ディレクトリでは、単発の攻略メモではなく、  
**認証機構のどこに設計上・実装上の弱点があったのか** を、観察・仮説・検証の流れで整理することを重視しています。

---

## このテーマで重視していること

Authentication のWriteupでは、特に次の点を意識しています。

- 認証フロー全体を状態遷移として捉えること
- レスポンス差分やエラーメッセージの揺れを観察すること
- 認証成功 / 失敗の判定根拠を明確にすること
- 「認証機能が存在すること」と「安全に設計されていること」は別だと理解すること
- 攻撃手順だけでなく、原因と対策まで整理すること

---

## 掲載している代表Writeup

このテーマでは、以下の3本を代表的なWriteupとして整備しています。

| Lab | 主に見てほしい点 | 主な観点 |
| --- | --- | --- |
| [Username enumeration via subtly different responses](./lab-username-enumeration.md) | わずかなレスポンス差分から有効なユーザー名を特定する流れ | response diff / enumeration / brute-force |
| [Password reset broken logic](./lab-password-reset-broken-logic.md) | パスワードリセット処理のロジック不備を突く検証 | reset flow / hidden parameter / server-side validation |
| [2FA broken logic](./lab-2fa-broken-logic.md) | 多要素認証があっても設計不備で突破できることの確認 | 2FA flow / state management / auth logic |

---

## これらのLabを選んだ理由

この3本は、Authentication の中でも異なる切り口を持っており、  
認証機構に対する理解をバランスよく示しやすいと考えています。

### 1. Username enumeration via subtly different responses
このLabでは、**レスポンス差分の観察** が中心になります。  
見た目にはほぼ同じ失敗レスポンスでも、文言や長さのわずかな違いから、  
有効なユーザー名の存在を推測できることを確認できます。

このWriteupでは、以下の力を示しやすいと考えています。

- 小さな差分を観察する力
- Burp Intruder を使った比較
- 成功条件を段階的に切り分ける力

### 2. Password reset broken logic
このLabでは、**パスワードリセット機構のロジック不備** を扱います。  
ログイン画面そのものではなく、その周辺機能である reset flow に問題がある点が重要です。

このWriteupでは、以下の観点が出しやすいです。

- hidden parameter やリクエスト構造の観察
- サーバ側で再検証されるべき情報が信頼されている問題
- 「補助機能の不備が認証破壊につながる」ことの理解

### 3. 2FA broken logic
このLabでは、**2FA が存在していても、それだけでは安全性を保証しない** ことを確認できます。  
多要素認証の導入有無ではなく、**どのように状態遷移を管理しているか** が本質であると分かるLabです。

このWriteupでは、以下の点を示しやすいです。

- 認証フローの状態管理に着目する視点
- パラメータの意味を読み解く力
- 認証処理を画面単位ではなくフロー全体で理解する姿勢

---

## Authentication で重要だと考えている観点

Authentication を学ぶ中で、特に重要だと感じたのは次の観点です。

### レスポンス差分
認証失敗時の文言、レスポンス長、リダイレクト有無、ステータスコードの違いは、  
username enumeration や認証状態の漏えいにつながることがあります。

### 状態遷移
ログイン、2FA、パスワードリセットなどの機能は、  
個別画面ではなく **一連の状態遷移** として観察する必要があります。  
途中のチェックが不十分であれば、認証フロー全体が破綻します。

### サーバ側検証
hidden field やクライアント側の入力値に重要な意味を持たせすぎると、  
サーバ側の再検証不足によって認証回避や不正変更につながります。

### 防御機能の“存在”ではなく“成立”
2FA やブルートフォース対策が「ある」ことと、  
それが攻撃に対して **正しく成立している** ことは別問題です。  
Authentication では、この差を丁寧に見極めることが重要だと考えています。

---

## Writeupの記録方針

各Writeupでは、以下の流れを統一しています。

```text
観察
→ 仮説
→ 検証
→ 成功条件の確認
→ 原因分析
→ 対策整理
→ 学んだこと
