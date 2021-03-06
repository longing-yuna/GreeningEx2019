# 苗の持ち運び
苗は、アクションキーで運ぶことができる。

## 処理の流れ
- 立っている状態で、アクションキーを押して、最寄りが苗だった時に持ち運び状態に遷移
- アクションキーをもう一度押すか、水キーを押すと、苗を足元に置く
- 苗を置く場所は1mごとに調整
- すでに何か置いてある場所には置けない
- 段差のあるところには置かない
- 高い段差の手前の場合は足元に植える
- 苗を持っている状態ではジャンプできない
- 苗を持っている状態では、ツタの昇り降りはできない
- 苗を持っている状態で水に落ちると、苗を取り落として苗は落下

## アクションキー
アクションキーは状況に応じで行動が変化する。ステラの前方の一定の範囲を探索して、見つけた行動オブジェクトのうち、もっともステラに近いものについて行動を起こす。まずは探索する範囲を調べる処理を実装する。

### アクションキーの仕様
現在有効なオブジェクトは、将来的に光らせたりエッジを描画するなどして分かるようにしたいので、常時選択しておくようにしたい。そのため、ActionBoxにスクリプトを設定して、OnTrigger????で、常に接触しているオブジェクトを見つけるようにする。

- ステラから一定の相対座標の場所を常に監視して、行動が可能なオブジェクトをリストアップし、そのうちの最寄りのものを見つけておく
- 行動できる相手のオブジェクトには、Actableクラスを継承したクラスを作成してアタッチする。このクラスは`CanAction`プロパティーと`Action()`メソッドを実装する
- 行動範囲にあるオブジェクトがActableクラスを持ち、かつ、CanActionがtrueの場合、行動候補にリストアップする
- アクションキーが押されたら、検出済みの最寄りのオブジェクトのAction()メソッドを呼び出すことで、行動を開始する。どのような行動をするかは、Actable.Action()が管理するので、ステラのアクションキーの処理はここまでで終わり


### 探索範囲トリガーの作成
- Hierarchyウィンドウで空のゲームオブジェクトを作成して、`ActionBox`という名前にしておく
- ActionBoxに*BoxCollider*をアタッチする
- ActionBoxのTransformのPositionは0,0,0としておいて、位置と大きさはBoxColliderのCenterとSizeで調整して、行動相手を検出したい範囲をコライダーが示すようにする

![行動範囲の当たり判定](Images/ActionKey00.png)

- 調整ができたら、ProjectウィンドウからStellaプレハブをHierarchyにドラッグ＆ドロップして開く
- Hierarchyウィンドウにおいて、ActionBoxをドラッグして、Stellaにドロップして子供にする
- Inspectorウィンドウで、ActionBoxに以下の設定をする
  - TagとLayerにActionを追加して、どちらもActionにする
  - Is Triggerにチェック
- 変更を反映させるために、StellaのOverridesからApply Allをクリックしてすべて適用

以上が完了したら、HierarchyウィンドウからStellaオブジェクトを削除する。

### 接触の設定
不要なオブジェクトが反応しないようにレイヤーを設定する。Actionが検出したいのは、*Nae*, *MapCollision*, *MapTrigger*の3種類のレイヤーのみなので、Physics設定でそのように設定する。

![レイヤー間の当たり判定](Images/ActionKey01.png)


### スクリプト実装
検出したアクションオブジェクトを管理するためのActableクラスを作成する。

```cs
using UnityEngine;

namespace GreeningEx2019
{
    /// <summary>
    /// アクションキーで動作する対象のオブジェクトは、このクラスを継承して実装します。
    /// </summary>
    public abstract class Actable : MonoBehaviour
    {
        /// <summary>
        /// 行動可能な時、trueを返します。
        /// </summary>
        public bool CanAction { get; protected set; }

        /// <summary>
        /// 行動を実行します。
        /// </summary>
        public abstract void Action();

        /// <summary>
        /// 選択された時に呼び出します。
        /// </summary>
        public virtual void Select() { }

        /// <summary>
        /// 選択を解除します。
        /// </summary>
        public virtual void Deselect() { }
    }
}
```

ActionBoxスクリプトを新規に作成して、ActionBoxオブジェクトにアタッチしてコードを作成する。実装するのは以下のような項目。

- ActionBoxに触れているアクション可能なオブジェクトのインスタンスを保持する配列を定義
- Physics.BoxCastNonAlloc()を使って、設定したActionBoxの範囲内にあるオブジェクトを列挙して、最寄りのオブジェクトを返すメソッドを作成
- BoxCastNonAllocのCenterを設定する際には、XのオフセットをStellaMove.forwardVector.xを掛けて左右フリップを反映させること

#### メモ
当初、OnTriggerEnter()などで作成してみたのだが、トリガーを発動してくれないことが判明した。CharacterControllerだと子供のColliderは動作しない可能性があったため、BoxCastNonAllocに変更した。

