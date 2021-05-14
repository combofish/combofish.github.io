---
layout: post
title: "Android RadioGroup Error: java.lang.NullPointerException: findViewById&lt;RadioButton&gt;(checkedId) must not be null"
date: "2021-05-14 11：10：00"
tag: RadioGroup RadioButton Android Kotlin Java
---
- Error java.lang.NullPointerException: findViewById&lt;RadioButton&gt;(checkedId) must not be null
  - [情景](#org8fc74e5)
  - [错误原因](#orgf991045)
  - [解决办法一](#orgc2102ad)
  - [解决办法二](#org3428ee0)


<a id="org63bc53e"></a>

# 错误 java.lang.NullPointerException: findViewById&lt;RadioButton&gt;(checkedId) must not be null


<a id="org8fc74e5"></a>

## 情景

在 Android 开发时，使用到了 RadioGroup ：单项选择按钮组。 它可以包含多个 RadioButton，即单选按钮，它们共同为用户提供一种多选一的选择方式。 在多个 RadioButton 被同一个 RadioGroup 包含的情况下，多个 RadioButton 之间自动形成互斥关系，仅有一个可以被选择。 单选按钮的使用方法和 CheckBox 的使用方法高度相似，其事件监听接口使用的是 RadioGroup.OnCheckedChangeListener()，使用 setOnCheckedChangeListener() 方法将监听器设置到单选按钮上。

-   UI 布局部分代码

```xml
<RadioGroup
    android:layout_below="@id/title1"
    android:id="@+id/check_group"
    android:layout_centerHorizontal="true"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:gravity="center"
    android:orientation="horizontal">

  <RadioButton
      android:id="@+id/check_button_1"
      android:layout_width="wrap_content"
      android:layout_height="wrap_content"
      android:text="@string/value1" />

  <RadioButton
      android:id="@+id/check_button_2"
      android:layout_width="wrap_content"
      android:layout_height="wrap_content"
      android:text="@string/value2" />
</RadioGroup>

<Button
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:gravity="center"
    android:layout_below="@id/check_group"
    android:layout_centerHorizontal="true"
    android:id="@+id/btn_clear"
    android:text="@string/clear_check_group"/>
```


<a id="orgf991045"></a>

## 错误原因

-   单选按钮组的响应函数

```kotlin
inner class MyRadioGroupCheckedChangedListener() : RadioGroup.OnCheckedChangeListener {
    override fun onCheckedChanged(group: RadioGroup?, checkedId: Int) {
	Log.i(TAG, "Checked: $checkedId")
	radioGroupValue = findViewById<RadioButton>(checkedId).text as String
	Log.i(TAG,"Now RadioGroup's value is: ${radioGroupValue}")
    }
}
```

-   清空按钮的响应函数

```kotlin
override fun onClick(v: View?) {
    when (v?.id) {
	R.id.btn_clear -> {
	    Log.i(TAG, "Clear Button pressed!")
	    radioGroupTest.clearCheck()
	}
    }
}
```

错误原因是，在清空全部的单项选择按钮后，单选按钮组的响应函数 OnCheckedChanged() 会被执行，而此时回调函数的参数 checkedId 为 -1。而在代码里，直接把此时为 -1 的 checkedId 传给了 findviewbyid()。 系统找不到资源，所以报错。


<a id="orgc2102ad"></a>

## 解决办法一

-   回避 checkedId 为 1 的情况

```kotlin
inner class MyRadioGroupCheckedChangedListener() : RadioGroup.OnCheckedChangeListener {
    override fun onCheckedChanged(group: RadioGroup?, checkedId: Int) {
	Log.i(TAG, "Checked: $checkedId")
	if (checkedId.equals(-1)) return
	radioGroupValue = findViewById<RadioButton>(checkedId).text as String
	Log.i(TAG,"Now RadioGroup's value is: ${radioGroupValue}")
    }
}
```

在 checkedId 为 -1 时，直接把函数返回。不进行后续操作。


<a id="org3428ee0"></a>

## 解决办法二

-   不使用 RadioGroup.clearcheck() 方法。用不可见的 RadioButton 来模拟清空所有单选框。

在布局文件中添加一个不可见的 RadioButton。即 visibility='gone'。这样这个组件在视图层等于没有。

```xml
<RadioButton
    android:id="@+id/blankRadioButton"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:button="@null"
    android:visibility="gone" />
```

然后将清空的函数为选中这个不可见的占位单选框。

```kotlin
override fun onClick(v: View?) {
    when (v?.id) {
	R.id.btn_clear -> {
	    Log.i(TAG, "Clear Button pressed!")
	    // radioGroupTest.clearCheck()
	    clearRadioGroup()
	}
    }
}

private fun clearRadioGroup() {
    blankRadioButton.isChecked = true
}
```
