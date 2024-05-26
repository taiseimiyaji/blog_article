---
title: React Hook Formを使ってみる
tags: [react]
createDate: 2024-05-26
updateDate: 2024-05-26
slug: react-hook-form
draft: false
---

## はじめに

今回はreact-hook-formを使ってみます。業務ではVue3を中心に利用していますが、副業として稼働する際や、個人開発でreactを使うことが多いので、関連したライブラリのキャッチアップ目的で記事にしました。

## react-hook-formとは

Reactアプリケーションでフォームを簡単に扱うためのライブラリです。  
バリデーションやエラー処理を簡単に行うことができます。

## 公式サイトのコンセプト

公式サイトはこちら
[https://react-hook-form.com/](https://react-hook-form.com/)

公式サイトには以下のようなコンセプトが記載されています。  
ここでは一部を抜粋して紹介します。  
詳細は具体例やスクリーンショットがあるので、公式サイトを参照してください。

### Less code. More performant

React Hook Formは、不要な再レンダリングを削除しつつ、記述するコード量を削減します。

### Isolate Re-renders

コンポーネントの再レンダリングを分離して、ページやアプリのパフォーマンスを向上させます。

### Subscriptions

パフォーマンスは、フォームの構築に関するUXの重要な側面です。  
フォーム全体を再レンダリングすることなくここの入力とフォームの状態をサブスクライブすることができます。

## React Hook Formを使わない場合のフォーム実装を考える

Reactでフォームを実装する場合の典型例を考えてみます。  
今回はnameとemailの2つの入力フォームを実装することを考えます。  

```jsx
import React, { useState } from 'react';

const SimpleForm = () => {
  const [formData, setFormData] = useState({
    name: '',
    email: ''
  });

  const [errors, setErrors] = useState({});

  const validate = () => {
    const errors = {};
    if (!formData.name) {
      errors.name = 'Name is required';
    }
    if (!formData.email) {
      errors.email = 'Email is required';
    } else if (!/\S+@\S+\.\S+/.test(formData.email)) {
      errors.email = 'Email is invalid';
    }
    return errors;
  };

  const handleChange = (e) => {
    const { name, value } = e.target;
    setFormData({
      ...formData,
      [name]: value
    });
  };

  const handleSubmit = (e) => {
    e.preventDefault();
    const validationErrors = validate();
    if (Object.keys(validationErrors).length === 0) {
      console.log(formData);
    } else {
      setErrors(validationErrors);
    }
  };

  return (
    <form onSubmit={handleSubmit}>
      <div>
        <label>Name:</label>
        <input
          type="text"
          name="name"
          value={formData.name}
          onChange={handleChange}
        />
        {errors.name && <p>{errors.name}</p>}
      </div>
      <div>
        <label>Email:</label>
        <input
          type="email"
          name="email"
          value={formData.email}
          onChange={handleChange}
        />
        {errors.email && <p>{errors.email}</p>}
      </div>
      <button type="submit">Submit</button>
    </form>
  );
};

export default SimpleForm;

```

このようにuseStateを使うことでフォーム入力フィールドの値が更新されるたびにコンポーネントが際レンダリングされます。  
また、formDataという状態で管理することでバリデーションやエラーハンドリングを簡単に行うことができます。

## React Hook Formを使ったフォーム実装

```jsx
import React from 'react';
import { useForm } from 'react-hook-form';

const SimpleFormWithHook = () => {
  const { register, handleSubmit, formState: { errors } } = useForm();

  const onSubmit = (data) => {
    console.log(data);
  };

  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      <div>
        <label>Name:</label>
        <input
          type="text"
          {...register('name', { required: 'Name is required' })}
        />
        {errors.name && <p>{errors.name.message}</p>}
      </div>
      <div>
        <label>Email:</label>
        <input
          type="email"
          {...register('email', { 
            required: 'Email is required', 
            pattern: {
              value: /\S+@\S+\.\S+/,
              message: 'Email is invalid'
            }
          })}
        />
        {errors.email && <p>{errors.email.message}</p>}
      </div>
      <button type="submit">Submit</button>
    </form>
  );
};

export default SimpleFormWithHook;
```

このようにReact Hook Formを使うことで、useStateを使った実装よりもコードが簡潔になります。

特徴的な処理は以下の通りです。

- useForm()でフォームの状態を管理する  
ここではuseFormフックを利用して、`register`、`handleSubmit`、`formState`を取得しています。また、register関数を使ってフォームの入力フィールドを登録しています。

- フォームのバリデーション
`register`関数の第2引数にバリデーションルールを渡すことで、バリデーションを設定することができます。

単純なフォームだと上記のようなコードで実装できます。特にフォームが複雑になったり、数が多くなってくるとReact Hook Formを使うことでコード量を大幅に削減できるので、ぜひ使ってみてください。

## React Hook Form + Zodを使ったフォーム実装

ここからは、React Hook FormとZodを組み合わせてフォームを実装する方法を紹介します。  

個人的にはバリデーションライブラリとしてZodを使うことが多いので、React Hook Formと組み合わせを模索してみます。

今回のzodの用途としては、通常のバリデーション用途というよりは、フォームのスキーマを定義してわかりやすく分割することと、任意の構造に変換するという2つの目的で使います。

実現には、公式で用意されている`@hookform/resolvers`を使います。

[https://github.com/react-hook-form/resolvers](https://github.com/react-hook-form/resolvers)

zod以外にもさまざまなresolverがサポートされています。  
[https://github.com/react-hook-form/resolvers?tab=readme-ov-file#supported-resolvers](https://github.com/react-hook-form/resolvers?tab=readme-ov-file#supported-resolvers)

```jsx
import React from 'react';
import { useForm } from 'react-hook-form';
import { zodResolver } from '@hookform/resolvers/zod';
import * as z from 'zod';

// Zodスキーマの定義とデータ整形
const schema = z.object({
  date: z.string().nonempty('Date is required'),
  time: z.string().nonempty('Time is required'),
}).transform((data) => {
  const dateTimeString = `${data.date}T${data.time}`;
  return { dateTime: new Date(dateTimeString) };
});

const SimpleFormWithDate = ({ onSubmit }) => {
  const { register, handleSubmit, formState: { errors } } = useForm({
    resolver: zodResolver(schema),
  });

  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      <div>
        <label>Date:</label>
        <input
          type="date"
          {...register('date')}
        />
        {errors.date && <p>{errors.date.message}</p>}
      </div>
      <div>
        <label>Time:</label>
        <input
          type="time"
          {...register('time')}
        />
        {errors.time && <p>{errors.time.message}</p>}
      </div>
      <button type="submit">Submit</button>
    </form>
  );
};

export default SimpleFormWithDate;
```

呼び出し側のコンポーネントからはonSubmit用の関数を渡すようにすると再利用性が高まります。

```jsx
import React from 'react';
import SimpleFormWithDate from './SimpleFormWithDate';

const ParentComponent = () => {
  const handleFormSubmit = (data) => {
    console.log('Submitted DateTime:', data.dateTime);
  };

  return (
    <div>
      <h1>Parent Component</h1>
      <SimpleFormWithDate onSubmit={handleFormSubmit} />
    </div>
  );
};

export default ParentComponent;
```

このようにReact Hook FormではバリデーションルールをView部分に書いていたのに対して、zodを利用することでスキーマとして分離することができます。

また、transformメソッドを使うことで、フォームの入力値を任意の構造に変換することができます。

本来のzodの用途としては、バリデーション用途がメインですが、このようにフォームのスキーマを定義してわかりやすく書くことができるようになります。

## まとめ

- React Hook Formを使うことでフォームの実装が簡単に書くことができます。
- バリデーションルールの部分はzodのようなライブラリを使うことでresolversを使って分離することができます。

## 参考

- [https://zenn.dev/uzimaru0000/articles/react-hook-form-with-zod](https://zenn.dev/uzimaru0000/articles/react-hook-form-with-zod)
- [【2022年】 React Hook FormでValidationライブラリはどれにするか?](https://zenn.dev/longbridge/articles/8a167c53ff026c)
- [ZodとReact Hook Formを使用したシンプルかつ安全なフォームの実装例](https://note.com/cyberz_cto/n/n0d608b9380d3)

## 所感

普段の業務ではVue3を中心に触っていますが、副業や個人開発ではReactを使うことが多いので、フォームの実装のアイデアとしてReact Hook Formを使ってみました。  
比較的単純に書けることと、個人的にはほぼ必須となるバリデーションライブラリとしてZodを使うことで、フォームの実装をスッキリ書く方法を模索できたと思います。