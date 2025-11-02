# Dignus.Log

**Attribute-driven, zero-GC logging system focused on performance and extensibility.**  
Provides fast format rendering, pluggable targets (console, file, database), and DI-backed configuration.

---

## Overview

`Dignus.Log` is a high-performance, allocation-free logging framework that integrates with the Dignus ecosystem.  
It offers a flexible, attribute-driven configuration model with built-in console, file, and database targets.  
All log operations are buffered and designed for zero-GC execution in high-frequency environments.

---

## Highlights

- **Zero-GC Logging:** Buffered and pooled string operations for allocation-free output  
- **Flexible Formatting:** `${symbol[:format]}` pattern pipeline (e.g., datetime, level, message, callsite)  
- **Attribute-Based Mapping:** `[LogElement("...")]`, `[LogTarget("...")]` enable automatic discovery  
- **Multi-Target Routing:** Log messages distributed to multiple outputs per configuration rules  
- **Dependency Injection Integration:** Targets resolved via `Dignus.DependencyInjection`  
- **Graceful Shutdown:** Buffered writes automatically flushed and targets disposed on exit  

---

## Built-in Format Elements

| Element | Description |
| :--- | :--- |
| `datetime` | Timestamp with customizable format |
| `level` | Log severity level |
| `message` | Log message text |
| `callsite` | File path and line number (`filePath : lineNumber`) |
| `callerFileName` / `callerFilePath` / `callerLineNumber` | Detailed caller info |
| `literal` | Static text literal for fixed separators or symbols |

---

## Built-in Targets

| Target | Key | Description |
| :--- | :---: | :--- |
| **Console** | `console` | Writes logs to standard output with per-level colors and custom format |
| **File** | `file` | Rolling log files (`Hour` / `Day` / `Week` / `Month`), archive rotation, auto-flush, keep-open |
| **Database** | `database` | Writes to any ADO.NET-compatible provider using parameterized queries |

---

## Example Usage

### Manual Configuration
```csharp
var logPath = "./logs/UnityLogFile.txt";
var archivePath = "./logs/archive/UnityLogFile.{#}.txt";

var logConfig = new LogConfiguration();

var fileTarget = new FileLogTarget()
{
    ArchiveFileName = archivePath,
    ArchiveRollingType = FileRollingType.Day,
    AutoFlush = true,
    KeepOpenFile = true,
    LogFileName = logPath,
    MaxArchiveFile = 7,
    LogFormatRenderer = new LogFormatRenderer()
};
fileTarget.LogFormatRenderer.SetLogFormat("${datetime} | ${message} | ${callerFileName} : ${callerLineNumber}");

var fileRule = new LoggerRule("unity logger", LogLevel.Fatal, fileTarget);
logConfig.AddLogRule("file rule", fileRule);

var unityTarget = new UnityLogTarget();
var unityRule = new LoggerRule("unity logger", LogLevel.Debug, unityTarget);
logConfig.AddLogRule("unity console rule", unityRule);

LogBuilder.Configuration(logConfig);
LogBuilder.Build();

LogHelper.SetLogger(LogManager.GetLogger("unity logger"));
LogHelper.Debug("log initialized");
```

### XML Configuration
```csharp
LogBuilder.Configuration(LogConfigXmlReader.Load($"DignusLog.{stage}.config"))
           .Build();
LogHelper.Debug("example message");
```

### Typical Output
```csharp
2025-11-02 19:45:22.481 | log initialized | MainSceneController.cs : 83
2025-11-02 19:45:23.509 | user connected  | NetworkManager.cs : 42
2025-11-02 19:45:24.731 | error occurred  | BattleScene.cs : 120
```

### Configuration Example (XML)
```xml
<?xml version="1.0" encoding="utf-8"?>
<configuration>
  <targets>
    <target id="consoleConfig" logTarget="console" format="${datetime} | ${level} | ${message} ${callerFileName} : ${callerLineNumber}">
      <option category="color" id="Debug" value="DarkGray"/>
      <option category="color" id="Info" value="Green"/>
      <option category="color" id="Error" value="DarkRed"/>
      <option category="color" id="Fatal" value="Red"/>
    </target>

    <target id="fileConfig" logTarget="file" format="${datetime} | ${level} | ${message} | ${callerFileName} : ${callerLineNumber}">
      <option category="fileName" value="./logs/LogFile.txt"/>
      <option category="archiveFileName" value="./logs/archive/log.{#}.txt"/>
      <option category="archiveRolling" value="Day"/>
      <option category="maxArchiveFiles" value="7"/>
      <option category="keepOpenFile" value="true"/>
      <option category="autoFlush" value="true"/>
    </target>

    <target id="databaseConfig" logTarget="database">
      <database dbConnection="test"
                dbProvider="MySql.Data.MySqlClient.MySqlConnection, MySql.Data"
                keepConnectionOpen="true"
                dbCommand="insert into log (logged, level, message)
                           values (@logged, @level, @message);">
        <option category="parameter" id="@level" value="${level}"/>
        <option category="parameter" id="@logged" value="${datetime}"/>
        <option category="parameter" id="@message" value="${message}"/>
        <option category="parameter" id="@callsite" value="${callerFileName} : ${callerLineNumber}"/>
      </database>
    </target>
  </targets>

  <loggers>
    <logger id="consoleConfig" minLogLevel="Debug"/>
    <logger id="fileConfig" minLogLevel="Info"/>
    <!-- <logger id="databaseConfig" minLogLevel="Info"/> -->
  </loggers>
</configuration>
```

## Design Principles

- **Zero Allocation:** All formatting and dispatching minimize GC pressure  
- **Optimized Rendering:** Log elements mapped to fast cached delegates for zero reflection overhead
- **Extensible Pipeline:** Add new log elements or targets with simple attribute declarations  
- **Safe Concurrency:** Buffered writes with lock-free signaling and atomic flush  
- **Simple Integration:** Single configuration file, automatically bound through DI  

---

**Dignus.Log** — high-performance, extensible, and allocation-free logging engine.  
Ideal for performance-critical systems, real-time servers, and Unity-based projects.