---
date: 2026-06-05
project:
tags:
  - Unity
---

## 前提

今回は、ObjectPool内のオブジェクトを利用する際の初期化・リセット・破棄などの詳細には触れず、ObjectPool自体の仕組みに焦点を当てます。

## 問題点

1. 借りたオブジェクトが返却されない
2. 借りたオブジェクトが誤って破棄される
3. プールが大量のオブジェクトを保持し続ける

## 最もシンプルな実装から始める

- 貸し出し
- 返却

```csharp
public class ObjectPool
{
    private Queue<GameObject> _pool;
    ObjectPool()
    {
        _pool = new Queue<GameObject>(_capacity);
        MakeMore(50);       
    }

    private void MakeMore(int capacity)
    {
        for (int i = 0; i < capacity; i++)
        {
            var one = new GameObject();
            one.name = $"xxx_ObjectPool_{i}";
            _pool.Enqueue(one);
        }
    }

    public GameObject GetObject()
    {
        if (_pool.Count <= 0)
        {
            MakeMore(10);
        }

        return _pool.Dequeue();
    }

    public void ReleaseObject(GameObject gameObject)
    {
        if (gameObject)
        {
            _pool.Enqueue(gameObject);      
        }
    }
}
```

## 借りたオブジェクトが返却されなかった場合

> [!note]
> 上記の例では、最もシンプルなプールを実装しました。しかし、実際の開発では、借りたオブジェクトの返却を忘れることがあります。貸し出し中のオブジェクトを記録しておけば、返却されていないオブジェクトを特定できます。

```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class ObjectPool
{
    private Queue<GameObject> _pool;
    // 貸し出し中のオブジェクトを記録する
    private Dictionary<int, GameObject> _history;
    ObjectPool()
    {
        _pool = new Queue<GameObject>();
        _history = new Dictionary<int, GameObject>();
        MakeMore(50);
    }

    private void MakeMore(int capacity)
    {
        for (int i = 0; i < capacity; i++)
        {
            var one = new GameObject();
            one.name = $"xxx_ObjectPool_{i}";
            _pool.Enqueue(one);
        }
    }

    public GameObject GetObject()
    {
        if (_pool.Count <= 0)
        {
           MakeMore(10);
        }

        GameObject gameObject = _pool.Dequeue();
        _history[gameObject.GetInstanceID()] = gameObject;
        return gameObject;
    }

    public void ReleaseObject(GameObject gameObject)
    {
        if (gameObject)
        {
            _history.Remove(gameObject.GetInstanceID());
            _pool.Enqueue(gameObject);      
        }
    }

    // 返却されていないオブジェクトを報告する
    public void LogLeakObject()
    {
        foreach (var pair in _history)
        {
            Debug.Log($"Object {pair.Key} was not returned.");
        }
    }
}
```

## 貸し出し中のオブジェクトが破棄された場合

> [!note]
> 上記のコードでは、貸し出し中のオブジェクトを記録できます。では、そのオブジェクトが返却される前に誤って破棄された場合は、どのように検知すればよいでしょうか。

