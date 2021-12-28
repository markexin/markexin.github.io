---
title: ES2022 feature class static initialization blocks
tags: 趣味杂谈
---

## ES2022 feature: class static initialization blocks



### Introduction 简介

Ron Buckton 在 *ECMAScript* 规范中提出  [“Class static initialization blocks” ](https://github.com/tc39/proposal-class-static-block)已经进入到第四阶段，预计将写入2022年ECMAScript中。

当我们创建Class类，通常我们可以有两种方式创建 constructs：

*  Field: 创建（并可选择初始化）实例属性；
* 构造函数Constructor：在设置完成之前执行的代码块。

为了设置类的静态部分，我们只有静态字段。 ECMAScript 提案为类引入了**static initialization blocks** 。



大致上，对于静态类来说，构造函数对于实例来说是一样的。



### 目录

* 为什么我们需要**static blocks**在class类中？
* 一个更复杂的例子
* 支持类静态块的引擎
* JavaScript 是否变得很像 Java 和/或一团糟？
* 结论



#### 1. 为什么我们需要**static blocks**在class类中？

为了方便外部函数调用内部属性，我们通常会这样设置：

```javascript
class Translator {
  static translations = {
    yes: 'ja',
    no: 'nein',
    maybe: 'vielleicht',
  };
  static englishWords = extractEnglish(this.translations);
  static germanWords = extractGerman(this.translations);
}
function extractEnglish(translations) {
  return Object.keys(translations);
}
function extractGerman(translations) {
  return Object.values(translations);
}
```

在这种情况下，使用外部函数 extractEnglish() 和 extractGerman() 效果很好，因为我们可以看到它们是从**类内部调用的**，并且它们完全**独立于类**。但是，如果我们想同时设置两个静态字段，事情就会变得不那么优雅：

```javascript
class Translator {
  static translations = {
    yes: 'ja',
    no: 'nein',
    maybe: 'vielleicht',
  };
  static englishWords = [];
  static germanWords = [];
  static _ = initializeTranslator( // (A)
    this.translations, this.englishWords, this.germanWords);
}
function initializeTranslator(translations, englishWords, germanWords) {
  for (const [english, german] of Object.entries(translations)) {
    englishWords.push(english);
    germanWords.push(german);
  }
}
```

这次有几个问题：

* 调用 initializeTranslator() 是一个额外的步骤，必须在创建类之后在类之外执行。或者通过变通方法执行（A 行）。
* initializeTranslator() 无权访问 Translator 的私有属性。

当我们使用 **static initialization blocks**（A 行），我们有一个更优雅的解决方案。

```javascript
class Translator {
  static translations = {
    yes: 'ja',
    no: 'nein',
    maybe: 'vielleicht',
  };
  static englishWords = [];
  static germanWords = [];
  static { // (A)
    for (const [english, german] of Object.entries(this.translations)) {
      this.englishWords.push(english);
      this.germanWords.push(german);
    }
  }
}
```



#### 2.一个更复杂的例子

**static initialization blocks** 的细节是相对合乎逻辑的（相对于更复杂的实例成员规则）：

* 每个类可以有多个静态块。
* 静态块的执行与静态字段初始值设定项的执行交错执行。
* 超类的静态成员在子类的静态成员之前执行。

以下代码演示了这些规则：

```javascript
class SuperClass {
  static superField1 = console.log('superField1');
  static {
    assert.equal(this, SuperClass);
    console.log('static block 1 SuperClass');
  }
  static superField2 = console.log('superField2');
  static {
    console.log('static block 2 SuperClass');
  }
}

class SubClass extends SuperClass {
  static subField1 = console.log('subField1');
  static {
    assert.equal(this, SubClass);
    console.log('static block 1 SubClass');
  }
  static subField2 = console.log('subField2');
  static {
    console.log('static block 2 SubClass');
  }
}

// Output:
// 'superField1'
// 'static block 1 SuperClass'
// 'superField2'
// 'static block 2 SuperClass'
// 'subField1'
// 'static block 1 SubClass'
// 'subField2'
// 'static block 2 SubClass'
```



#### 3. 支持类静态块的引擎

- V8: unflagged in v9.4.146 ([source](https://github.com/tc39/proposal-class-static-block#stage-4-entrance-criteria))
- SpiderMonkey: behind a flag in v92, intent to ship unflagged in v93 ([source](https://github.com/tc39/proposal-class-static-block#stage-4-entrance-criteria))
- TypeScript: v4.4 ([source](https://devblogs.microsoft.com/typescript/announcing-typescript-4-4-rc/))



#### 4. JavaScript 是否变得很像 Java 和/或一团糟？

这是一个小功能，不会与其他功能竞争。我们已经可以通过带有 ` static _ = ... ` 解决方法的运行。**static initialization blocks** 意味着不再需要这种解决方法。除此之外，类只是JavaScript程序员的众多工具之一。有些人使用它，有些人不使用，还有很多替代方法。即使是使用类的JavaScript代码也经常使用函数，并且趋向于轻量级。



#### 5.结论

**static initialization blocks** 是一个相对简单的特性，它完善了类的静态属性。粗略地说，它是实例构造函数的静态版本。当我们必须设置多个静态字段时，它可以发挥很大的作用。

