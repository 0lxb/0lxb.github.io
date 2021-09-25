---
layout:     post
title:      "A beginners' guide away from scanf()"
subtitle:   "远离scanf的入门指南"
author:     "qiuxi"
catalog: true
tags:
    - scanf
    - c
---

远离scanf的入门指南

如果你正准备学习C语言编程，本文中关于scanf函数的输入方法正好可以关注。

## 0. scanf()存在什么问题？

> 规则0：除非你确切的知道你在做什么，否则不要使用scanf()。


## 1. 读取一个数字
* 案例1
```c {.line-numbers}
#include <stdio.h>

int main(void)
{
	int a;
	printf("enter a number: ");
	scanf("%d", &a);
	printf("You entered %d.\n", a);
}
```

```
$ ./example1
enter a number: 42
You entered 42.
```

```
$ ./example1
enter a number: abcdefgh
You entered 1.
```

* 案例2
```c {.line-numbers}
#include <stdio.h>

int main(void)
{
	int a;
	printf("enter a number: ");
	while (scanf("%d", &a) != 1)
	{
		// input was not a number, ask again:
		printf("enter a number: ");
	}
	printf("You entered %d.\n", a);
}
```

```
$ ./example2
enter a number: abc
enter a number: enter a number: enter a number: enter a number: enter a
number: enter a number: enter a number: enter a number: enter a number:
enter a number: enter a number: enter a number: enter a number: enter a
number: enter a number: enter a number: enter a number: enter a number:
enter a number: enter a number: enter a number: enter a number: enter a
number: enter a number: enter a number: enter a number: enter a number:
enter a number: enter a number: enter a number: enter a number: enter a
number: enter a number: enter a number: enter a number: enter a number:
enter a number: enter a number: enter a number: enter a number: enter a
number: enter a number: enter a number: enter a number: enter a number:
enter a number: enter a number: enter a number: enter a number: enter a
number: enter a number: enter a number: ^C
```

> 规则1: scanf()不是用于读取输入，而是用于解析输入。

## 2. 读取一个字符串
* 案例3
```c {.line-numbers}
#include <stdio.h>

int main(void)
{
	char name[12];
	printf("What's your name? ");
	scanf("%s", name);
	printf("Hello %s!\n", name);
}
```

```
$ ./example3
What's your name? Paul
Hello Paul!
```

```
$ ./example3
What's your name? Christopher-Joseph-Montgomery
Hello Christopher-Joseph-Montgomery!
[1]    41147 abort      ./example3
```

> 规则2: 不小心使用 scanf()可能会很危险。始终将字段宽度用于解析为字符串的转换（如 %s）。

* 案例4
```c {.line-numbers}
#include <stdio.h>

int main(void)
{
	char name[40];
	printf("What's your name? ");
	scanf("%39s", name);
	printf("Hello %s!\n", name);
}
```

```
$ ./example4
What's your name? Martin Brown
Hello Martin!
```
> 尽管scanf()格式字符串看起来与printf()格式字符串非常相似，但它们的语义通常略有不同。

* %[a-z]: parse as long as the input characters are in the range a - z.
* %[ny]: parse as long as the input characters are y or n.
* %[^.]: The ^ negates the list, so this means parse as long as there is no . in the input.

* 案例5
```c {.line-numbers}
#include <stdio.h>

int main(void)
{
	char name[40];
	printf("What's your name? ");
	scanf("%39[^\n]", name);
	printf("Hello %s!\n", name);
}
```

```
$ ./example5
What's your name? 
Hello ÿ¦e!
```

* 案例6
```c {.line-numbers}
#include <stdio.h>

int main(void)
{
	char name[40];
	printf("What's your name? ");
	scanf(" %39[^\n]", name);
	//     ^ note the space here, matching any whitespace
	printf("Hello %s!\n", name);
}
```

## 3. fgets

* 案例7
```c {.line-numbers}
#include <stdio.h>

int main(void)
{
	char name[40];
	printf("What's your name? ");
	if (fgets(name, 40, stdin))
	{
		printf("Hello %s!\n", name);
	}
}
```

```
$ ./example7
What's your name? Bob
Hello Bob
!
```

* 案例8
```c {.line-numbers}
#include <stdio.h>
#include <string.h>

int main(void)
{
	char name[40];
	printf("What's your name? ");
	if (fgets(name, 40, stdin))
	{
		name[strcspn(name, "\n")] = 0;
		printf("Hello %s!\n", name);
	}
}
```

```
$ ./example8
What's your name? Bob Belcher
Hello Bob Belcher!
```

* 案例9
```c {.line-numbers}
#include <stdio.h>
#include <stdlib.h>

int main(void)
{
	int a;
	char buf[1024]; // use 1KiB just to be sure

	do {
		printf("enter a number: ");
		if (!fgets(buf, 1024, stdin)) {
			// reading input failed, give up:
			return 1;
		}

		// have some input, convert it to integer:
		a = atoi(buf);
	} while (a == 0); // repeat until we got a valid number

	printf("You entered %d.\n", a);
}
```

```
$ ./example9
enter a number: foo
enter a number: bar
enter a number: 15x
You entered 15.
```

* 案例10
```c {.line-numbers}
#include <stdio.h>
#include <stdlib.h>
#include <errno.h>

int main(void)
{
	long a;
	char buf[1024]; // use 1KiB just to be sure
	int success; // flag for successful conversion

	do {
		printf("enter a number: ");
		if (!fgets(buf, 1024, stdin)) {
			// reading input failed:
			return 1;
		}

		// have some input, convert it to integer:
		char *endptr;

		errno = 0; // reset error number
		a = strtol(buf, &endptr, 10);
		if (errno == ERANGE) {
			printf("Sorry, this number is too small or too large.\n");
			success = 0;
		} else if (endptr == buf) {
			// no character was read
			success = 0;
		} else if (*endptr && *endptr != '\n') {
			// *endptr is neither end of string nor newline,
			// so we didn't convert the *whole* input
			success = 0;
		} else {
			success = 1;
		}
	} while (!success); // repeat until we got a valid number

	printf("You entered %ld.\n", a);
}
```

```
$ ./example10
enter a number: 565672475687456576574
Sorry, this number is too small or too large.
enter a number: ggggg
enter a number: 15x
enter a number: 0
You entered 0.
```

## 5. scanf()

> 规则4: scanf()是一个非常强大的函数。

* 案例11
```c {.line-numbers}
#include <stdio.h>

int main(void)
{
	int a;
	int rc;
	printf("enter a number: ");
	while ((rc = scanf("%d", &a)) == 0) {// Neither success (1) nor EOF
		// clear what is left, the * means only match and discard:
		scanf("%*[^\n]");
		// input was not a number, ask again:
		printf("enter a number: ");
	} if (rc == EOF) {
		printf("Nothing more to read - and no number found\n");
	} else {
		printf("You entered %d.\n", a);
	}
}
```

* 案例12
```c {.line-numbers}
#include <stdio.h>

int main(void)
{
	char name[40];
	printf("What's your name? ");
	if (scanf("%39[^\n]%*c", name) == 1) {// We expect exactly 1 conversion
		printf("Hello %s!\n", name);
	}
}
```

## 参考
http://sekrit.de/webdocs/c/beginners-guide-away-from-scanf.html
https://wizardforcel.gitbooks.io/re-for-beginners/content/Part-I/Chapter-07.html
http://c.biancheng.net/view/160.html
