---
layout: post
title: "CPP implements a custom template stack and overloads input and output operators"
date: "2021-11-06 17: 43: 00"
tag: cpp stack templates
---
- [CPP 实现自定义模板栈](#org553b67c)
  - [具体结构](#org45d063e)
  - [重载输入输出操作符](#orgd9f4854)
  - [类模板配合友元函数实现](#org5907979)
    - [类内实现](#orge826ba3)
    - [类外实现1](#org5564af8)
    - [类外实现2](#org46977c2)
  - [完整代码](#org0293847)


<a id="org553b67c"></a>

# CPP 实现自定义模板栈

栈（stack）又名堆栈，它是一种运算受限的线性表。限定仅在表尾进行插入和删除操作的线性表。这一端被称为栈顶，相对地，把另一端称为栈底。 向一个栈插入新元素又称作进栈、入栈或压栈，它是把新元素放到栈顶元素的上面，使之成为新的栈顶元素； 从一个栈删除元素又称作出栈或退栈，它是把栈顶元素删除掉，使其相邻的元素成为新的栈顶元素。
![栈的结构](/images/CPP-custom_template-stack/stack.jpeg)

<a id="org45d063e"></a>

## 具体结构

实现一个特定类型的栈只能用于存储特定类型，功能很有限。就想写成模板的形式，这样通用性会更强一些。

-   先定义模板栈类的结构

```cpp
const int defaultSize = 16;
template <typename T> class CStack {
public:
  CStack(int);
  ~CStack();
  bool empty();
  bool full();
  int size() const;            /* 返回栈已使用的空间个数 */
  int capacity() const;        /* 返回栈的容量，开辟的空间数组的总大小 */
  bool push(T);                /* 入栈操作 */
  bool pop(T &);               /* 出栈操作 */
  T operator[](int);

private:
  void increaseCapacity();     /* 用于在栈空间满时，自动扩充栈 */

  T *m_pData;                  
  int m_nTop;                  /* 指向栈顶 */
  int m_nBottom;               /* 指向栈底 */
  int m_nSize;                 /* 栈的空间大小 */
};
```

-   实现构造函数和析构函数

```cpp
template <typename T> CStack<T>::CStack(int size) {
  if (size <= 0)
    size = defaultSize;

  m_nSize = size;
  m_pData = new T[size];
  m_nTop = m_nBottom = 0;
}

template <typename T> CStack<T>::~CStack() { delete[] m_pData; }
```

-   实现入栈和出栈操作

```cpp
/* size() const */
template <typename T> int CStack<T>::size() const { return m_nTop - m_nBottom; }

template <typename T> bool CStack<T>::full() { return m_nTop >= m_nSize; }

/* empty() */
template <typename T> bool CStack<T>::empty() {
  return (m_nTop == m_nBottom) ? true : false;
}

template <typename T> bool CStack<T>::push(T ele) {
  if (full())
    increaseCapacity();

  m_pData[m_nTop++] = ele;
  return true;
}

template <typename T> bool CStack<T>::pop(T &ele) {
  if (empty())
    return false;

  ele = m_pData[--m_nTop];
  return true;
}
```

-   实现索引访问

```cpp
template <typename T> T CStack<T>::operator[](int idx) {
  if (idx < m_nBottom || idx >= m_nTop)
    std::exit(-1);

  return m_pData[idx];
}
```

-   实现栈空间自增

```cpp
template <typename T> int CStack<T>::capacity() const { return m_nSize; }

template <typename T> void CStack<T>::increaseCapacity() {
  m_nSize = m_nSize * 2;
  T *tmp_nData = new T[m_nSize];
  for (int i = 0; i < m_nTop; i++) {
    tmp_nData[i] = m_pData[i];
  }
  delete[] m_pData;
  m_pData = tmp_nData;
}
```


<a id="orgd9f4854"></a>

## 重载输入输出操作符

在C++中，I/O使用了流的概念-字符（或字节）流。我们可以用 cin 从命令行读取信息，也可以用 cout 把信息输出到标准输出流。 那么我们能不能用流操作符实现对自定义栈的操作呢，用 >> 来读取信息到栈，用 << 来从栈中起出数据。

-   先看代码

```cpp
void printStack(CStack<string> &sta) {
  cout << "Print stack: ";
  for (int i = 0; i < sta.size(); i++) {
    cout << sta[i] << " ";
  }
  cout << endl;
}

int main() {
  std::stringstream ss1("hello"),ss2("world");
  CStack<string> sta(3);
  ss1 >> sta;
  ss2 >> sta;

  printStack(sta);
  cout << sta << endl;
  printStack(sta);
  return 0;
}
```

-   结果如下

```shell
Print stack: hello world 
world
Print stack: hello
```


<a id="org5907979"></a>

## 类模板配合友元函数实现

<a id="orge826ba3"></a>

### 类内实现

-   在模板类内部直接声明友元函数，不涉及函数模板

这种情况只能在模板类内部一起把函数的定义写出来，不能在外部实现，因为外部需要类型参数，而需要类型参数就是模板了。 其实这种情况相当于一般的模板类的成员函数，也就是相当于一个函数模板

```cpp
template <typename T> class CStack {
  // other code
public:
  /* 全局函数配合友元 类内实现 */
  friend std::ostream &operator<<(std::ostream &out, CStack<T> &cstack) {
    T ele;
    cstack.pop(ele);
    out << ele;
    return out;
  }
  friend std::istream &operator>>(std::istream &in, CStack<T> &s) {
    T ele;
    in >> ele;
    s.push(ele);
    return in;
  }
};
```


<a id="org5564af8"></a>

### 类外实现1

-   在模板类内部声明对应版本的友元函数模板实例化时，需要前置声明

该方法声明一个函数模板，并保持该函数的参数类型和该模板类的实例一个类型。 注意 <> 符号的使用，一是，表明此友元函数是函数模板；二是，此模板使用的模板类型参数为当前模板类的类型参数class。

不加 <> 会报错：friend declaration ‘std::ostream& operator<<(std::ostream&, CStack<T>&)’ declares a non-template function

```cpp
/* 在 Stack 类外 */
template <typename T> class CStack; /* 要对类进行声明 要声明在模板之前 */

template <typename T> std::ostream &operator<<(std::ostream &, CStack<T> &);
template <typename T> std::istream &operator>>(std::istream &, CStack<T> &);

/* 在 Stack 类内部 */
template <typename T> class CStack {
  /* other code */
public:
  /* 全局函数配合友元 类外实现 */
  friend std::ostream &operator<<<>(std::ostream &out, CStack<T> &);
  friend std::istream &operator>><T>(std::istream &in, CStack<T> &);
};

/* 函数具体实现 */
template <typename T>
inline std::ostream &operator<<(std::ostream &out_stream, CStack<T> &cstack)
{
  T ele;
  cstack.pop(ele);
  out_stream << ele;
  return out_stream;
}

template <typename T>
inline std::istream &operator>>(std::istream &in_stream, CStack<T> &cstack) {
  T ele;
  in_stream >> ele;
  cstack.push(ele);
  return in_stream;
}
```


<a id="org46977c2"></a>

### 类外实现2

-   在模板类内部声明友元的函数模板

这种方式更为灵活，它不会要求参数类型与模板类实例是一个类型，但是一般情况下我们也是按照一个类型使用的。

```cpp
template <typename T> class CStack {
  /* other code */
public:
  template <typename U>
  friend std::ostream &operator<<(std::ostream &out, CStack<U> &);

  template <typename U>
  friend std::istream &operator>>(std::istream &in, CStack<U> &);
};

/* 具体实现 */
template <typename T>
inline std::ostream &operator<<(std::ostream &out_stream, CStack<T> &cstack) {
  T ele;
  cstack.pop(ele);
  out_stream << ele;
  return out_stream;
}

template <typename T>
inline std::istream &operator>>(std::istream &in_stream, CStack<T> &cstack) {
  T ele;
  in_stream >> ele;
  cstack.push(ele);
  return in_stream;
}
```


<a id="org0293847"></a>

## 完整代码

[custom_template_stack.h</sub>](https://github.com/combofish/chips-get/blob/master/CPP/MyStack/custom_template_stack.h)
