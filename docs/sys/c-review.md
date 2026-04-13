# C 语言复习

## 字符串

### 函数

主要是一堆字符串相关函数

### 输入字符串的两种方式-GPT

> 如果想要控制台向指定字符串变量输入一个字符串，该怎么做？

要在 C 语言中从控制台输入一个字符串并存储到指定的字符串变量中，可以使用 `scanf` 或 `fgets` 函数。以下是两种常见的方法：

1. **使用 `scanf`**

```c
#include <stdio.h>

int main() {
    char str[100];  // 定义一个足够大的字符数组

    printf("请输入一个字符串：");
    scanf("%99s", str);  // 限制输入的最大字符数，防止缓冲区溢出
    printf("您输入的字符串是：%s\n", str);

    return 0;
}
```

- **注意**：`scanf` 只会读取到第一个空格为止，因此适合用于输入单个单词。

2. **使用 `fgets`**

```c
#include <stdio.h>

int main() {
    char str[100];  // 定义一个足够大的字符数组

    printf("请输入一个字符串：");
    fgets(str, sizeof(str), stdin);  // 从标准输入读取一行字符串
    str[strcspn(str, "\n")] = 0;  // 去掉换行符

    printf("您输入的字符串是：%s\n", str);

    return 0;
}
```

- `fgets` 可以读取一整行，包括空格，非常适合输入包含空格的字符串。
- `strcspn` 用于去掉字符串末尾的换行符（如果用户按下 Enter）。

**总结**

- 使用 `scanf` 更适合输入单个单词，不处理空格。
- 使用 `fgets` 更灵活，可以处理带空格的完整行输入。

## 输入输出

### 缓冲区问题

参考链接：https://blog.csdn.net/junbopengpeng/article/details/16810451

通过 `getchar()` 可以解决缓冲区问题

## 指针

### 基本

```c
/*
「&」被称为「取地址运算符」
「*」有两种含义：「指针声明符」、「解引用运算符」

联想 Go：
- 之所以阐述「*」的两种含义，是因为在 Go 中「*」的两种含义易混淆。
- 例如 Go 中声明变量 v *struct{}：
  - 如果将「*」视为「指针声明符」，那么 v 即为地址，引用类型；
  - 如果将「*」视为「解引用运算符」，对 struct{} 进行解引用，那么 v 易被误解为值类型。 
*/

// 实际变量的声明
int var = 20;

// 指针的声明
int *ip; // 此处「*」为「指针声明符」

ip = &var;

/*
其中
	ip 是内存地址
	*ip 是实际变量
	&var 是内存地址
	var 是实际变量
*/

printf("ip  变量存储的地址: %p\n", ip  );
printf("*ip 变量的值:      %d\n", *ip ); // 此处「*」为「解引用运算符」
printf("var 变量的地址:    %p\n", &var);
printf("var 变量的值:      %d\n", var );
```

### 数组

```c
// 打印数组
void printArray(int arr[], int n) {
	for (int i = 0; i < n; i++) {
		printf("%d ", arr[i]);
	}
	printf("\n");
}
```

```c
// 扩容数组容量的函数
int *expandArray(int *array, int *size) {
	*size *= 2;  // 将数组容量翻倍
	int *new_array = realloc(array, *size * sizeof(int)); // 重新分配内存
	if (new_array == NULL) {
		fprintf(stderr, "内存重新分配失败\n");
		free(array); // 如果内存分配失败，释放原有内存
		exit(1);
	}
	return new_array;
}
```

面对这两个函数所产生的疑惑：

> 为什么一个`*array`、一个`array[]`？

二者等价，都意味着该参数传递的是一个指针，且该指针指向数组的第一个元素。

> 为什么使用函数时传递的明明是`array`指针，参数却是`*array`实际数组呢？

不知道原理，只知道 GPT 说数组会退化为指针，指针指向数组第一个元素。暂时会用就行了。

> 第二个函数的 int 和函数名前的 * 是干什么的？

函数名前加 \* 表示该函数的返回值是一个指针；int 表示该指针指向 int 类型。

> 返回的明明是一个数组指针，以 Java 的思路来看指针应该指向 `int[]` ，为什么指向 int 呢？