## 苗用のアクションを作成する
アクションを呼び出す処理ができたら、苗のアクションを作成してアタッチする。まずは単純に呼び出しが成功しているかを確認するだけのものを用意する。

- `NaeActable`スクリプトを新規に作成して、Actableを継承させる
- 持ち運びが可能な苗のプレハブに、NaeActableをアタッチする
- 以下のようなコードを実装する

```cs
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

namespace GreeningEx2019
{
    public class NaeActable : Actable
    {
        private void Awake()
        {
            CanAction = true;
        }

        public override void Action()
        {
            Debug.Log("Action");
        }
    }
}
```

スクリプトができたら、苗のプレハブに作成したNaeActableスクリプトをアタッチしてPlayする。苗に近づいてアクションキーを押した時に、**Action**とログに表示されればここまで成功である。

## 持ち上げ動作
持ち上げの動作の流れを検討する。

- 持ち上げアニメ
  - 操作不可
- 持ち上げ中
  - 静止と歩き
  - 落下
  - ジャンプ
- 置くアニメ
  - 操作不可
- 置き終わったら立ちに移行

### 持ち上げアニメ
- NaeActable.Action()から開始する
- しゃがみ終わった時と、持ち上げ終わった時の2回、アニメからイベントを呼ぶので、しゃがんだ時に苗を手にくっつける処理、立ち上がった時に持ちアニメへの移行を登録する
- 苗のPickupメソッドに右手のtransformを渡して、持ち上げ状態にする
- アニメが終わったら、持ち上げ中の静止へ

### 持ち上げ中
静止と歩きは、移動範囲や振る舞いが異なるためStellaWalkを継承して別クラスで実装する。落下とジャンプは現状のものを利用する。

歩きの違いは以下の通り。

- 足元のチェックをして、苗を置くべき場所に壁があったら進めなくする
- アクションボタンがもう一度押されたら、苗を置く動作へ
- 苗を置く予定の場所に、薄く苗を表示する

### 苗を置く
苗は高いところから落としたり、すでに何か植わっている場所に置くことはできない。置けない場合は、半分ぐらい動作をしたら戻して歩きに戻るようにする。

- 置けるかどうかによって、アニメとその後の処理を分けて登録
- アニメが完了したら、苗を持ったまま歩きに戻るか、苗を置いて通常歩きに戻す
- 通常歩きへの戻し方
  - 苗を置くためのメソッドを呼び出して、手から外して地面に固定する
  - 苗を持っているフラグを外す

以上を元に実装していく。

## 苗を持ち上げる実装
### 持ち上げ用のScriptableObjectを作成する
StellaLiftUpスクリプトを作成する。まずは流れを確認するために、以下の機能で実装する。

- Initで以下を実行
  - 速度0の歩きアニメに変更する
  - イベントに、苗を持つ処理のためのメソッドを登録
  - アニメのLiftUpトリガーを設定
- UpdateActionでは、歩きを停止させるために、myVelocityのxに0を設定して、Move()を呼び出しておく
- 苗を持つメソッド
  - デバッグ用に何か表示
  - 苗を持つ状態のアニメを設定するためのメソッドを作成して、アニメイベントに登録
- 苗を持つ状態のアニメを設定するためのメソッド
  - 苗歩きへ移行するメソッドをアニメイベントに登録する
  - アニメのNaeをtrueにする
- 苗歩きへ移行するメソッド
  - デバッグ用に何か表示
  - ステラのアニメのNaeをtrueに設定
  - ステラの動作をNaeWalkに変更

呼び出しが多いが、苗を奇麗に持たせるために、アニメの完了直前に苗を持たせた状態を設定したいためにこのようになった。

NaeActableのActionメソッドに、ステラの行動をLiftUpに変更する処理を実装する。苗持ち上げ処理は後で実装するので、一先ずStellaActionWalkを設定しておく。

Playして苗を拾ってみる。ログが表示されればOK。

### 苗を持ち上げる
NaeActableに、苗をステラに持たせるためのメソッドHold()を実装する。

- Colliderを保存する変数を定義しておいて、Awake()でGetComponentする
- オフセット値を定義
- 持ち上げられているフラグを定義して、falseで初期化、Hold()でtrueにする
- Hold()メソッドに以下を実装
  - ステラのじょうろのピボットオブジェクトのTransformを受け取って記録しておく
  - コライダーを無効にする
  - ホールドフラグをtrueにする
- LateUpdate()を追加して、持ち上げられている時に、ピボットオブジェクトの座標にオフセットを加えた位置に移動させて、平行になるように回転を設定する

以上できたら、StellaLiftUpスクリプトの苗を持つメソッドから上記のHold()を呼び出すようにする。Playして動作確認する。

### アニメの調整
持ち上げて立ち上がった辺りの動きをうまく調整しないと、ステラがガクっとブレるように見えてしまう。AnimatorのLiftUpからStandへ戻す時のトランジションを、以下のように調整して奇麗に繋がるように調整した。

