# Dignus.Log

**Attribute-driven, zero-GC logging system focused on performance and extensibility.**  
Provides fast format rendering, pluggable targets (console, file, database), and DI-backed configuration.

---

## Highlights

- **Zero-GC Logging:** Buffered and pooled string operations for allocation-free output  
- **Flexible Formatting:** `${symbol[:format]}` pattern pipeline (e.g., datetime, level, message, callsite)  
- **Attribute-Based Mapping:** `[LogElement("...")]`, `[LogTarget("...")]` enable automatic discovery  
- **Multi-Target Routing:** Log messages distributed to multiple outputs per configuration rules  
- **Dependency Injection Integration:** Targets are resolved via `Dignus.DependencyInjection`  
- **Graceful Shutdown:** Buffered writes automatically flushed and targets disposed on exit  

---

## Built-in Format Elements

- `datetime` — timestamp with customizable format  
- `level` — log severity level  
- `message` — log message content  
- `callsite` — file path and line number (`filePath : lineNumber`)  
- `callerFileName`, `callerFilePath`, `callerLineNumber` — detailed caller info  
- `literal` — static text literal for fixed separators or symbols  

---

## Built-in Targets

| Target | Key | Description |
| :--- | :---: | :--- |
| **Console** | `console` | Writes logs to standard output with per-level colors and custom format |
| **File** | `file` | Rolling log files (`Hour` / `Day` / `Week` / `Month`), archive rotation, auto-flush, keep-open |
| **Database** | `database` | Writes to any ADO.NET-compatible provider using parameterized queries |

---

## Design Principles

- **Zero Allocation:** All formatting and dispatching minimize GC pressure  
- **Precompiled Rendering:** Log elements mapped to compiled delegates instead of reflection  
- **Extensible Pipeline:** Add new log elements or targets with simple attribute declarations  
- **Safe Concurrency:** Buffered writes with lock-free signaling and atomic flush  
- **Simple Integration:** Single configuration file, automatically bound through DI  

---

## Configuration Example

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

## Summary

- Attribute-driven, zero-GC logging engine  
- Configurable multi-target output (console, file, database)  
- Flexible format syntax with expression-free rendering  
- Safe concurrent write with buffered flushing  
- Ideal for performance-critical applications and real-time services