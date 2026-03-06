# PortSwigger Web Security Academy Writeup Portfolio

## 概要
本リポジトリは、PortSwigger Web Security Academy を通じて学習した  
Webアプリケーション脆弱性の検証過程を整理したポートフォリオです。

単なるLabの解答集ではなく、以下の流れを重視して記録しています。

- HTTP通信・画面挙動・レスポンス差分の観察
- 脆弱性の成立条件に関する仮説立案
- Burp Suite を用いた検証手順の整理
- 成功条件の根拠の明確化
- 開発者視点での原因分析と対策整理

---

## このポートフォリオの目的
本ポートフォリオの目的は、Webアプリケーションの脆弱性について  
「解けたこと」だけでなく、「どのように考えて検証したか」を記録し、  
再現性のある形で整理することにあります。

特に、以下の点を示すことを意識しています。

- 脆弱性の種類ごとの特徴を理解していること
- レスポンス差分や認証フローを観察し、成立条件を説明できること
- Burp Suite を用いて検証手順を組み立てられること
- 脆弱性の原因を開発者視点でも捉えられること

---

## 学習テーマ
- Authentication
- Access Control
- SQL Injection
- OS Command Injection
- Path Traversal

各テーマの概要は、テーマ別READMEに整理しています。

---

## 特に見てほしいWriteup
代表的なWriteupとして、以下を重点的に整備しています。

- Authentication
  - Username enumeration via subtly different responses
  - 2FA bypass using a brute-force attack

- Access Control
  - Unprotected admin functionality

- SQL Injection
  - Login bypass
  - UNION attack

- OS Command Injection
  - Basic command injection

これらのWriteupでは、観察・仮説・検証・成功条件・原因・対策までを一連の流れでまとめています。

---

## 記録方針
各Writeupでは、次の観点を統一して記録しています。

1. 目的  
2. 脆弱性の概要  
3. 攻撃の結論  
4. 事前観察  
5. 仮説  
6. 検証手順  
7. 成功条件と根拠  
8. 原因  
9. 対策  
10. 学んだこと  

この構成により、単発の攻略メモではなく、  
再現性と説明可能性を重視した技術記録となるようにしています。

---

## 使用ツール・環境
- Burp Suite
  - Repeater
  - Intruder
- Browser DevTools
- PortSwigger Web Security Academy Lab Environment
- Markdown / GitHub

---

## リポジトリ構成
```text
writeups/
├─ authentication/
├─ access-control/
├─ sqli/
├─ command-injection/
└─ path-traversal/