- Transition Durationが設定されていると苗を持つ前に腕を戻してしまうので0にする
- Transition Offsetを調整して、立ち上がってからの動きが滑らかに繋がるようにする。0.7辺りできれいに繋がった

以上で、持ち上げる動作は完了。

## 苗を置く予定の場所
苗を置ける場所のルールがややこしいため、混乱を避けるために常に置ける場所を表示するとともに、移動に反映させられるようにする。

- 苗を置く場所を、単位で丸め込んで決定
- 苗を置きたい場所にあるものを列挙
  - 置けないものがあったら、置けない状態にする
    - マーカーオブジェクトを非表示にする
    - 水キーやアクションキーを見ない
  - 何もない場合、床までの高さを確認
    - ステラの足元との差が置ける範囲を越えている場合、置けない状態にする
- 置ける状態なら、水キーとアクションキーをチェックして、押されていたら置く動作へ移行
- 移動先を算出
- 移動先の苗を置く座標を算出して、前の座標と比較
  - 同じならそのまま移動
  - 違う場合、新しい置きたい場所にあるものを列挙して、床があったら移動をキャンセルする

### 苗を置く候補の表現
苗を置く先を表す状態にするマテリアルを作成して、NaeActableに持たせておく。マーカーとして利用するオブジェクトは、ステラが苗を持った時に、持ったオブジェクトを複製して、そのマテリアルをマーカーのものに切り替える。

### 苗歩き
StellaWalkを継承したStellaNaeWalkスクリプトを作成する。StellaNaeWalkで必要になるメンバをprivateからprotectedに変更しつつ、UpdateAction()を上書きして、苗運び用に書き換える。

### 苗を置く場所の決定
オブジェクトを置く場所は、アクション範囲と、持ったオブジェクトの当たり判定Xの半分の場所を基準とする。苗を拾った時に算出する。

- ActionBoxの中心へのオフセットX + halfSize.x + 拾ったオブジェクトのColliderのExtents.xが、基準距離
- ステラの中心座標に、向いている方向を掛けた基準距離を足して座標を求める
- 求めた座標に基準倍率を掛けたあとRound()してから、基準倍率で割る
- 求めた座標が、基準距離より遠くなったら、基準倍率分を減らす
- 求めたX座標に対して、ステラの足元の座標から下方向にレイを飛ばして、見つけた地面の高さをYにする

以上で、苗を置く候補座標が算出できる。移動前と移動候補の2回求めるので、関数にしておく。

### 置けない判定
苗を置く座標が求まったら、そこに苗の持っている当たり判定をCastして、範囲内のオブジェクトを列挙するメソッドを作成する。

- 高さがステラの足元から一定以上低かったら置けない
- Yが問題なければ、苗を置く予定の座標の範囲内のオブジェクトを列挙
- 何かオブジェクトがあれば置けない
- 以上から、置けない状況になっていたら以下を処理
  - 置けないフラグを設定
  - マーカーを非表示
  - 水やアクションキー判定を飛ばす

苗の当たり判定は、BoxCollider, SphereCollider, CapsuleColliderがある。これらを区別せずに処理するために、デリゲートを利用する。`NaeActable`に、基本となるメソッドの型を以下のようにデリゲートで定義する。デリゲートについては以下を参照。

[++C++; // 未確認飛行 C. デリゲート](https://ufcpp.net/study/csharp/sp_delegate.html)


### 移動不能チェック
移動先の座標を求めて、地面に重なっていないかどうかのチェックを行う。

- 移動候補座標で、苗を置く候補座標を求める
- 候補座標が変わっていたら、そこのオブジェクトを列挙
- 床があったら移動をキャンセル

## 苗を置く
苗を置くアクションを作成する。

- Init()で以下を実行
  - 苗を実際にマップに置くメソッドの登録をする
  - 苗を置くアニメの再生
- アニメイベント呼び出しから、以下で苗を置く
  - 苗のPut()メソッドを呼び出す
    - 苗をステラから切り離す
    - StellaMoveから、苗を置く場所を読み取ってそこに配置
    - 苗の当たり判定を戻す
  - ステラの苗持ちフラグをfalseにする
  - アニメの苗持ちフラグをfalseにする
  - 歩きへ移行するメソッドを、アニメイベントへ登録

## 拾うときと置く時に座標を補正する
その場で苗を拾ったり、置いたりすると、座標がズレて不自然である。拾ったり、置いたりする前の動作として、不自然じゃない位置まで移動してから置く処理を入れる。

- 持ち上げと置く処理の共通点が多いので、置く処理は持ち上げを継承するようにする
- TargetWalkとActionの列挙子を定義
- Init()で、目的座標の設定と目的地歩きの設定
- UpdateAction()に、switchで状態分けを実施。目的座標への歩きと、完了したら拾ったり置いたりする処理の開始を設定

以上のようにして、補正歩きを組み込む。
