# Lab: Password reset broken logic

> **分類**: Authentication  
> **難易度**: Apprentice  
> **学習テーマ**: password reset / broken logic / hidden parameter tampering  
> **使用ツール**: Burp Suite（Proxy / Repeater）, Browser DevTools

---

## 1. 概要

このLabでは、**パスワードリセット機能のロジック不備**を利用して、他ユーザーのパスワードを不正に変更できることを確認する。  
主なポイントは、パスワード再設定URLに含まれる **reset token** が、再設定フォーム送信時には実質的に検証されておらず、さらに **username** が hidden input としてクライアントから送信されている点にある。

このWriteupでは、以下を中心に整理する。

- パスワードリセット処理のどこに問題があるか
- どの観察結果から token 未検証を疑ったか
- どのような手順で Carlos のパスワードを変更したか
- なぜこの挙動が脆弱性として成立するか
- 開発者視点では何を修正すべきか

---

## 2. このLabで問われていること

このLabのゴールは、**Carlos のパスワードを攻撃者が任意の値に変更し、その新しいパスワードでログインして My account ページへアクセスすること**である。

利用可能な初期情報は以下の通り。

- 自分の認証情報: `wiener:peter`
- 標的ユーザー: `carlos`

---

## 3. 脆弱性の要点

このLabで扱う脆弱性は、**password reset broken logic** である。

本来、パスワードリセット機能では、メールで送信される reset token が「誰のパスワードを、どの期限内で、どの1回だけ変更できるか」をサーバ側で厳密に管理し、**再設定フォーム送信時にも再度その token を検証する必要がある**。

しかしこのLabでは、再設定フォーム送信時の `POST /forgot-password?temp-forgot-password-token=...` において、**token を URL と body の両方から削除してもパスワード変更が通る**。さらに、送信される `username` は hidden input に依存しており、これを `carlos` に変更することで、他ユーザーのパスワード変更が成立する。

### 3.1 本質

この脆弱性の本質は、**パスワードリセット token が再設定確定時にサーバ側で検証されておらず、hidden input の `username` を信頼している点**にある。

### 3.2 典型的な原因

- reset token をメールリンク生成時にしか使わず、最終送信時に再検証していない  
- `username` のような重要な識別子を hidden input でクライアントに持たせている  
- 「正しいフローでページに到達したユーザーはその後も信用できる」と誤って想定している  

---

## 4. 事前観察

まず、自分のアカウント `wiener:peter` を使って、通常のパスワードリセットフローを確認した。

### 4.1 対象機能

- エンドポイント: `/forgot-password`
- 入力項目: `username`, `new-password-1`, `new-password-2`
- 関連要素: reset email, URL query parameter, hidden input, temp reset token

### 4.2 初期挙動

通常のフローでは、以下の流れでパスワード再設定が行われる。

1. 「Forgot your password?」から自分のユーザー名を送信する  
2. Email client で reset link を確認する  
3. reset link を開くと、token 付きのパスワード再設定フォームが表示される  
4. 新しいパスワードを2回入力して送信すると、パスワード変更が成立する  

### 4.3 差分として気づいた点

Burp の HTTP history を確認すると、再設定フォーム送信時のリクエストは **`POST /forgot-password?temp-forgot-password-token=...`** となっており、フォーム内には **`username` が hidden input** として含まれていた。

この時点で、以下の2点が不自然だと感じた。

- token が URL と body に現れているが、送信時に本当に検証されているか不明  
- `username` がサーバ側セッションではなく hidden input に依存している

---

## 5. 仮説

観察結果から、**再設定フォーム送信時には reset token が実質的に検証されておらず、`username` を変更すれば別ユーザーのパスワードを変更できる**と考えた。

そのため、以下の二段階で検証する方針を取った。

1. 自分のリセットフローで token を削除してもパスワード変更が通るか確認する  
2. token を削除したまま `username` を `carlos` に変更し、Carlos のパスワード変更が成立するか確認する  

---

## 6. 検証手順

### 6.1 手順1: token が本当に検証されているか確認

**目的**  
再設定フォーム送信時に reset token がサーバ側で検証されているかを確認する。

**実施内容**  
- 「Forgot your password?」から自分のユーザー名 `wiener` を送信した  
- Email client で reset link を開き、新しいパスワードを設定した  
- その際の **`POST /forgot-password?temp-forgot-password-token=...`** を Burp Repeater に送った  
- URL の query parameter と request body の両方から `temp-forgot-password-token` の値を削除して再送した  

**確認ポイント**  
- token を消してもパスワード変更が成立するか  
- サーバが token 不正として拒否するか

**結果**  
token の値を URL と body の両方から削除しても、パスワード変更は成立した。

**解釈**  
この時点で、**再設定確定時に token が検証されていない**ことが分かった。

---

### 6.2 手順2: `username` を変更して他ユーザーのパスワードを更新

**目的**  
hidden input の `username` を改ざんして、Carlos のパスワードを変更できるか確認する。

**実施内容**  
- ブラウザから再度自分のパスワードリセットフローを開始した  
- 再び **`POST /forgot-password?temp-forgot-password-token=...`** を Burp Repeater に送った  
- URL と body の両方から `temp-forgot-password-token` の値を削除した  
- hidden input の `username` を `wiener` から **`carlos`** に変更した  
- `new-password-1` と `new-password-2` に任意の新しいパスワードを設定して送信した  

