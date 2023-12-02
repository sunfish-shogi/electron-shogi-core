# electron-shogi-core

electron-shogi-core は [electron-shogi](https://github.com/sunfish-shogi/electron-shogi) (Electron将棋) で使用している将棋の局面や棋譜を取り扱う実装です。
TypeScript や JavaScript のアプリケーションに組み込むことができます。

KIF や KI2、CSA、JKF、SFEN/USI といった幅広いフォーマットでの入出力に対応しており、分岐や消費時間、コメントなど、それらのフォーマットで用いられるほとんどの表現に対応しています。

## インストール

```
npm install electron-shogi-core
```

## 基本的な使用方法

```.ts
import fs from "node:fs";
import { importKIF } from "electron-shogi-core";

/*
 * .kifu ファイルを読み込む。
 */
// NOTE: .kif ファイルの場合は Shift-JIS から UTF-8 に変換する必要がある。
const data = fs.readFileSync("sample.kifu", "utf-8");
const record = importKIF(data);
if (record instanceof Error) {
  throw record;
}

/*
 * 指し手を表示用の形式で出力する。
 *
 * 出力:
 *   1 ☗２六歩
 *   2 ☖８四歩
 *   3 ☗２五歩
 *   4 ☖８五歩
 *   5 ☗７八金
 *   6 中断
 */
while (record.goForward()) {
  // NOTE: record.current は現在の指し手、record.position は現在の局面を表す。
  console.log(record.current.ply, record.current.displayText);
}

// 指し手を追加する。
const move = record.position.createMoveByUSI("4a3b");
if (!move) {
  throw new Error("Invalid SFEN");
}
// NOTE: 合法手チェックをしない場合は ignoreValidation オプションを指定する。
if (!record.append(move)) {
  throw new Error("Illegal move");
}
```

## 読み取り専用型

接頭辞 `Immutable` が付いた interface 型は各クラスの読み取り専用インターフェースです。
値を変更するコードを書きにくくなるだけで、変更されないことを保証するものではありません。

## 補助的な関数や定数

- `function detectRecordFormat` 文字列から棋譜のフォーマットを判別します。
- `enum InitialPositionSFEN` 一般的に用いられる初形配置の SFEN です。
- `function countExistingPieces` 局面に配置された駒を種類ごとに数えます。
- `function countNotExistingPieces` 平手と比べて不足している駒の枚数を数えます。
- `function getBlackPlayerName` 先手または下手の対局者名をフルネーム優先で取得します。
- `function getWhitePlayerName` 後手または上手の対局者名をフルネーム優先で取得します。
- `function getBlackPlayerNamePreferShort` 先手または下手の対局者名を省略表記優先で取得します。
- `function getWhitePlayerNamePreferShort` 後手または上手の対局者名を省略表記優先で取得します。

## 注意点

文字コードの変換機能はありません。Shift_JIS でエンコードされた .kif ファイルや .ki2 ファイルがあることに注意してください。

CSA 形式棋譜フォーマットでは 1 つのファイルに複数の棋譜を埋め込むことが可能ですが、 electron-shogi-core では最初の 1 件しか読み込みません。
全ての棋譜を読み込む場合は、セパレーターが書かれている行で分割してから個別に読み込ませてください。

## ライセンス

[MIT License](LICENSE)
