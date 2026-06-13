# App Clip 配信用ドメインの設定

このフォルダの `apple-app-site-association`(拡張子なし、JSON)を GitHub Pages か独自ドメインに公開する。

## 必要事項

- **配信パス**: `/.well-known/apple-app-site-association`(ルート直下でも可、Apple は両方探す)
- **Content-Type**: `application/json`(GitHub Pages はファイル名と内容から自動判定するため、拡張子なしでも JSON として配信される)
- **HTTPS 必須**(GitHub Pages は自動で HTTPS 提供)
- **キャッシュ**: Apple は AASA を CDN 経由でキャッシュする。変更後の反映には数時間〜数日かかる場合あり

## GitHub Pages での公開手順

```bash
# 例: ローカルで作業する場合
cd ~/Documents/iPhone アプリ開発/008-OneDigitField/AppClip-Domain
mkdir -p .well-known
cp apple-app-site-association .well-known/

# 新規 git リポジトリ作成して GitHub に push
git init
git add .
git commit -m "Initial App Clip AASA"
git branch -M main
git remote add origin https://github.com/<your-username>/onedigitfield-clip.git
git push -u origin main
```

GitHub の Repository Settings → Pages → Source = main / root → Save。

公開後の確認:

```bash
curl -i https://<your-username>.github.io/.well-known/apple-app-site-association
```

`Content-Type: application/json` が返り、JSON 本文が見えれば OK。

## ファイルの中身(apple-app-site-association)

```
{
  "applinks": {
    "details": [
      { "appIDs": ["AQVH2U33P9.info.hrathnir.app008.clip"], "components": [{ "/": "/play" }] }
    ]
  },
  "appclips": {
    "apps": ["AQVH2U33P9.info.hrathnir.app008.clip"]
  }
}
```

- `AQVH2U33P9` は Apple Developer の Team ID(現状 project.pbxproj の DEVELOPMENT_TEAM から確認可)
- `info.hrathnir.app008.clip` は App Clip Bundle ID(Xcode で新ターゲット作成時に指定)
- `/play` は起動 URL のパス(`https://<domain>/play?xxx` のようにスキャンされる)

## Bundle ID 登録(Apple Developer Portal)

1. https://developer.apple.com/account → Identifiers → ＋ → App IDs → App
2. **Bundle ID**: `info.hrathnir.app008.clip` を **Explicit** で登録
3. **Capabilities**: Associated Domains にチェック
4. **Parent Application**: `info.hrathnir.app008` を選択(App Clip 用 ID 登録時の必須項目)

その後 Xcode の Signing & Capabilities で対応する Provisioning Profile が自動取得される。

## App Clip カード(QR/招待)用画像

App Store Connect で **App Clip Experience** を作成する際に以下が必要:

- アプリ名(短いキャッチコピー、20 文字以内)
- 招待カード用画像 1800×1200 px
- ヘッダー画像 1200×580 px
- 起動 URL: `https://<your-domain>/play`

これらは Apple のレビュー対象。

## QR / NFC への展開(将来)

App Clip Code(専用 QR 風コード)は App Store Connect で生成可能。
名刺、ポスター、Web ページなどに貼り付ければ、対応 iPhone でかざすだけで起動。
