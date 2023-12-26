---
title: "超越语法：利用ClangChecker拓展进行高级代码分析，提升漏洞检测和预防能力"
date: 2023-06-21 11:19:44
img: https://wangxs020202.gitee.io/pbad/background/page_8.png
top: false
summary: 本文将介绍如何通过对ClangChecker拓展进行高级代码分析，提升漏洞检测和预防能力。
categories: 笔记
tags:
  - LLVM
  - Clang
  - ClangChecker
  - 静态分析
  - 漏洞检测
  - 预防
  - 教程
  - 笔记
---

### 序幕

ClangChecker是Clang的一个静态分析框架，可以用于检测C/C++代码中的错误。本文将介绍如何通过对ClangChecker拓展进行高级代码分析，提升漏洞检测和预防能力。

### ClangChecker

1. 简介

   ClangChecker是Clang的一个静态分析框架，可以用于检测C/C++代码中的错误。ClangChecker的工作原理是：通过对源码进行语法分析，生成抽象语法树（AST），然后对AST进行遍历，检测错误。ClangChecker的检测能力可以通过拓展进行扩展，本文将介绍如何通过对ClangChecker拓展进行高级代码分析，提升漏洞检测和预防能力。
2. 获取

   ClangChecker是LLVM的一部分，可以参考上一篇文章：[定制你的编译过程：像专业人士一样从源码构建和安装LLVM&Clang](https://sirxs.cn/2023/06/21/page-id-8/)。
3. 拓展

   ClangChecker的检测能力可以通过拓展进行扩展，本文将介绍如何通过对ClangChecker拓展进行高级代码分析，提升漏洞检测和预防能力。

### 拓展ClangChecker

1. 在已有的检查器包——`alpha.core`中添加自定义的检查器，那么在`Checkers.td`文件中添加如下内容：

   ```td
   def MainCallChecker : Checker<"MainCall">,
     HelpText<"Check for calls to main">,
     Documentation<NotDocumented>;
   ```

   修改后的内容如下：

   ```td
   let ParentPackage = CoreAlpha in {
   
   // 省略 ...
   
   def MainCallChecker : Checker<"MainCall">,
     HelpText<"Check for calls to main">,
     Documentation<NotDocumented>;
   
   } // end "alpha.core"
   ```

   *注：*

    - `Checkers.td`文件位于`clang/include/clang/StaticAnalyzer/Checkers/`目录中。
    - `clang -cc1 -analyzer-checker-help`和`clang -cc1 -analyzer-checker-help-alpha`
      等命令所显示的检查器列表来源于`Checkers.td`文件。
    - `def MainCallChecker`，表示检查器的注册名称为`MainCallChecker`。
    - `"MainCall"`，表示检查器的名称为`MainCall`。也就是说，可以通过`-analyzer-checker=alpha.core.MainCall`标志来启用该检查器。
    - `HelpText`选项，用于指定该检查器对应的描述。从而，在执行类似于`-help`命令时显示。
    - `Documentation`选项，用于指定检查器文档的`URI`地址。

   **需要注意的是，** 修改的`Checkers.td`文件在**重新编译安装 Clang** 后才生效。

2. 添加检查器的实现代码

   在`clang/lib/StaticAnalyzer/Checkers/`目录中新建`MainCallChecker.cpp`文件，内容如下：

   ```c++
   #include "clang/StaticAnalyzer/Checkers/BuiltinCheckerRegistration.h"
   #include "clang/StaticAnalyzer/Core/BugReporter/BugType.h"
   #include "clang/StaticAnalyzer/Core/Checker.h"
   #include "clang/StaticAnalyzer/Core/CheckerManager.h"
   #include "clang/StaticAnalyzer/Core/PathSensitive/CallEvent.h"
   #include "clang/StaticAnalyzer/Core/PathSensitive/CheckerContext.h"
   
   using namespace clang;
   using namespace ento;
   
   namespace {
   
   class MainCallChecker : public Checker<check::PreCall> {
   public:
     void checkPreCall(const CallEvent &Call, CheckerContext &Ctx) const;
   
   private:
     mutable std::unique_ptr<BugType> BT;
   };
   
   } // anonymous namespace
   
   void MainCallChecker::checkPreCall(const CallEvent &Call,
                                      CheckerContext &Ctx) const {
     if (const IdentifierInfo *II = Call.getCalleeIdentifier()) {
       if (II->isStr("main")) {
         if (!BT) {
           BT.reset(new BugType(this, "Call to main", "Example checker"));
         }
         ExplodedNode *Node = Ctx.generateErrorNode();
         auto Report
           = std::make_unique<PathSensitiveBugReport>(*BT, BT->getDescription(), Node);
         Ctx.emitReport(std::move(Report));
       }
     }
   }
   
   void ento::registerMainCallChecker(CheckerManager &Mgr) {
     Mgr.registerChecker<MainCallChecker>();
   }
   
   bool ento::shouldRegisterMainCallChecker(const CheckerManager &mgr) {
     return true;
   }
   ```

   *注：*

   - 新建的源文件名称不必是`MainCallChecker.cpp`，也可以是其他名称。
   - 检查器的类名不必与在`Checkers.td`文件中定义的检查器的注册名称保持一致。
   - `void ento::registerXXX(CheckerManager &Mgr)`中的`XXX`必须与在`Checkers.td`文件中定义的检查器的注册名称保持一致。
   - `Mgr.registerChecker();`中的`XXX`必须与检查器的类名保持一致。
   - `ento::shouldRegisterXXX(const CheckerManager &mgr)`中的`XXX`必须与在`Checkers.td`文件中定义的检查器的注册名称保持一致。

3. 修改 CMakeLists.txt

   在`clang/lib/StaticAnalyzer/Checkers/CMakeLists.txt`文件中添加如下内容：

   ```
   MainCallChecker.cpp
   ```

   *注：*`MainCallChecker.cpp`即上文中新建的用于存放检查器实现代码的源文件。

   修改后的内容如下：

   add_clang_library(  # 省略 ... MainCallChecker.cpp # 省略 ...)

4. 编译安装 Clang

   可以参考上一篇文章：[定制你的编译过程：像专业人士一样从源码构建和安装LLVM&Clang](https://sirxs.cn/2023/06/21/page-id-8/)。

5. 查看自定义的检查器

   ```shell
   clang -cc1 -analyzer-checker-help-alpha | grep MainCall  alpha.core.MainCall      (Enable only for development!) Check for calls to main
   ```

   从上面的输出可以看出：

    - `alpha.core.MainCall`，即我们自定义的检查器的完整名称。
    - `Check for calls to main`，即我们在`Checkers.td`文件中为自定义的检查器所添加的描述。

6. 编写测试用例 `Example_Test.c`

   ```c
   typedef int (*main_t)(int, char **);
   int main(int argc, char **argv) {
     main_t foo = main;
     int exit_code = foo(argc, argv);   // actually calls main()!
     return exit_code;
   }
   ```

   上述测试程序存在这样的错误：通过函数指针变量`foo`调用`main`函数。

7. 运行自定义的检查器

   ```shell
   clang --analyze -Xanalyzer -analyzer-checker=alpha.core.MainCall Example_Test.c
   ```

   `output`

   ```shell
   Example_Test.c:4:19: warning: Call to main [alpha.core.MainCall]
     int exit_code = foo(argc, argv);   // actually calls main()!
                     ^~~~~~~~~~~~~~~
   1 warning generated.
   ```

