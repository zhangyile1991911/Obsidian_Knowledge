---
date: 2026-06-25
project:
tags:
  - Unity
  - 技術日本語
---

## 概要

### ゲーム開発におけるUI管理の課題

> [!tip] ゲーム開発では、UIが増えるにつれて次のような問題が発生しやすくなります。

1. UIの表示順を管理しにくい
2. UI要素とクラス内の変数を手動で紐付ける必要があり、UI構成が変わるたびに修正が発生する
3. UIのライフサイクル管理が複雑になる
4. 頻繁に生成・破棄によりメモリ断片化とGCスパイクを避ける
5. バグ発生時に原因を特定しにくい
6. ホーム画面のように、アプリのライフサイクル中は破棄したくないUIが存在する

### 対策

> [!tip] 上記の課題に対して、次の方針で対応します。

1. `Bottom`、`Center`、`Top`、`Tip`など、表示順を制御するレイヤーを事前に定義する
2. オブジェクト名に応じて、UI要素の紐付けコードを自動生成する
3. LRU（Least Recently Used）アルゴリズムを利用して、UIオブジェクトのライフサイクルを管理する
4. 実行時に内部変数や状態を確認しやすいように、可視化ツールを用意する
5. UIオブジェクトを破棄する際、タグや属性に応じて特別な処理を行う

## 仕組みの設計

このUI管理基盤は、主に`UIWidget`、`UIWindow`、`UIManager`の3つのクラスで構成されます。

- `UIWidget`は、最小単位のUI要素です。
- `UIWindow`は、複数の`UIWidget`を持つ画面単位のクラスです。
- 同じ名前の`UIWindow`と`UIWidget`は、それぞれ1つだけ存在します。
- `UIManager`は、LRUアルゴリズムを利用して`UIWindow`を管理します。

> [!info]
> LRUは、Least Recently Used Cacheの略です。簡単に言えば、最近使われていないものから優先的に破棄する仕組みです。

## メリット

1. よく使う画面の生成・破棄を減らし、同じインスタンスを再利用できる
2. メモリ使用量を一定範囲内に抑えやすい
3. 長時間使われていない画面を自動的に破棄できる
4. UI要素と変数を自動で紐付けられる
5. 管理ツールによって、内部変数や状態を確認しやすくなる

## 例で説明する

ここでは、最大4つのWindowを保持できるスタックを例にします。

### 1. ログイン画面を開く

ユーザーがゲームを起動すると、ログイン画面が表示されます。

ロジック側では、スタックに空きがあるため`LoginWindow`を追加します。

```text
|-------------------|
|                   |
|                   |
|                   |
|    LoginWindow    |
|-------------------|
```

### 2. キャラクター画面を開く

ユーザーがキャラクター一覧画面を開き、育成素材を確認します。

ロジック側では、スタックに空きがあるため`CharaWindow`を追加します。

```text
|-------------------|
|                   |
|                   |
|    CharaWindow    |
|    LoginWindow    |
|-------------------|
```

### 3. バッグ画面を開く

ユーザーがバッグ画面を開き、所持アイテム数を確認します。その後、素材を購入するためにショップ画面へ遷移しようとします。

ロジック側では、スタックに空きがあるため`BagWindow`を追加します。

```text
|-------------------|
|                   |
|     BagWindow     |
|    CharaWindow    |
|    LoginWindow    |
|-------------------|
```

### 4. ショップ画面を開く

ユーザーがショップ画面を開きます。しかし、購入したいアイテムを忘れたため、バッグ画面に戻ります。

ロジック側では、スタックに空きがあるため`ShopWindow`を追加します。

```text
|-------------------|
|    ShopWindow     |
|     BagWindow     |
|    CharaWindow    |
|    LoginWindow    |
|-------------------|
```

### 5. バッグ画面を再利用する

ユーザーがバッグ画面を開いた直後に、メールボックスのログイン報酬を思い出します。

ロジック側では、`BagWindow`の使用情報を更新します。ただし、使用回数だけを増やすと問題が起きる可能性があります。例えば、バッグ画面を何度も開くと使用回数が非常に大きくなり、スタックに常駐しやすくなります。

そのため、使用回数だけでなくタイムスタンプも加味します。これにより、たとえ画面が100回以上開かれていても、最近使われていなければ破棄対象にできます。

### 6. メール画面を開く

ユーザーがメール画面を開きます。

この時点でスタックは満杯のため、最も古い`LoginWindow`を削除し、`MailWindow`を追加します。

```text
|-------------------|
|     MailWindow    |
|    ShopWindow     |
|     BagWindow     |
|    CharaWindow    |
|-------------------|
```

> [!info]
> 常駐画面は破棄されず、メモリ上に残り続けます。例えば、ホーム画面などが該当します。

## UIクラスとUI要素の紐付けを自動生成する

### 1. 接頭辞を付ける

画面を組み立てる際、コードから取得したいオブジェクトに接頭辞を付けます。

