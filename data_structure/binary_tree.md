# 二叉树

## 知识点

### 二叉树遍历

- 先序遍历：首先访问<u>根</u>，再先序遍历左（右）子树，最后先序遍历右（左）子树。
- 中序遍历：首先中序遍历左（右）子树，再访问<u>**根**</u>，最后中序遍历右（左）子树。
- 后序遍历：首先后序遍历左（右）子树，再后序遍历右（左）子树，最后访问<u>根</u>。

注意点

- 以根访问顺序决定是什么遍历
- 左子树都是优先右子树

#### 递归模板

- 递归实现二叉树遍历非常简单，不同顺序区别仅在于访问父结点顺序

二叉树的每个节点由键 key、值 value 与左右子树 left/right 组成，这里我们把节点声明为一个泛型结构。

```rust
use std::rc::Rc;
use std::cell::RefCell;

// 定义二叉树节点结构体
#[derive(Debug, PartialEq, Eq)]
pub struct TreeNode {
    pub val: i32,
    pub left: Option<Rc<RefCell<TreeNode>>>,
    pub right: Option<Rc<RefCell<TreeNode>>>,
}

// 访问节点的函数，这里仅仅是打印节点值
fn visit(node: &TreeNode) {
    println!("{}", node.val);
}

// 前序遍历
fn preorder_rec(root: Option<Rc<RefCell<TreeNode>>>) {
    if let Some(node) = root {
        let node = node.borrow();
        visit(&node); // 访问当前节点
        preorder_rec(node.left.clone()); // 递归遍历左子树
        preorder_rec(node.right.clone()); // 递归遍历右子树
    }
}

// 中序遍历
fn inorder_rec(root: Option<Rc<RefCell<TreeNode>>>) {
    if let Some(node) = root {
        let node = node.borrow();
        inorder_rec(node.left.clone()); // 递归遍历左子树
        visit(&node); // 访问当前节点
        inorder_rec(node.right.clone()); // 递归遍历右子树
    }
}

// 后序遍历
fn postorder_rec(root: Option<Rc<RefCell<TreeNode>>>) {
    if let Some(node) = root {
        let node = node.borrow();
        postorder_rec(node.left.clone()); // 递归遍历左子树
        postorder_rec(node.right.clone()); // 递归遍历右子树
        visit(&node); // 访问当前节点
    }
}
```

#### [前序非递归](https://leetcode-cn.com/problems/binary-tree-preorder-traversal/)

- 本质上是图的 DFS 的一个特例，因此可以用栈来实现

```rust
use std::rc::Rc;
use std::cell::RefCell;

impl Solution {
    // 前序遍历的迭代实现
    pub fn preorder_traversal(root: Option<Rc<RefCell<TreeNode>>>) -> Vec<i32> {
        let mut ans = Vec::new(); // 用于存放遍历结果
        let mut stack = vec![root]; // 栈用于存放接下来需要访问的节点

        // 当栈不为空时，循环继续
        while let Some(node_option) = stack.pop() {
            // if let检查node_option是否包含一个值。如果是，它会将这个值绑定到node_ref变量，并执行if块中的代码。
            if let Some(node_ref) = node_option {
                let node = node_ref.borrow(); // 获取节点的不可变引用
                ans.push(node.val); // 访问当前节点
                // 如果有右子树，先压入右子树（保证左子树先处理）
                if node.right.is_some() {
                    stack.push(node.right.clone());
                }
                // 如果有左子树，再压入左子树
                if node.left.is_some() {
                    stack.push(node.left.clone());
                }
            }
        }
        ans
    }
}

```

使用`Option<Rc<RefCell<TreeNode>>>`处理节点的所有权和可变性，确保在遍历过程中能够安全、有效地访问和修改树结构。

我们没有使用`.take()`来移除节点的所有权，因为这样做会改变原始树的结构。相反，我们使用`.clone()`来复制`Option`内部的`Rc`指针，这样就可以保持原始树不变，同时避免了使用`borrow_mut`造成的潜在可变借用冲突。

#### [中序非递归](https://leetcode-cn.com/problems/binary-tree-inorder-traversal/)

