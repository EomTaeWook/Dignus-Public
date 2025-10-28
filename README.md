
# Dignus Library

> Licensed under the [MIT License](./LICENSE)  
> © 2021 EomTaeWook
> 

**High-performance modular framework ecosystem focused on runtime efficiency and GC avoidance.**  
All modules are designed for performance-critical environments with zero-copy and zero-GC architecture.

---

## Modules

| Module | Description |
| :--- | :--- |
| [**Dignus.Core**](./publish/Readme/Dignus.Core.md) | Foundation layer with allocation-free collections, deterministic coroutine scheduling, lightweight DI, and framework utilities |
| [**Dignus.Sockets**](./publish/Readme/Dignus.Sockets.md) | Event-driven async socket framework with zero-copy I/O and expression-compiled protocol pipeline |
| [**Dignus.Log**](./publish/Readme/Dignus.Log.md) | Attribute-driven logging system with configurable targets and zero-GC rendering |
| [**Dignus.Unity**](./publish/Readme/Dignus.Unity.md) | Lightweight Unity integration with coroutine, pooling, DI, and reactive scene architecture |

---

## Highlights

- **Zero Allocation:** Designed to eliminate GC overhead during runtime  
- **Zero-Copy Architecture:** Direct buffer access across networking layers  
- **Expression Tree Compilation:** Reflection-free execution for critical paths  
- **Thread-Safe Design:** Synchronized variants for concurrent environments  
- **Extensible and Modular:** Each module can operate independently or as part of a full-stack system  

---


# Share

1.ExcelToJson

- Excel의 Sheet는 Data와 Define으로 나눠진다

- Convert excel to json

	#Define

	- [Define] Data Sheet를 정의한다.

		- Name : Data의 헤더 이름. 식별자입니다.

		- Count : Name 컬럼의 개수

		- Required : 필수 항목 체크

		- Type : 데이터 타입

			- Int32

			- Int64

			- double
		
			- Class

				- Member를 가져야합니다.

			- Member
				
				- 클래스의 하위 필드입니다.

			- String

			- Bool

			- enum

				- Name에는 EnumType의 이름을 적어줍니다. 

				- 개발자에게 Enum 추가를 요청해야합니다.

		- RefTemplate, Ref
		
			- 다른 엑셀의 템플릿을 참조합니다.

			- Excel의 파일명 입니다.

		- RefTemplateField, RefField
		
			- 참조하는 템플릿의 필드명입니다.

		- ConditionField 
		
			- 해당 조건을 확인할 필드명을 적습니다.

		- Condition
		
			- is : {ConditionType}, Ref : {RefTemplate}, RefField : Name

			- is : Damage, Ref: SkillEffectsDamage, RefField : Name
	
2.JsonToCSharp

	- Convert json to CSharp code

	- make {template}.cs file

© 2021–2026 EomTaeWook