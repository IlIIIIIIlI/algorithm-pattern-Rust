# 快速开始

## 数据结构与算法

数据结构是一种数据的表现形式，如链表、二叉树、栈、队列等都是内存中一段数据表现的形式。
算法是一种通用的解决问题的模板或者思路，大部分数据结构都有一套通用的算法模板，所以掌握这些通用的算法模板即可解决各种算法问题。

后面会分专题讲解各种数据结构、基本的算法模板、和一些高级算法模板，每一个专题都有一些经典练习题，完成所有练习的题后，你对数据结构和算法会有新的收获和体会。

先介绍两个算法题，试试感觉~

### [示例 1：strStr](https://leetcode-cn.com/problems/implement-strstr/)

> 给定一个  haystack 字符串和一个 needle 字符串，在 haystack 字符串中找出 needle 字符串出现的第一个位置 (从 0 开始)。如果不存在，则返回  -1。

- 思路：核心点遍历给定字符串字符，判断以当前字符开头字符串是否等于目标字符串

```Rust
impl Solution {
    // Solution 的公共方法，符合问题预期的接口。
    // 它接受 String 类型的拥有所有权的 haystack 和 needle。
    // 返回类型为 i32，按照问题的规范。
    pub fn str_str(haystack: String, needle: String) -> i32 {
        // 在公共方法内部，将拥有所有权的 String 参数转换为字符串切片 (&str)
        // 通过借用进行转换。这个转换是必要的，因为内部逻辑（在 str_str_impl 中）
        // 是基于字符串切片操作的，而不是拥有所有权的 String。
        // 这种方法避免了获取函数参数的所有权，
        // 允许我们使用引用来操作。
        Solution::str_str_impl(&haystack, &needle)
    }

    // 一个私有辅助函数，执行实际的计算。
    // 这个函数被设计为使用字符串切片 (&str)，这对于只读操作如字符串搜索来说更为高效。
    fn str_str_impl(haystack: &str, needle: &str) -> i32 {
        // 检查 needle 是否为空字符串。
        // 根据问题定义，如果 needle 为空，
        // 函数应返回 0，表示在 haystack 的开始位置就匹配到了。
        if needle.is_empty() { return 0; }

        // 使用 find 方法搜索 haystack 中 needle 的第一次出现。
        // find 方法返回一个 Option<usize> 类型：
        // - Some(index) 如果找到了 needle，其中 index 是起始位置。
        // - None 如果没有找到 needle。
        // 然后使用 Option 上的 map_or 方法来处理这两种情况：
        // - 如果是 Some(index)，则转换为 index 的 i32 类型。
        // - 如果是 None（没有找到 needle），则返回 -1。
        haystack.find(needle).map_or(-1, |v| v as i32)
    }
}
```

需要注意点

- 通过在接口边界将 String 转换为 &str，代码高效处理字符串数据，无需不必要的克隆或所有权转移。。
- 错误处理：
  - Rust 通过 Result 和 Option 枚举强制进行错误处理，这与 Python 的异常处理机制不同。
  - 需要习惯使用 match 或 if let 表达式来处理这些枚举的值。
- 不可变性：
  - Rust 中的变量默认是不可变的。如果你需要修改变量的值，需要在声明时使用 mut 关键字。

### [示例 2：subsets](https://leetcode-cn.com/problems/subsets/)

> 给定一组不含重复元素的整数数组 nums，返回该数组所有可能的子集（幂集）。

- 思路：这是一个典型的应用回溯法的题目，简单来说就是穷尽所有可能性，算法模板如下

```go
result = []
func backtrack(选择列表,路径):
    if 满足结束条件:
        result.add(路径)
        return
    for 选择 in 选择列表:
        做选择
        backtrack(选择列表,路径)
        撤销选择
```

- 通过不停的选择，撤销选择，来穷尽所有可能性，最后将满足条件的结果返回。答案代码：

```Rust
impl Solution {
    // 定义一个公共的静态方法 `subsets`，该方法接收一个 Vec<i32> 类型的参数 `nums`，
    // 表示一组整数，返回一个 Vec<Vec<i32>> 类型，表示所有可能的子集合。
    pub fn subsets(nums: Vec<i32>) -> Vec<Vec<i32>> {
        // 定义一个动态数组 `result` 用于存储最终的所有子集结果。
        let mut result = Vec::new();

        // 定义一个内部的递归函数 `backtrack` 用于回溯搜索子集。
        // 它接受一个整数数组的切片 `nums`、一个起始索引 `start`、一个当前路径的可变引用 `route`，
        // 以及一个最终结果的可变引用 `result`。
        fn backtrack(nums: &[i32], start: usize, route: &mut Vec<i32>, result: &mut Vec<Vec<i32>>) {
            // 将当前路径的一个克隆添加到结果集中。
            // 这一步确保了在每次递归调用中，当前探索的路径都会被记录下来。
            result.push(route.clone());

            // 从 `start` 索引开始遍历 `nums` 数组。
            for i in start..nums.len() {
                // 将当前数字添加到路径中。
                route.push(nums[i]);
                // 递归调用 `backtrack` 函数，i + 1 保证下一次调用时，搜索的范围向前推进了一步。
                backtrack(nums, i + 1, route, result);
                // 回溯：将路径中最后一个数字移除，探索不包含当前数字的其他路径。
                route.pop();
            }
        }

        // 定义一个动态数组 `route` 用于记录当前搜索的路径。
        let mut route = Vec::new();
        // 调用 `backtrack` 函数开始搜索，从索引 0 和空路径开始。
        backtrack(&nums, 0, &mut route, &mut result);

        // 返回最终收集到的所有子集结果。
        result
    }
}
```

说明：后面会深入讲解几个典型的回溯算法问题，如果当前不太了解可以暂时先跳过

## 面试注意点

我们大多数时候，刷算法题可能都是为了准备面试，所以面试的时候需要注意一些点

- 快速定位到题目的知识点，找到知识点的**通用模板**，可能需要根据题目**特殊情况做特殊处理**。
- 先去朝一个解决问题的方向！**先抛出可行解**，而不是最优解！先解决，再优化！
- 代码的风格要统一，熟悉各类语言的代码规范。
  - 命名尽量简洁明了，尽量不用数字命名如：i1、node1、a1、b2
- 常见错误总结
  - 访问下标时，不能访问越界
  - 空值 nil 问题 run time error

## 练习

- [ ] [strStr](https://leetcode-cn.com/problems/implement-strstr/)
- [ ] [subsets](https://leetcode-cn.com/problems/subsets/)
