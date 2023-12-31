
# Parser Binary Template (PBT) - Syntax Doc
**PBT**, 'Parser Binary Template',  is a file extension created by [Kojo Bailey](https://github.com/KojoBailey) for his [tool](https://github.com/KojoBailey/CC2-Parser-Plus) that parses **binary data** files into JSON and back. As the name implies, they act as **templates** for the parser to read and **act** based on.

PBTs are intended to be similar to [010 Editor](https://www.sweetscape.com/010editor/) **BT** files, and like BTs, are very similar in syntax to [**C**](https://en.wikipedia.org/wiki/C_(programming_language)), although with built-in functions, types, and more features that will prove useful to users. 

This doc outlines all the **syntax** for in PBT files, outlining the similarities to BT and C files, and explaining the stuff that is new or different. There are also many **examples** included to help make your understanding as clear as possible. Since PBT is a custom language, syntax highlighting from **C/C++** will be used, although note that some things may not be highlighted appropiately anyway.

## Structure
First of all, PBTs do not require any access point other than the start of the file itself. This means no `int main()` or anything to start. Trying to do this will simply create a new function that you'll have to manually call.

Next, although the syntax of PBT is based on C, it does not have support for C or C++ libraries, and `#include` will therefore not work with them. Similarly, not all C++ `#` preprocessors will work, although some and more do as documented below.

Likewise, do not expect everything from C's standard library to work. Namespaces aren't used either, so you aren't able to write things like `std::string` (however, see below for the supported data types, as `string` is one of them).

## Data Types
PBT data types reflect those used to interpret **binary data**. Plus, as a handy feature, there are a range of **options** to choose from for each individual data type.

The **full list** of supported data types is as follows:
 - **Signed Byte** (8-bits) → `int8`, `i8`, `byte`
 - **Unsigned Byte** (8-bits) → `uint8`, `u8`, `ubyte`
 - **Signed Short** (16-bits) → `int16`, `i16`, `short`
 - **Unsigned Short** (16-bits) → `uint16`, `u16`, `ushort`, `word`
 - **Signed Int** (32-bits) → `int32`, `i32`, `int`, `long`, `long32`
 - **Unsigned Int** (32-bits) → `uint32`, `u32`, `uint`, `ulong`, `ulong32`, `dword`, `dword32`
 - **Signed Int64** (64-bits) → `int64`, `i64`, `longlong`, `long64`
 - **Unsigned Int64** (64-bits) → `uint64`, `u64`, `ulonglong`, `ulong64`, `qword`, `dword64`, `dwordlong`
 - **Pointer** (see below) → `ptr`, `pointer`
 - **Boolean** (see below) → `bool`, `boolean`
 - **Float** (32-bits) → `float`
 - **Double** (64-bits) → `double`, `float64`
 - **Half Float** (16-bits) → `half`, `halffloat`, `float16`
 - **Char** (8-bits) → `char`
 - **String** (see below) → `string`, `str`, `chars`
 - **Raw Hex Bytes** (see below) → `raw`, `hex`

All of these names can also be used in **uppercase** (e.g. `UINT64`). Plus, you can add up to **2 underscores** (`__`) before each name (e.g. `__int64` - Windows stuff).

Which name you use for each data type is completely up to you, and you're free to mix them up as you please.

```cpp
uint32 Char_ID;
longlong Health;
double xPos;
float yPos;
```

### Pointer

This is just a variation of the **unsigned integer**, and is **64-bit** by default (a `uint64`).

To change the size of a pointer, you can either:
- Specify in the type name, like `ptr32` or `pointer64`.
- Use `#pointer` at the top of the file. This works with any numerical data type.

### Boolean

These are **32-bit** by default, but can be changed by specifying the number of bits (e.g. `bool64` for 64-bits or 8 bytes).

### Char & String
Unlike C++ chars and strings, **both quotation marks** can be used **interchangably** in cases where it **doesn't matter**.

```cpp
char Costume = "C";
string Name = 'Order of Darkness';
```

Where it *does* matter though, they will have **different effects**.

```cpp
'A' + 2 = 'C'
"A" + 2 = "A2"
```

Despite these changes though, `char` and `string` are still treated separately.
- `char` is limited to one character and will only be read and written as such.
- `string` can be any length, and is automatically read from a file until reaching an empty byte (`'\0'` or `0x00`). When writing, an empty byte will also be dropped at the end of the string within the output file.

In cases where strings don't have space between them, you can choose to specify the string's length in the type's name.

```cpp
// Data is "5bct01Bruno BucciaratiSticky Fingers\0";
string6 Char_ID;  // "5bct01"
str16 Char_Name;  // "Bruno Bucciarati"
chars Stand_Name; // "Sticky Fingers"
```

Notice that the last string doesn't need a specified length since the string ends with an empty byte (`\0`) anyway. Also notice how this syntax works for any varation of the `string` data type.

Although the default delimiter for strings is `\0`, you can change this using `#str_delim`.

### Raw Hex Bytes

These are **64-bit** by default, but can be changed by specifying the number of bits (e.g. `hex8` for 8-bits or 1 byte). Only 1, 2, 4, and 8 bytes are allowed.

```cpp
// Data is 0x0EFA9D42
hex32 a; // 0x0EFA
raw16 b; // 0x9D
hex c;   // 0x42 [end of data]
```

Byte data is **stored** in an **unsigned integer** of corresponding size. However, when **written** to JSON, it is converted to a **string**. It will then be converted back when put into binary.

All of this conversion is done in **big endian** (left-to-right) for the littlest confusion.

## Variables
Like with BT files, variables **defined normally** will be **read** from whatever input file in the **order** which they are written, and can then be used as if they were normal variables without any effect on the reading afterwards.

Then, `local` variables have **no impact** on the reading.


### Local Variables
```cpp
uint32 Entry_Count;
for (local int i = 0; i < Entry_Count; i++) {
	// Define entry variables
}
```
```cpp
uint32 Entry_Count;
local int = 0;
while (i < Entry_Count) {
	// Define entry variables
	i++;
}
```
By default, defined `local` variables are **guaranteed** to be **empty** (`0` for numerical variables and `""` for chars and strings).

However, you *cannot* have **raw hex data** for `local` variables. Instead, use unsigned integers. Also, keep in mind that `int` still means `int32` in case you want to pass values between data types.

### Variable Names
When converting to JSON, any underscores (`_`) will be automatically turned into spaces.

```cpp
int Test_Variable;
```
```json
{ "Test Variable": 0 }
```

However, in order to provide the most flexibility with JSON display names while still keeping PBT script names easy-to-use, you have the option to put any string after the variable's name to determine the JSON display name.

```cpp
int TestVar "Test Variable";
local int store = TestVar;
```
```json
{ "Test Variable": 0 }
```

Because of this, you more or less have full control over the displayed variable names, similar to the `<name="text">` tag in BT files, although with different syntax and a bit more control.

```cpp
uint64 Count "Entry Count";
for (local int i; i < Count; i++) {
	string Char_ID "Character ID " + (i + 1);
	char Alt printf("Costume #%c", 'A' + i);
}
```
```jsonc
{
	"Entry Count": 392,
	"Character ID 1": "1jnt01",
	"Costume ID 1": "A",
	"Character ID 2": "1zpl01",
	"Costume ID 2": "B",
	// ...
}
```

### Definitions
Both normal and local variables can be custom-defined, although be aware that assigning a value to a normal variable will override whatever value it read from the input file.

```cpp
uint32 HP "Health"; // 80
HP = 100; // Now 100
```
```json
{ "Health": 100 }
```

## Preprocessors \#
As mentioned towards the top, not all C/C++ `#` preprocessors work in PBT, and the ones that do may have some differences.

Here is the full list of available preprocessors grouped by relevance, with explanations for each further down:
- `#file`, `#extension`
- `#include`, `#include_all`, `#public_all`, `#private_all`
- `#define`
- `#pointer`
- `#str_delim`

### #file
Initially, the parser will search for a PBT of the same name as the input file (minus the file extensions of course). If it finds a PBT, it will then look for the `#file` tag to determine whether the file truly matches or not.

Therefore, `#file` should contain the intended matching file name in a string.

```cpp
// MainModeParam.bt
#file "MainModeParam.bin.xfbin";
```

Note that `#file` must end with a semicolon (`;`). This is because you can state more than one file as well.

```cpp
#file "MainModeParam.bin.xfbin", "MainModeParam.binary";
```

To make this more useful, you can use `__FILE__` which returns a string containing the full name of the input file.

```cpp
#file "MainModeParam.bin.xfbin", "MainModeParam.binary";
#include "xfbin.bt"

if (__FILE__ == "MainModeParam.bin.xfbin") {
	GetXfbinChunk(1);
}
```

You can also use `__NAME__` if you only want the file name without any extension(s).

### #extension
This is similar to `#file`, although only checks the **file extension**. Similarly, you can use `__EXTENSION__` to grab the file extension.

Note that `#file` takes **priority** over this, and `#extension` will only be used when no matching `#file` can be found, or if the **user** chooses to specifically parse by extension instead.

### #include
Used similarly as C, although to include **`.pbt`** files instead of `.h` or `.hpp`.

All **functions** are **public** by default, allowing use in any PBT that includes its file of origin. This can be changed, however, by setting it to `private` instead.

```cpp
// xfbin.bt
uint64 GetChunkPos(int chunk) {
	return /* a position */;
}

private void DoThing() {
	/* do something */
}


// MainModeParam.bt
#include "xfbin.bt"

void DoThing() {
	/* do something else */
}
// Note that this won't be shared with xfbin.bt as it's not included

GetChunkPos(1);
DoThing();
```

On the other hands, all **variables** are **private** by default, but can be done so by using `public`.

```cpp
// xfbin.bt
public uint64 TotalFileSize "Total File Size";
public local uint64 TrueDataSize = TotalFileSize - 42;

// MainModeParam.bt
#include "MainModeParam.bt"
/* TotalFileSize and TrueDataSize can now be used */
```

Changing a variable in one file *will* affect that variable in other files too. This can be prevented by simply creating a **private copy** of that variable.

`#public_all` will set everything in a PBT to public, and `#private_all` will set everything to private. You can also use `#include_all "file.pbt"` to override and take everything as public.

On that note, `public` and `private` do not exist for security or gatekeeping. They are simply descriptors for the preset availability of variables and functions.

### #pointer
Sets the default type for the pointer data type to use.

```cpp
#pointer uint32 // Sets pointers to uint32
#pointer i64    // Sets pointers to int64
#pointer double // Sets pointers to double

#pointer string // This won't work
#pointer char   // This will set to uint8 instead
#pointer ptr    // This will just default uint64
```

### #str_delim
Changes the string delimiter from default `\0` to something else.

```cpp
#string_delim '\n' // Separates strings by new-lines instead
#string_delim "XX" // Separates strings by 2 'X's
```