被 Java 害的不浅，**所谓的数组的指针，实际上是指向数组首元素的地址**，数组首元素是 int ，所以返回值就是 int 。

### 其他

#### 安全性

由于传递的是指向原数组的指针，函数中的任何操作都会对原数组生效。如果只需要读取数组内容，不想修改数组，可以使用`const`来指明指针是只读的。提高安全性，防止函数内对数组的意外修改。

```c
void myFunction(const int *arr, int n) {
    // code
}
```

#### 多维数组

举个例子就好，一通百通。

```c
void myFunction(int arr[][4], int rows) {
    // code
}
```

```c
void myFunction(int (*arr)[4], int rows) {
    // code
}
```

二者等价。

### 字符串

> 涉及数组、字符串、指针

```c
int numbers[10] = {1, 2, 3, 4, 5, 6, 7, 8, 9, 10};
```

```c
char word[10] = "hello";
```

```c
char *words[3] = {"apple","see","her"}
```

存储 int 类型元素的数组直接声明就好。

C 语言中，字符串本质上是以`'\0'`结尾的字符数组。

没有直接的字符串类型，而是通过字符数组和字符指针来表示字符串。

**`char *` 实际上表示数组每一个元素存储的是一个指向字符数组的指针**；例如 word[0] 实际上是一个指向 "apple" 首地址的指针。（与函数的返回值有异曲同工之妙）

### 函数

注意观察这两个函数的声明参数与使用时传递的参数。

```c
void func(int n) {
    n = n + 1;  // 仅仅修改函数内部的n
}

int main() {
    int n = 5;
    func(n);  // 这里传递的是n的值，即5
    printf("n = %d\n", n);  // 输出 n = 5
    return 0;
}
```

```c
void func(int *n) {
    *n = *n + 1;  // 通过解引用操作，修改指针指向的原始变量的值
}

int main() {
    int n = 5;
    func(&n);  // 这里传递的是n的地址，即&n
    printf("n = %d\n", n);  // 输出 n = 6
    return 0;
}
```

第一种方式叫做**按值传递**，函数创建新的本地变量存储它，不会改变原始变量；

第二种方式叫做**按地址传递**，函数直接根据内存地址操作原始变量。

#### 联想 Java

C 语言中，函数参数的传递，本质上只有一种方式 - **按值传递**。

但是，通过传递指针，可以模拟出**按引用传递**的效果。

联想 Java 中对象的使用

* 对于**基本类型参数**来说，是**按值传递**，拷贝并创建本地变量；
* 对于**引用类型参数**来说，是**按引用传递**，传递的是堆内存中对象的引用（地址的拷贝），因此对引用类型对象的修改会影响到元对象。

## 结构体

C中的结构体是其它数据类型（变量）的一个集合，它们储存在一块内存中，然而你可以通过独立的名字来访问每个变量。

1. 定义结构体
   - 进阶：嵌套结构体
2. 初始化结构体
   - 三种方式
3. 当结构体作为函数参数传递
   - 值传递、指针传递
4. 进阶：
   1. 创建结构体对象，并为结构体动态分配内存
   2. 清理结构体内存

### 定义

```c
struct tag { // tag 可选，可将其比作 Java 中的类名，例如可用 struct tag 声明结构体实例
    member-list
    member-list
    member-list  
    ...
} obj1 ; // variable 可选，相当于直接实例化了一个 obj1 的结构体，可同时声明多个
```

typedef 关键字，为类型取一个新的名字

```c
// BYTE 相当与 unsigned char 的缩写
typedef unsigned char BYTE;
```

```c
/*
Simple 相当与 struct {
    int a;
    char b;
    double c;
}的缩写，结合 typedef 关键字，比起 struct tag ，使声明更加清晰
*/
typedef struct {
    int a;
    char b;
    double c;
} Simple;
```

### 初始化&函数参数传递

