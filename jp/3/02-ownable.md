---
title: Ownable コントラクト
actions: ["答え合わせ", "ヒント"]
requireLogin: true
material:
  editor:
    language: sol
    startingCode:
      "zombiefactory.sol": |
        pragma solidity ^0.4.19;

        // 1. ここにインポートせよ

        // 2. ここで継承せよ:
        contract ZombieFactory {

            event NewZombie(uint zombieId, string name, uint dna);

            uint dnaDigits = 16;
            uint dnaModulus = 10 ** dnaDigits;

            struct Zombie {
                string name;
                uint dna;
            }

            Zombie[] public zombies;

            mapping (uint => address) public zombieToOwner;
            mapping (address => uint) ownerZombieCount;

            function _createZombie(string _name, uint _dna) internal {
                uint id = zombies.push(Zombie(_name, _dna)) - 1;
                zombieToOwner[id] = msg.sender;
                ownerZombieCount[msg.sender]++;
                NewZombie(id, _name, _dna);
            }

            function _generateRandomDna(string _str) private view returns (uint) {
                uint rand = uint(keccak256(_str));
                return rand % dnaModulus;
            }

            function createRandomZombie(string _name) public {
                require(ownerZombieCount[msg.sender] == 0);
                uint randDna = _generateRandomDna(_name);
                randDna = randDna - randDna % 100;
                _createZombie(_name, randDna);
            }

        }
      "zombiefeeding.sol": |
        pragma solidity ^0.4.19;

        import "./zombiefactory.sol";

        contract KittyInterface {
          function getKitty(uint256 _id) external view returns (
            bool isGestating,
            bool isReady,
            uint256 cooldownIndex,
            uint256 nextActionAt,
            uint256 siringWithId,
            uint256 birthTime,
            uint256 matronId,
            uint256 sireId,
            uint256 generation,
            uint256 genes
          );
        }

        contract ZombieFeeding is ZombieFactory {

          KittyInterface kittyContract;

          function setKittyContractAddress(address _address) external {
            kittyContract = KittyInterface(_address);
          }

          function feedAndMultiply(uint _zombieId, uint _targetDna, string _species) public {
            require(msg.sender == zombieToOwner[_zombieId]);
            Zombie storage myZombie = zombies[_zombieId];
            _targetDna = _targetDna % dnaModulus;
            uint newDna = (myZombie.dna + _targetDna) / 2;
            if (keccak256(_species) == keccak256("kitty")) {
              newDna = newDna - newDna % 100 + 99;
            }
            _createZombie("NoName", newDna);
          }

          function feedOnKitty(uint _zombieId, uint _kittyId) public {
            uint kittyDna;
            (,,,,,,,,,kittyDna) = kittyContract.getKitty(_kittyId);
            feedAndMultiply(_zombieId, kittyDna, "kitty");
          }

        }
      "ownable.sol": |
        /**
         * @title Ownable
         * @dev The Ownable contract has an owner address, and provides basic authorization control
         * functions, this simplifies the implementation of "user permissions".
         */
        contract Ownable {
          address public owner;

          event OwnershipTransferred(address indexed previousOwner, address indexed newOwner);

          /**
           * @dev The Ownable constructor sets the original `owner` of the contract to the sender
           * account.
           */
          function Ownable() public {
            owner = msg.sender;
          }


          /**
           * @dev Throws if called by any account other than the owner.
           */
          modifier onlyOwner() {
            require(msg.sender == owner);
            _;
          }


          /**
           * @dev Allows the current owner to transfer control of the contract to a newOwner.
           * @param newOwner The address to transfer ownership to.
           */
          function transferOwnership(address newOwner) public onlyOwner {
            require(newOwner != address(0));
            OwnershipTransferred(owner, newOwner);
            owner = newOwner;
          }

        }
    answer: >
      pragma solidity ^0.4.19;

      import "./ownable.sol";

      contract ZombieFactory is Ownable {

          event NewZombie(uint zombieId, string name, uint dna);

          uint dnaDigits = 16;
          uint dnaModulus = 10 ** dnaDigits;

          struct Zombie {
              string name;
              uint dna;
          }

          Zombie[] public zombies;

          mapping (uint => address) public zombieToOwner;
          mapping (address => uint) ownerZombieCount;

          function _createZombie(string _name, uint _dna) internal {
              uint id = zombies.push(Zombie(_name, _dna)) - 1;
              zombieToOwner[id] = msg.sender;
              ownerZombieCount[msg.sender]++;
              NewZombie(id, _name, _dna);
          }

          function _generateRandomDna(string _str) private view returns (uint) {
              uint rand = uint(keccak256(_str));
              return rand % dnaModulus;
          }

          function createRandomZombie(string _name) public {
              require(ownerZombieCount[msg.sender] == 0);
              uint randDna = _generateRandomDna(_name);
              randDna = randDna - randDna % 100;
              _createZombie(_name, randDna);
          }

      }
