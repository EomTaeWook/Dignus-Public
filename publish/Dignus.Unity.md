# Dignus.Unity

**Lightweight Unity extension built on top of the Dignus framework.**  
Provides dependency injection, coroutine management, pooling, and DI-ready scene architecture for scalable Unity projects.

---

## Overview

`Dignus.Unity` is a lightweight and extensible Unity framework built to streamline large-scale game architecture.  
It offers dependency injection, coroutine scheduling, resource and object pooling, and scene-based architecture with reactive data binding.

---

## Core Features

| Feature | Description |
| :--- | :--- |
| **Dependency Injection** | Lightweight DI via `DignusUnityServiceContainer` |
| **Scene Architecture** | Structured scene controllers with `SceneControllerBase<TScene, TModel>` |
| **Coroutine Manager** | High-performance coroutine scheduler using `DignusUnityCoroutineManager` |
| **Object Pooling** | Reuse `GameObject` and `Component` instances via `DignusUnityObjectPool` |
| **Resource Management** | Centralized prefab and asset handling with `DignusUnityResourceManager` |
| **Reactive Binding** | `BindableProperty<T>` system for UI and gameplay data synchronization |
| **Singleton System** | Simple and persistent lifecycle management for runtime managers |
| **Async Scene Flow** | `DignusUnitySceneManager` for async scene transitions and initialization |

---

## Manager Overview

### DignusUnityCoroutineManager
```csharp
DignusUnityCoroutineManager.Start(MyCoroutine());
```

Efficient coroutine handler that executes enumerators or coroutine handles with optional delay and completion callbacks.
Internally updates via FixedUpdate to ensure stable timing without GC allocations.

### DignusUnityObjectPool
```csharp
var bullet = DignusUnityObjectPool.Instance.Pop(prefab);
DignusUnityObjectPool.Instance.Push(bullet);
```

Reuses and manages pooled GameObject instances to minimize runtime allocations.
Supports both GameObject and Component-based access.

### DignusUnityResourceManager
```csharp
var prefab = DignusUnityResourceManager.Instance.LoadAsset<MyPrefab>();
```

Loads assets from Unity’s Resources folder with internal caching and prefab-path attribute support.

### DignusUnitySceneManager
```csharp
DignusUnitySceneManager.Instance.LoadScene(SceneType.LobbyScene.ToString());
```

Handles asynchronous scene transitions, scene lifecycle events, and current scene registration.

### UnityServiceContainer
```csharp
var container = DignusUnityServiceContainer.RegisterDependencies(typeof(LobbySceneController).Assembly);
container.Build();
```

Dependency injection container specialized for Unity — supports runtime object construction with argument injection.

## Example: Lobby Scene Architecture
### Controller
```csharp
[Injectable(LifeScope.Singleton)]
public class LobbySceneController : SceneControllerBase<LobbyScene, LobbySceneModel>
{
    private readonly GameClientService _gameClientService;
    private readonly UserService _userService;

    public LobbySceneController(GameClientService gameClientService, UserService userService)
    {
        _userService = userService;
        _gameClientService = gameClientService;
    }

    public void OnAwake()
    {
        Model.CurrentPlayer = new GamePlayer()
        {
            AccountId = _userService.GetUserModel().AccountId,
            Nickname = _userService.GetUserModel().Nickname
        };
    }

    public void RoomListRequest(int page, int size)
    {
        _gameClientService.Send(Packet.MakePacket(CGSProtocol.GetRoomList, new GetRoomList()
        {
            Page = page,
            ItemSize = size
        }));
    }

    public override void Dispose()
    {
        Model.LobbyRoomInfos.Clear();
    }
}

```

### Model
```csharp
public class LobbySceneModel : ISceneModel
{
    public Dictionary<int, ArrayQueue<RoomListItemUI>> LobbyRoomInfos { get; set; } = new();
    public GamePlayer CurrentPlayer { get; set; }
    public List<PlayerModel> RoomMembers { get; set; }
    public int JoinRoomNumber { get; set; }
}
```

### Scene
```csharp
public class LobbyScene : SceneBase<LobbySceneController>
{
    private LobbyUI _lobbyUI;

    protected override void OnAwakeScene()
    {
        SceneController.OnAwake();
        _lobbyUI = UIManager.Instance.AddUI<LobbyUI>();
        _lobbyUI.Init(SceneController);
    }

    public override void OnDestroyScene()
    {
        UIManager.Instance.RemoveUI(_lobbyUI);
        SceneController.Dispose();
    }
}
```


### Example Scene Flow
```csharp
// Dependency Setup
var container = DignusUnityServiceContainer.RegisterDependencies(typeof(LobbySceneController).Assembly);
container.Build();

// Load Scene with DI Controller
DignusUnitySceneManager.Instance.LoadScene<LobbyScene>(SceneType.LobbyScene);

```

### Highlights

Zero-GC Scene Flow – Optimized for stable runtime performance

Reactive UI – BindableProperty updates UI automatically

Extensible Architecture – Seamlessly integrates with Dignus.Core and Dignus.DependencyInjection

Unity-Native Lifecycle – Works directly with Awake, Start, and OnDestroy

Production Ready – Designed for scalability and maintainability