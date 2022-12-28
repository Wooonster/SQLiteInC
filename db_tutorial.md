# db_tutorial

## Part 1  Introduction

### `size_t`

- an **unsigned** integral data type, to represent the size of objects in bytes

  (C中任何对象所能达到的最大长度)

- designed to contain the size of the biggest object of the host system, guaranteed to transfer in different machines

- used as the return type of `sizeof` and arguments type of `malloc`, `strlen` etc.

- good to be used as the index type in a loop or of an array

  ```c
  for (size_t n = 0; n < N; ++n)
  {
      a[n] = n;
  }
  ```

- use **`%zu`** to print or use `%u, %lu` in some cases

  `size_t` is unsigned and used to represent **positive integral**, therefore there may have problems when printing minus integral

  ```c
  size_t a = -5;
  printf('%d \n', a);  // error
  printf('%zu \n', a);  // 18446744....
  
  size_t b = 5;
  printf('%d \n', b);  // 5
  printf('%zu \n', b);  // 5
  ```

  *`Warning:`* Since this computer is `x64`, therefore it would be better to use `%ld` to print.

- `size_of` and `int`

  `size_of` differs from 32 and 64 bits rack and is **unsigned**, it has the same bytes when the machine is 32 bits

### `ssize_t`

Mostly same as `size_t`, but `ssize_t` is signed integral type. 

It's the same as `int` when on a 32 bits machine and same as `long` on 64 bits machine

### `getline()`

Read an entire line from stream, storing the address of the buffer containing the text into `*lineptr`

```c
#include <stdio.h>
ssize_t getline(char **restrict lineptr, size_t *restrict n, FILE *restrict stream);

// a typical getline() statement
getline(&buffer, &size, stdin);
```

**Argues**

- `char **restrict lineptr / &buffer`:  the address of the first character position where the input string will be stored.
- `size_t *restrict n / &size`: the address of the variable that holds the size of the input buffer, another pointer.
- `FILE *restrict stream / stdin`: the input file handle. So you could use `getline()` to read a line of text from a file, but when `stdin` is specified, standard input is read.

**Return**

What `getline()` returns is the **size** of the buffer that it gets, the string is stored in `buffer`

###  `strcmp()`

The `strcmp()` compares two strings character by character. If the strings are equal, the function returns 0. Defined in `string.h`

**Prototype:**

```c
int strcmp (const char* str1, const char* str2);
```

**Return:**

| Return Value | Remarks                                    |
| ------------ | ------------------------------------------ |
| 0            | 2 strings are equal                        |
| >0           | non-matching and `str1`(in ASCII) > `str2` |
| <0           | non-matching and `str1` < `str2`           |

### `strncmp()`

This function compares at most the first **n** bytes of `str1` and `str2`.

**Prototype:**

```c
// n − The maximum number of characters to be compared.
int strncmp (const char *str1, const char *str2, size_t n);
```

**Return:**

| Return Value | Remarks                                    |
| ------------ | ------------------------------------------ |
| 0            | 2 strings are equal                        |
| >0           | non-matching and `str1`(in ASCII) > `str2` |
| <0           | non-matching and `str1` < `str2`           |

### `malloc()`

This function allocates the requested memory and returns a pointer to it. Defined in `stdlib.h`

**Prototype:**

```c
void *malloc(size_t size);
```

**Return** 

This function returns a pointer to the allocated memory, or NULL if the request fails.

## Part 2 

### Introduction to `SQLite`