**確認ポイント**  
- `username=carlos` でパスワード変更が受理されるか  
- 変更後のパスワードで Carlos にログインできるか

**結果**  
リクエストは受理され、その後、設定した新しいパスワードで **Carlos のアカウントにログインできた**。  
最終的に My account ページへアクセスして Lab を解決した。

**解釈**  
パスワードリセット機能が、**token ではなく hidden input の `username` を事実上の決定要素として扱っていた**ことが分かった。

---

## 7. 攻撃の結論

最終的には、**再設定フォーム送信時に token が検証されないこと**を利用し、hidden input の `username` を `carlos` に変更することで、Carlos のパスワードを任意の値に変更して Lab を解決した。

### 最短要約

- 自分のパスワードリセットフローを観察し、`POST /forgot-password?temp-forgot-password-token=...` を取得した  
- token を URL と body の両方から削除しても、パスワード変更が成立することを確認した  
- hidden input の `username` を `carlos` に変更し、新しいパスワードを設定した  
- その新しいパスワードで Carlos にログインし、My account にアクセスした  

---

## 8. 成功条件と根拠

成功したと判断した根拠は次のとおりである。

- token を削除しても再設定リクエストが受理された  
- `username=carlos` に変更したリクエストで、Carlos のパスワードが実際に変更された  
- 設定した新しいパスワードで Carlos のアカウントにログインできた  
- My account ページへアクセスして Lab を完了できた  

これにより、単なるフォーム改ざんではなく、**Carlos の認証情報が実際に攻撃者の制御下に置かれたこと**を確認できた。

---

## 9. なぜ成立したのか（原因分析）

このLabで脆弱性が成立した原因は、**パスワード再設定の最終確定処理が token によって保護されておらず、クライアント送信の `username` を信頼していたこと**にある。

### 9.1 認証設計上の問題

パスワードリセットは、認証の補助機能ではなく、**実質的に本人確認を代替する強い認証機構**である。  
したがって、reset token は「このユーザーのパスワードを変更してよい」という認可そのものであり、再設定フォーム送信時に必ず再確認されなければならない。

しかしこのLabでは、「正しいリンクからフォームに到達した」という前提だけで処理が進み、**最終送信時の認可確認が欠落していた**。

### 9.2 実装上の問題

- `temp-forgot-password-token` を送信時に再検証していない  
- `username` を hidden input でクライアントに持たせている  
- 最終的に「誰のパスワードを変えるか」をサーバ側で固定していない  

---

## 10. 対策

### 10.1 実装面

- reset token を **再設定フォーム送信時にも必ずサーバ側で検証**する  
- token から対象ユーザーを特定し、`username` をクライアント入力に依存させない  
- token は高エントロピー・短寿命・ワンタイムで運用し、使用後は即時失効させる  

### 10.2 運用・設計面

- パスワードリセットを認証機能と同等の重要度でレビューする  
- hidden input や query parameter に重要な認可情報を持たせない  
- 補助機能だからといって認可確認を省略しない  

### 10.3 このLabに即した改善案

このLabでは、まず **`username` hidden field を廃止し、token からサーバ側で対象ユーザーを決定する設計**に改めるべきである。  
そのうえで、フォーム送信時に token を再検証し、期限・未使用・対象ユーザー一致を確認してからパスワード変更を確定すべきである。

---

## 11. このLabで学んだこと

このLabを通じて、**パスワードリセット機能はログイン画面と同じくらい重要な認証機構であり、そこにロジック不備があると直接アカウント乗っ取りにつながる**と理解できた。

また、以下の点が重要だと分かった。

- token が存在していても、**最終送信で再検証されなければ意味がない**  
- hidden input は「見えない」だけであり、**信頼できる入力ではない**  
- 正しい手順を一度通ったユーザーが、その後も正しい入力を送るとは限らない  

### 今後に活かせる観点

- Password reset 系では、**token がどの段階で検証されているか**を必ず確認する  
- hidden input の `username`, `email`, `userId` などを重点的に疑う  
- 「この値は本来サーバ側で決まるべきではないか」という視点で認証補助機能を見る  

---

## 12. 使用ツール

- **Burp Suite**
  - Proxy
  - Repeater
- **Browser DevTools**

---

## 13. 補足メモ

### ハマった点

- 最初は reset token が URL に入っているため、安全に見えやすい  
- しかし実際には、**フォーム送信時に token を消しても通るか**を試さないと本質が見えにくい  

### 再現時の注意点

- token は **URL と body の両方** から削除して確認する  
- `username` hidden field を書き換える前に、まず「token が未検証であること」を確認する  
- 最終的な成功判定は、**Carlos の新しいパスワードで本当にログインできるか**で行う  

### 追加で確認したいこと

- Password reset poisoning のように、reset link 生成自体が壊れているケースとの違い  
- 実サービスで token 再検証漏れをどのようにテストするか  
- 補助認証機能を設計レビューするときの観点整理  

---

## 14. まとめ

このLabでは、**パスワード再設定の最終送信時に reset token が検証されていない**ことを利用し、hidden input の `username` を `carlos` に変更することで、他ユーザーのパスワードを任意の値に変更できた。

重要だったのは、パスワードリセットを単なる補助機能ではなく、**本人確認を代替する認証機構**として見ることだった。  
このWriteupでは、**観察 → 仮説 → 検証 → 成功条件 → 原因分析 → 対策** の流れで整理することで、ロジック不備がどのように直接アカウント乗っ取りにつながるかを明確にできた。
