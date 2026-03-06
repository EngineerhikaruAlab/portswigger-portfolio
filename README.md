<div align="center">

# PortSwigger Web Security Academy Writeup Portfolio

**観察・仮説・検証・原因分析を重視した  
Webアプリケーション脆弱性学習のポートフォリオ**

![Status](https://img.shields.io/badge/status-active-111111?style=for-the-badge)
![Focus](https://img.shields.io/badge/focus-Web%20Security-111111?style=for-the-badge)
![Tools](https://img.shields.io/badge/tools-Burp%20Suite%20%7C%20DevTools-111111?style=for-the-badge)
![Language](https://img.shields.io/badge/language-Japanese-111111?style=for-the-badge)

</div>

---

## 概要

本リポジトリは、**PortSwigger Web Security Academy** を通じて学習した  
Webアプリケーション脆弱性の検証過程を整理したWriteupポートフォリオです。

単なるLabの解答集ではなく、以下の流れを重視して記録しています。

- HTTPリクエスト / レスポンスや画面挙動の観察
- 脆弱性の成立条件に関する仮説立案
- Burp Suite を用いた検証手順の整理
- 成功条件の根拠の明確化
- 開発者視点での原因分析と対策整理

> **このポートフォリオで重視していること**  
> 「解けたこと」そのものではなく、**どのように観察し、どう仮説を立て、何を根拠に成功と判断したか** を再現可能な形で残すことを重視しています。

---

## 目次

- [概要](#概要)
- [このポートフォリオの目的](#このポートフォリオの目的)
- [特に見てほしいWriteup](#特に見てほしいwriteup)
- [検証方針](#検証方針)
- [学習テーマ](#学習テーマ)
- [リポジトリ構成](#リポジトリ構成)
- [使用ツール・環境](#使用ツール環境)
- [このポートフォリオで示したいこと](#このポートフォリオで示したいこと)
- [今後の課題](#今後の課題)
- [補足](#補足)

---

## このポートフォリオの目的

本ポートフォリオの目的は、Webアプリケーションの脆弱性について  
**「Labを解けた」という結果だけでなく、「どのように考えて検証したか」を技術文書として整理すること** にあります。

特に、以下の点を示すことを意識しています。

- 脆弱性の種類ごとの特徴を理解していること
- レスポンス差分や認証フローを観察し、成立条件を説明できること
- Burp Suite を用いて検証手順を組み立てられること
- 脆弱性の原因を開発者視点でも捉えられること
- 学習内容をMarkdownで再利用可能な形に整理できること

---

## 特に見てほしいWriteup

代表的なWriteupとして、以下を重点的に整備しています。

| 分類 | Lab | 主に見てほしい点 |
| --- | --- | --- |
| Authentication | Username enumeration via subtly different responses | レスポンス差分の観察、列挙の成立条件整理 |
| Authentication | 2FA bypass using a brute-force attack | 認証フロー分析、Burp Intruder の活用 |
| Access Control | Unprotected admin functionality | 認可不備の発見、管理機能への到達確認 |
| SQL Injection | Login bypass | 認証回避の成立条件、入力処理の問題整理 |
| OS Command Injection | Basic command injection | コマンド実行の確認、サーバ側挙動の理解 |

---

## 検証方針

各Writeupでは、次の流れを統一して記録しています。

```text
観察
→ 仮説
→ 検証
→ 成功条件の確認
→ 原因分析
→ 対策整理
→ 学んだこと