![例|400](https://zhangyile1991911.github.io/assets/img/ui_framework/s7.png)

### 2. 拡張コマンドを実行する

エディタ上の拡張コマンドを利用します。

![例|409](https://zhangyile1991911.github.io/assets/img/ui_framework/s8.png)

### 3. オブジェクトを枠内にドラッグする

対象オブジェクトを指定された枠内にドラッグします。

![例|418](https://zhangyile1991911.github.io/assets/img/ui_framework/s9.png)

### 4. 自動生成ボタンを押す

> [!tip] 自動生成ボタンを押すと、プレハブに対応するコードが生成されます。
#### プレハブ
![例|425](https://zhangyile1991911.github.io/assets/img/ui_framework/s11.png)

#### 生成されるコード
```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.UI;
using TMPro;
using SuperScrollView;

/// <summary>
/// Auto Generated Class!!!
/// </summary>
[UI("Assets/Resources/Prefab/Windows/HomeWindow.prefab")]
public partial class HomeWindow : UIWindow
{
    public Transform Ins_BG;
    public RectTransform RT_Center;
    public Transform Ins_CommuCenterWidget;
    public Transform Ins_TopResWidget;

    protected override void OnBind(GameObject go)
    {
        uiGo = go;
        Ins_BG = go.transform.Find("Ins_BG").GetComponent<Transform>();
        RT_Center = go.transform.Find("RT_Center").GetComponent<RectTransform>();
        Ins_CommuCenterWidget = go.transform.Find("RT_Center/Ins_CommuCenterWidget").GetComponent<Transform>();
        Ins_TopResWidget = go.transform.Find("Ins_TopResWidget").GetComponent<Transform>();

        base.OnBind(go);
    }
}
```

## デバッグツール

![例|425](https://zhangyile1991911.github.io/assets/img/ui_framework/tool.png)

デバッグツールでは、現在管理されているWindowの状態を確認できます。

1. 順位：LRUに基づく現在のWindow順位
2. 名前：Window名
3. Component数：Windowが持つComponentの数
4. 所属階層：現在のWindowが所属しているレイヤー
5. 表示状態：現在のWindowが表示中かどうか
6. 使用回数：Windowが開かれた回数
7. 常駐：メモリ上に常駐するかどうか

## 基底クラスの拡張

複数のクラスが同じコードや機能を持つ場合は、共通処理を基底クラスに切り出すと管理しやすくなります。

```csharp
class AWidget : UIWidget
{
    protected PlayableDirector rootPD;
    protected Animator rootAnimator;

    public override void OnCreate()
    {
        rootPD = uiTran.GetComponent<PlayableDirector>();
        rootAnimator = uiTran.GetComponent<Animator>();
    }

    public override void OnShow(UIOpenParam openParam)
    {
        // オブジェクトの表示時にアニメーションを再生する
        rootPD.Play();
    }
    
    public override void OnHide(UIOpenParam openParam)
    {
        // オブジェクトの非表示時にアニメーションを再生する
        rootPD.Play();
    }
}

class BWidget : UIWidget
{
    protected PlayableDirector rootPD;
    protected Animator rootAnimator;

    protected override void OnCreate()
    {
        rootPD = uiTran.GetComponent<PlayableDirector>();
        rootAnimator = uiTran.GetComponent<Animator>();
    }

    protected override void OnShow(UIOpenParam openParam)
    {
    }
    
    protected override void OnHide(UIOpenParam openParam)
    {
    }
}
```

### 共通コードを基底クラスに切り出す

```csharp
class UIPDWidget : UIWidget
{
    protected PlayableDirector rootPD;
    protected Animator rootAnimator;
    
    // 子クラスを生成する前に、必要なComponentが揃っているか確認する
#if UNITY_EDITOR
    [UIRequirement]
    public static bool RequirementChecker(GameObject go)
    {
        var pd = go.GetComponent<PlayableDirector>();
        if (!pd)
        {
            UnityEngine.Debug.LogError($"UIPDComponent requires PlayableDirector Component: {go.name}");
            return false;
        }

        var ar = go.GetComponent<Animator>();
        if (!ar)
        {
            UnityEngine.Debug.LogError($"UIPDComponent requires Animator Component: {go.name}");
            return false;
        }

        return true;
    }
#endif
    
    protected override void OnCreate()
    {
        rootPD = uiTran.GetComponent<PlayableDirector>();
        rootAnimator = uiTran.GetComponent<Animator>();
    }

    protected override void OnShow(UIOpenParam openParam)
    {
    }
    
    protected override void OnHide(UIOpenParam openParam)
    {
    }
}
```

### 生成前に基底クラスを選択する

![例|425](https://zhangyile1991911.github.io/assets/img/ui_framework/parentclasschoice.png)

### 要件が満たされない場合

> [!warning]
> 必要なComponentが不足している場合はエラーを表示し、クラスとプレハブを生成しません。

## Windowのライフサイクル管理を改善する

> [!tip]
> 新しいAttributeを追加することで、Windowのライフサイクルを明確に表現できます。

```csharp
// このWindowは自動的に破棄される
[UILifeTime(UILifeTimeType.Transient)]
public partial class TestWindow : UIWindow
{

}

// このWindowはメモリに常駐する
[UILifeTime(UILifeTimeType.Permanent)]
public partial class TestWindow : UIWindow
{

}
```

## Enumを廃止する

ClassとEnumの1対1の関係を維持するのは手間がかかります。また、クラスを削除した際に対応するEnumの削除を忘れると、バグが発生する可能性があります。

そのため、可能であればEnumではなく、Attributeや型情報を利用して管理した方が安全です。