```rust
impl Solution {
    // 中序遍历的迭代实现
    pub fn inorder_traversal(root: Option<Rc<RefCell<TreeNode>>>) -> Vec<i32> {
        let mut s = Vec::new(); // 栈用于存放接下来需要访问的节点
        let mut inorder = Vec::new(); // 用于存放遍历结果
        let mut node = root; // 当前访问的节点

        // 当栈不为空或者当前节点不为空时，循环继续
        // 从栈中弹出一个节点时，必须首先确保它的所有左子树都已经被访问过了。
        while s.len() > 0 || node.is_some() {
            if let Some(node_ref) = node {
                s.push(node_ref.clone()); // 将当前节点压入栈中
                node = node_ref.borrow().left.clone(); // 移动到左子树
            } else {
                node = s.pop(); // 从栈中取出下一个节点
                if let Some(node_ref) = node {
                    let node_borrowed = node_ref.borrow();
                    inorder.push(node_borrowed.val); // 访问当前节点
                    node = node_borrowed.right.clone(); // 移动到右子树
                }
            }
        }

        inorder
    }
}
```

#### [后序非递归](https://leetcode-cn.com/problems/binary-tree-postorder-traversal/)

```rust
impl Solution {
    // 后序遍历的迭代实现
    pub fn postorder_traversal(root: Option<Rc<RefCell<TreeNode>>>) -> Vec<i32> {
        let mut ans = Vec::new(); // 用于存放遍历结果
        let mut stack = Vec::new(); // 栈用于存放节点
        let (mut node, mut last_visit) = (root.clone(), None);

        while stack.len() > 0 || node.is_some() {
            while let Some(node_ref) = node {
                stack.push(node_ref.clone());
                node = node_ref.borrow().left.clone();
            }
            // 查看栈顶节点而不弹出
            let node_ref = stack.last().unwrap().clone();
            let node_borrowed = node_ref.borrow();
            // 如果右子树存在且未被访问，或者没有右子树，访问当前节点
            if node_borrowed.right.is_none() || Some(node_borrowed.right.as_ref().unwrap().clone()) == last_visit {
                ans.push(node_borrowed.val);
                last_visit = stack.pop();
                node = None; // 避免再次进入左子树的循环
            } else {
                node = node_borrowed.right.clone();
            }
        }
        ans
    }
}
```

注意点

- **进入左子树**:
  - 使用`while let`循环沿左子树深入，将途径的每个节点推入栈中，直到遇到空节点。
- **访问节点的条件**:
  - 从栈中查看顶部节点(`peek`)，但不立即弹出，以便确定是否可以访问该节点或需转向右子树。
  - 判断是否可以访问当前节点（即是否已经访问了该节点的右子树），依赖于最近一次访问的节点(`last_visit`)。
- **处理`.as_ref().unwrap()`**:
  - `.as_ref()`将`Option<Rc<RefCell<TreeNode>>>`转换为`Option<&Rc<RefCell<TreeNode>>>`，使其可以被借用而不是获取所有权。
  - `.unwrap()`尝试从`Option`类型中取出值。如果是`Some`，返回内部值；如果是`None`，则会导致 panic。这里使用`.unwrap()`，因为通过逻辑保证了此处`Option`不会是`None`。
- **访问和转移逻辑**:
  - 如果当前节点的右子树为`None`或最近访问的节点是当前节点的右子树，则访问当前节点，并将其从栈中移除。
  - 访问节点后，将`last_visit`更新为当前节点，以指示最近访问过此节点。
  - 如果当前节点的右子树存在且未被访问，则将当前节点更新为其右子树，准备下一轮遍历。
- **注意点**:
  - 通过`clone()`复制`Rc`指针，以在不转移所有权的情况下共享访问。这是管理树结构中节点的常见 Rust 做法。

#### DFS 深度搜索-从下向上（分治法）

```Python
impl Solution {
    // 前序遍历的递归实现
    pub fn preorder_traversal(root: Option<Rc<RefCell<TreeNode>>>) -> Vec<i32> {
        // 检查根节点是否为空
        if let Some(node_ref) = root {
            let node = node_ref.borrow();
            let val = node.val; // 访问当前节点
            let left_result = Self::preorder_traversal(node.left.clone()); // 递归遍历左子树
            let right_result = Self::preorder_traversal(node.right.clone()); // 递归遍历右子树

            // 将当前节点的值与左右子树的遍历结果合并
            std::iter::once(val).chain(left_result.into_iter()).chain(right_result.into_iter()).collect()
        } else {
            Vec::new() // 如果根节点为空，返回空向量
        }
    }
}
```