![SQLite Architecture (https://www.sqlite.org/arch.html)](https://cstack.github.io/db_tutorial/assets/images/arch2.gif)

### `typedef`

Used to give a new name to replace a type or a user defined data type.

```c
typedef unsigned char BYTE;
BYTE b1, b2; // same as unsigned char b1, b2;
```

### `enum`

An enumeration type is a data type that consist of integral constants.

```c
// define
enum flag { const1, const2, const3...};

// declare
enum flag arg_name;
// or
enum flag { const1, const2 } arg_name;
```

By default, `const1` to `constN` are 0 to n. They are also changeable.

## Part 3 

### Inserting a row

Define a simple schema to store data.

| **column** | type         |
| ---------- | ------------ |
| id         | Integer      |
| username   | varchar(32)  |
| email      | varchar(255) |

For now, store the data in memory and the insert statement is like `insert 1 wonster wooonster@outlook.com`.

Instead of using B-Tree as the real `SQLite` did, now arrange the data as an array.

- Store rows in blocks of memory called pages.
- Each page store as many rows as it fits.
- Rows are serialized into a compact representation with each page.
- Pages are only allocated as needed.

> 可以理解为，将data存储在一个数组里。
>
> 这是个二维数组，第一层为page，第二层为每个page所能包含的最大的rows。page只在需要所生成。

Serialized row looks like

| column   | size(bytes) | offset |
| -------- | ----------- | ------ |
| id       | 4           | 0      |
| username | 32          | 4      |
| email    | 255         | 36     |
| total    | 291         |        |

### Grammar in C

#### `uint8_t`

This is a nickname of type `unsign char`. In C, there are 6 fundamental data type which are

-  Numeric type
  - Integer: `short`, `int`, `long`
  - Float: `float`, `double`
- Character type: `char`

And all of these are defined like this in C: (in `C99` `stdlib.h` )

```c
   /* 7.18.1.1 */

    /* exact-width signed integer types */
typedef   signed           char int8_t;//给有符号char，取别名为int8_t
typedef   signed short     int int16_t;//给有符号短整型short int，取别名int16_t
typedef   signed           int int32_t;//给有符号整型short int，取别名int32_t
typedef   signed       __INT64 int64_t;

    /* exact-width unsigned integer types */
typedef unsigned          char uint8_t;//给无符号char，取别名为uint8_t
typedef unsigned short     int uint16_t;//给无符号短整型short int，取别名为uint16_t
typedef unsigned           int uint32_t;//给无符号整型short int，取别名为uint32_t
typedef unsigned       __INT64 uint64_t;

    /* 7.18.1.2 */
//和上面一样，取其他别名
    /* smallest type of at least n bits */
    /* minimum-width signed integer types */
typedef   signed          char int_least8_t;
typedef   signed short     int int_least16_t;
typedef   signed           int int_least32_t;
typedef   signed       __INT64 int_least64_t;

    /* minimum-width unsigned integer types */
typedef unsigned          char uint_least8_t;
typedef unsigned short     int uint_least16_t;
typedef unsigned           int uint_least32_t;
typedef unsigned       __INT64 uint_least64_t;

    /* 7.18.1.3 */

    /* fastest minimum-width signed integer types */
typedef   signed           int int_fast8_t;
typedef   signed           int int_fast16_t;
typedef   signed           int int_fast32_t;
typedef   signed       __INT64 int_fast64_t;

    /* fastest minimum-width unsigned integer types */
typedef unsigned           int uint_fast8_t;
typedef unsigned           int uint_fast16_t;
typedef unsigned           int uint_fast32_t;
typedef unsigned       __INT64 uint_fast64_t;

```

*`_t` in these names means these data type are defined by `typedef`*

> ANSI C 提供了3种字符类型，分别是`char`、`signed char`、`unsigned char`，`char`相当于`signed char`或者`unsigned char`，但是**取决于编译器**
>
> 这样取别名为uint8_t / uint16_t / uint32_t /uint64_t 的好处是？
> 1，从名字一眼就可以看出，使用多少位来存数据，统一了“标准”
> 2，为了程序的可扩展性, 假如我们此刻将来我们需要的数据大小变成了`64bit`时,我们只需要将`typedef long long size_t`就可以了, 不然我们可要修改好多好多的地方了，当自己设计一个int类型保存某种数据时,但你又没把握将来是不是要用long int时你可以引用一个自己定义的数据类型
>
> 原文链接：https://blog.csdn.net/weixin_45456099/article/details/120974270

#### `memcpy()` 

```c
void *memcpy(void* dest, const void* src, size_t n);
```

Copy the n character from memory `src` to memory `dest`.

#### `sizeof()`

Use to compute the size of its operand and return the size of a variable or an expression.

**Calculate a `struct` type**

```c
struct stru {
    char a;
    int b;
    float c;
    double d;
}

printf(sizeof(stru));
```

在计算一个结构体的大小时，需要考虑对齐的问题，即*变量存放的起始地址相对于结构的起始地址的偏移量*。

1. 偏移量

   **偏移量指的是结构体变量中成员的地址和结构体变量地址的差**

   **结构体大小等于最后一个成员的偏移量加上最后一个成员的大小**

2. 对齐

   > 存储变量时地址要求对齐，编译器在编译程序时会遵循两条原则：  
   >
   > （1）结构体变量中成员的偏移量必须是成员大小的整数倍（0被认为是任何数的整数倍）   
   >
   > （2）结构体大小必须是所有成员大小的整数倍，也即所有成员大小的公倍数

3. 其他

   结构体大小必须是所有成员大小的整数倍，也即所有成员大小的公倍数

```c
struct stru{
	char a;   //第一个成员a的偏移量为0
	int b;    //第二个成员b的偏移量是第一个成员的偏移量加上第一个成员的大小（0+1=1，但是必须是第二个变量类型长度4的倍数，即为4）
	float c;  //第三个成员c的偏移量是第二个成员的偏移量加上第二个成员的大小（4+4=8，是第三个变量类型长度4的倍数，即为8）
	double d; //第四个成员d的偏移量是第三个成员的偏移量加上第三个成员的大小（8+4=12，但是必须是第四个变量类型长度8的倍数，即为16） 
	int e;	  //第五个成员e的偏移量是第四个成员的偏移量加上第四个成员的大小（16+8=24，是第五个变量类型长度4的倍数，即为24）  
};
//最后计算结构体大小等于最后一个成员（第五个）的偏移量（24）加上最后一个成员的大小（4）。
//即24+4(double)=28
//另外结构体大小必须是所有成员大小的整数倍，也即所有成员大小的公倍数。
//大于等于28并且1,4,4,8,4的公倍数------>32 

```

## Part 4

### 'Weird characters'

```shell
db > insert 1 aaaaa... aaaaa...
Executed.
db > select
(1, aaaaa...aaa\�, aaaaa...aaa\�)
Executed.
db >
```

**problem**

Strings in C always ends with a null character, since the previous code didn't allocate space for that, there comes the weird characters.

**solution**

```c
 const uint32_t COLUMN_EMAIL_SIZE = 255;
 typedef struct {
   uint32_t id;
-  char username[COLUMN_USERNAME_SIZE];
-  char email[COLUMN_EMAIL_SIZE];
+  char username[COLUMN_USERNAME_SIZE + 1];
+  char email[COLUMN_EMAIL_SIZE + 1];
 } Row;
```