```c
#include <stdio.h>

// 一、定义一个结构体类型
typedef struct {
        char name[50];
        int age;
        float salary;
} Employee;

// 三、使用结构体的函数

// 1. 值传递：结构体副本，不会修改原结构体。
void print_employee(Employee emp) {
    printf("Name: %s, Age: %d, Salary: %.2f\n", emp.name, emp.age, emp.salary);
}

// 2. 指针传递：高效，允许修改原结构体。
void update_salary(Employee *emp, float new_salary) {
    emp->salary = new_salary;
}

int main() {

    // 二、初始化结构体的三种方式

    // 1. 声明并直接赋值
    Employee emp = {"John Doe", 30, 50000.0};

    // 2. 分步赋值
    Employee emp2;
    emp.age = 30;
    emp.salary = 50000.0;
    snprintf(emp.name, sizeof(emp.name), "John Doe"); // 避免缓冲区溢出
  
    // 3. 结构体数组的初始化
    Employee emps[3] = {
        {"Alice", 28, 60000.0},
        {"Bob", 35, 70000.0},
        {"Charlie", 25, 40000.0}
    };

    // 调用函数
    print_employee(emp); // 值传递
    update_salary(&emp, 60000.0); // 指针传递
    print_employee(emp);

    return 0;
}
```

### 动态分配内存

```c
#include <stdio.h>
#include <assert.h>
#include <stdlib.h>
#include <string.h>

// 1. 结构体的定义
struct Person {
    char *name;   // 姓名，动态分配内存的字符串
    int age;      // 年龄
    int height;   // 身高
    int weight;   // 体重
};

// 创建结构体 Person 的函数
struct Person *Person_create(char *name, int age, int height, int weight)
{
    // 2. 为结构体动态分配内存
    
    // 动态分配内存，用于存储结构体实例
    struct Person *who = malloc(sizeof(struct Person));
    // 确保分配成功，避免空指针错误
    assert(who != NULL);

    // 使用 strdup 动态复制字符串 name（深拷贝而不是浅拷贝）
    who->name = strdup(name); // strdup 动态分配内存并复制字符串，需在后续用 free 释放

    // 3. 结构体的访问
    
    // 初始化其他字段
    who->age = age;
    who->height = height;
    who->weight = weight;

    // 返回创建的结构体指针
    return who;
}

// 4, 清理结构体内存
void Person_destroy(struct Person *who)
{
    // 确保传入的指针有效
    assert(who != NULL);

    // 多级释放：who->name 是单独动态分配的，必须先释放 name 的内存，再释放 who。
    free(who->name); // 释放 strdup 分配的内存
    free(who); // 释放结构体本身的内存
}

// 5. 打印结构体内容
void Person_print(struct Person *who)
{
    printf("Name: %s\n", who->name);     // 打印姓名
    printf("\tAge: %d\n", who->age);    // 打印年龄
    printf("\tHeight: %d\n", who->height); // 打印身高
    printf("\tWeight: %d\n", who->weight); // 打印体重
}

int main(int argc, char *argv[])
{
    // 6. 创建结构体实例
    // 创建多个 Person 对象，分别存储在堆内存中。
    struct Person *joe = Person_create("Joe Alex", 32, 64, 140);
    struct Person *frank = Person_create("Frank Blank", 20, 72, 180);

    // 7. 打印结构体的地址
    printf("Joe is at memory location %p:\n", joe);
    Person_print(joe);

    printf("Frank is at memory location %p:\n", frank);
    Person_print(frank);

    // 8. 修改结构体字段
    joe->age += 20;        // 年龄加 20
    joe->height -= 2;      // 身高减 2
    joe->weight += 40;     // 体重加 40
    Person_print(joe);     // 打印修改后的内容

    frank->age += 20;      // 年龄加 20
    frank->weight += 20;   // 体重加 20
    Person_print(frank);   // 打印修改后的内容

    // 每个实例有独立的内存块，使用 Person_destroy 分别释放。
    Person_destroy(joe);
    Person_destroy(frank);

    return 0;
}
```

注意事项：

1. **动态内存管理：**

   - `malloc` 和 `strdup` 的每一次调用都需要匹配 `free`。
   - 忘记释放内存会导致内存泄漏。

