---
title: Keccak256と型キャスト
actions: ["答え合わせ", "ヒント"]
material:
  editor:
    language: sol
    startingCode: |
      pragma solidity ^0.4.19;

      contract ZombieFactory {

          uint dnaDigits = 16;
          uint dnaModulus = 10 ** dnaDigits;

          struct Zombie {
              string name;
              uint dna;
          }

          Zombie[] public zombies;

          function _createZombie(string _name, uint _dna) private {
              zombies.push(Zombie(_name, _dna));
          }

          function _generateRandomDna(string _str) private view returns (uint) {
              // ここから始めるの�
          }

      }
    answer: >
      pragma solidity ^0.4.19;


      contract ZombieFactory {

          uint dnaDigits = 16;
          uint dnaModulus = 10 ** dnaDigits;

          struct Zombie {
              string name;
              uint dna;
          }

          Zombie[] public zombies;

          function _createZombie(string _name, uint _dna) private {
              zombies.push(Zombie(_name, _dna));
          }

          function _generateRandomDna(string _str) private view returns (uint) {
              uint rand = uint(keccak256(_str));
              return rand % dnaModulus;
          }

      }
---

`_generateRandomDna`関数を（セミ）ランダムな`uint`で返したいのだが、何か良いアイ
ディアがあるかな？

イーサリアムには SHA3 のバージョンの一つである`keccak256`が組み込まれている。ハ
ッシュ関数は基本的には、文字列をランダムな 256 ビットの 16 進数にマッピングする
機能だ。文字列をほんの少しでも変更すれば、ハッシュは大きく変わるから気をつけるよ
うにな。

イーサリアムのいろいろな場面で使用できるが、ここでは擬似乱数生成に使用していくぞ
。

例：

```
//6e91ec6b618bb462a4a6ee5aa2cb0e9cf30f7a052bb467b0ba58b8748c00d2e5
keccak256("aaaab");
//b1f078126895a1424524de5321b339ab00408010b7cf0e6ed451514981e58aa9
keccak256("aaaac");
```

見てわかると思うが、入力する文字が１文字違うだけで、戻り値が全く別物になることが
わかったかな。

> 注：ブロックチェーンでの**安全な**乱数の生成は非常に難しい課題です。ここで紹介
> する方法は安全なものではありませんが、ゾンビ DNA の作成のチュートリアルではセ
> キュリティを考慮する必要はないので、この方法で十分です。

## 型キャスト

場合によっては、データ型を変更する必要があるときがある。下の例で考えてみるぞ：

```
uint8 a = 5;
uint b = 6;
// a * b はuint8ではなくuintで返すから、エラーになる：
uint8 c = a * b;
// 正しく動作させるために、bをuint8に型キャストさせる：
uint8 c = a * uint8(b);
```

この例では`a * b`は`uint`を返すが、`uint8`で格納しようとしているから、問題が発生
することになる。`uint8`にキャストすることで、正常に動作する上にコンパイラもエラ
ーを吐き出すことがなくなる。

# それではテスト�

`_generateRandomDna`関数の中身を書いてみよ！以下の点に従って書くように：

1. コードの最初の行は `_str`の`keccak256`ハッシュを取得し、擬似乱数の 16 進数を
   生成し、それを`uint`に型キャストして、 `rand`という`uint`に格納せよ。

2. DNA は 16 桁になるようにしたい（`dnaModulus`を覚えているか？）。そこで次の行
   では上で求めた値の`dnaModulus`による剰余(`%`)を `return`するようにせよ。
