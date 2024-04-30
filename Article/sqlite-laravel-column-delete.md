---
title: SQLiteとLaravel11でのカラム削除の挙動でつまづいた話
tags: [SQLite, Laravel]
createDate: 2024-04-25
updateDate: 2022-04-25
slug: sqlite-laravel-column-delete
---

## はじめに

今回は2024年3月12日にリリースされたLaravel11のうち、移行時につまづいたSQLiteの仕様についてまとめます。

[Laravel Upgrade Guide](https://laravel.com/docs/11.x/upgrade)

## 概要

Laravel11ではSQLiteは3.35.0以上のバージョンが必須となりました。  
この3.35.0というバージョンは2021年3月12日にリリースされたものです。  
この3.35.0でカラムの削除周りの挙動が変更されたのが今回のサポートバージョンの変更の理由のようです。

## カラム削除の挙動について

### 3.35.0以前のSQLiteでの挙動

3.35.0以前は`ALTER TABLE DROP COLUMN`のサポートがなかったので、カラムを削除する場合には以下の手順を踏む必要がありました。

1. 既存のテーブル名を変更
2. 新しいテーブルを元々のテーブル名で作成
3. 古いテーブルから新しいテーブルにレコードをコピー
4. 古いテーブルを削除

### 3.35.0以降のSQLiteでの挙動

`ALTER TABLE DROP COLUMN`のサポートが追加されたので、以下のようなSQLの実行が可能になります。

[https://www.sqlite.org/lang_altertable.html#altertabdropcol](https://www.sqlite.org/lang_altertable.html#altertabdropcol)

```sql
ALTER TABLE table_name DROP COLUMN column_name;
```

ほかのDBMS同様に、下記のような場合に削除できない制約があります。

- 列が主キー、またはその一部である
- 列にユニーク制約が設定されている
- 列にインデックスが設定されている
- 列の名前が部分インデックスのWHERE句に含まれている
- 列がテーブル内の他の列に関連付けられたCHECK制約に利用されている
- 列が外部キー制約の参照先である
- 列が生成列の式で利用される
- 列がトリガーまたはビューで利用される

## 外部キー制約の削除時の挙動

SQLiteでは、外部キー制約の削除はサポートされておらず、SQLite3.35.0時点でも削除できません。

その代わりに、外部キー制約を削除するためには、次の方法を使用する必要があります。

```sql
PRAGMA foreign_keys=OFF;
```

## 発生する問題

上記の通り、カラムの削除はサポートされましたが、外部キー制約の削除はサポートされていません。

Laravelのマイグレーションでは、外部キー制約の削除を直接行うことができないため、エラーが発生するケースがあります。  

特にSQLite3.35.0以前ではテーブルを再作成していたため、外部キー制約は削除されていましたが、3.35.0以降では外部キー制約が残ったままになるため、外部キー制約関連のエラーが発生します。

[Laravel 10 SQLiteGrammar](https://github.com/laravel/framework/blob/10.x/src/Illuminate/Database/Schema/Grammars/SQLiteGrammar.php#L403)


```php
    /**
     * Compile a drop column command.
     *
     * @param  \Illuminate\Database\Schema\Blueprint  $blueprint
     * @param  \Illuminate\Support\Fluent  $command
     * @param  \Illuminate\Database\Connection  $connection
     * @return array
     */
    public function compileDropColumn(Blueprint $blueprint, Fluent $command, Connection $connection)
    {
        if ($connection->usingNativeSchemaOperations()) {
            $table = $this->wrapTable($blueprint);

            $columns = $this->prefixArray('drop column', $this->wrapArray($command->columns));

            return collect($columns)->map(fn ($column) => 'alter table '.$table.' '.$column
            )->all();
        } else {
            $tableDiff = $this->getDoctrineTableDiff(
                $blueprint, $schema = $connection->getDoctrineSchemaManager()
            );

            foreach ($command->columns as $name) {
                $tableDiff->removedColumns[$name] = $connection->getDoctrineColumn(
                    $this->getTablePrefix().$blueprint->getTable(), $name
                );
            }

            return (array) $schema->getDatabasePlatform()->getAlterTableSQL($tableDiff);
        }
    }
```

[Laravel 11 SQLiteGrammar](https://github.com/laravel/framework/blob/11.x/src/Illuminate/Database/Schema/Grammars/SQLiteGrammar.php#L450)

```php
    /**
     * Compile a drop column command.
     *
     * @param  \Illuminate\Database\Schema\Blueprint  $blueprint
     * @param  \Illuminate\Support\Fluent  $command
     * @param  \Illuminate\Database\Connection  $connection
     * @return array
     */
    public function compileDropColumn(Blueprint $blueprint, Fluent $command, Connection $connection)
    {
        $table = $this->wrapTable($blueprint);

        $columns = $this->prefixArray('drop column', $this->wrapArray($command->columns));

        return collect($columns)->map(fn ($column) => 'alter table '.$table.' '.$column)->all();
    }
```

Laravel10までのコードでは、`usingNativeSchemaOperations`でスキーマのチェックを行って、スキーマ操作が利用可能であればそのまま実行、そうでなければDoctrineを利用してスキーマ変更を処理していました。

Laravel11ではシンプルな処理に変わって、SQLコマンドを作成しているだけの処理になっています。

## 回避策

先述したように外部キー制約の設定自体をオフにすることで一応回避はできますが、必要な外部キー制約まで無効になってしまうため、本質的な対応ではないです。

```sql
PRAGMA foreign_keys=OFF;
```

## 記事中に出てきたDB用語

今回の記事はDBに関する話題が多かったので、DB用語について簡単にまとめておきます。

### 主キー

データベースのテーブル内の各行を一意に識別するために使用される列。  
複数の列を設定することもでき、その場合は複合主キーと呼ばれる。  
主キーに設定された列の値は一意でなければならず、NULLを含むことができません。  
つまり後述するユニーク制約、NOT NULL制約が自動的に設定される。

### ユニーク制約

指定された列の値がテーブル内で一意であることを保証する制約。  
これにより、特定の列に重複したデータが存在することを防ぐ事ができる。  
主キーと似ているが、NULL値の扱いに違いがあり、ユニーク制約の列は複数のNULL値を許容することができる  

### インデックス

データベースの検索や並べ替えの操作を高速化するためにテーブルのデータに作成されるデータ構造。  
一般的にはインデックスを適用すると、データへのアクセス速度が向上する。  
メリットだけでなく、書き込み操作時にオーバーヘッドが増えるため、使用する場面を選ぶ必要がある。

### 部分インデックス

テーブルの全データではなく、特定の条件を満たすデータのみにインデックスを作成する方法。  
インデックスのサイズをより小さくする事ができ、検索速度を向上させることができる。

### CHECK制約

列に入力されるデータが特定の条件を満たすことを要求する制約。  
この制約を使用することで、データの整合性と正確性を保証できる。  
例えば、年齢が0以上でなければならないといった条件を設定することができる。  

### 外部キー制約

あるテーブルの列が別のテーブルの特定の列に関連づけられていることを定義するための制約。  
外部キーが参照する列は、その列が属するテーブルにおいて一意の値を持つ必要がある。  
通常、外部キーは他のテーブルの主キーを参照する事が多いが、ユニーク制約が設定されている任意の列も参照することができる。

### 生成列

他の列の値から計算される列。  
これにより、データを冗長に保存することなく、必要に応じて特定のデータを動的に生成することができる。  
MySQLでは5.7.6から、PostgreSQLでは12からサポートされている。  
SQLiteの場合は3.31.0(2020/01/22 リリース)からサポートされている。  

### トリガー

トリガーは、データベースのテーブルに対する特定の操作（INSERT、UPDATE、DELETEなど）が行われたときに自動的に実行されるプロシージャ(DB内に定義された一連のSQL命令)。  
これを利用することで、データの変更を監視したり、特定のビジネスルールを強制したりすることが可能。  

### ビュー

一つまたは複数のテーブルから抽出されたデータの仮想表。  
ビューは実際にデータを格納するのではなく、定義されたSQLクエリに基づいてデータを表示するための窓口のようなイメージ。  
ビューを利用することで、複雑なクエリを単純化し、データアクセスを容易にすることができる。
