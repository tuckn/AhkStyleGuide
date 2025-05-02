# AhkStyleGuide

A pragmatic coding-style guide for AutoHotkey v2.

- インデントはスペースで4つ

```ahk
if ObjOwnPropCount(obj) {
    local clone := {}
}
```

- `if`の条件は`()`で囲まないのを基本とする。

```ahk
if obj is Array {
    local clone := []
    for , v in obj {
        clone.Push(Util.CloneObjectDeeply(v))
    }
    return clone
}
```

- しかし複数条件で改行を伴う場合は次のように`()`で囲む。
- この時の複数条件のインデントは4で、閉じの`)`の前で改行する。

```ahk
if (StrLen(pathStr) < 2
    || SubStr(pathStr, 1, 1) != quote
    || SubStr(pathStr, -1) != quote
) {
    pathStr := quote . pathStr . quote
}
```

- `if`は内容が単一であっても`{}`で囲む
- `{`の後に改行する。

```ahk
if InStr(targetPath, "/") ; x
    return targetPath

if InStr(targetPath, "/") { ; o
    return targetPath
}

if err.Count {
    MsgBox(err.Count . " error(s); last error at line #" . err.Last.Line)
} else {
    MsgBox("No errors")
}
```

- `if-else`の書き方

```ahk
if IsDone {
    ...
} else if x < y {
    ...
} else {
    ...
}
```

- `if`、`else`、`return`、`while`、`for`、`break`、`continue`、`try`、`catch`、など他の言語においても一般的な制御後部は小文字で書く
- `Loop`はAutoHotkey独自色が強いがこれも`loop`のように小文字にする

```ahk
loop 3 {
    MsgBox("Iteration number is " . A_Index)
    Sleep(100)
}
```

- `while`、`loop`の書き方は`if`と同じ

```ahk
while counter < 10 {
    ...
    ..
}
```

- むしろ`for`は`()`が使えない点に留意する
- なので`if`も`()`をつけない書き方を標準とした。

```ahk
for (k, v in obj) { ; x
    clone.Push(Util.CloneObjectDeeply(v))
}

for k, v in obj { ; o
    clone.Push(Util.CloneObjectDeeply(v))
}
```

- `try catch`の書き方

```ahk
try {
    ...
} catch Error {
    ...
} finally {
    ...
}

; or

try {
    ...
} catch Error as e {
    ...
} finally {
    ...
}
```

- ローカル変数は`local`で宣言して使用する。
- 変数名はlowerCameCase

```ahk
ParseLParam(lParam) {
    local stringAddress := NumGet(lParam + 2 * A_PtrSize, "UPtr")
    local copyOfData := StrGet(stringAddress)
    ...
```

- グローバル変数は、`global`で宣言して使用する。
- 変数名は`g_`から開始し、lowerCameCase
- 配列の変数名は複数形にする

```ahk
global g_tests := [
    "Test_ExecAndGetStdout",
    "Test_Clone",
    "Test_EncloseQuotes"
]
```

- 定数の変数には、UPPER_CASEを用いる
- AutoHotkeyには`const`宣言はないので、この形にすることで定数ということを強調する
- また定数には、`g_`をつけない

```ahk
global STD_OUTPUT_HANDLE := -11
```

- `InStr`や`MsgBox`などAutoHotkeyで定義済みの関数はUpperCamelCaseを使う
- 関数表記を使い、`MsgBox, AutoHotkey`などの表記は使わない

```ahk
MsgBox, AutoHotkey v2 ; x
MsgBox("AutoHotkey v2") ; o

ExitApp 0 ; x
ExitApp(0) ; o
```

- 関数を定義するコードは専用のファイルを作成し、ファイル名は`Function`で開始する
- 関数名はUpperCamelCase
- JsDocにならったブロックコメントで説明を行う

```Function_AppHandler.ahk
/**
 * @function PreprocessBeforeSendKeyToWin
 * @description pre-process before send key to window.
 * @param {} [targetWin="activeWin"] "activeWin"/"cursorWin"/Object
 * @return {Associative Array} See WindowUtil.GetActiveWindowInfo
 */
PreprocessBeforeSendKeyToWin(targetWin="activeWin") {
    local win
    if (targetWin = "activeWin") {
        win := WindowUtil.GetActiveWindowInfo()
    } else if (targetWin = "cursorWin") {
        win := WindowUtil.GetWindowInfoUnderCursor()
    } else {
        win := targetWin
    }

    return win
}
...
..
```

- class定義するコードは1クラスごとに専用のファイルを作成し、ファイル名は`Class_<ClassName>.ahk`とする
- `class`宣言は小文字
- class名はUpperCamelCase

```Class_Util.ahk
class Util {
    /**
      * @method CloneObjectDeeply
      * @description Deep-clone an associative/array object (recursively).
      * @syntax clonedObj := Util.CloneObjectDeeply(obj)
      * @param {Object} obj
      * @return {Object}
      */
    static CloneObjectDeeply(obj) {
        if !IsObject(obj) {
            ...
```

- ホットーを定義するコードは専用のファイルを作成し、ファイル名は`Hotkey`で開始する
- ホットキーは次のように`{}`で囲む
- `{`の後に改行する

```Hotkey_JisKeyboard.ahk
sc07B & 1:: {
    Send("{Blind}+{1}")
    return
}
```

- AutoHotkey.exeに渡す専用のファイルをホットーキーファイルやクラスファイルとは別に用意し、そこに必要な設定を書く
- ファイル名は`Run`で開始する
  - テストコードの場合、`Test`で開始する
- 先頭に`#Requires AutoHotkey v2.0`を書く
- ホットーのファイルは`#Include`の定義の後方にを配置する

```Run_TucknHotkey.ahk
#Requires AutoHotkey v2.0
#SingleInstance FORCE
#NoTrayIcon

SendMode Event

#Include %A_ScriptDir%\Class_Util.ahk
#Include %A_ScriptDir%\Function_AppHandler.ahk
#Include %A_ScriptDir%\Hotkey_JisKeyboard.ahk ; クラスや関数より後方に配置する
```

- クラス、関数、ホットキー用のファイルは`.\Libs`に配置する
- テスト用のコードは別ファイルにして、`.\Test`に配置する
- テスト対象のファイルごとにテストコードを別ファイルとしてわけ、そのファイル名は、テスト対象のベースファイル名の末尾に`.test`をつけたものとする。

```
AhkUtil\
  Run_TucknHotkey.ahk
  README.md
  Libs\
    Class_Util.ahk
    Function_AppHandler.ahk
    Hotkey_JisKeyboard.ahk
  Test\
    Test_Util.ahk
    Class_Util.test.ahk ; コード先頭に`#Include ..\Libs\Class_Util.ahk`を書く
    Function_AppHandler.test.ahk ; コード先頭に`#Include ..\Libs\Function_AppHandler.ahk`を書く
```