```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

// GameObjectの破棄を監視する
internal class ObjectWatchDog : MonoBehaviour
{
    public ObjectPool ParentPool;

    int instanceID;
    void Start()
    {
        instanceID = GetInstanceID();
    }

    void OnDestroy()
    {
        // 所属するプールに通知する
        ParentPool?.ReportIllegalDestroy(instanceID);
    }
}

public class ObjectPool
{
    private Queue<GameObject> _pool;
    // 貸し出し中のオブジェクトを記録する
    private Dictionary<int, GameObject> _history;
    ObjectPool()
    {
        _pool = new Queue<GameObject>();
        _history = new Dictionary<int, GameObject>();
        
        MakeMore(50);
    }

    private void MakeMore(int capacity)
    {
        for (int i = 0; i < capacity; i++)
        {
            var one = new GameObject();
            one.name = $"xxx_ObjectPool_{i}";
            // 初期化時に監視用のComponentを追加する
            ObjectWatchDog dog = one.AddComponent<ObjectWatchDog>();
            dog.ParentPool = this;
            _pool.Enqueue(one);
        }
    }

    public GameObject GetObject()
    {
        if (_pool.Count <= 0)
        {
            MakeMore(10);
        }

        GameObject gameObject = _pool.Dequeue();
        _history[gameObject.GetInstanceID()] = gameObject;
        return gameObject;
    }

    public void ReleaseObject(GameObject gameObject)
    {
        if (gameObject)
        {
            _history.Remove(gameObject.GetInstanceID());
            _pool.Enqueue(gameObject);      
        }
    }
    
    // 返却されていないオブジェクトを報告する
    public void LogLeakObject()
    {
        foreach (var pair in _history)
        {
            Debug.Log($"Object {pair.Key} was not returned.");
        }
    }
    
    // オブジェクトが予期せず破棄された場合に呼び出す
    public void ReportIllegalDestroy(string name,int instanceId)
    {
        _history.Remove(name);
        Debug.LogWarning($"Object {name} was destroyed without being returned.");
    }
}
```

## 借りたオブジェクトを自動的に返却する

> [!note]
> ここまでは、オブジェクトの返却と破棄について考えてきました。しかし、利用者が常に返却を意識しなければならない設計は、扱いにくく、返却漏れも起こりやすくなります。そこで、オブジェクトを自動的に返却する仕組みを検討します。

```csharp
// IDisposableを利用し、借りたオブジェクトを自動的にプールへ返却する。
//
// 検討事項：ここでは、なぜstructではなくclassを利用するのか。
//
// ObjectHandler h1 = pool.Get();
// ObjectHandler h2 = h1;
//
// classの場合、h1とh2は同じインスタンスを参照する。
// 一方、structの場合は値がコピーされるため、h1とh2は別の値になる。
// その結果、h1.Release()を呼び出しても、h2の状態には反映されない。

public class ObjectHandler : IDisposable
{
    public ObjectPool ParentPool { get; private set; }
    public GameObject Instance { get; private set; }

    public int InstanceId { get; private set;}
    public string name { get; private set;}
    private bool _hasReleased = false;
    public ObjectHandler(ObjectPool pool,GameObject instance)
    {
        ParentPool = pool;
        Instance = instance;
        InstanceId = instance.GetInstanceID();
        name = instance.name;
    }
    
    // 手動で返却する
    public void Release()
    {
        ParentPool.ReleaseObject(this);
        _hasReleased = true;
    }
    
    public void Dispose()
    {
        // 明示的に返却されていない場合は、自動的に返却する
        if (!_hasReleased)
        {
            Release();    
        }
    }
}

// GameObjectの破棄を監視する
internal class ObjectWatchDog : MonoBehaviour
{
    public ObjectHandler handler;
    int instanceId;
    private void Start()
    {
        instanceId = GetInstanceID();
    }
    private void OnDestroy()
    {
        ParentPool?.ReportIllegalDestroy(handler);
    }
}

public class ObjectPool
{
    private Queue<GameObject> _pool;
    // 貸し出し中のオブジェクトを記録する
    private Dictionary<int, GameObject> _history;
    ObjectPool()
    {
        _pool = new Queue<GameObject>();
        _history = new Dictionary<string, GameObject>();
        MakeMore(50);
    }

    private void MakeMore(int capacity)
    {
        for (int i = 0; i < capacity; i++)
        {
            var one = new GameObject();
            one.name = $"xxx_ObjectPool_{i}";
            // 初期化時に監視用のComponentを追加する
            ObjectWatchDog dog = one.AddComponent<ObjectWatchDog>();
            dog.ParentPool = this;
            _pool.Enqueue(one);
        }
    }

    public ObjectHandler GetObject()
    {
        if (_pool.Count <= 0)
        {
            return null;
        }

        GameObject gameObject = _pool.Dequeue();
        _history[gameObject.GetInstanceID()] = gameObject;
        ObjectHandler handler = new ObjectHandler(this, gameObject);
        return handler;
    }
    
    public void ReleaseObject(ObjectHandler handler)
    {
        if (handler)
        {
            _history.Remove(handler.Instance.GetInstanceID());
            _pool.Enqueue(handler.Instance);      
        }
    }

    public void LogLeakObject()
    {
        foreach (var pair in _history)
        {
            Debug.Log($"Object {pair.Key} was not returned.");
        }
    }
    
    public void ReportIllegalDestroy(ObjectHandler handler)
    {
        _history.Remove(handler.InstanceID);
        Debug.LogWarning($"Object {handler.name} was destroyed without being returned.");
    }
}
```

