# OpenCmd

Windows 11 の右クリックメニューに「管理者としてコマンドプロンプトを開く」を追加するレジストリ設定ツール。

## 機能

右クリックメニューから、現在のフォルダパスで管理者権限のコマンドプロンプトを直接開くことができます。

対応する右クリック操作:
- **フォルダ内の空白部分**を右クリック → そのフォルダのパスでCMDを開く
- **フォルダアイコン**を右クリック → そのフォルダのパスでCMDを開く
- **ドライブ**を右クリック → そのドライブのルートでCMDを開く

## インストール

1. `install.reg` をダブルクリック
2. 「レジストリ エディターに情報を追加しますか？」→「はい」をクリック
3. 完了

## アンインストール

1. `uninstall.reg` をダブルクリック
2. 「レジストリ エディターに情報を追加しますか？」→「はい」をクリック
3. 完了

## 使い方

1. エクスプローラーで任意のフォルダを開く
2. フォルダ内の空白部分（またはフォルダアイコン）を右クリック
3. 「その他のオプションを確認」をクリック（Windows 11の場合）
4. 「管理者としてコマンドプロンプトを開く」をクリック
5. UACプロンプトで「はい」をクリック
6. そのフォルダのパスで管理者権限のCMDが開き、`claude -c --dangerously-skip-permissions` のテキストが表示される

> **補足**: Windows 11では、このメニュー項目は「その他のオプションを確認」（Shift+右クリックでも表示可能）の中に表示されます。Windows 11のモダンメニュー（トップレベル）への追加はCOM拡張+MSIX署名が必要なため、レジストリ方式ではレガシーメニュー配下に表示されます。

## 動作の仕組み

メニュー選択時に以下の処理が実行されます:

```
PowerShell -windowstyle hidden -Command "Start-Process cmd.exe -ArgumentList '/s /k pushd \"%V\" && echo claude -c --dangerously-skip-permissions' -Verb RunAs"
```

1. PowerShellを非表示で起動
2. `Start-Process ... -Verb RunAs` でUAC昇格を要求
3. `cmd.exe /s /k pushd "%V"` で対象フォルダに移動したCMDを開く
4. `echo claude -c --dangerously-skip-permissions` でコマンドテキストを画面に表示
5. `pushd` はUNCパス（`\\server\share`）やドライブ間移動にも対応

## 動作環境

- Windows 10 / Windows 11
- PowerShell 5.1以上（Windows標準搭載）

## ライセンス

MIT License