注意点：

> DFS 深度搜索（从上到下） 和分治法区别：前者一般将最终结果通过指针参数传入，后者一般递归返回结果最后合并

#### [BFS 层次遍历](https://leetcode-cn.com/problems/binary-tree-level-order-traversal/)

```Python
impl Solution {
    // 层次遍历的实现
    pub fn level_order(root: Option<Rc<RefCell<TreeNode>>>) -> Vec<Vec<i32>> {
        let mut levels = Vec::new(); // 存储最终的层序遍历结果
        if root.is_none() {
            // 如果根节点为空，则直接返回空的结果集
            return levels;
        }

        let mut bfs = VecDeque::new(); // 使用VecDeque作为BFS的队列
        bfs.push_back(root.unwrap()); // 将根节点加入队列

        // 当队列不为空时，继续遍历
        while bfs.len() > 0 {
            let level_size = bfs.len(); // 记录当前层的节点数
            let mut current_level = Vec::new(); // 用于存储当前层的节点值

            for _ in 0..level_size {
                // 对当前层的每个节点进行遍历
                let node = bfs.pop_front().unwrap(); // 从队列中取出节点
                let node_borrowed = node.borrow(); // 借用节点，访问其值和子节点
                current_level.push(node_borrowed.val); // 将节点值加入当前层结果集

                // 如果左子节点存在，则加入队列等待后续遍历
                if let Some(left) = node_borrowed.left.clone() {
                    bfs.push_back(left);
                }
                // 如果右子节点存在，同样加入队列
                if let Some(right) = node_borrowed.right.clone() {
                    bfs.push_back(right);
                }
            }

            // 将当前层的结果加入到最终结果集中
            levels.push(current_level);
        }
        levels // 返回层序遍历的结果
    }
}
```

### 分治法应用

先分别处理局部，再合并结果

适用场景

- 快速排序
- 归并排序
- 二叉树相关问题

分治法模板

- 递归返回条件
- 分段处理
- 合并结果

## 常见题目示例

### [maximum-depth-of-binary-tree](https://leetcode-cn.com/problems/maximum-depth-of-binary-tree/)

> 给定一个二叉树，找出其最大深度。

- 思路 1：分治法

```Python
impl Solution {
    // 计算二叉树的最大深度
    pub fn max_depth(root: Option<Rc<RefCell<TreeNode>>>) -> i32 {
        match root {
            // 如果当前节点为空，返回深度为0
            None => 0,
            // 如果当前节点非空
            Some(node) => {
                // 递归计算左子树的最大深度
                let left_depth = Solution::max_depth(node.borrow().left.clone());
                // 递归计算右子树的最大深度
                let right_depth = Solution::max_depth(node.borrow().right.clone());
                // 当前节点的深度为左右子树深度的最大值加1
                1 + std::cmp::max(left_depth, right_depth)
            },
        }
    }
}
```

通过`std::cmp::max(left_depth, right_depth)`来比较左右子树的深度并取较大值。

- 思路 2：层序遍历

```Python
impl Solution {
    // 计算二叉树的最大深度
    pub fn max_depth(root: Option<Rc<RefCell<TreeNode>>>) -> i32 {
        if root.is_none() {
            // 如果根节点为空，深度为0
            return 0;
        }

        let mut bfs = VecDeque::new(); // 使用VecDeque作为队列进行广度优先搜索
        bfs.push_back(root.unwrap()); // 将根节点加入队列
        let mut depth = 0; // 初始化深度为0

        while bfs.len() > 0 {
            // 开始新的一层，深度加1
            depth += 1;
            let level_size = bfs.len(); // 当前层的节点数量

            for _ in 0..level_size {
                // 遍历当前层的每个节点
                let node = bfs.pop_front().unwrap();
                let node_borrowed = node.borrow();

                // 如果左子节点存在，加入队列
                if let Some(left) = node_borrowed.left.clone() {
                    bfs.push_back(left);
                }
                // 如果右子节点存在，加入队列
                if let Some(right) = node_borrowed.right.clone() {
                    bfs.push_back(right);
                }
            }
        }

        depth // 返回树的最大深度
    }
}
```

