---
title: 数式演算
actions: ["答え合わせ", "ヒント"]
material:
  editor:
    language: sol
    startingCode: |
      pragma solidity ^0.4.19;

      contract ZombieFactory {

          uint dnaDigits = 16;
          // ここにdnaModulusを定義するの�

      }
    answer: >
      pragma solidity ^0.4.19;


      contract ZombieFactory {

          uint dnaDigits = 16;
          uint dnaModulus = 10 ** dnaDigits;

      }
---Solidity で使う数式は誰でもわかるような簡単なものだ。他のプログラム言語と全く
同じだと思ってもらっていい：

- 加算（足し算）: `x + y`
- 減算（引き算）: `x - y`,
- 乗算（掛け算）: `x * y`
- 除算（割り算）: `x / y`
- 剰余（余り）: `x % y` _(例えば、`13 % 5` は `3`になる。なぜかというと、13 を 5
  で割ると、余りが 3 だからだ.)_

Solidity は**_指数演算子_**もサポートしている。(例 "x の y 乗"、 x^y):

```
uint x = 5 ** 2; // 5^2 = 25 と同様
```

# テストの実行

ゾンビの DNA が 16 桁の数字だと確認するために、別の`uint`を作成して 10^16 と設定
せよ。後のレクチャーでは剰余演算子である `%`を使用して整数を 16 桁に縮小できる。

1. `dnaModulus`という名前の `uint`を作成し、**10 の`dnaDigits`乗**に設定せよ。
