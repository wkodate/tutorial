JVMはどのように動くのか
-----

http://gihyo.jp/dev/serial/01/jvm-arc

## 第1回　JVMはどのようにメモリ空間を利用するのか

http://gihyo.jp/dev/serial/01/jvm-arc/0001
* JVMはメモリ空間を分けて使用する
    * Javaヒープ
        * Javaのプログラム内で使用されるメモリ空間
        * 建物のイメージ。JVMの種類や選択したGCの種類で変わるJavaヒープ
    * Cヒープ
        * JVMがネイティブライブラリで使用するメモリ空間
        * 庭のイメージ。Javaヒープ以外で使われるメモリ空間
    * スレッドスタック
        * JVMが持つスレッドの情報を格納。スレッドはGCやファイナライザを実行
        * 同じく庭のイメージ
* ヒープの空き容量を超えるサイズのオブジェクトを生成するとOOMになる


## 第2回　ヒープが再利用される仕組みを理解する

http://gihyo.jp/dev/serial/01/jvm-arc/0002

* ガベージコレクタは、不要なオブジェクトを回収してヒープを開放する仕組み
* 参照がないオブジェクトが対象になる
* JVMが必要なタイミングで実行
* 参照が残ってGCの対象とならないオブジェクトが増え続ける状態がメモリリーク
* GCで回収される前に処理を行いたい場合にはファイナライザを使う。時間がかかる処理を書くとGCに追いつけないので注意


## 第3回　システムトラブルの原因はGCの実装を知れば見えてくる

http://gihyo.jp/dev/serial/01/jvm-arc/0003

* システムが無反応になってしまう原因
    * CPUなどのリソース不足
    * コネクションやスレッドプール不足。待ち状態
    * GC設定の誤り
* 3の対策のために、GCの実行頻度や停止時間を考えなければならない
* 回収対象でないオブジェクトの量とヒープサイズを検討する必要がある
* STW(Stop The World)は、GCによってアプリケーションが止まる時間
* 停止に対するアプローチとして、スループットの向上もしくはレスポンスタイムの短縮のどちらを重視するか
    * スループット向上
→どこかのタイミングでまとめて停止させる
    * レスポンスタイムの短縮
→細かく停止させる


## 第4回　3つのGCを使い分けて停止時間を最小にする

http://gihyo.jp/dev/serial/01/jvm-arc/0004

* 3つのGC。シリアルGC、パラレルGC、コンカレントGC
* シリアルGC
    * すべてのアプリケーションスレッドを停止し、1つのスレッドでGCを実行する
    * アプリケーションの停止時間が長くなってしまう
* パラレルGC
    * すべてのアプリケーションスレッドを停止し、複数のスレッドでGCを実行する
    * シリアルGCと比較して停止時間が短くなる
    * GCスレッド間の同期にオーバーヘッドがある
* コンカレントGC
    * アプリケーションスレッドと同時にGCスレッドを動作
    * 停止時間なくGCを行うことができる
    * GCスレッド間の同期にオーバーヘッドがあるが停止時間は短縮する
    * すべてのフェーズがコンカレントで処理できるわけではないため、完全に停止時間をゼロにはできない

## 第5回　チューニングのために理解しておきたいGCの4つのアルゴリズム

http://gihyo.jp/dev/serial/01/jvm-arc/0005

* GCチューニングの流れは、モニタリング→分析→チューニング
* 4つのアルゴリズム。マークスイープGC、コンパクション、コピーGC、世代別GC
* マークスイープGC
    * 2つのフェーズに分けて実行
    * マークフェーズ
        * 生きているオブジェクトをマーク
    * スイープフェーズ
        * マークされていないオブジェクトを削除
    * 空き領域が断片化してしまう
* コンパクション
    * バラバラに配置されたオブジェクトを連続して配置。空き領域の場所を見つけやすくする
* コピーGC
    * GCとコンパクションを同時に行う
    * From領域とTo領域に分ける。新しいオブジェクトはFrom領域に作られ、生きているオブジェクトをコピーGCがTo領域にコピーする。その後、To領域がFrom領域として使われるようになる。これによって断片化を防ぐ
    * From領域しか使えないため、使える容量が半分になる問題がある
* 世代別GC
    * 使われる時間でオブジェクトを分けて管理、GCする
    * 全てのオブジェクトをGC対象とすることを一世代ヒープと呼ぶ
    * オブジェクトがGCされた回数を年齢(age)と呼び、年齢が若い領域をYoung領域、古い領域をOld領域と呼ぶ
    * それぞれの世代で管理するのでGCの実行時間が減る。古い世代はなかなかGCされない

## 第6回　HotSpot JVMのヒープ構造の仕組みを把握する

http://gihyo.jp/dev/serial/01/jvm-arc/0006

* HotSpotはOracleに提供されているJVM。世代別GCを採用。ヒープはNew領域とTenured領域に分かれる
* New領域を、Eden領域、Survivor 0領域、Survivor 1領域の3つに分かれてコピーGCを行う
* HotSpotのヒープ構造は、短期=Eden領域、中期=Survivor領域、長期=Tenured領域に分ける。コピーGCで実現すると、EdenはFrom領域、SurvivorはFrom領域とTo領域に分ける

## 第7回　HotSpot JVMではどのようにオブジェクトが移動するのか

http://gihyo.jp/dev/serial/01/jvm-arc/0007

* Eden領域がいっぱいになるとマイナーGCが行われる
    * 1回目のコピーGCの構成は、Eden領域とSurvivor0領域がFrom領域、Survivor1領域がTo領域
    * 2回目のコピーGCの構成は、Eden領域とSurvivor1領域がFrom領域、Survivor0領域がTo領域
    * 以降はこれを繰り返す
* Eden領域は常にFrom領域でTo領域にはならない
* 停止時間を短くするためには、寿命の短いオブジェクトをEden領域で削除できるようなヒープ設計にする

## 第8回　イレギュラーなヒープの動作を理解する

http://gihyo.jp/dev/serial/01/jvm-arc/0008

* イレギュラーなパターン。オブジェクトが早くTenured領域に移動してしまう
* Tenured領域はメジャーGCの対象になるので停止時間が長くなる
* Tenured領域の空き容量が生成したいオブジェクトのサイズよりも大きい場合、いきなりTenured領域に割り当てられてしまう
* Tenured領域の空き容量が生成したいオブジェクトのサイズよりも小さい場合、Tenured領域のメジャーGCが実行されて空き容量が増え、この空き容量がオブジェクトサイズを超えた場合はTenured領域に割り当てられる
* New領域を増やす、ヒープサイズを拡大するなどの対策が必要

## 第9回　HotSpot JVMのGCを選択しよう

http://gihyo.jp/dev/serial/01/jvm-arc/0009

* HotSpot JVMのGCは、シリアルGC、パラレルGC、CMS(コンカレントマークスイープGC)、G1GC(ガベージファーストGC)の4つのGCがある
* シリアルGC
    * シングルスレットでマークスイープとコンパクションを実行
    * シングルコアならデフォルト
    * GCに時間がかかり、停止時間も長くなってしまうため、積極的に使うことは少ない
* パラレルGC
    * マルチスレットでマークスイープとコンパクションを実行
    * マルチコアではデフォルト
    * シングルスレッドに比べてスループットが高い
* CMS
    * アプリケーションを同時に実行できるフェーズと、アプリケーションを止めて実行するフェーズに分かれている
    * スレッド間のやりとりでCPUを使うのでスループットが下がる
    * アプリケーション全体の停止時間が短い
* G1GC
    * リージョンと呼ばれる領域に分割して使用する。アプリケーションと同時に実行できるフェーズがある