2. **结构体的动态内存分配**
   - 动态分配的结构体只为结构体本身的大小分配内存，结构体中的指针成员需要单独分配。
   - 与动态数组分配内存是两码事（要熟悉[C内存管理](https://www.runoob.com/cprogramming/c-memory-management.html)）。

3. **检查指针合法性：**
   - 在操作指针前使用 `assert` 或显式检查是否为 `NULL`。
   
4. **深拷贝和浅拷贝：**
   - `strdup` 是深拷贝，适合动态分配的字符串。
   - 如果只是简单赋值指针（浅拷贝），容易引发悬空指针问题（两个指针指向同一块内存。如果其中一个指针释放了这块内存，另一个指针就变成了悬空指针）。
5. **内存对齐：**
   - 结构体在内存中会有字节对齐，需注意内存大小。

## 内存管理

C 语言很重要的一部分就是内存管理，指针、动态数组、动态分配结构体都与其相关，主要涉及 `malloc()` `realloc()` `free()` 等函数。

菜鸟教程写的很详细

https://www.runoob.com/cprogramming/c-memory-management.html

## 流

在计算机科学中，“流”指的是一个连续的数据序列，它可以被程序读取或写入。流可以来自或指向不同的数据源或数据目的地，比如文件、内存、网络连接或标准输入输出设备（如键盘和显示屏）。流在数据处理中的概念类似于现实生活中水流的方式，数据像水一样从一个地方流动到另一个地方。

### 类型

1. **输入流（Input Stream）**：
   - 输入流用于从数据源（如文件、网络或键盘）读取数据到程序中。
   - 在 C 语言中，常用 `fscanf`、`fgets` 等函数从输入流读取数据。

2. **输出流（Output Stream）**：
   - 输出流用于将数据从程序写入到目的地（如文件、网络或显示屏）。
   - 在 C 语言中，可以使用 `fprintf`、`fputs` 等函数将数据写入输出流。

### 作用

- **抽象**：流提供了一种抽象，使得程序员可以使用统一的方式处理来自不同源的数据。无论数据来自硬盘、内存还是网络，程序中的数据读写接口可以保持一致。
- **缓冲**：大多数流实现都提供缓冲机制，这有助于提高数据处理的效率。缓冲允许操作系统在真正执行数据传输前积累足够的数据，减少每次传输的开销。
- **方便的数据操作**：流支持多种数据操作，包括基本的字节读写，以及复杂的格式化操作。这使得数据的处理变得灵活且高效。

### 标准流

在 C 语言的标准库中，存在几个预定义的标准流对象，它们分别是：
- `stdin`：标准输入流，通常用于从键盘读取输入。
- `stdout`：标准输出流，用于向显示屏输出数据。
- `stderr`：标准错误流，专用于输出错误信息，即使 `stdout` 被重定向，错误信息仍然可以看到。

这些流在程序启动时自动创建，并且在程序结束时自动关闭，提供了一种方便的方式来处理常见的输入和输出操作。

### 文件&流-相关库函数

与文件或流相关的库函数通常以 f 开头，其中 `f` 代表 "file" 或 "file stream"，可操作任意文件流。

这些函数可以通过指定流（通常是指向 FILE 文件类型的指针）对文件进行读写操作。

也可以通过指定标准输入输出流进行控制台输入输出，`printf` `scanf` 是 C 语言提供的默认操作标准输入与标准输出的函数。

#### 常见

`fopen`：打开文件

`fclose`：关闭文件

`fprintf`：向文件写入格式化的输出

`fscanf`：从文件读取格式化的输入

`fgets`：从文件中读取字符串，直到行结束

`fputs`：向文件写入字符串

`fread`：从文件读取数据到缓冲区

`fwrite`：将数据从缓冲区写入文件

`fseek`：在文件中移动文件指针到指定位置

`ftell`：返回当前文件指针的位置

`rewind`：将文件指针重置回文件的开始

#### 示例

fprintf

```c
// fprintf 函数用于向指定的流写入格式化字符串。
fprintf(filePtr, "Hello, world!\n"); // filePtr 是指向文件的指针，使用 fprintf 向文件写入文本
```

fgets

```c
fgets(str, sizeof(str), stdin);  // 从标准输入读取一行字符串
```

```c
// 使用 fgets 从文件读取文本，直到遇到换行符或达到指定字符数限制。
while(fgets(buffer, 100, filePtr) != NULL) {
    printf("%s", buffer);  // 打印读取的内容
}
```

## 文件读写

菜鸟教程：https://www.runoob.com/cprogramming/c-file-io.html

### 入门示例

#### 写入文件

首先，我们将创建一个文件，并写入一些文本。这里使用`fopen`函数打开文件，`fprintf`函数写入文件，最后使用`fclose`函数关闭文件。

```c
#include <stdio.h> // 标准输入输出
#include <stdlib.h> // 文件操作与内存管理函数

int main() {
    FILE *filePtr;  // 文件指针，FILE 是 C 语言中用于表示文件流的数据类型。

    // 这行代码尝试打开（或创建）一个名为 "example.txt" 的文件，并将其用于写入（"w" 模式）。
    // 如果文件已经存在，它将被覆盖。
    // 如果文件打开成功，filePtr 将指向这个文件。
    filePtr = fopen("example.txt", "w");

    // 如果文件无法打开，打印错误并退出程序
    if (filePtr == NULL) {
        printf("Error opening file!\n");
        return 1;
    }

    // fprintf 函数用于向指定的流写入格式化字符串。
    // 使用 fprintf 向文件写入文本
    fprintf(filePtr, "Hello, world!\n");
    fprintf(filePtr, "This is a test file.\n");
    fprintf(filePtr, "File handling in C is fun!\n");

    fclose(filePtr);  // 关闭文件
    return 0;
}
```

#### 读取文件

接下来，我们将打开刚才创建的文件，并读取其中的内容，使用`fopen`打开文件，`fgets`读取文件中的行，最后使用`fclose`关闭文件。

```c
#include <stdio.h>
#include <stdlib.h>

int main() {
    FILE *filePtr;
    char buffer[100];  // 创建一个字符数组，用于存储读取的文本

    filePtr = fopen("example.txt", "r");  // 打开文件用于读取
    if (filePtr == NULL) {
        printf("Error opening file!\n");
        return 1;  // 如果文件无法打开，打印错误并退出程序
    }

    // 使用 fgets 从文件读取文本，每次读取一行
    while(fgets(buffer, 100, filePtr) != NULL) {
        printf("%s", buffer);  // 打印读取的内容
    }

    fclose(filePtr);  // 关闭文件
    return 0;
}
```

#### 综合说明

- **写文件**：首先确定用 `"w"` 模式打开文件，这会创建新文件或者覆盖旧文件。使用 `fprintf` 可以像标准输出一样向文件中写入格式化的文本。
- **读文件**：使用 `"r"` 模式打开文件进行读取。`fgets` 用来读取文件中的一行，这个函数会在读到换行符或者达到缓冲区限制时停止读取。
- 更复杂的文件操作，如二进制文件读写、文件定位操作等。

## 函数指针

- 声明&使用
  - 无参
  - 有参

- 回调函数：可以简单理解为 **“把函数作为参数传递给另一个函数”**，然后在适当的时候调用这个传递进去的函数。（有点类似与 Java 中的函数式编程，二者核心思想都是将函数作为参数传递）之所以叫回调函数是因为该函数是延迟执行的，只有在接收它的函数执行时，它才会被执行。
- 函数指针可以存储在结构体中，模仿"对象"行为

无参

```c
#include <stdio.h>

void say_hello() {
    printf("Hello, World!\n");
}

int main() {
    void (*func_ptr)(); // 声明一个函数指针，指向返回类型为 void、无参数的函数
    func_ptr = say_hello; // 将函数地址赋值给指针
    func_ptr(); // 调用函数指针
    return 0;
}
```

有参

```c
#include <stdio.h>

int add(int a, int b) {
    return a + b;
}

int multiply(int a, int b) {
    return a * b;
}

int main() {
    int (*operation)(int, int); // 声明函数指针，指向返回值为 int 的函数
    operation = add; // 指向 add 函数
    printf("Addition: %d\n", operation(5, 3)); // 调用函数指针

    operation = multiply; // 指向 multiply 函数
    printf("Multiplication: %d\n", operation(5, 3)); // 调用函数指针

    return 0;
}
```

回调函数

```c
#include <stdio.h>

void greet_user(const char *name, void (*callback)(const char *)) {
    printf("Hello, %s!\n", name);
    callback(name); // 调用回调函数
}

void callback_function(const char *name) {
    printf("%s, this is a callback!\n", name);
}

int main() {
    greet_user("Alice", callback_function);
    return 0;
}
```

函数指针作为函数参数（个人认为类似 Java 的函数式编程）

```c
#include <stdio.h>

void sort(int *arr, int n, int (*compare)(int, int)) {
    for (int i = 0; i < n - 1; i++) {
        for (int j = 0; j < n - i - 1; j++) {
            if (compare(arr[j], arr[j + 1]) > 0) {
                int temp = arr[j];
                arr[j] = arr[j + 1];
                arr[j + 1] = temp;
            }
        }
    }
}

int ascending(int a, int b) {
    return a - b;
}

int descending(int a, int b) {
    return b - a;
}

int main() {
    int arr[] = {5, 2, 9, 1, 6};
    int n = sizeof(arr) / sizeof(arr[0]);

    sort(arr, n, ascending); // 使用升序排序
    printf("Ascending: ");
    for (int i = 0; i < n; i++) printf("%d ", arr[i]);
    printf("\n");

    sort(arr, n, descending); // 使用降序排序
    printf("Descending: ");
    for (int i = 0; i < n; i++) printf("%d ", arr[i]);
    printf("\n");

    return 0;
}
```

函数指针与结构体结合（模拟对象）

```c
#include <stdio.h>

typedef struct {
    void (*say_hello)(const char *name); // 函数指针
} Greeter;

void greet(const char *name) {
    printf("Hello, %s!\n", name);
}

int main() {
    Greeter greeter = {greet}; // 初始化结构体
    greeter.say_hello("Bob"); // 通过函数指针调用
    return 0;
}
```

## 宏

**基本使用**

```c
// 1. 常数宏
#define PI 3.1415926
// 2. 函数宏
#define SQUARE(x) ((x)*(x))
#define  message_for(a, b)  \
    printf(#a " and " #b ": We love you!\n")
// 3. 条件编译
#define WINDOWS
int main()
{
    // 1. 常数宏
    printf("PI = %f\n", PI);
    // 2. 函数宏
    printf("SQUARE(5) = %d\n", SQUARE(5));
    message_for(Java, Golang);
    // 3. 条件编译
    #ifdef WINDOWS
    printf("This is WINDOWS\n");
    #else
    printf("This is not WINDOWS\n");
    #endif
    // 4. 预定义宏
    printf("File :%s\n", __FILE__ ); // 当前文件名
    printf("Date :%s\n", __DATE__ ); // 当前日期
    printf("Time :%s\n", __TIME__ ); // 当前时间
    printf("Line :%d\n", __LINE__ ); // 当前行号
    printf("ANSI :%d\n", __STDC__ ); // 是否是ANSI标准
    return 0;
}
```

**重点**

1. 宏只不过是一个文本替换工具

2. ANSI C 定义了一些预定义宏，可以直接使用

3. 宏可以模拟函数

4. 条件编译（重点）：

    - 跨平台开发，编译器会自动识别当前运行环境，提供一些预定义可使用的宏。

        ```c
        #include <stdio.h>
        
        // 模拟一个跨平台代码
        int main() {
            #ifdef _WIN32
                printf("This is Windows.\n");
            #elif __linux__
                printf("This is Linux.\n");
            #elif __APPLE__
                printf("This is macOS.\n");
            #else
                printf("Unknown platform.\n");
            #endif
        
            return 0;
        }
        ```

    - 开发过程中，调试代码可能需要打印额外的日志信息，但在最终发布的版本中不希望这些日志信息出现。通过条件编译，可以方便地控制这些代码是否包含在程序中。

        ```c
        #include <stdio.h>
        
        #define DEBUG 1  // 切换为 0 表示发布版本
        
        int main() {
            #ifdef DEBUG
                printf("This is a debug message.\n");
            #endif
            printf("This is the main program.\n");
        
            return 0;
        }
        ```

## 联合体

联合体的定义语法与结构体类似，使用 union 关键字代替 struct。

不同在于：联合体的所有成员共享同一块内存空间，即联合体中成员的存储是互斥的，声明并初始化时只能初始化其中的一个成员。

作用/应用场景：

- **处理复杂的数据结构**：联合体可以与结构体结合使用，形成复杂的数据结构。这在处理多种不同类型的数据时非常有用。例如：
    - RV32I 不同类型的指令，定义在一个联合体中，正确存储的同时还方便调用。
    - 网络协议中的数据包解析。