### [balanced-binary-tree](https://leetcode-cn.com/problems/balanced-binary-tree/)

> 给定一个二叉树，判断它是否是高度平衡的二叉树。

- 思路 1：分治法，左边平衡 && 右边平衡 && 左右两边高度 <= 1，

```Python
impl Solution {
    // 检查二叉树是否平衡
    pub fn is_balanced(root: Option<Rc<RefCell<TreeNode>>>) -> bool {
        // 定义一个辅助函数来计算树的深度，同时判断是否平衡
        fn depth(root: Option<Rc<RefCell<TreeNode>>>) -> (i32, bool) {
            match root {
                None => (0, true),
                Some(node) => {
                    let node_ref = node.borrow();
                    // 递归地计算左子树的深度和是否平衡
                    let (dl, bl) = depth(node_ref.left.clone());
                    // 递归地计算右子树的深度和是否平衡
                    let (dr, br) = depth(node_ref.right.clone());

                    // 当前树的深度为左右子树深度的最大值加1
                    let depth = std::cmp::max(dl, dr) + 1;
                    // 当前树是否平衡取决于左右子树是否都平衡，以及左右子树的深度差是否不超过1
                    let balanced = bl && br && (dl - dr).abs() < 2;

                    (depth, balanced)
                }
            }
        }

        // 使用辅助函数计算根节点的深度和是否平衡
        let (_, is_balanced) = depth(root);
        // 返回整棵树是否平衡
        is_balanced
    }
}
```

