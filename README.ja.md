# TurboWarp Photogrammetry App

[English](README.md) | **日本語**

校正したカメラを持って移動し、WebGPUで空間を逐次再構成するアプリです。

## 現在の内容

turbowarp-app-templateから生成した初期雛形です。用途固有の機能は未実装です。

- 共通app-shellを利用したモード選択・案内・エラー表示。
- 展開済みSB3ソースと、緑の旗で状態変数を更新する起動確認スクリプト。
- SB3と配布ページのビルド、SHA-256の記録、CI。

配布ページはTurboWarpプレイヤーを内蔵せず、起動確認用SB3のダウンロードを提供します。開発サーバーでダウンロードする際は事前にbuild:sb3を実行してください。

## 実装予定

- レンズ校正プロファイルをcamera-calibration-appが作ったファイルから読み込む。撮影条件に適合しないもの、適合を判定できないものは適用前に拒否する。
- スタンドアロンでは固定ステレオrigから開始し、移動しながら深度・点群・メッシュを逐次更新する。単眼は初期化条件と実寸スケールの根拠を確認してから後続段階で追加する。
- クラスターではペアリング・時刻対応・配置校正を通してから再構成を開始する。品質不足・未校正・切断を成功と表示しない。
- 固定rigの相対姿勢は配置校正の結果を使い、移動中の姿勢はvisual-trackingが担当する。rigの構成が変わったら配置校正をやり直す。
- カメラrigの撮影開始・停止、追跡喪失・復帰、撮影範囲と品質を案内する。
- キーフレームと校正・時刻情報を保存し、再構成結果を書き出す。
- 実機で形状誤差・軌跡誤差・遅延・GPUメモリを検証する。追跡30Hz／形状更新5–10Hzは仮目標とする。

## モード

- **スタンドアロン**：1台のPCで完結する。ペアリングと時刻同期を必要とせず、レンズ校正だけを前提に再構成を始める。
- **クラスター**：複数PCのカメラを使う。QR搬送ペアリング、時刻対応、共通基準への配置校正を経てから再構成する。

## 依存と責務

- camera-source：画像と撮影設定、内部校正プロファイルの契約。
- camera-calibration-app：内部校正プロファイルの生成側。ファイルで受け取り、本appは校正手順とOpenCVを持たない。
- time-space-sync：時刻対応と固定rigの配置。クラスターで使う。
- webrtc-qrcode-pairing／webrtc：クラスターのペアリングと接続。スタンドアロンでは使わない。
- visual-tracking：移動中の姿勢と疎な地図。
- photogrammetry：深度推定と融合・形状生成。
- aframe：姿勢・投影設定と再構成結果の描画。

実際の依存はpackage.jsonのturbowarp-app-shell 0.2.0のみです。上記の用途固有の接続は予定であり、未公開の初期拡張に依存しません。追加時には拡張のexact version、配布物hash、API manifest、評価順序を固定します。

## 構成と開発

Node.js >=22.18.0、pnpm 11.11.0。

```bash
corepack enable
pnpm install --frozen-lockfile
pnpm check
pnpm dev
```

- config/app.json：名前、モード、説明、実装予定。
- config/feature-flags.ts：起動時固定・既定OFFの実験機能フラグ。
- scripts/project.ts：起動確認用SB3の正本。
- apps/main/source：生成した展開済みSB3ソース。
- src：共通シェルを利用する配布ページ。
- public/downloads：生成SB3とrelease.json。
- dist：配布ページとダウンロードのビルド結果。

project.tsやtitleを変更したらpnpm source:updateで生成ソースを更新します。生成SB3・distはGit管理対象外です。アーカイブはsb3-toolchainで生成します。

## 段階導入と受け入れ基準

1. 関連GitHub Issueで既存実装の抽出対象、依存、DoD、切戻しを確定する。
2. 用途固有の経路を既定OFFで追加し、既存側は委譲へ置き換える。
3. 機材による統合検証で誤差・遅延・停止と復旧を記録する。
4. 本体拡張のアルゴリズムをアプリに重複実装しない。

初期雛形のDoDはpnpm check成功、SB3で緑の旗による状態更新、配布ページで説明・モード選択・SB3ダウンロードが確認できることです。カメラを使う用途機能の実機検証は未実施です。

## ロールバックとタスク管理

新経路はconfig/feature-flags.tsのフラグOFFで止め、移行中は旧アプリ経路と互換読取りを保持します。初期フラグをONにしても用途固有の機能は実装されません。

GitHub Issuesを進捗の正本とし、start/done/blockedを記録します。

## 抽出元

紙芝居アプリとrealtime-motion-capture-appから抽出した共通構成を利用しています。詳しくは[抽出記録](docs/extraction.md)を参照してください。

## ライセンス

MPL-2.0。packageは初期状態ではprivateです。