---

どれ、前のチャプターでセキュリティホールに気がついたか？

`setKittyContractAddress` は `external`だから、だれでも呼び出すことができるの�
！つまり、この関数を呼び出した奴はクリプトキティーズのコントラクトのアドレスを変
更することで、我々のアプリをめちゃくちゃにすることが可能なのだ。

もちろん、アドレスの更新ができるようにするつもりだが、誰でも更新できるようにした
いわけでない。

ではどうすれば良いかというと、よくあるやり方としては、コントラクトを`Ownable`（
所有可能）とすることだ。これはコントラクトには特別な権限を持つオーナー（所有者）
がいることを意味するものだ。

## OpenZeppelin の `Ownable` コントラクト

一つ例を見せてやろう。これは Solidity のライブラリにある **_OpenZeppelin_** とい
う`Ownable`コントラクトだ。OpenZeppelin は安全でしかもコミュニティで検証を経たス
マートコントラクトだ。これを DApp で使うことができる。このレッスンの後で
、OpenZeppelin のサイトをチェックしておくように。今後非常に役に立つだろう！

下のコントラクトをよく読むように。まだわからないことがいくつもあるが、あとで教え
るから今は気にすることはない。

```
/**
 * @title Ownable
 * @dev The Ownable contract has an owner address, and provides basic authorization control
 * functions, this simplifies the implementation of "user permissions".
 */
contract Ownable {
  address public owner;
  event OwnershipTransferred(address indexed previousOwner, address indexed newOwner);

  /**
   * @dev The Ownable constructor sets the original `owner` of the contract to the sender
   * account.
   */
  function Ownable() public {
    owner = msg.sender;
  }

  /**
   * @dev Throws if called by any account other than the owner.
   */
  modifier onlyOwner() {
    require(msg.sender == owner);
    _;
  }

  /**
   * @dev Allows the current owner to transfer control of the contract to a newOwner.
   * @param newOwner The address to transfer ownership to.
   */
  function transferOwnership(address newOwner) public onlyOwner {
    require(newOwner != address(0));
    OwnershipTransferred(owner, newOwner);
    owner = newOwner;
  }
}
```

新しいものがいくつかあるから、解説してやるぞ：

- コンストラクタ： `function Ownable()`は**_コンストラクタ_**だ。これは特別な関
  数で、コントラクトと同じ名前だ。コントラクトが最初に作成された時に、1 度だけ実
  行されるぞ。
- 関数修飾子：修飾子は半分関数のようなもので、他の関数を編集する際に使うものだ。
  通常は実行する前に要件をチェックするために使用するぞ。この例で言えば
  、`onlyOwner`は**owner(オーナー）だけ**が関数を実行できるように、制限をアクセ
  スするために使用されているのだ。関数修飾子については次のチャプターで詳しく説明
  してやろう。`_;` という奇妙なものについてもな。
- `indexed` キーワード：これは無視して良い。必要ない。

`Ownable`コントラクトは基本的には次のようになる。これを覚えておくようにな：

1. コントラクトが作られた時、コンストラクタが`owner` を `msg.sender` （実行した
   人物だ）に設定する。

2. `onlyOwner`修飾子を追加して、`owner`だけが特定の関数にアクセスできるように設
   定する。

3. 新しい`owner`にコントラクトを譲渡することも可能�

`onlyOwner`は誰もが皆必要としているものだから非常に一般的になった。だから
Solidity の DApp を開発するときには、皆がこの`Ownable`コントラクトをコピーペース
トしてから、最初のコントラクトの継承を始めているのだ。我々
も`setKittyContractAddress` を`onlyOwner`に制限したいから、同じ様にするのだぞ。

## それではテスト�

新しいファイルの `ownable.sol`にすでに`Ownable`のコードをコピーしてある。そこ
で`ZombieFactory` を継承させることにする。

1. `ownable.sol`を`import`するように我々のコードを変更せよ。なに、やり方を忘れて
   しまっただと？その場合は`zombiefeeding.sol`を見て思い出すのだ。

2. `ZombieFactory`コントラクトを編集して`Ownable`を継承させよ。やり方を忘れてい
   たら`zombiefeeding.sol` を見て思い出すように。
