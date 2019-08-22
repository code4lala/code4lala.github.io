---
layout: post
comments: true
title:  "C++类外调用类内的私有函数"
date:   2019-08-22 09:47:54 +0800
tags: C++ 原创 指针
lang: zh
---

<!--引用数学表达式js脚本-->
{% include mathjax_support.html %}

`原创文章`

# C++ 类外调用类内的私有函数

> (完全没什么用瞎折腾系列)

## 代码
{% highlight cpp %}
#include <iostream>
#include <stdint.h>
#if UINTPTR_MAX == 0xffffffff
/* 32-bit */
typedef uint32_t POINTER_BASE;
#elif UINTPTR_MAX == 0xffffffffffffffff
/* 64-bit */
typedef uint64_t POINTER_BASE;
#else
/* wtf */
typedef NULL POINTER_BASE;
#endif
using namespace std;
class A {
private:
	virtual void f(void) {
		cout << "Hello A::f()" << endl;
	}
	void g(void) {
		cout << "Hello A::g()" << endl;
	}
	static void j(void) {
		cout << "Hello A::j()" << endl;
	}
	virtual void i(void) {
		cout << "Hello A::i()" << endl;
	}
	A() {
		void(A::*pAF)(void) = &A::g;
		pf_a = (POINTER_BASE*)(*(POINTER_BASE*)(&pAF));
		void(*pSAF)(void) = &A::j;
		psf_a = (POINTER_BASE*)pSAF;
	}
	virtual ~A() {
		if (nullptr != obj) delete obj;
	}
	static A* obj;
public:
	static A* getInstance() {
		if (nullptr == obj) obj = new A();
		return obj;
	}
	POINTER_BASE *pf_a;
	POINTER_BASE *psf_a;
};
A* A::obj = nullptr;
int main() {
	A *a = A::getInstance();
	POINTER_BASE b = *((POINTER_BASE*)&(*a));
	typedef void(*P_FUN)(void);
	P_FUN pF;
	pF = (P_FUN)(*((POINTER_BASE*)(*(POINTER_BASE*)(&b)) + 1));
	pF(); // call private virtual void A::i(void)
	pF = (P_FUN)(a->pf_a);
	pF(); // call private void A::g(void)
	pF = (P_FUN)(a->psf_a);
	pF(); // call private static void A::j(void)
	return 0;
}

{% endhighlight %}

## 输出
{% highlight cpp %}
Hello A::i()
Hello A::g()
Hello A::j()
{% endhighlight %}

## 解析

1. 调用`A::i()`就是虚函数表查表，用vs debug一下就能看到一个叫`__vfptr`的变量(**v**irtual **f**unction **p**oin**t**e**r**)，如图

   ![虚函数表](https://s2.ax1x.com/2019/08/22/ma24QH.png)

2. 调用`A::g()`和`A::j()`差别不大，都是想办法把函数指针整出去，唯一的区别就是`j`函数加了static之后貌似没有命名空间的限制了