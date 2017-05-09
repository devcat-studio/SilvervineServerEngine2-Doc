# 서버에서 루아 스크립트 사용하기
`LuaVM` 클래스를 사용해서 루아 VM을 만들고 루아 코드를 실행할 수 있습니다.
    
## LuaVM 인스턴스 관리
C#에서 `LuaVM` 클래스의 인스턴스를 생성하면 그것과 짝을 이루는 루아 VM이 엔진 내부의 C++ 영역에 생성됩니다. 그런데 C#에서 `LuaVM` 인스턴스가 가비지 컬렉션될 때 C++ 영역의 루아 VM이 자동으로 제거되지 않습니다. C# `LuaVM` 인스턴스를 더이상 사용하지 않게 되면 반드시 Dispose() 를 호출하십시오. 그러지 않으면 서비스에 지장을 줄 수 있는 수준의 심각한 메모리 릭이 발생합니다.
 
## LuaVM 퍼블릭 메서드
### public T ExecuteString<T>(string source)
인자로 주어진 루아 프로그램을 루아 VM에 전달해서 실행합니다. 실행 결과는 루아 VM 안에서 json으로 인코딩되고, C#에서 JSON.net을 통해 T 형식으로 디코딩됩니다. 루아 프로그램이 두 개 이상의 값을 리턴하면 두번째 리턴값부터는 무시됩니다. 실행중 에러가 발생하거나 실행 결과 디코딩에 실패하면 C# 예외가 발생합니다.
 
### public void ExecuteString(string source)
위 메서드와 같은 일을 하는데, 리턴값을 무시합니다.
 
### public T CallGlobalFunction<T>(string fnName, params object[] arg)
인자로 주어진 이름의 루아 함수를 _G에서 찾아서 실행합니다. 전달한 인자들은 json으로 인코딩/디코딩되어 루아 VM으로 전달됩니다. 루아 함수가 리턴한 결과도 json으로 인코딩/디코딩되어 다시 C#으로 전달됩니다. 루아 함수가 두 개 이상의 값을 리턴한 경우 두번째 리턴값부터는 무시됩니다. 실행중 에러가 발생하거나 실행 결과 디코딩에 실패하면 C# 예외가 발생합니다.
 
### public void CallGlobalFunction(string fnName, params object[] arg)
위 메서드와 같은 일을 하는데, 리턴값을 무시합니다.
 
### public LoadedModuleSnapshot RecordLoadedModuleSnapshot()
require로 로드된 모듈들 각각의 원본 소스코드의 최종 수정 시각을 리턴합니다.
내부적으로는 package.loaded 모듈에 들어있는 이름들을 모두 얻고, package.path를 조회해서 파일을 실제로 찾은 경우에만 최종 수정 시각을 기록합니다. package.path를 일부 모듈 로드 이후에 변경하거나 커스텀 로더를 사용하는 경우 그 모듈의 수정 시각이 누락될 수 있음에 주의하십시오.
 
## C 확장 모듈
- cjson을 쓸 수 있습니다. 전역 json 테이블을 사용하면 됩니다.
- os.timer()를 쓸 수 있습니다. Gideros에 들어있는 것과 같은 동작을 합니다.
- luasocket을 쓸 수 있지만, 프로토타이핑/디버깅 용도로만 사용하십시오! luasocket의 I/O 모델은 실버바인 서버 엔진 2와 다르기 때문에 서버 성능을 크게 저하시키고 서비스 품질을 떨어뜨릴 수 있습니다.
- C 확장 모듈을 추가하고 싶다면 LuaVMMgr.cpp 의 InitializeApplicationSpecificExtensions에서 초기화하십시오.<br>
※ 현재 이 방식은 엔진 C++ 소스코드를 수정해야 하므로 권장하지 않습니다. dll로 C 확장 모듈을 만들 수 있도록 수정할 예정입니다.
- Core.class는 기본 지원되지 않습니다. 필요하다면 루아 코드에서 직접 정의해서 사용하십시오. Source/Sample.Gideros/ServerLua/Class.lua를 복사해 가시면 됩니다.
 
## 루아 VM 수정 내역
- Visual Studio Code 디버거 확장을 붙일 수 있도록 OP_HALT 패치를 적용했습니다.
- Tail call optimization을 제거했습니다.
- 한글 식별자를 사용할 수 있습니다.
- print가 입력 문자열을 UTF-8로 간주하고, 현재 콘솔 인코딩으로 자동 변환하여 출력합니다.