Leetcode 一个相当精妙的解法[Tab Liu](https://leetcode.cn/u/tab-liu/)

自底向上的递归解法。通过递归地计算每个节点的左右子树高度，我们可以在同一过程中判断子树是否平衡。若任一子树不平衡，即其高度差超过 1，函数立即返回-1，标识不平衡，从而避免进一步无谓的计算。

```rust
use std::rc::Rc;
use std::cell::RefCell;
impl Solution {
    fn heigh(node: Rc<RefCell<TreeNode>>) -> i32 {
        // 如果节点为叶子节点，返回高度1
        if node.borrow().left.is_none() && node.borrow().right.is_none() {
            return 1;
        }

        // 递归计算左子树的高度，如果左子树不平衡，则直接返回-1
        let l = if let Some(left) = &node.borrow().left {
            Self::heigh(left.clone())
        } else {
            0 // 如果左子节点不存在，高度为0
        };
        if l == -1 { // 如果左子树不平衡，提前结束
            return -1;
        }

        // 递归计算右子树的高度，同样，如果右子树不平衡，则直接返回-1
        let r = if let Some(right) = &node.borrow().right {
            Self::heigh(Rc::clone(right))
        } else {
            0 // 如果右子节点不存在，高度为0
        };
        if r == -1 { // 如果右子树不平衡，提前结束
            return -1;
        }

        // 检查当前节点的左右子树的高度差，如果大于1，说明不平衡，返回-1
        if (l - r).abs() > 1 {
            return -1;
        }

        // 如果当前节点平衡，返回当前节点的高度，即左右子树高度的最大值加1
        l.max(r) + 1
    }

    // 检查二叉树是否平衡的公开接口
    pub fn is_balanced(root: Option<Rc<RefCell<TreeNode>>>) -> bool {
        match root {
            Some(node) => Solution::heigh(node) != -1, // 如果高度不为-1，则树平衡
            None => true, // 空树默认为平衡
        }
    }
}

```

### [binary-tree-maximum-path-sum](https://leetcode-cn.com/problems/binary-tree-maximum-path-sum/)

> 给定一个**非空**二叉树，返回其最大路径和。

- 思路：分治法。最大路径的可能情况：左子树的最大路径，右子树的最大路径，或通过根结点的最大路径。其中通过根结点的最大路径值等于以左子树根结点为端点的最大路径值加以右子树根结点为端点的最大路径值再加上根结点值，这里还要考虑有负值的情况即负值路径需要丢弃不取。

```rust
use std::rc::Rc;
use std::cell::RefCell;
use std::cmp::max;
impl Solution {
    pub fn max_path_sum(root: Option<Rc<RefCell<TreeNode>>>) -> i32 {
        // 值-2147483648 是 i32 类型可以表示的最小整数值
        let mut max_sum = -2147483648;

        // dfs函数递归地计算每个节点作为路径终点的最大路径和，并更新全局最大路径和
        fn dfs(root: &Option<Rc<RefCell<TreeNode>>>, max_sum: &mut i32) -> i32 {
            match root {
                // 如果当前节点存在
                Some(node) => {
                    let node = node.borrow(); // 获取当前节点的借用
                    let val = node.val; // 当前节点的值

                    // 递归计算左子树的最大路径和，如果为负则取0（即不包括该子树）
                    let left_max = max(0, dfs(&node.left, max_sum));
                    // 递归计算右子树的最大路径和，同样，如果为负则取0
                    let right_max = max(0, dfs(&node.right, max_sum));

                    // 更新全局最大路径和：考虑通过当前节点连接左右子树的路径和
                    *max_sum = max(*max_sum, val + left_max + right_max);

                    // 返回以当前节点为终点的最大路径和给父节点
                    val + max(left_max, right_max)
                },
                // 如果当前节点不存在，返回0，表示没有路径贡献
                None => 0,
            }
        }

        // 从根节点开始递归，计算最大路径和
        dfs(&root, &mut max_sum);
        // 返回计算得到的全局最大路径和
        max_sum
    }
}
```

- 计算以每个节点为根的最大路径和，并更新一个全局的最大路径和。
- 对于每个节点，dfs 函数计算两个重要的值：
  - 以当前节点为终点的单侧最大路径和，这用于更新父节点的路径和。
  - 包括当前节点值和左右子树的最大路径和，如果这个总和大于当前记录的 max_sum，则更新 max_sum。
- max_sum 在递归过程中作为一个引用传递，使得每次计算得到更大的路径和时，都能实时更新全局的最大值。这种设计巧妙地避免了需要额外的全局变量或类属性来存储最大路径和的值。

TODO:

### [lowest-common-ancestor-of-a-binary-tree](https://leetcode-cn.com/problems/lowest-common-ancestor-of-a-binary-tree/)

> 给定一个二叉树, 找到该树中两个指定节点的最近公共祖先。

- 思路：分治法，有左子树的公共祖先或者有右子树的公共祖先，就返回子树的祖先，否则返回根节点

```Python
class Solution:
    def lowestCommonAncestor(self, root: 'TreeNode', p: 'TreeNode', q: 'TreeNode') -> 'TreeNode':

        if root is None:
            return None

        if root == p or root == q:
            return root

        left = self.lowestCommonAncestor(root.left, p, q)
        right = self.lowestCommonAncestor(root.right, p, q)

        if left is not None and right is not None:
            return root
        elif left is not None:
            return left
        elif right is not None:
            return right
        else:
            return None
```

### BFS 层次应用

### [binary-tree-zigzag-level-order-traversal](https://leetcode-cn.com/problems/binary-tree-zigzag-level-order-traversal/)

> 给定一个二叉树，返回其节点值的锯齿形层次遍历。Z 字形遍历

- 思路：在 BFS 迭代模板上改用双端队列控制输出顺序

```Python
class Solution:
    def zigzagLevelOrder(self, root: TreeNode) -> List[List[int]]:

        levels = []
        if root is None:
            return levels

        s = collections.deque([root])

        start_from_left = True
        while len(s) > 0:
            levels.append([])
            level_size = len(s)

            if start_from_left:
                for _ in range(level_size):
                    node = s.popleft()
                    levels[-1].append(node.val)
                    if node.left is not None:
                        s.append(node.left)
                    if node.right is not None:
                        s.append(node.right)
            else:
                for _ in range(level_size):
                    node = s.pop()
                    levels[-1].append(node.val)
                    if node.right is not None:
                        s.appendleft(node.right)
                    if node.left is not None:
                        s.appendleft(node.left)

            start_from_left = not start_from_left


        return levels
```

### 二叉搜索树应用

### [validate-binary-search-tree](https://leetcode-cn.com/problems/validate-binary-search-tree/)

> 给定一个二叉树，判断其是否是一个有效的二叉搜索树。

- 思路 1：中序遍历后检查输出是否有序，缺点是如果不平衡无法提前返回结果， 代码略

- 思路 2：分治法，一个二叉树为合法的二叉搜索树当且仅当左右子树为合法二叉搜索树且根结点值大于右子树最小值小于左子树最大值。缺点是若不用迭代形式实现则无法提前返回，而迭代实现右比较复杂。

```Python
class Solution:
    def isValidBST(self, root: TreeNode) -> bool:

        if root is None: return True

        def valid_min_max(node):

            isValid = True
            if node.left is not None:
                l_isValid, l_min, l_max = valid_min_max(node.left)
                isValid = isValid and node.val > l_max
            else:
                l_isValid, l_min = True, node.val

            if node.right is not None:
                r_isValid, r_min, r_max = valid_min_max(node.right)
                isValid = isValid and node.val < r_min
            else:
                r_isValid, r_max = True, node.val


            return l_isValid and r_isValid and isValid, l_min, r_max

        return valid_min_max(root)[0]
```

- 思路 3：利用二叉搜索树的性质，根结点为左子树的右边界，右子树的左边界，使用先序遍历自顶向下更新左右子树的边界并检查是否合法，迭代版本实现简单且可以提前返回结果。

```Python
class Solution:
    def isValidBST(self, root: TreeNode) -> bool:

        if root is None:
            return True

        s = [(root, float('-inf'), float('inf'))]
        while len(s) > 0:
            node, low, up = s.pop()
            if node.left is not None:
                if node.left.val <= low or node.left.val >= node.val:
                    return False
                s.append((node.left, low, node.val))
            if node.right is not None:
                if node.right.val <= node.val or node.right.val >= up:
                    return False
                s.append((node.right, node.val, up))
        return True
```

#### [insert-into-a-binary-search-tree](https://leetcode-cn.com/problems/insert-into-a-binary-search-tree/)

> 给定二叉搜索树（BST）的根节点和要插入树中的值，将值插入二叉搜索树。 返回插入后二叉搜索树的根节点。

- 思路：如果只是为了完成任务则找到最后一个叶子节点满足插入条件即可。但此题深挖可以涉及到如何插入并维持平衡二叉搜索树的问题，并不适合初学者。

```Python
class Solution:
    def insertIntoBST(self, root: TreeNode, val: int) -> TreeNode:

        if root is None:
            return TreeNode(val)

        node = root
        while True:
            if val > node.val:
                if node.right is None:
                    node.right = TreeNode(val)
                    return root
                else:
                    node = node.right
            else:
                if node.left is None:
                    node.left = TreeNode(val)
                    return root
                else:
                    node = node.left
```

## 总结

- 掌握二叉树递归与非递归遍历
- 理解 DFS 前序遍历与分治法
- 理解 BFS 层次遍历

## 练习

- [x] [maximum-depth-of-binary-tree](https://leetcode-cn.com/problems/maximum-depth-of-binary-tree/)
- [x] [balanced-binary-tree](https://leetcode-cn.com/problems/balanced-binary-tree/)
- [x] [binary-tree-maximum-path-sum](https://leetcode-cn.com/problems/binary-tree-maximum-path-sum/)
- [ ] [lowest-common-ancestor-of-a-binary-tree](https://leetcode-cn.com/problems/lowest-common-ancestor-of-a-binary-tree/)
- [ ] [binary-tree-level-order-traversal](https://leetcode-cn.com/problems/binary-tree-level-order-traversal/)
- [ ] [binary-tree-level-order-traversal-ii](https://leetcode-cn.com/problems/binary-tree-level-order-traversal-ii/)
- [ ] [binary-tree-zigzag-level-order-traversal](https://leetcode-cn.com/problems/binary-tree-zigzag-level-order-traversal/)
- [ ] [validate-binary-search-tree](https://leetcode-cn.com/problems/validate-binary-search-tree/)
- [ ] [insert-into-a-binary-search-tree](https://leetcode-cn.com/problems/insert-into-a-binary-search-tree/)
