---
title: クリーンアーキテクチャのUseCaseにおけるInput,Outputについて
tags: [design]
createDate: 2022-10-16
updateDate: 2022-10-16
slug: inputport
---


## はじめに
クリーンアーキテクチャにやっとの思いでなれ始めているこの頃ですが、日頃業務で実装をしていくにあたって疑問に感じることがあります。   
とはいえ大抵のドメインでは既存の実装に倣ってしまうこともあるんですが、自分の感じた疑問については自分の中で答えを出していきたい性格なのでこの機会にできるだけ言語化しておきたいと思います。   
クリーンアーキテクチャについては以前ブログ記事にもしたので良ければ参考にしてください。   
https://taisei-miyaji.hatenadiary.com/entry/2022/03/21/081313

この記事を書いたときは実際にコードを書いて身につけたというよりは一通り情報として理解していましたが、今は実装を通して理解が深まったかなと思います。


## 目的
Action(Controller)、UseCaseの実装時におけるInput,Outputについて実装の正解を自分なりに考えてみる。


## 導入

まずはいつものボブおじさんの図をもう一度確認しておきます。
![picture 1](/mages/8dbc4e9ea504ae68caeebf7261788baa807accda93f08a9ea19197b22ac05696.png)  


## Action,UseCaseの実装例
Laravelを例にActionとUseCaseの実装例を書いてみます。
あくまで雰囲気なので実際に動作しない可能性があります。

```PHP

/**
 * Action
 */
class Action implements Controller
{
    /**
     * ユースケース
     *
     * @var UseCaseInterface
     */
    Private UseCaseInterface $useCase;

    /**
     * コンストラクタ
     *
     * ユースケースをコンストラクタでインジェクション
     */
    public function __construct(
        UseCaseInterface $useCase
    ){
        $this->useCase = $useCase
    }

    public function __invoke(Request $request)
    {
        try{
            // 1. RequestからInputへ詰め替える
            $input = new Input(
                $request->param1(),
                $request->param2(),
                $request->param3()
            )
            $output = new Output();
            DB::beginTransaction();
            try{
                // 2. ユースケースを実行する
                $this->useCase->process($input, $output);
                DB::commit();
            } catch(UseCaseException $e){
                DB::rollBack();
                throw new InternalServerErrorException();
            }
        } catch(InvalidArgumentException $e){
            throw new BadRequestException();
        } catch(InternalServerErrorException $e){
            throw $e;
        }
        return $output->response();  // outputをレスポンスの形式に合わせる
    }
}

class UseCase
{
    public function process(InputPort $input, OutputPort $output): void
    {
        // ユースケースの処理
        $param1 = $input->param1(); // ユースケースで使用する値はInputPortから取り出す
        $output->output($result); // 処理結果はOutputPortにわたす。
    }
}


```

## 疑問1. ValueObjectの利用タイミング
まず、上記の例ではRequest内でValueObjectをnewしています。
Requestは下記の様な形になります。

```PHP
class Request extends FormRequest
{
    public function authorize(): bool
    {
        return true;
    }

    public function rules(): array
    {
        // バリデーション
    }

    public function param1(): string
    {
        return new ValueObject1($this->get('param1', ''));
    }
    
    public function param2(): string
    {
        return new ValueObject2($this->get('param1', ''));
    }
    
    public function param3(): string
    {
        return new ValueObject3($this->get('param1', ''));
    }
}
```

リクエスト内でインスタンスの生成を行います。
そのため、Controllerでは以下のように詰め替えます。
```PHP
// 1. RequestからInputへ詰め替える
$input = new Input(
    $request->param1(),
    $request->param2(),
    $request->param3()
)
```

ですが、Controllerで直接newすることもできます。
この場合はRequestではプリミティブ型の状態で返します。
```PHP
$input = new Input(
    new ValueObject1($request->param1()),
    new ValueObject2($request->param1()),
    new ValueObject1($request->param1()),
)
```

実装しているときにふとどちらの書き方がいいかなと悩むことがありました。


### エラーハンドリングについて
ValueObject内で投げられるエラーハンドリングについて考えてみます。   
リクエストでValueObjectをインスタンス化する場合、Inputに詰め替える際に`param1()`メソッドが呼ばれ、この中でValueObjectが生成されるため、リクエスト内部から例外がスローされます。   
コントローラーでValueObjectをインスタンス化する場合はValueObjectで投げられた例外を直接コントローラーでキャッチすることができます。   
基本的に例外はすべてのクラスで順番に受け渡していきたいと考えるとコントローラーでValueObjectをインスタンス化するほうが良さそうです。   

### 依存の方向について
クリーンアーキテクチャ的には依存の方向性が重要です。外部のレイヤから内側のレイヤに向かってのみ依存するようにしておくことが重要です。   
リクエストでValueObjectをインスタンス化する場合、リクエストはValueObjectに依存しますが、コントローラーはValueObjectに依存しません。   
コントローラーでValueObjectをインスタンス化する場合、リクエストはValueObjectに依存しませんがコントローラーはValueObjectに依存することになります。   

ここで依存の方向について考えると、ValueObjectは`Application Business Rules`層(図の赤色の層)になり、コントローラーやリクエストより内側にあります。   
つまり、依存の方向という観点からはどちらでも大丈夫そうです。

### 結論
エラーハンドリングの観点から、例外を順番に受け渡すという設計を目指すと、コントローラーでValueObjectをnewするほうが良さそうです。   
とはいえ、PHPは例外という仕組みがあってcatchできるので言語の機能を活かすという考えもあると思うので好みの範疇かなと思っています。

## 疑問2. Outputの利用について
クリーンアーキテクチャにおけるUseCaseは返り値ではなく、`OutputPort`を利用します。(クリーンアーキテクチャの図の右下参照)
UseCaseから直接返り値を返さない理由が気になったので調査してみました。

https://izumisy.work/entry/2019/12/12/000521

同じ内容についてまとめられたブログを見つけました。
このブログでは

>UsecaseがPresenterを呼び出すメリットは"UsecaseがPresenterをどう使うかを制御できること"

と挙げられています。


参考:   
https://nrslib.com/clean-flow-of-control/#outline__3   
https://gist.github.com/mpppk/609d592f25cab9312654b39f1b357c60   


ただ、上記参考記事をみると、一般的なWebアプリケーションを作る場合はPresenterは素直にレンダリングするだけのものになります。
となるとOutputを抽象化するメリットが薄れ、データのやり取りをむやみに冗長化させてしまうことになります。

>今現在一般的な、フロントエンド・アプリケーションをSPAとして作り、サーバはプレゼンター層でJSONを毎度返すだけというようなケースではOutput Portのような抽象化は必ずしも必要ではない場合があるし、むしろ使わないほうがポジティブなこともあると言える。

[参考ブログ](https://izumisy.work/entry/2019/12/12/000521)では上記のように書かれており、概ね同意できる内容かなと感じました。

Outputについては抽象化することは必須ではなさそうです。


## 所感
今回は普段実装するうえで感じた疑問について調べてみました。   
とはいえソフトウェアエンジニアリングは経験則からベストプラクティスを学ぶ分野だと思っています。   
思考停止することなく常に疑問を持って、その疑問を置き去りにしないように取り組み続けていきたいと思います。   