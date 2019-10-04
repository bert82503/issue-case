

Google Code Review Developer Guide
==================================
> Google, 2019-09-16


## 1.Introduction
**A code review is a process** where someone other than the author(s) of **a piece of code examines that code.**
代码审查是一个过程。

At Google we use code review to **maintain the quality of our code and products**. (目标)
在谷歌，我们使用代码审查来维护代码和产品的质量。

This documentation is the canonical description of Google's code review processes and policies.

This page is an overview of our code review process. There are two other large documents that are a part of this guide:
* [How To Do A Code Review](https://github.com/google/eng-practices/blob/master/review/reviewer): A detailed guide for code reviewers.
  如何进行代码审查：代码审查员的详细指南
* [The CL Author's Guide](https://github.com/google/eng-practices/blob/master/review/developer): A detailed guide for developers whose CLs are going through review.
  更改列表作者指南：针对更改列表正在进行审查的开发人员的详细指南

> **CL**: Stands for "changelist," which means one self-contained change that has been submitted to version control or which is undergoing code review.


## 2.What Do Code Reviewers Look For?
> 代码审查员在寻找什么？

Code reviews should look at:
* **Design**: Is the code well-designed and appropriate for your system?
  **设计**：代码是否经过精心设计并适合您的系统？
* **Functionality**: Does the code behave as the author likely intended? Is the way the code behaves good for its users?
  **功能**：代码的行为是否与作者可能的预期相同？代码对用户的行为方式是否正常？
* **Complexity**: Could the code be made simpler? Would another developer be able to easily understand and use this code when they come across it in the future?
  **复杂性**：代码可以变得更简单吗？是否有其他开发人员能够在将来遇到这些代码时轻松理解并使用？ (可读性)
* **Tests**: Does the code have correct and well-designed automated tests?
  **测试**：代码是否具有正确且设计良好的自动化测试？
* **Naming**: Did the developer choose clear names for variables, classes, methods, etc.?
  **命名**：开发人员是否为变量，类，方法等选择了明确的名称？
* **Comments**: Are the comments clear and useful?
  **注释**：注释是否清晰有用？
* **Style**: Does the code follow our [style guides](http://google.github.io/styleguide/)?
  **风格**：代码是否遵循我们的风格指南？
* **Documentation**: Did the developer also update relevant documentation?
  **文档**：开发人员是否也更新了相关文档？

See [How To Do A Code Review](https://github.com/google/eng-practices/blob/master/review/reviewer) for more information.


## 3.Picking the Best Reviewers
> 挑选最佳代码审查员


## 4.In-Person Reviews


## 5.See Also
* [How To Do A Code Review](https://github.com/google/eng-practices/blob/master/review/reviewer): A detailed guide for code reviewers.
* [The CL Author's Guide](https://github.com/google/eng-practices/blob/master/review/developer): A detailed guide for developers whose CLs are going through review.


[原文](https://github.com/google/eng-practices/blob/master/review/index.md)