## 破棄されたオブジェクトは自動返却できない

`OnDestroy`が呼び出された時点では、対象のGameObjectはすでに破棄処理中です。そのため、オブジェクトをプールへ戻して再利用することはできません。

```csharp
internal class ObjectWatchDog : MonoBehaviour
{
    public ObjectPool ParentPool;
    private void OnDestroy()
    {
        // この時点では破棄処理が始まっているため、回収して再利用することはできない
        ParentPool?.ReportIllegalDestroy(this.gameObject);
    }
}
```

## プールを自動的に拡張・縮小する

```csharp
public class GameObjectPool
{
    public ObjectHandler Get()
    {
        // 貸し出し時に、現在の貸出数と一定期間内の最大貸出数を記録する
        _currentBorrowed++;
        _maxBorrowed = Mathf.Max(_currentBorrowed, _maxBorrowed);
    }
    public void Release(ObjectHandler handler)
    {
        _currentBorrowed--;
    }

    public void LateUpdate()
    {
        // 設定した時間が経過したら、必要なプールサイズを計算する
        _smoothedPeak = alpha * _maxBorrowed + (1.0f - alpha) * _smoothedPeak;
        int targetPoolNum = Mathf.CeilToInt(_smoothedPeak * 1.2f);
        ShrinkOrExpandTo(targetPoolNum);
    }

    private void ShrinkOrExpandTo(int targetPoolNum)
    {
        int current = _pool.Count;
        
        if (current == targetPoolNum)
            return;
        
        float changePercent = Mathf.Abs(targetPoolNum - current) / (float)current;
        // 頻繁なサイズ変更を避ける
        if (changePercent < 0.25f)
            return;

        if (targetPoolNum < current)
        {
            // 縮小
            int toRemove = current - targetPoolNum;
            for (int i = 0; i < toRemove; i++)
            {
                var obj = _pool.Dequeue();
                Destroy(obj);
            }
            Debug.Log($"{name}: removed {toRemove} objects from the pool.");
        }
        else
        {
            // 拡張
            int toAdd = targetPoolNum - current;
            MakeMore(toAdd);
            Debug.Log($"{name}: added {toAdd} objects to the pool.");
        }
    }
}
```

## 貸し出し中のオブジェクトを可視化・集計する

貸し出し中のオブジェクト一覧や貸出数の推移を可視化できれば、返却漏れの調査や適切なプールサイズの決定に役立ちます。

## 衝突判定中のオブジェクトが回収された場合

プールで管理されているオブジェクトが衝突判定領域内にある状態で回収された場合、衝突判定側ではどのように扱うべきでしょうか。回収時にColliderを無効化する、接触状態を明示的に解除するなど、利用側とのライフサイクル管理を検討する必要があります。

## まとめ

ここまで、シンプルな実装から始め、実際の開発で起こりやすい問題とその対策を段階的に検討してきました。もちろん、現時点の設計は完全ではなく、検討すべき点がまだ残っています。

今後の課題は次のとおりです。

- [x] プールを自動的に拡張・縮小する
- [x] 貸し出し中のオブジェクトを可視化・集計する
- [ ] テスト結果に基づき、初期化時の適切なオブジェクト数を設定できるようにする
