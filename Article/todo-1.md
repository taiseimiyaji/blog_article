---
title: 再帰的なデータ構造で閉包テーブルを使用したToDoリストを作ってみる ~その1~ 目標設定とDB設計
tags: [個人開発]
createDate: 2022-11-05
updateDate: 2022-11-05
slug: todo-01
---


## はじめに

[前回の記事](https://taisei-miyaji.hatenadiary.com/entry/2022/10/27/232126)で半年間の振り返りをしました。このブログも約半年間継続できていろんなことをインプットできているかなと思います。
そろそろアウトプットとしてゴリゴリ実装したいなという気持ちになっているので、インプットと並行してなにか個人開発に挑戦してみようかと思います。

## ToDoリストを開発する上での目標

以前の記事からサンプルとしてちらほらToDoリストをあげていました。入社して早い段階でフロントエンドの勉強のためにVue.jsで簡易的なToDoリストを作ったんですが、それを改良する形で機能を追加していこうと思います。   
未完成のまま放置することを避けるために今回の開発における目標を優先度を決めて設定しておきたいとおもいます。まずは小さく目標設定してver.1.0として作りきってしまってから色々と機能追加をしていこうと思います。
なお、この設計自体に誤りがあったり方針を見直したり場合は適宜修正加筆をします。

### 目標

**MUST**

- 使用する技術スタックはLaravel,Vue.js,MySQL等を使用
- タスクをカード化して移動できるカンバンの実現(Trelloを参考にします)
- タスクの親子関係の実現(Redmineを参考にします)
- ドメイン駆動開発、クリーンアーキテクチャの採用(設計の思想についてのブログ記事化)


**WILL**

- テスト駆動開発を意識したユニットテストの整備
- 規模が小さいため、ドキュメント整備について色々と試してみて、よければプロダクトへの導入を検討してみる
- フロントエンドのカタログ化(storybook)
- その他実装経験が薄い部分の機能追加(ログイン機能やメール送信機能など)

## 設計について
機能設計やDB設計等の方針についてはせっかくこれまでいろんな技術書を読んできたのでこれまで読んだものと照らし合わせながら根拠を持って決定していきたいと思います。`この本(記事)のこの部分を参考にこう決定した`といったような内容をブログとして公開できればと思います。

## DB設計について
まずはタスクの仕様を整理してそこからDB設計をしたいと思います。
その後書籍「SQLアンチパターン」を改めて確認し、アンチパターンになっていないかを確認していきたいと思います。

まずは実現したいタスクの仕様を整理します。

タスク持つ基本的な情報

- ID
- タスクの名前
- タスクにかかる予定工数
- タスクの完了期限
- タスクの内容

上記に加えて、今回はカンバン形式でのタスク管理を実現したいため、`タスクのステータス`を持つようにし、これをカラムとしてタスクカードを移動させられるようにします。
つまり、`タスクのステータス=カラム名`としたいと思います。

タスクの親子関係の管理のために持つプロパティも考えたいと思います。
まず、直近の親子関係を取得したいです。
ただ、Redmineを参考にすると、先祖すべて、もしくは子孫すべてを参照する事ができます。このことから、先祖と子孫すべてに対してリレーションがあることが伺えます。このような再起的な構造を実現するために今回は書籍「SQLアンチパターン」にて紹介されている**閉包テーブル(Closure Table)**を使用したいと思います。   
以前に[ブログ](https://taisei-miyaji.hatenadiary.com/entry/2022/08/19/230714)でも内容について軽く触れましたが、今回はその復習とともに実際に有効に使用できるかを実装したいと思います。


### テーブル構造

まずは作成するテーブルを考えます。

- tasks   
タスクに関わる属性を格納するテーブルです。

- task_status  
タスクの持つステータスを管理するテーブルです。ステータスとタスクは1対多の関係になります。そのため、このテーブルにはステータスのidとステータス名を格納しておき、taskテーブルのほうにstatus_idを持つようにして関係を表現します。

- tree_paths  
閉包テーブルです。
それぞれのtaskに関して、その祖先と子孫の関係を格納しているテーブルです。


## 閉包テーブルの復習
再帰的な構造をもつデータに対しての解決策です。
再帰的な関係にあるデータ(今回の場合はtask)には子孫関係の情報を持たせず、子孫関係の情報は別のテーブルにもたせます。

tree_pathsテーブル(子孫関係の情報を持つテーブル)  
taskについて子孫関係を持つ場合の閉包テーブルがどのようになるか考えてみます。

|ancestor|descendant|
|---|---|
|1|1|
|1|2|
|1|3|
|1|4|
|1|5|
|1|6|
|1|7|
|2|2|
|2|3|
|3|3|
|4|4|
|4|5|
|4|6|
|4|7|
|5|5|
|6|6|
|6|7|
|7|7|


上記のような形で、それぞれのtaskに関する祖先と子孫の関係を格納します。
このような形式のテーブルをどのように用いるのかパッとイメージができなかったのでそれぞれのケースごとにSQLをどう書くか確認していきます。


- あるレコードの子孫をすべて取得する場合  
tree_pathsテーブルで例えば4の子孫のtaskのレコードを取得する場合のSQL
先祖が4のレコードを検索します。

```SQL
SELECT tasks.*
FROM tasks
    INNER JOIN tree_paths ON tasks.id = tree_paths.descendant
    WHERE tree_paths.ancestor = 4;
```

- あるレコードの先祖をすべて取得する場合  
6の先祖を取得したい場合は逆に子孫が6のレコードを検索します。

```SQL
SELECT tasks.*
FROM tasks
    INNER JOIN tree_paths ON tasks.id = tree_paths.ancestor
    WHERE tree_paths.descendant = 6;
```

- あるレコードに子を追加する場合  
8というレコードを5の子として追加する場合  
まず、新しい行を追加する場合には自己参照の行を追加します。  
次に、task5を子孫として参照する行の集合(task5が自己参照する行を含めて)を取得して子孫を新しいtaskの番号で更新したものを新たに挿入します。

個人的にここが少しわかりにくかったところなんですが、先述した取得時のクエリを見直すと理解しやすかったです。  
具体的にはあるレコード`x`からみた子孫を取得する場合には先祖が`x`であるレコードを検索します。
今回はtask5の子として8を追加するので、追加時には

1. task5を子孫として参照する行をすべて取得
2. その子孫を8に置き換えたレコードを用意して追加(これまでtask5を子孫としているレコードは新たに8も子孫に含めるため)

という手順になります。

```SQL
INSERT INTO tasks(id, name, costs, deadline, detail)
    VALUES(8, 'newTask', 12, '2022-11-04', '新しいタスク');

INSERT INTO tree_paths (ancestor, descendant)
    SELECT tree_paths.ancestor, 8
    FROM tree_paths
    WHERE tree_paths.descendant = 5
    UNION ALL
        SELECT 8, 8;

```

- ある末尾の葉レコードを削除する場合  
例えばtask7を削除する場合、子孫としてtask7を参照しているすべての行を削除します。
これも先程同様に取得時のレコードを考えると削除するべきレコードがどういうものになるか理解できると思います。

```SQL
DELETE FROM tree_paths WHERE descendant = 7;
```

- あるサブツリー全体を削除する場合
例えばtask4とその子孫をすべて削除するには、子孫としてtask4を参照しているすべての行と、task4の子孫を子孫として参照するすべての行を削除します。

```SQL
DELETE FROM tree_paths
WHERE descendant IN (SELECT x.id FROM
                    (SELECT descendant AS id
                    FROM tree_paths
                    WHERE ancestor = 4) AS x);
```

- サブツリーを移動する場合
tree_pathsテーブルはtaskレコードの関連性を格納しているテーブルのため、レコードを削除してもtask自体は削除されません。   
これを利用することで、柔軟に各taskの関連付けを変更できます。

まずはサブツリーのトップのノードとそのノードの子孫の先祖を参照する行を削除して先祖からサブツリーを外します。
例えば、task6をtask4の子からtask3の子に移動する場合を考えてみます。  
移動するtaskの自己参照レコードを削除しないよう注意が必要です。
6のすべての先祖とそれらの子孫を削除します。

```SQL
DELETE FROM tree_paths
WHERE descendant IN (SELECT x.id FROM(SELECT descendant AS id
                    FROM tree_paths
                    WHERE ancestor = 6) AS x)
                AND ancestor IN (SELECT y.id FROM (SELECT ancestor AS id
                FROM tree_paths
                WHERE descendant = 6
                AND ancestor != descendant) AS y)
```

次に、移動先の先祖とサブツリーの子孫の組み合わせを表す行を挿入します。
新しい移動先の先祖とサブツリーのすべてのノードの組み合わせ行を生成するために、デカルト積を生成するCROSS JOIN構文を使用できます。

```SQL
INSERT INTO tree_paths (ancestor, descendant)
    SELECT supertree.ancestor, subtree.descendant
    FROM tree_paths AS supertree
        CROSS JOIN tree_paths AS subtree
        WHERE supertree.descendant = 3
            AND subtree.ancestor = 6;
```

上記SQLで、task3を含むtask3の先祖と、task6を含むtask6の子孫のパスを挿入します。
結果としてtask6から開始されるサブツリーはtask3の子に移動します。

### 閉包テーブルの欠点
閉包テーブルは先祖や子孫の一覧を比較的カンタンなクエリで取得できます。  
一方で、隣接リストや経路列挙といったデータ構造と比べると直近の子や親へのクエリがやや複雑になります。この問題への対処としてtree_pathsテーブルへpath_length属性を追加します。path_lengthにはノードの自己参照を0、直近の子は1、孫は2といったように割り当てることが可能です。
例えば以下のようにtask4の子を簡単に取得できます。

```SQL
SELECT *
FROM tree_paths
WHERE ancestor = 4 AND path_length = 1;
```

## 今回作成するToDoリストのテーブル設計

- taskテーブル

```SQL
CREATE TABLE `task` (
  `id` char(26) COLLATE utf8mb4_unicode_ci NOT NULL,
  `name` varchar(255) COLLATE utf8mb4_unicode_ci NOT NULL,
  `detail` varchar(1000) COLLATE utf8mb4_unicode_ci NOT NULL,
  `deadline` datetime NOT NULL,
  `cost` int NOT NULL,
  `status_id` varchar(255) COLLATE utf8mb4_unicode_ci NOT NULL,
  `created_at` timestamp NULL DEFAULT NULL,
  `updated_at` timestamp NULL DEFAULT NULL,
  `deleted_at` timestamp NULL DEFAULT NULL,
  PRIMARY KEY (`id`),
  CONSTRAINT `task_id_foreign` FOREIGN KEY (`id`) REFERENCES `task_status` (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
```

- task_statusテーブル

```SQL
CREATE TABLE `task_status` (
  `id` char(26) COLLATE utf8mb4_unicode_ci NOT NULL,
  `name` varchar(50) COLLATE utf8mb4_unicode_ci NOT NULL,
  `created_at` timestamp NULL DEFAULT NULL,
  `updated_at` timestamp NULL DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
```

- tree_paths

```SQL
CREATE TABLE `tree_paths` (
  `ancestor` int NOT NULL,
  `descendant` int NOT NULL,
  `path_length` int NOT NULL,
  PRIMARY KEY (`ancestor`,`descendant`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
```

## 所感
今回はこれまで勉強してきた内容でなにか作ってみるための第一歩としてToDoリストを制作することに決め、その際の目標を決めました。  
また、永続化のためにDBの設計について考えました。  
参考としたのは以前記事にもした「SQLアンチパターン」中心にしています。  
再帰的データ構造の実現のために閉包テーブルを作成し、その復習も兼ねて記事にしました。  
次回はこのテーブル構造からtaskエンティティの作成とRepositoryの作成を行いたいと思います。   
インプットしてきたことがアウトプットの役に立っている実感があり、モチベ的にも高くなってる気がします。   
引き続き楽しみながらプログラミングしていこうと思います。