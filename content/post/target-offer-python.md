---
title: 剑指Offer——Python 题解
date: 2019-03-28T20:40:22+08:00
permalink:
tags: ["剑指Offer", "Python"]
categories: ["算法"]
---
剑指 Offer 题目的 Python 题解。

<!--more-->

## 3. 二维数组中的查找

Q: 在一个二维数组中，每一行都按照从左到右递增的顺序排序，每一列都按照从上到下递增的顺序排序。请完成一个函数，输入这样的一个二维数组和一个整数，判断数组中是否含有该整数。

A: 首先选取数组中右上角的数字。如果该数字等于要查找的数字，查找过程结束；如果该数字大于要查找的数字，就删除这个数字所在的列（左移一位）；如果该数字小于要查找的数字，就删除这个数字所在的行（下移一位）。

```python
class Solution:

    def find(self, arr, target):
        if not arr:
            return False

        row = len(arr)
        col = len(arr[0])

        i, j = col - 1, 0
        while i >= 0 and j < row:
            if arr[i][j] == target:
                return True
            elif arr[i][j] > target:
                i -= 1
            else:
                j += 1

        return False
```

## 4. 替换空格

Q: 实现一个函数，把字符串中的每个空格替换成"%20"。例如输入 we are happy，输出 we%20are%20happy。

A: 先计算最终需要给出的长度，然后建立两个指针p1,p2，p1指向原始字符串的末尾，p2指向替换后的字符串的末尾。同时移动p1,p2, 将p1指的内容逐个复制到p2,当p1遇到空格时，在p2处插入%20，p1向前移动一个位置，p2向前移动3个位置，当p1和p2位置重合时，全部替换完成。

```python
class Solution:

    def replace_blank(self, s):
        """
        先计算最终需要给出的长度，然后建立两个指针p1,p2，p1指向原始字符串的末尾，
        p2指向替换后的字符串的末尾。同时移动p1,p2, 将p1指的内容逐个复制到p2,
        当p1遇到空格时，在p2处插入%20，p1向前移动一个位置，p2向前移动3个位置，
        当p1和p2位置重合时，全部替换完成。
        """
        assert s and isinstance(s, str) is True

        blank_num = len([1 for i in s if i == ' '])
        new_string_length = len(s) + 2 * blank_num
        new_str = new_string_length * [None]
        index_origin, index_new = len(s) - 1, new_string_length - 1
        while index_new >= 0 and index_new >= index_origin:
            if s[index_origin] != ' ':
                new_str[index_new] = s[index_origin]
                index_new -= 1
                index_origin -= 1
            else:
                new_str[index_new - 2:index_new + 1] = '%20'
                index_new -= 3
                index_origin -= 1

        return ''.join(new_str)
```

## 5. 反向打印链表

Q: 输入一个链表的头结点，从尾到头反过来打印出每个结点的值。

A: 利用栈的结构特点。

```python
from collections import deque


class Node:

    def __init__(self, x=None):
        self.val = x
        self.next = None


class Solution:

    def print_reverse(self, head):
        d = deque()
        while head:
            d.appendleft(head.val)
            head = head.next

        return d
```

## 6. 重建二叉树

Q: 输入某二叉树的前序遍历和中序遍历的结果，请重建出该二叉树。假设输入的前序遍历和中序遍历的结果中都不含重复的数字。例如输入前序遍历序列{1,2,4,7,3,5,6,8}和中序遍历序列{4,7,2,1,5,3,8,6}，则重建二叉树并返回。

A: 根据前序遍历和中序遍历的特点，前序遍历的第一个结点肯定是根结点；在中序遍历中，根结点在中间，左子树的结点在根结点的左边，右子树的结点在根结点的右边。然后我们可以通过递归的方式，分别对左子树和右子树进行同样的过程。

```python
class TreeNode:

    def __init__(self, x=None):
        self.val = x
        self.left = None
        self.right = None


class Solution:

    def reconstruct_binary_tree(self, pre_order, in_order):
        assert set(pre_order) == set(in_order)
        if not pre_order or not in_order:
            return

        root = TreeNode(pre_order[0])
        index = in_order.index(pre_order[0])
        root.left = self.reconstruct_binary_tree(pre_order[1:index + 1],
                                                 in_order[:index])
        root.right = self.reconstruct_binary_tree(pre_order[index + 1:],
                                                  in_order[index + 1:])
        return root


s = Solution()
pre_order = [1, 2, 4, 7, 3, 5, 6, 8]
in_order = [4, 7, 2, 1, 5, 3, 8, 6]
tree = s.reconstruct_binary_tree(pre_order, in_order)
```

## 7. 用两个栈实现队列

Q: 用两个栈实现一个队列。

A: 两个栈 stack1 和 stack2，push 的时候直接往 stack1 中 push，pop 的时候需要分情况。如果 stack2 为空，就把 stack1 中的元素逐个 push 到 stack2 中，然后进行 pop 操作；如果 stack2 不为空，直接 pop。

```python
class Solution:
    def __init__(self):
        self.stack1 = []
        self.stack2 = []

    def push(self, x):
        self.stack1.append(x)

    def pop(self):
        if not self.stack2:
            self.stack2 = self.stack1[::-1]
        return self.stack2.pop()
```

## 8. 旋转数组的最小数字

Q: 把一个数组最开始的若干个元素搬到数组末尾，称之为数组的旋转。输入一个递增排序的数组的旋转，输出旋转数组的最小元素。例如数组{3,4,5,1,2}为{1,2,3,4,5}的一个旋转，该数组的最小值为1。

A: 旋转数组的首元素肯定不小于最后一个元素，找一个中间点，如果中间点元素比首元素大，说明最小元素在中间点的后面；如果中间点元素比尾元素小，说明最小元素在中间点前面。然后循环。但是在一次循环中，首元素小于尾元素，说明该数组是排序的，首元素就是最小数字，如果出现首元素、尾元素、中间值三者相等，则只能在此区域中顺序查找。

```python
class Solution:

    def minNumberInRotateArray(self, rotateArray):
        if len(rotateArray) == 0:
            return 0
        front, rear = 0, len(rotateArray) - 1
        midIndex = 0
        while rotateArray[front] >= rotateArray[rear]:
            if rear - front == 1:
                midIndex = rear
                break
            midIndex = (front + rear) // 2
            # 如果中间和左边、右边数值都相等，需要顺序查找
            if rotateArray[front] == rotateArray[rear] and rotateArray[
                front] == rotateArray[midIndex]:
                return self.MinInOrder(rotateArray, front, rear)
            # 如果中间元素大于最左边元素，将左边指针移动到中间；如果中间元素小于最右边元素，就将右边指针移到中间
            if rotateArray[midIndex] >= rotateArray[front]:
                front = midIndex
            elif rotateArray[midIndex] <= rotateArray[rear]:
                rear = midIndex
        return rotateArray[midIndex]

    def MinInOrder(self, array, front, end):
        result = array[0]
        for i in range(front, end + 1):
            if array[i] < result:
                result = array[i]
        return result
```

## 9 斐波那契数列

Q: 输入一个整数 n，请你输出斐波那契数列的第 n 项。
相关问题：

* 一只青蛙一次可以跳上1级台阶，也可以跳上2级。求该青蛙跳上一个n级的台阶总共有多少种跳法。
* 一只青蛙一次可以跳上1级台阶，也可以跳上2级……它也可以跳上n级。求该青蛙跳上一个n级的台阶总共有多少种跳法。
* 我们可以用2x1的小矩形横着或者竖着去覆盖更大的矩形。请问用 n 个2x1的小矩形无重叠地覆盖一个2xn的大矩形，总共有多少种方法？

```python
class Solution:

    def Fibonacci(self, n):
        a, b = 0, 1
        count = 2
        if n == 0:
            return a
        elif n == 1:
            return b
        else:
            while count <= n:
                a, b = b, a + b
                count += 1
        return a

    def jumpFloor(self, number):
        """青蛙跳台阶, 每次可以跳1级或2级"""
        arr = [1, 2]
        if number >= 3:
            for i in range(3, number + 1):
                arr[(i + 1) % 2] = arr[0] + arr[1]
        return arr[(number + 1) % 2]

    def jumpFloorII(self, number):
        ans = 1
        if number >= 2:
            for i in range(number - 1):
                ans = ans * 2
        return ans
    
    def rectCover(self, number):
        if number == 0:
            return 0
        arr = [1, 2]
        if number >= 3:
            for i in range(3, number + 1):
                arr[(i + 1) % 2] = arr[0] + arr[1]
        return arr[(number + 1) % 2]
```

## 10. 二进制中 1 的个数

Q: 请实现一个函数，输入一个整数，输出该数二进制表示中 1 的个数，例如把 9 表示成二进制是 1001， 有 2 位是 1。因此输入 9，该函数输出 2。

A: 非零整数 n 和 n-1 进行按位与运算，得到的结果相当于把整数 n 的二进制表示中最右边的一个 1 变成 0。**负数的补码等于原码取反加一；补码转原码等于减一取反**。

```python
class Solution:

    def number_of_1(self, num):
        count = 0
        if num < 0:
            num = num & 0xffffffff
        while num:
            count += 1
            num = (num - 1) & num
        return count
```

## 11. 数值的整数次方

Q: 给定一个double类型的浮点数base和int类型的整数exponent。求base的exponent次方。

A: 根据公式 $$ a^n = a^{n/2} \times a^{n/2} , \quad n为偶数 \\[2ex] a^n=a^{(n-1)/2} \times a^{(n-1)2}, \quad n为奇数 $$，利用位与运算代替了求余运算法%和乘除法来判断一个数是奇数还是偶数。

```python
class Solution:
    
    def power(self, base, exponent):
        """
        :param base: 底数
        :param exponent: 指数
        :return: 
        """

        def equal(num1, num2):
            if (num1 - num2 < -0.0000001) and (num1 - num2 > 0.0000001):
                return True
            else:
                return False

        if equal(base, 0.0) and exponent < 0:
            return 0.0

        abs_exponent = abs(exponent)

        def power_with_exponent(base, exponent):
            if exponent == 0:
                return 1
            if exponent == 1:
                return base

            result = power_with_exponent(base, exponent >> 1)
            result *= result
            if (exponent & 0x1) == 1:
                result *= base
            return result

        result = power_with_exponent(base, abs_exponent)
        if exponent < 0:
            return 1.0 / result
        return result
```

## 12. 打印 1 到最大的 n 位数

Q: 输入数字n, 按顺序打印从1最大的n位十进制数。比如输入3, 则打印出 1、2、3 到最大的3位数即999。

A: n 位所有十进制数就是 n 个从 0 到 9 的全排列。利用递归就能解决。

```python
class Solution:

    def Print1ToMaxOfNDigits_0(self, n):
        if n <= 0:
            return

        number = ['0'] * n
        while not self.Increment(number):
            self.PrintNumber(number)

    def Increment(self, number):
        is_overflow = False
        nTakeOver = 0
        nLength = len(number)

        for i in range(nLength - 1, -1, -1):
            nSum = int(number[i]) + nTakeOver
            if i == nLength - 1:
                nSum += 1

            if nSum >= 10:
                if i == 0:
                    is_overflow = True
                else:
                    nSum -= 10
                    nTakeOver = 1
                    number[i] = str(nSum)
            else:
                number[i] = str(nSum)
                break

        return is_overflow

    def Print1ToMaxOfNDigits(self, n):
        if n <= 0:
            return

        numbers = ['0'] * n
        for i in range(10):
            numbers[0] = str(i)
            self.Print1ToMaxOfNDigitsRecursively(numbers, n, 0)

    def Print1ToMaxOfNDigitsRecursively(self, number, length, index):
        if index == length - 1:
            self.PrintNumber(number)
            return
        for i in range(10):
            number[index + 1] = str(i)
            self.Print1ToMaxOfNDigitsRecursively(number, length, index + 1)

    def PrintNumber(self, number):
        is_beginning_zero = True
        nLength = len(number)

        for i in range(nLength):
            if is_beginning_zero and number[i] != '0':
                is_beginning_zero = False
            if not is_beginning_zero:
                print('%s' % number[i], end='')
        print()
```

## 13. 在 O(1) 时间内删除链表的结点

Q: 给定单向链表的头指针和一个结点指针,定义一个函数在O(1)时间删除该结点

A: 将删除结点的下一个结点的内容复制到需要删除结点上，然后把下一个结点删除。如果删除的结点位于链表尾部，仍然需要从链表头开始遍历进行删除操作；如果是头结点且链表只有一个结点，那么还需要同时把链表头结点设为 None。

```python
class ListNode:
    def __init__(self, x=None):
        self.val = x
        self.next = None

    def __del__(self):
        self.val = None
        self.next = None


class Solution:

    def DeleteNode(self, pListHead, pToBeDeleted):
        if not pListHead or not pToBeDeleted:
            return

        if pToBeDeleted.next is not None:  # 如果删除的结点不是尾结点
            pNext = pToBeDeleted.next
            pToBeDeleted.val = pNext.val
            pToBeDeleted.next = pNext.next
            pNext.__del__()
        elif pListHead == pToBeDeleted:  # 如果是头结点
            pToBeDeleted.__del__()
            pListHead.__del__()
        else:  # 尾结点
            pNode = pListHead
            while pNode.next != pToBeDeleted:
                pNode = pNode.next
            pNode.next = None
            pToBeDeleted.__del__()
```

## 14. 调整数组顺序使奇数位于偶数前面

Q: 输入一个整数数组，实现一个函数来调整该数组中数字的顺序，使得所有的奇数位于数组的前半部分，所有的偶数位于数组的后半部分。

```python
class Solution:

    def ReorderOddEven(self, arr):
        if not arr or len(arr) <= 1:
            return arr

        start, end = 0, len(arr) - 1
        while start <= end:
            while arr[start] & 1 == 1:
                start += 1
            while arr[end] & 1 == 0:
                end -= 1
            arr[start], arr[end] = arr[end], arr[start]
        arr[start], arr[end] = arr[end], arr[start]
        return arr
```

## 15. 链表中倒数第 k 个结点

Q: 输入一个链表，输出该链表中倒数第 k 个结点。

A: 设置快、慢两个指针，先让快指针移动 k-1 步，然后快、慢指针同时移动。当快指针移动到链表尾部时，慢指针所在位置就是倒数第 k 个结点。

```python
class Solution:
    def find_last_k_node(self, head, k):
        if not head or k <= 0:
            return

        slow, fast = head, head
        for _ in range(k - 1):
            if fast.next is not None:
                fast = fast.next
            else:
                return

        while fast.next:
            slow = slow.next
            fast = fast.next
        return slow
```

## 16. 反转链表

Q: 定义一个函数，输入一个链表的头结点，反转该链表并输出反转后链表的头结点。

```python
class ListNode:
    def __init__(self, x):
        self.val = x
        self.next = None

    def __str__(self):
        return str(self.val)


class Solution:
    # 返回ListNode
    def reverse_list(self, head):
        if not head or not head.next:
            return head
        pre = None
        while head:
            tmp = head.next  # 缓存当前节点的后一个节点
            head.next = pre  # 把当前节点的向前指针当成向后指针
            pre = head
            head = tmp
        return pre
    
    def reverse_recursion(self, head):
        if not head or not head.next:
            return head

        new_head = self.reverse_recursion(head.next)
        head.next.next = head
        head.next = None
        return new_head
```

## 17. 合并两个排序链表

Q: 输入两个递增排序的链表，合并这两个链表并使新链表中的结点仍然是递增排序的。

A: 每个链表建立一个指针，合并时，比较头节点大小，小的作为合并后链表的头节点，再比较剩余部分和另一个链表的头节点，取小的，然后一直循环此过程。

```python
class Solution:

    def merge(self, head1, head2):

        if not head1 and not head2:
            return
        elif not head1:
            return head2
        elif not head2:
            return head1

        if head1.val <= head2.val:
            merge_head = head1
            merge_head.next = self.merge(head1.next, head2)
        else:
            merge_head = head2
            merge_head.next = self.merge(head1, head2.next)
        return merge_head
```

## 18. 树的子结构

Q: 输入两个二叉树 A 和 B，判断 B 是不是 A 的子结构（空树不是任意一个树的子结构）

A: 在树 A 中查找和 B 根结点一致的值，然后判断树 A 中以该结点为根结点的子树，是不是和树 B 有相同的结构。

```python
class TreeNode:

    def __init__(self, x):
        self.val = x
        self.left = None
        self.right = None


class Solution:

    def HasSubtree(self, pRoot1, pRoot2):
        result = False
        if pRoot1 and pRoot2:
            if pRoot1.val == pRoot2.val:
                result = self.DoesTree1HaveTree2(pRoot1, pRoot2)
            if not result:
                result = self.HasSubtree(pRoot1.left, pRoot2)
            if not result:
                result = self.HasSubtree(pRoot1.right, pRoot2)
        return result

    def DoesTree1HaveTree2(self, pRoot1, pRoot2):
        if pRoot2 is None:
            return True
        if pRoot1 is None:
            return False
        if pRoot1.val != pRoot2.val:
            return False

        return self.DoesTree1HaveTree2(pRoot1.left, pRoot2.left) and self.DoesTree1HaveTree2(pRoot1.right, pRoot2.right)
```

## 19. 二叉树的镜像

Q: 输入一个二叉树，该函数输出它的镜像

A: 先前序遍历这棵树的每个节点，如果遍历到的节点有子节点，就交换它的子节点，当交换完所有非叶节点的左右子节点后，得到树的镜像。

```python
class Solution:

    def MirrorRecursively(self, root):
        if not root:
            return
        if not root.left and not root.right:
            return

        root.left, root.right = root.right, root.left

        self.MirrorRecursively(root.left)
        self.MirrorRecursively(root.right)
```

## 20. 顺时针打印矩阵

Q: 输入一个矩阵，按照从外向里以顺时针的顺序依次打印出每一个数字。例如，如果输入如下矩阵：`[[ 1,  2,  3,  4], [ 5,  6,  7,  8], [ 9, 10, 11, 12], [13, 14, 15, 16]]`，则依次打印出数字 1,2,3,4,8,12,16,15,14,13,9,5,6,7,11,10。

A: 

把矩阵看成由若干个顺时针方向的圈组成，每一圈按照从左向右，从上到下，从右向左，从下到上的顺序打印。

(1) 循环条件

columns > start * 2 and rows > start * 2

(2) 打印每一条边的前提条件

从左向右：子矩阵至少有一个元素
从上到下：子矩阵至少有两行，start < endY
从右向左：子矩阵至少两行两列，start < endY and start < endX
从下到上：子矩阵至少三行两列，start < endY - 1 and start < endX

(3) 确定每一条边的元素

已知 start 后，可以确定每个圈的 endX 和 endY。
endX = columns - 1 - start
endY = rows - 1 - start

从左到右：`[start, endX]`
从上到下：`[start+1, endY]`
从右到左：`[endX-1, start]`
从下到上：`[endY-1, start+1]`

```python
class Solution:
    # matrix类型为二维列表，需要返回列表
    def printNumber(self, number):
        print(number, '', end='')

    def printMatrix(self, matrix):
        if not isinstance(matrix, list):
            return
        rows = len(matrix)
        columns = len(matrix[0])
        start = 0
        while rows > start * 2 and columns > start * 2:
            self.PrintMatrixInCircle(matrix, columns, rows, start)
            start += 1
        print('')

    def PrintMatrixInCircle(self, matrix, columns, rows, start):
        endX = columns - 1 - start
        endY = rows - 1 - start

        # 从左到右打印一行
        for i in range(start, endX + 1):
            number = matrix[start][i]
            self.printNumber(number)

        # 从上到下打印一行，终止行号大于起始行号
        if start < endY:
            for i in range(start + 1, endY + 1):
                number = matrix[i][endX]
                self.printNumber(number)

        # 从右到左打印一行，终止行号大于起始行号 && 终止列号大于起始列号
        if start < endX and start < endY:
            for i in range(endX - 1, start - 1, -1):
                number = matrix[endY][i]
                self.printNumber(number)

        # 从下到上打印一行
        if start < endX and start < endY - 1:
            for i in range(endY - 1, start, -1):
                number = matrix[i][start]
                self.printNumber(number)
```

## 21. 包含 min 函数的栈

Q: 定义栈的数据结构，请在该类型中实现一个能够得到栈的最小元素的 min 函数。在该栈中，调用 min、push 及 pop 的时间复杂度都是 O(1)。

A: 利用辅助栈，把每次最小的元素（之前最小元素和新压入栈的元素两者中较小的值）

```python
class Solution:
    def __init__(self):
        self.stack = []
        self.minStack = []

    def push(self, v):
        self.stack.append(v)
        if self.minStack == [] or v < self.min():
            self.minStack.append(v)
        else:
            tmp = self.min()
            self.minStack.append(tmp)

    def pop(self):
        if self.stack == [] or self.minStack == []:
            return
        self.stack.pop()
        self.minStack.pop()

    def min(self):
        return self.minStack[-1]
```

## 22. 栈的压入、弹出队列

Q: 
> 输入两个整数序列，第一个序列表示栈的压入顺序，请判断第二个序列是否为该栈的弹出顺序。
> 假设压入栈的所有数字均不相等。例如序列1,2,3,4,5是某栈的压入顺序，序列4,5,3,2,1是该压栈序
> 列对应的一个弹出序列，但4,3,5,1,2就不可能是该压栈序列的弹出序列。
>（注意：这两个序列的长度是相等的）

A: 如果下一个弹出的数字刚好是栈顶元素，那么直接弹出。如果下一个弹出的元素不在栈顶，我们把压栈序列中还没有入栈的数字压入辅助栈，直到把下一个需要弹出的数字压入栈顶为止。如果所有数字都压入栈了仍然没有找到下一个弹出的数字，那么该序列不可能是一个弹出序列。

```python
class Solution:
    def isPopOrder(self, pushOrder, popOrder):
        if pushOrder == [] or popOrder == []:
            return False

        stack = []
        for v in pushOrder:
            stack.append(v)
            while len(stack) > 0 and stack[-1] == popOrder[0]:
                stack.pop()
                popOrder.pop(0)

        if len(stack) == 0:
            return True
        else:
            return False
```

## 23. 从上往下打印二叉树

Q: 从上往下打印出二叉树的每个节点，同层节点从左至右打印。

A: 从上到下打印二叉树的规律：每一次打印一个结点的时候，如果该结点有子结点，则把该结点的子结点放到
一个队列的末尾。接下来到队列的头部取出最早进入队列的结点，重复前面的打印操作，直至队列中所有的
结点都被打印出来为止。从本质上来说就是广度遍历二叉树。

```python
class Solution:
    def printFromTopToBottom(self, root):
        tmp = []
        result = []
        if not root:
            return result

        tmp.append(root)
        while len(tmp) > 0:
            current = tmp.pop(0)
            result.append(current.value)
            if current.left:
                tmp.append(current.left)
            if current.right:
                tmp.append(current.right)

        return result
```

## 24. 二叉搜索树的后序遍历序列

Q: 输入一个整数数组，判断该数组是不是某二叉搜索树的后序遍历的结果。如果是则返回 true，否则返回 false。假设输入的数组的任意两个数字都互不相同。二叉搜索树对于每一个非叶子节点 n, 其左子树下的每个后代节点的值都小于结点 n 的值。其右子树下的每个后代节点的值都大于结点 n 的值。

A: 根据后序遍历的特点，尾元素一定是根节点，同时小于尾元素的值是左子树，大于尾元素的值是右子树。且序列前半部分小于尾元素，后半部分大于尾元素，可以将序列划分为左子树序列和右子树序列，然后递归。

```python
class Solution:
    def verifySquenceOfBST(self, sequence):
        if not sequence:
            return False

        root = sequence[-1]
        length = len(sequence)
        if length > 1 and (min(sequence[:-1]) > root or max(sequence[:-1]) < root):
            return True

        index = 0
        # 在二叉搜索树中左子树的的结点小于根结点
        for i in range(length-1):
            index = i
            if sequence[i] > root:
                break

        # 在二叉搜索树中右子树的结点大于根结点
        for j in range(index, length - 1):
            if sequence[j] < root:
                return False

        # 判断左子树是不是二叉搜索树
        left = True
        if index > 0:
            left = self.verifySquenceOfBST(sequence[:index])

        # 判断右子树是不是二叉搜索树
        right = True
        if index < length - 1:
            right = self.verifySquenceOfBST(sequence[index:length-1])
        return left and right
```

## 25. 二叉树中和为某一值的路径

Q: 输入一颗二叉树和一个整数，打印出二叉树中结点值的和为输入整数的所有路径。路径定义为从树的根结点开始往下一直到叶结点所经过的结点形成一条路径。

A: 用前序遍历的方式访问二叉树的节点，当访问到一个节点时，将该节点加到路径中，并累加节点的值。直到访问到符合要求的节点或者访问到叶节点，然后递归访问该节点的父节点，在函数退出时要删除当前节点，并减去当前节点的值。

```python
class Solution:
    def findPath(self, root, expected_sum):
        if not root:
            return []

        if root.left is None and root.right is None:
            if root.value == expected_sum:
                return [[root.value]]
            else:
                return []

        tmp = self.findPath(root.left, expected_sum - root.value) + self.findPath(root.right, expected_sum - root.value)
        return [[root.value] + v for v in tmp]
```

## 26. 复杂链表的复制

Q: 输入一个复杂链表（每个节点中有节点值，以及两个指针，一个指向下一个节点，另一个特殊指针指向任意一个节点），返回结果为复制后复杂链表的head。

A: 
* 第一步：复制原始链表中的每个结点，将复制的结点到每个结点的后面。
* 第二步：如果原始链表中的每个结点的 sibling 指向 s，那么对应的复制结点的 sibling 指向 s 的
下一个结点。
* 第三步：把第二步得到的链表拆分成两个，奇数位置的结点组成原链表，偶数为止的结点组成复制出来的
链表。

```python
class ComplexListNode:
    def __init__(self, name=None):
        self.name = name
        self.next = None
        self.sibling = None


class Solution:
    def clone(self, head):
        if not head:
            return

        self.cloneNodes(head)
        self.connectSiblingNodes(head)
        return self.reconnectNodes(head)

    def cloneNodes(self, head):
        """第一步：复制原始链表中的每个结点，将复制的结点到每个结点的后面"""
        node = head
        while node:
            cloneNode = ComplexListNode()
            cloneNode.name = node.name
            cloneNode.next = node.next
            node.next = cloneNode
            node = cloneNode.next

    def connectSiblingNodes(self, head):
        """
        第二步：如果原始链表中的每个结点的 sibling 指向 s，那么对应的复制结点的 sibling
        指向 s 的下一个结点
        """
        node = head
        while node:
            cloneNode = node.next
            if node.sibling is not None:
                cloneNode.sibling = node.sibling.next
            node = cloneNode.next

    def reconnectNodes(self, head):
        """
        第三步：把第二步得到的链表拆分成两个，奇数位置的结点组成原链表，偶数为止的结点组成
        复制出来的链表
        """
        node = head
        cloneNodeHead = cloneNode = node.next
        node.next = cloneNodeHead.next
        node = node.next

        while node:
            cloneNode.next = node.next
            cloneNode = cloneNode.next
            node.next = cloneNode.next
            node = node.next

        return cloneNodeHead
```

## 27. 二叉搜索树与双向链表

Q: 输入一棵二叉搜索树，将该二叉搜索树转换成一个排序的双向链表。要求不能创建任何新的结点，只能调整树中结点指针的指向。

A: 根结点、左子树和右子树。在把左、右子树都转换成排序的双向链表之后在和根结点链接起来，整棵二叉搜
索树也就转换成了排序的双向链表。可以使用中序遍历算法。由于遍历和转换过程是一样的，因此可以采用
递归。

```python
class TreeNode:
    def __init__(self, v):
        self.val = v
        self.left = None
        self.right = None


class Solution:
    def convert(self, root):
        if root is None:
            return
        if root.left is None and root.right is None:
            return root

        self.convert(root.left)
        left = root.left

        # 连接根和左子树的最大结点
        if left:
            while left.right:
                left = left.right
            root.left, left.right = left, root

        self.convert(root.right)
        right = root.right

        # 连接根和右子树的最小结点
        if right:
            while right.left:
                right = right.left
            root.right, right.left = right, root

        # 获取双向链表的头结点
        while root.left:
            root = root.left

        return root
```

## 28. 字符串的排列

Q: 输入一个字符串，打印出该字符串中字符的所有排列。例如输入字符串 abc，则打印出 abc、acb、bac、bca、cab 和 cba。

A: 固定第一个元素，求出后面所有字符的排列，重新固定第一个元素，再求后面字符的排列，依次换第一个元素，典型递归问题。

```python
class Solution:
    def permutation(self, s):
        if len(s) <= 1:
            return list(s)

        str_list = []
        for i in range(len(s)):
            for j in self.permutation(s[:i] + s[i+1:]):
                str_list.append(s[i] + j)
        return str_list

    def group(self, s):
        """组合"""
        if len(s) <= 1:
            return list(s)

        str_list = []
        for i in range(len(s)):
            str_list.append(s[i])
            for j in self.group(s[i+1:]):
                str_list.append(s[i] + j)
        return str_list
```

## 29. 数组中出现次数超过一半的数字

Q: 数组中有一个数字出现的次数超过数组长度的一半，请找出这个数字。例如输入一个长度为9的数组
{1,2,3,2,2,2,5,4,2}。由于数字2在数组中出现了5次，超过数组长度的一半，因此输出2。

A: 第一种思路：出现次数超过一半的数字，一定位于数组中间的位置，找到数组中间位置的数字，然后再顺序检索这个数字的出现次数是否超过一半；第二种思路：出现次数超过一半的数，它的出现次数比其他所有数字出现次数的总和还要多，保存两个值，数组中的数字和它的出现次数。如果下一个数字等于该数字，那么出现次数加一，如果不相等，次数减一，当次数为0时，保存下一个数字，并重置出现次数为1，我们要找的数字就是最后一次把次数重置为1的时候，保存的数字。最后要检查得到的元素出现次数是否超过一半。

```python
class Solution:
    def MoreThanHalfNum(self, numbers):
        """基于Partition函数的O(n)算法"""
        length = len(numbers)
        if length == 1:
            return numbers[0]

        mid = length >> 1  # 中位数
        start = 0
        end = length - 1
        index = self.Partition(numbers, length, start, end)
        while index != mid:
            if index > mid:
                end = index - 1
                index = self.Partition(numbers, length, start, end)
            else:
                start = index + 1
                index = self.Partition(numbers, length, start, end)
        result = numbers[mid]
        if not self.CheckMoreThanHalf(numbers, length, result):
            result = 0
        return result

    def Partition(self, numbers, length, start, end):
        if not numbers or start < 0 or end >= length:
            return
        if end == start:
            return end
        pivot = numbers[start]
        left = start + 1
        right = end

        while left < right:
            while numbers[left] <= pivot and left < right:
                left += 1
            while numbers[right] >= pivot and left < right:
                right -= 1

            numbers[left], numbers[right] = numbers[right], numbers[left]
        numbers[right], numbers[start] = numbers[start], numbers[right]
        return right

    def CheckMoreThanHalf(self, numbers, length, number):
        """检查查找到中位数的元素出现次数是否超过所有元素数量的一半"""
        times = 0
        for i in range(length):
            if numbers[i] == number:
                times += 1

        return True if times * 2 > length else False

    def MoreThanHalfNum_2(self, numbers):
        """根据数组特点找出O(n)的算法"""
        if not numbers:
            return 0

        length = len(numbers)
        result = numbers[0]
        times = 1
        for i in range(1, length):
            if times == 0:
                result = numbers[i]
                times = 1
            elif numbers[i] == result:
                times += 1
            else:
                times -= 1
        if not self.CheckMoreThanHalf(numbers, length, result):
            result = 0
        return result
```

## 30. 最小的 k 个数

Q: 输入 n 个整数，找出其中最小的K个数。例如输入4,5,1,6,2,7,3,8这8个数字，则最小的4个数字是1,2,3,4。

A: 第一种思路：基于划分的方法，使比第k个数字小的数都位于数组的左边，大的位于数组右边。第二种思路：基于二叉树或堆。首先把前k个数字构建一个堆，从第k+1个数字开始遍历，遍历到的元素如果小于堆的最大元素，则交换，继续遍历到结束，最后剩下的就是最小的k个数。

```python
class Solution:

    def GetLeastNumbers(self, arr, k):
        """O(n)"""
        if not arr or k > len(arr):
            return []
        res = self.quick_sort(arr)
        return res[:k]

    def quick_sort(self, arr):
        if len(arr) < 2:
            return arr
        else:
            mid = arr[0]
            left = [i for i in arr[1:] if i <= mid]
            right = [i for i in arr[1:] if i > mid]
            final = self.quick_sort(left) + [mid] + self.quick_sort(right)
            return final

    def GetLeastNumbers_2(self, arr, k):
        """O(nlogk)，适合处理海量数据"""
        import heapq
        if not arr or k > len(arr):
            return []
        result = []
        for item in arr:
            if len(result) < k:
                result.append(item)
            else:
                result = heapq.nlargest(k, result)
                if item < result[0]:
                    result[0] = item
        return result[::-1]
```

## 31. 连续子数组的最大和

Q: 输入一个整形数组，数组里有正数也有负数。数组中一个或连续的多个整数组成一个子数组。求所有子数组的和的最大值。要求时间复杂度为 O(n)。例如输入 [1,-2,3,10,-4,7,2,-5]，和最大的子数组为 [3,10,-4,7,2]，最大和为 18。

A: 对于连续子数组，可以用一个数值来存储当前的和，如果当前和小于0，那么在进行到下一个元素时，直接把当前和赋值为下一个元素，如果当前和大于0，则累加下一个元素，同时用一个列表存储最大值并随时更新。（动态规划思想）

```python
class Solution:

    def FindGreatestSumOfSubArray(self, arr):
        if not arr:
            return

        cur_sum, great_sum = 0, arr[0]
        for idx, value in enumerate(arr):
            if cur_sum <= 0:
                cur_sum = value
            else:
                cur_sum += value
            if cur_sum > great_sum:
                great_sum = cur_sum
        return great_sum

    def FindGreatestSumOfSubArray_2(self, arr):
        """动态规划"""
        if not arr:
            return
        result = [0] * len(arr)
        for i in range(len(arr)):
            if i == 0 or result[i - 1] <= 0:
                result[i] = arr[i]
            else:
                result[i] = result[i - 1] + arr[i]
        return max(result)
```

## 32. 从 1 到 n 整数中 1 出现的次数

Q: 输入一个整数 n，求从 1 到 n 这 n 个整数的十进制表示中 1 出现的次数。例如输入 12，从 1 到 12 这些整数中包含 1 的数字有 1，10，11 和 12，1 一共出现了 5 次。

A: 

方法一：累加1-n中每个整数中1出现的次数，每次通过对10取余，判断整数的个位是否为1，如果数字大于10，则除以10以后，再通过取余计算判断各位是否为1。

方法二：设定整数点（如1、10、100等等）作为位置点m（对应n的各位、十位、百位等等），分别对每个数位上有多少包含1的点进行分析。

* 根据设定的整数位置，对n进行分割，分为两部分，高位n//m，低位n%m
* 当i表示百位，且百位对应的数>=2,如n=31456,m=100，则a=314,b=56，此时百位为1的次数有a/10+1=32（最高两位0~31），每一次都包含100个连续的点，即共有(a/10+1)*100个点的百位为1
* 当i表示百位，且百位对应的数为1，如n=31156,m=100，则a=311,b=56，此时百位对应的就是1，则共有a/10(最高两位0-30)次是包含100个连续点，当最高两位为31（即a=311），本次只对应局部点00~56，共b+1次，所有点加起来共有（a/10*100）+(b+1)，这些点百位对应为1
* 当i表示百位，且百位对应的数为0,如n=31056,m=100，则a=310,b=56，此时百位为1的次数有a/10=31（最高两位0~30）
* 综合以上三种情况，当百位对应0或>=2时，有(a+8)/10次包含所有100个点，还有当百位为1(a%10==1)，需要增加局部点b+1
* 之所以补8，是因为当百位为0，则a/10==(a+8)/10，当百位>=2，补8会产生进位位，效果等同于(a/10+1)

```python
class Solution:
    def NumberOf1Between1AndN(self, n):
        """O(logn)"""
        ones, m = 0, 1
        while m <= n:
            ones += (n // m + 8) // 10 * m + (n // m % 10 == 1) * (n % m + 1)
            m *= 10
        return ones

    def NumberOf1Between1AndN_2(self, n):
        ones, m = 0, 1
        while m <= n:
            if ((n // m) % 10) != 0 and ((n // m) % 10) != 1:
                ones += (n // 10 // m + 1) * m
            elif ((n // m) % 10) == 1:
                ones += (n // m // 10) * m + n % m + 1
            m *= 10
        return ones

    def NumberOf1Between1AndN_3(self, num):
        """O(n*logn)"""
        count = 0
        for n in range(1, num + 1):
            while n:
                if n % 10 == 1:
                    count += 1
                n = n // 10
        return count
```

## 33. 把数组排成最小的数

Q: 输入一个正整数数组，把数组里所有数字拼接起来排成一个数，打印能拼接出的所有数字中最小的一个。例如输入数组{3，32，321}，则打印出这三个数字能排成的最小数字为321323。

A: 将数组中的数字全部转换成字符串存储在一个新的数组中，然后比较每两个数字串的拼接的mn和nm的大小，如果mn小于nm，则m更小，反之n更小。然后把小的数放入一个新的列表中输出。

```python
from functools import cmp_to_key


class Solution:
    def PrintMinNumber(self, numbers):
        key = cmp_to_key(lambda x, y: int(x + y) - int(y + x))
        return ''.join(sorted([str(n) for n in numbers], key=key))
```

## 34. 丑数

Q: 把只包含质因子 2、3 和 5 的数称作丑数（Ugly Number）。例如 6、8 都是丑数，但 14 不是，因为它包含质因子 7。习惯上我们把 1 当做是第一个丑数。求按从小到大的顺序的第 N 个丑数。

A: 建立一个长度为 n 的数组保存 N 个丑数，在这 N 个数中，某个位置要求的丑数一定是前面某个丑数乘以 2、3或者5 得到的结果。分别记录乘以 2 能得到的最大丑数M<sub>2</sub>, 乘以 3 以后能得到的最大丑数M<sub>3</sub>, 乘以 5 以后能得到的最大丑数M<sub>5</sub>. 那么下一个丑数一定是这三个最大丑数中最小的那个。因为是按照顺序存放的丑数，所以对于乘以2来说，肯定存在一个丑数T<sub>2</sub>, 排在它之前的每一个丑数乘以 2 得到的结果都会小于已有的最大丑数，记住该丑数的位置，每次生成新的丑数时，更新它。对于乘以 3 或者 5，同理。

```python
class Solution:

    def GetUglyNumber(self, index):
        if not index:
            return

        uglyNumbers = [1] * index
        nextIndex = 1

        index2 = 0
        index3 = 0
        index5 = 0

        while nextIndex < index:
            min_val = min(uglyNumbers[index2] * 2, uglyNumbers[index3] * 3,
                         uglyNumbers[index5] * 5)
            uglyNumbers[nextIndex] = min_val

            while uglyNumbers[index2] * 2 <= uglyNumbers[nextIndex]:
                index2 += 1
            while uglyNumbers[index3] * 3 <= uglyNumbers[nextIndex]:
                index3 += 1
            while uglyNumbers[index5] * 5 <= uglyNumbers[nextIndex]:
                index5 += 1
            nextIndex += 1

        return uglyNumbers[-1]
```

## 35. 第一个只出现一次的字符

Q: 在字符串中找出第一个只出现一次的字符。如输入 "abaccdeff"，则输出 'b'。

A: 先遍历一遍字符串，用一个hash表存放每个出现的字符和字符出现的次数。在遍历一遍字符串，找到hash值等于1的即可。

```python
class Solution:
    def FirstNotRepeatingChar(self, s):
        if s is None or len(s) <= 0:
            return

        hashTable = {}
        for i in s:
            if i not in hashTable:
                hashTable[i] = 0
            hashTable[i] += 1

        for i in s:
            if hashTable[i] == 1:
                return i
        return
```

## 36. 数组中的逆序对

Q: 在数组中的两个数字如果前面一个数字大于后面的数字，则这两个数字组成一个逆序对。输入一个数组，求出这个数组中的逆序对的总数。例如在数组{7, 5, 6, 4}中，一共存在 5 个逆序对，分别是(7,6)、(7,5)、(7,4)、(6,4)、(5,4)。

A: 先将原序列排序，然后从排完序的数组中取出最小的，它在原数组中的位置表示有多少比它大的数在它前面，每取出一个在原数组中删除该元素，保证后面取出的元素在原数组中是最小的，这样其位置才能表示有多少比它大的数在它前面，即逆序对数。或者用归并排序。

```python
class Solution:
    def InversePairs(self, data):
        """O(nlogn)时间 + O(n)空间"""
        length = len(data)
        if data is None or length < 0:
            return 0

        copy = [data[i] for i in range(length)]
        count = self.InversePairsCore(data, copy, 0, length - 1)
        return count

    def InversePairsCore(self, data, copy, start, end):
        if start == end:
            copy[start] = data[start]
            return 0
        length = (end - start) // 2
        left = self.InversePairsCore(copy, data, start, start + length)
        right = self.InversePairsCore(copy, data, start + length + 1, end)

        # i 初始化为前半段最后一个数字的下标
        i = start + length
        # j 初始化为后半段最后一个数字的下标
        j = end

        indexCopy = end
        count = 0
        while i >= start and j >= (start + length + 1):
            if data[i] > data[j]:
                copy[indexCopy] = data[i]
                indexCopy -= 1
                i -= 1
                count += j - (start + length)
            else:
                copy[indexCopy] = data[j]
                indexCopy -= 1
                j -= 1

        while i >= start:
            copy[indexCopy] = data[i]
            indexCopy -= 1
            i -= 1
        while j >= (start + length + 1):
            copy[indexCopy] = data[j]
            indexCopy -= 1
            j -= 1
        return left + right + count

    def InversePairs2(self, data):
        if not data:
            return 0

        count = 0
        copy = [item for item in data]
        copy.sort()

        for i in range(len(copy)):
            count += data.index(copy[i])
            data.remove(copy[i])

        return count
```

## 37. 两个链表的第一个公共结点

Q: 输入两个链表，找出它们的第一个公共结点。

A: 

> 方法一: 时间复杂度O(mn)，空间复杂度O(1)

在第一个链表上顺序遍历每个结点，每遍历到一个结点的时候，在第二个链表上顺序遍历每个结点。

> 方法二: 时间复杂度O(m+n)，空间复杂度O(m+n)

利用辅助栈，分别将两个链表的结点放入栈里，这样两个链表的尾结点就位于两个栈的栈顶。接下来比较两个栈顶的结点是否相同。如果相同，就把栈顶弹出接着比较下一个栈顶，直到找到最后一个相同的结点。

> 方法三：时间复杂度O(m+n)，空间复杂度O(1)

首先遍历两个链表得到它们的长度。在第二次遍历的时候，在较长的链表上先走若干步，接着再同时在两个链表遍历，找到第一个的相同的结点就是它们的第一个公共结点。

```python
class ListNode:
    def __init__(self, x):
        self.val = x
        self.next = None


class Solution:
    def FindFirstCommonNode(self, pHead1, pHead2):
        nLength1 = self.GetListLength(pHead1)
        nLength2 = self.GetListLength(pHead2)
        nLengthDif = abs(nLength1 - nLength2)

        if nLength1 > nLength2:
            pListHeadLong = pHead1
            pListHeadShort = pHead2
        else:
            pListHeadLong = pHead2
            pListHeadShort = pHead1

        for i in range(nLengthDif):
            pListHeadLong = pListHeadLong.next

        while pListHeadLong and pListHeadShort and pListHeadLong != pListHeadShort:
            pListHeadLong = pListHeadLong.next
            pListHeadShort = pListHeadShort.next

        pFirstCommonNode = pListHeadLong
        return pFirstCommonNode

    def GetListLength(self, pHead):
        nLength = 0
        while pHead != None:
            pHead = pHead.next
            nLength += 1
        return nLength
```

## 38. 数字在排序数组中出现的次数

Q: 统计一个数字在排序数组中出现的次数。例如输入排序数组 [1,2,3,3,3,3,4,5]和数字3，由于3在这个数组中出现了4次，因此输出4。

A: 首先用二分查找法找到数组中第一个 k。如果中间的数字和 k 相等，我们判断这个数字是不是第一个 k。如果位于中间数字的前面一个数字不是 k，那么此时中间的数字就刚好是第一个 k。如果前面一个数字也是 k，那么我们仍需要在数组的前半段进行查找。最后一个 k 的思路也是如此。时间复杂度为 O(logn)。

```python
class Solution:
    def get_number_of_k(self, data, k):
        number = 0
        if data:
            length = len(data)
            first = self.get_first_k(data, length, k, 0, length - 1)
            last = self.get_last_k(data, length, k, 0, length - 1)
            if first > -1 and last > -1:
                number = last - first + 1
        return number

    def get_first_k(self, data, length, k, start, end):
        if start > end:
            return -1

        mid = (start + end) // 2
        mid_num = data[mid]

        if mid_num == k:
            if mid > 0 and data[mid - 1] != k or mid == 0:
                return mid
            end = mid - 1
        elif mid_num > k:
            end = mid - 1
        else:
            start = mid + 1
        return self.get_first_k(data, length, k, start, end)

    def get_last_k(self, data, length, k, start, end):
        if start > end:
            return -1

        mid = (start + end) // 2
        mid_num = data[mid]
        if mid_num == k:
            if mid < length - 1 and data[mid + 1] != k or mid == length - 1:
                return mid
            start = mid + 1
        elif mid_num > k:
            start = mid + 1
        else:
            end = mid - 1
        return self.get_last_k(data, length, k, start, end)


l = [1, 2, 3, 3, 3, 3, 4, 5]
s = Solution()
print(s.get_number_of_k(l, 3))
```

## 39.1 二叉树的深度

Q: 输入一棵二叉树的根结点，求该树的深度。从根结点到叶子结点依次经过的结点（含根、叶子结点）形成树的一条路径，最长路径的长度为树的深度。

A: 利用递归，如果一个树只有一个节点，那么深度为1，如果存在左子树和右子树，深度为左右子树中深度较深的加 1。

```python
class TreeNode:
    def __init__(self, x):
        self.val = x
        self.left = None
        self.right = None


class Solution:

    def TreeDepth(self, root):
        """递归，时间复杂度O(n), 空间复杂度O(logn)"""
        if not root:
            return 0
        return max(self.TreeDepth(root.left), self.TreeDepth(root.right)) + 1

    def TreeDepth2(self, root):
        """非递归，利用栈"""
        if not root:
            return 0
        depth = 0
        stack, flag = [], []
        node = root
        while node or stack:
            while node:
                stack.append(node)
                flag.append(0)
                node = node.left
            if flag[-1] == 1:
                depth = max(depth, len(stack))
                stack.pop()
                flag.pop()
                node = None
            else:
                node = stack[-1]
                node = node.right
                flag.pop()
                flag.append(1)
        return depth
```

## 39.2 二叉树是否为平衡树

Q: 输入一颗二叉树的根结点，判断该树是不是平衡二叉树。

A: 第一种思路：基于二叉树深度的求解方法，在遍历树的每个结点的时候，得到每个结点左右子树的深度。如果每个结点的左右子树的深度相差都不超过 1，就是平衡二叉树。但是这种方法时间效率不高。第二种思路是使用后序遍历的方式，在遍历每个结点的时候同时记录当前结点的深度，最后遍历到根结点的时候，就判断树是否为平衡二叉树。

```python
class TreeNode:
    def __init__(self, x):
        self.val = x
        self.left = None
        self.right = None


class Solution:

    def __init__(self):
        self.flag = True
        self.depth = 0

    def isBalanced(self, root):
        self.getDepth(root)
        return self.flag

    def getDepth(self, root):
        if not root:
            return 0

        left = self.getDepth(root.left)
        right = self.getDepth(root.right)
        if abs(left - right) > 1:
            self.flag = False

        return 1 + (left if left > right else right)

    def getDepth2(self, root):
        """递归，时间复杂度O(n), 空间复杂度O(logn)"""
        if not root:
            return 0
        return max(self.getDepth2(root.left), self.getDepth2(root.right)) + 1

    def isBalanced2(self, root):
        if not root:
            return True
        if abs(self.getDepth2(root.left)-self.getDepth2(root.right)) > 1:
            return False
        return self.isBalanced2(root.left) and self.isBalanced2(root.right)


pNode1 = TreeNode(1)
pNode2 = TreeNode(2)
pNode3 = TreeNode(3)
pNode4 = TreeNode(4)
pNode5 = TreeNode(5)
pNode6 = TreeNode(6)
pNode7 = TreeNode(7)

pNode1.left = pNode2
pNode1.right = pNode3
pNode2.left = pNode4
pNode2.right = pNode5
pNode3.right = pNode6
pNode5.left = pNode7

s = Solution()
print(s.isBalanced(pNode1))
print(s.isBalanced2(pNode1))
```

## 40. 数组中只出现一次的数字

Q: 一个整型数组里除了两个数字以外，其他的数字都出现了两次。找出这两个只出现一次的数字，要求时间复杂度是 O(n)，空间复杂度为 O(1)。

A: 首先从头到尾依次异或数组中的每一个数字，最后结果肯定是两个只出现一次数字的异或结果，其结果也不为0。也就是说二进制表示中至少有一位 1。在结果数字中找到第一个为 1 的位的位置，计为第 n 位。然后根据第 n 位是否是 1 将原数组分为两个子数组，然后对每个子数组进行异或运算就能找到两个只出现一次的数字。

```python
class Solution:

    def FindNumsAppearOnce(self, arr):
        if not arr:
            return []
        resultExclusiveOr = 0
        for i in arr:
            resultExclusiveOr ^= i

        indexOf1 = self.FindFirstBitIs1(resultExclusiveOr)
        num1, num2 = 0, 0
        for j in range(len(arr)):
            if self.IsBit1(arr[j], indexOf1):
                num1 ^= arr[j]
            else:
                num2 ^= arr[j]
        return [num1, num2]

    def FindFirstBitIs1(self, num):
        indexBit = 0
        while num & 1 == 0 and indexBit <= 32:
            indexBit += 1
            num = num >> 1
        return indexBit

    def IsBit1(self, num, indexBit):
        num = num >> indexBit
        return num & 1


l = [2, 4, 3, 6, 3, 2, 5, 5]
s = Solution()
print(s.FindNumsAppearOnce(l))
```

## 41.1 和为 s 的两个数字

Q: 输入一个递增排序的数组和一个数字 s，在数组中查找两个数，使得它们的和正好是 s。如果有多对数字的和等于 s，输出两个数的乘积最小的。

A: 两个指针，一个指向起点，一个指向终点，然后对两个数字求和，如果大于s，则前移后一个指针，如果小于s，则把前一个指针后移。

```python
class Solution:
    def FindNumbersWithSum(self, arr, target):
        if not arr:
            return []
        start, end = 0, len(arr) - 1
        while start < end:
            val = arr[start] + arr[end]
            if val < target:
                start += 1
            elif val > target:
                end -= 1
            else:
                return [arr[start], arr[end]]
        return []


arr = [1, 2, 4, 7, 11, 15]
s = Solution()
print(s.FindNumbersWithSum(arr, 15))
```

## 41.2 和为 s 的连续正数序列

Q: 输入一个正数 s，打印出所有和为 s 的连续正数序列（至少含有两个数）。例如输入15，打印出1～5、4～6和7～8三个连续序列。

A: 设置 small 和 big 两个数表示序列的最小值和最大值。首先将 small 和 big 分别设置为 1 和 2。如果 small 到 big 的序列和大于 s，就去掉序列中最小的值，也就是增加 small。如果 small 到 big 的序列和小于 s，就增大 big。因为序列至少两个数字，所以 small 最多到 (1+s)/2 为止。

```python
class Solution:

    def FindContinuousSequence(self, target):
        if target < 3:
            return []

        small, big = 1, 2
        mid = (1 + target) / 2
        cur = small + big
        result = []
        while small < mid:
            if cur > target:
                cur -= small
                small += 1
            else:
                if cur == target:
                    result.append([small, big])
                big += 1
                cur += big
        return result


s = Solution()
print(s.FindContinuousSequence(15))
```

## 42.1 翻转单词顺序

Q: 输入一个英文句子，翻转句子中单词的顺序，但单词内的顺序不变（标点符号和普通字母一样处理）

A: 第一步把输入的字符串完全翻转，第二步从前往后依次遍历新字符串，遇到空格，就把空格前的字符串翻转，添加空格，继续遍历，遍历到结尾的时候，因为最后一个字符串后没有空格，所以最后要再翻转它。

```python
class Solution:

    def reverse(self, s):
        if not s:
            return ''
        start, end = 0, len(s) - 1
        while start < end:
            s[start], s[end] = s[end], s[start]
            start += 1
            end -= 1
        return s

    def reverse_sentence(self, s):
        if not s:
            return ''

        s = list(s)
        new_s = self.reverse(s)
        begin, end = 0, 0
        length = len(s)
        while end < length:
            if end == length - 1:
                new_s[begin:] = self.reverse(new_s[begin:])
                break
            if new_s[begin] == ' ':
                begin += 1
                end += 1
            elif new_s[end] == ' ':
                new_s[begin:end] = self.reverse(new_s[begin:end])
                begin = end
            else:
                end += 1

        return ''.join(new_s)


test_string = 'I am a student.'
s = Solution()
print(s.reverse_sentence(test_string))
```

## 42.2 左旋转字符串

Q: 字符串的左旋转操作是把字符串前面的若干个字符转移到字符串的尾部。

A: 将字符串根据旋转的位数将字符串分为两部分，然后对每部分进行翻转，最后对整个字符串再进行一次翻转即可。

```python
class Solution:

    def reverse(self, s):
        if not s:
            return ''
        start, end = 0, len(s) - 1
        while start < end:
            s[start], s[end] = s[end], s[start]
            start += 1
            end -= 1
        return s

    def left_rotate_string(self, s, n):
        if not s:
            return ''

        new_s = list(s)
        new_s[:n] = self.reverse(new_s[:n])
        new_s[n:] = self.reverse(new_s[n:])
        new_s = self.reverse(new_s)

        return ''.join(new_s)


test_string = 'abcdefg'
s = Solution()
print(s.left_rotate_string(test_string, 2))
```

## 43. n 个骰子的点数

Q: 把 n 个骰子仍在地上，所有骰子朝上一面的点数之和为 s。输入 n，打印出 s 的所有可能的值出现的概率。

A: 
第一种方法是基于动态规划。可将f(n, s) 表示n个骰子点数的和为s的排列情况总数。n个骰子点数和为s的种类数只与n-1个骰子的和有关。因为一个骰子有六个点数，那么第n个骰子可能出现1到6的点数。所以第n个骰子点数为1的话，f(n,s)=f(n-1,s-1)，当第n个骰子点数为2的话，f(n,s)=f(n-1,s-2)，…，依次类推。当在增加一个骰子时，f(n,s)=f(n-1,s-1)+f(n-1,s-2)+f(n-1,s-3)+f(n-1,s-4)+f(n-1,s-5)+f(n-1,s-6)。

第二种方法使用两个数组来存储骰子点数的每一个总数出现的次数。在第一次循环中，第一个数组中的第 n 个数字表示骰子和为 b 出现的次数。在第二次循环中，加上一个新骰子，此时和为 n 的骰子出现的次数应该等于上一次循环中骰子点数和为 n-1、n-2、n-3、n-4、n-5 和 n-6 的次数的总和。将另一个数组中的第 n 个数字设为前一个数组对应的第 n-1、n-2、n-3、n-4、n-5 和 n-6 之和。（其实和第一种思路是一样的）

```python
class Solution:

    def print_probability(self, n):
        dice = n
        total = 6 ** n
        for target in range(n, 6 * n + 1):
            count = self.get_N_count(dice, target)
            probability = count / total * 100
            print(
                "n={}, target={}, count={}, p={:.2f}%".format(n, target, count,
                                                              probability))

    def get_N_count(self, n, target):
        if n < 1 or target < n or target > 6 * n:
            return 0
        elif n == 1:
            return 1

        count = self.get_N_count(n - 1, target - 1) + \
                self.get_N_count(n - 1, target - 2) + \
                self.get_N_count(n - 1, target - 3) + \
                self.get_N_count(n - 1, target - 4) + \
                self.get_N_count(n - 1, target - 5) + \
                self.get_N_count(n - 1, target - 6)
        return count

    def print_probability2(self, n):
        if n < 1:
            return
        MAX_VALUE = 6
        storage = [[], []]
        storage[0] = [0] * (MAX_VALUE * n + 1)
        flag = 0

        # 1 个骰子时每个点数的和
        for i in range(1, MAX_VALUE + 1):
            storage[flag][i] = 1
        # 外层循环为每次增加一个骰子数量
        for dice in range(2, n + 1):
            storage[1-flag] = [0] * (MAX_VALUE * n + 1)
            # 内层循环为骰子和的范围
            for target in range(dice, MAX_VALUE * dice + 1):
                dice_value = 1  # 骰子的值
                # 第n个数字设为前一个数组对应的第n - 1、n - 2、n - 3、n - 4、n - 5、n - 6 之和
                while dice_value < target and dice_value <= MAX_VALUE:
                    storage[1-flag][target] += storage[flag][target-dice_value]
                    dice_value += 1
            flag = 1 - flag

        total = MAX_VALUE ** n
        for target in range(n, MAX_VALUE * n + 1):
            count = storage[flag][target]
            probability = count / total * 100
            print(
                "n={}, target={}, count={}, p={:.2f}%".format(n, target, count,
                                                              probability))
```

## 44. 扑克牌的顺子

Q: 从扑克牌中随机抽取 5 张牌，判断是不是一个顺子，即这 5 张牌是不是连续的，2～10 为数字本身，A 为 1，J 为 11，Q 为 12，K 为 13，大王、小王可以看成任意数字。

A: 把这5张牌组成的顺子看成数组，所以先对数组排序，然后求出大小王即0的个数，然后除去0之外的数字，求出在数组中的数字间隔，然后比较间隔和0的个数，如果出现成对的，肯定不是顺子，再求间隔的时候需要鉴别。

```python
class Solution:

    def IsContinuous(self, arr):
        """假定大王、小王为0"""
        if not arr:
            return False
        # 转换
        mapping = {'A': 1, 'J': 11, 'Q': 12, 'K': 13}
        for i in range(len(arr)):
            if arr[i] in mapping:
                arr[i] = mapping[arr[i]]

        arr = sorted(arr)
        number_of_zero = 0
        number_of_gap = 0
        length = len(arr)

        # 统计0的个数
        idx = 0
        while idx < length and arr[idx] == 0:
            number_of_zero += 1
            idx += 1

        # 统计间隔的数目
        small = number_of_zero
        big = small + 1
        while big < length:
            # 出现对子, 不可能是顺子
            if arr[small] == arr[big]:
                return False

            number_of_gap += arr[big] - arr[small] - 1
            small = big
            big += 1
        return False if number_of_gap > number_of_zero else True


l1 = ['A', 3, 2, 5, 0]
l2 = [0, 3, 1, 6, 4]
s = Solution()
print(s.IsContinuous(l1))
print(s.IsContinuous(l2))
```

## 45. 圆圈中最后剩下的数字

Q: 0,1,...,n-1 这 n 个数字排成一个圆圈，从数字 0 开始每次从这个圆圈里删除第 m 个数字，求出这个圆圈里剩下的最后一个数字。(约瑟夫环)

A: 第一种方法是用环形链表模拟圆圈；第二种是分析每次被删除的数字的规律并直接计算出圆圈中最后剩下的数字。推导过程可以参考[约瑟夫环——公式法（递推公式）](https://blog.csdn.net/u011500062/article/details/72855826)

```python
from collections import deque


class Solution:

    def LastRemaining(self, n, m):
        """常规解法"""
        if n < 1 or m < 1:
            return -1

        d = deque(range(n))
        start, last = 0, -1
        while d:
            k = (start + m - 1) % n
            last = d[k]
            d.remove(last)
            n -= 1
            start = k
        return last

    def LastRemaining2(self, n, m):
        """
        利用数学规律，每删除一个人等于将这个数组向前移动了 m 位
        """
        if n < 1 or m < 1:
            return -1
        remain_idx = 0
        for i in range(1, n+1):
            remain_idx = (remain_idx + m) % i
        return remain_idx


s = Solution()
print(s.LastRemaining(5, 3))
print(s.LastRemaining2(5, 3))
```

## 46. 求 1+2+...+n

Q: 求 1+2+...+n，要求不能使用乘除法、for、while、if、else、switch、case 等关键字及条件判断语句。

A: 利用两个函数，一个函数充当递归函数的角色，另一个函数处理终止递归的情况，如果对n连续两次做去反运算，那么非零的n转换为True，0转换为False。利用这一特性终止递归。还可以利用python中的and特性，a and b，a为False，返回a，a为True，就返回b

```python
class Solution:

    def custom_sum(self, n):
        return n and self.custom_sum(n - 1) + n

    def custom_sum2(self, n):
        return self.sum_n(n)

    def sum_0(self, n):
        return 0

    def sum_n(self, n):
        cond = {False: self.sum_0, True: self.sum_n}
        return n + cond[bool(n)](n-1)


s = Solution()
print(s.custom_sum(5))
print(s.custom_sum2(5))
```

## 47. 不用加减乘除做加法

Q: 写一个函数，求两个整数之和，要求在函数体内不得使用+、-、*、/四则运算符号。

A: 两个数异或：相当于每一位相加，而不考虑进位；两个数相与，并左移一位：相当于求得进位；将上述两步的结果相加。注意：在使用Python实现的过程中，对于正整数是没有问题的，但是对于负数，会出现死循环情况。因为在Python中，对于超出32位的大整数，会自动进行大整数的转变，这就导致了在右移位过程中，不会出现移到了0的情况，也就会造成了死循环。已经知道了右移过程中大整数的自动转化，导致变不成0，那么只需要在移动的过程中加一下判断就行了。一个int可表示的无符号整数为4294967295，对应的有符号为-1。因此最后我们可以判断符号位是否为1。

```python
class Solution:

    def Add(self, num1, num2):
        while num2 != 0:
            res = num1 ^ num2
            num2 = (num1 & num2) << 1
            num1 = res & 0xffffffff
        return num1 if num1 >> 31 == 0 else num1 - 2 ** 32


s = Solution()
print(s.Add(1, 2))
```

## 49. 把字符串转换成整数

Q: 写一个函数，实现把字符串转换为整数。不能使用类似的库函数

A:  主要是区分输入和合法性，比如输入一个None，输入一个空字符串 “ ”，或者输入的字符串中含有“+”或者“-”，或者输入的字符串中含有除去“+”“-” 数字的非数字字符。

```python
class Solution:

    def StrToInt(self, s):
        """如果输出是0, 通过检查flag判断输入不合法还是输入直接是'0'"""
        flag = False
        if not s:
            return 0
        stack = []
        mapping = {'0': 0, '1': 1, '2': 2, '3': 3, '4': 4, '5': 5, '6': 6,
                   '7': 7, '8': 8, '9': 9}
        for idx, value in enumerate(s):
            if value in mapping:
                stack.append(mapping[value])
            elif value in ['+', '-'] and idx != 0 or value not in ['+', '-']:
                return 0

        if len(stack) == 1 and stack[0] == 0:
            flag = True
            return 0
        result = 0
        for item in stack:
            result = result * 10 + item

        return -result if s[0] == '-' else result


test_string = '-12356'
s = Solution()
print(s.StrToInt(test_string))
```

## 50.1 二叉树中两个结点的最低公共祖先

待补充

## 51. 数组中重复的数字

Q: 在一个长度为n的数组里的所有数字都在0到n-1的范围内。数组中某些数字是重复的，但不知道有几个数字是重复的。也不知道每个数字重复几次。请找出数组中任意一个重复的数字。例如，如果输入长度为7的数组{2,3,1,0,2,5,3}，那么对应的输出是重复的数字2或者3。

A: 

> 方法一: 时间复杂度O(nlogn)，空间复杂度O(1)

首先将数组进行排序，然后顺序扫描数组，直到发现有重复的数字。

> 方法二: 时间复杂度O(n), 空间复杂度O(n)

利用哈希表，顺序扫描数组，如果哈希表中没有该数字，就加到哈希表中，否则就是重复的数字。

> 方法三: 时间复杂度O(n), 空间复杂度O(1)

由于数组中的数字都是在 0 到 n-1 之间。如果数组中没有重复的数字，那么数组排序后，下标为 i 对应的数值就是 i。

顺序扫描数组，当扫描到下标为 i 的数字时，比较这个数字 m 是否等于下标 i。如果是，接着扫描下一个数字。如果不是，用 m 和下标为 m 的数字进行比较。如果 m 和下标为 m 的数字相等，就找到一个重复的数字。如果不相等，那么将 m 和下标为 m 的数字进行交换。然后重复这个过程，直到找到一个重复的数字。

```python
class Solution:
    def duplicate(self, numbers):
        if numbers is None or len(numbers) <= 0:
            return False, []
        for num in numbers:
            if num < 0 or num >= len(numbers):
                return False, []

        for idx in range(len(numbers)):
            while idx != numbers[idx]:
                if numbers[idx] == numbers[numbers[idx]]:
                    return True, [numbers[idx]]
                else:
                    numbers[numbers[idx]], numbers[idx], = numbers[idx], numbers[numbers[idx]]
        return False, []
```

## 52. 构建乘积数组

Q: 给定一个数组A[0,1,...,n-1],请构建一个数组B[0,1,...,n-1],其中B中的元素B[i]=A[0]*A[1]*...*A[i-1]*A[i+1]*...*A[n-1]。不能使用除法。

A: 把 B[i] 分成两部分， 一部分是 A[0,…,i-1] 的连乘，记为 C[i]，一部分是 A[i+1,…,n-1] 的连乘，记为 D[i]，所以，C[i]=C[i-1] * A[i-1]， D[i]=D[i+1] * A[i+1]。

```python
class Solution:
    def multiply(self, array):
        if array is None or len(array) <= 0:
            return
        length = len(array)
        result = [1] * length
        tmp = 1
        # 计算下三角
        for i in range(1, length):
            result[i] = result[i - 1] * array[i - 1]
        # 计算上三角
        for i in range(length - 2, -1, -1):
            tmp *= array[i + 1]
            result[i] *= tmp
        return result


array = [1, 2, 3, 4]
s = Solution()
print(s.multiply(array))
```

## 53. 正则表达式匹配

Q: 请实现一个函数用来匹配包括'.'和'*'的正则表达式。模式中的字符'.'表示任意一个字符，而'*'表示它前面的字符可以出现任意次（包含0次）。在本题中，匹配是指字符串的所有字符匹配整个模式。例如，字符串"aaa"与模式"a.a"和"ab*ac*a"匹配，但是与"aa.a"和"ab*a"均不匹配。

A: 如果字符串中的第一个字符和模式中的第一个字符相匹配，那么在字符串和模式上都向后移动一个字符。然后匹配剩余的字符串。否则返回 False。当模式中的第二个字符是“*”时，一个选择是在模式上向后移动两个字符，相当于将“*”和它前面的字符被忽略掉，因为“*”可以匹配字符串中 0 个字符。如果模式中的第一个字符和字符串的第一个字符匹配后，在字符串向后移动一个字符后，模式上有两个选择：向后移动两个字符或者保持模式不变。

```python
class Solution:
    def match(self, s, pattern):
        if not s:
            return False
        if not pattern:
            return False
        if s == pattern:
            return True
        index = 0
        if len(pattern) > 1 and pattern[index + 1] == '*':
            # 第一种情况：'a' 和 'a*a'，星号代表0个a，因此模式需要往后移动两个字符继续判断
            # 第二种情况：'aaa' 和 'a*a'，星号代表多个a，因此字符串需要往后移动一个字符继续判断
            # 第三中情况：'abc' 和 'a*bc'，星号代表1个a，因此字符串往后移动一位，模式移动两位继续判断
            if s[index] == pattern[index] or pattern[index] == '.':
                return self.match(s, pattern[index + 2:]) or \
                       self.match(s[index + 1:], pattern) or \
                       self.match(s[index + 1:], pattern[index + 2:])
            else:
                # 如果字符串中的第一个字符和模式中的第一个字符不匹配，那么模式必须往后移动两个字符来继续进行判断
                return self.match(s, pattern[index + 2:])

        # 如果字符串中的第一个字符和模式中的第一个字符相匹配，那么字符串和模式都往后移动一个字符
        if s[index] == pattern[index] or pattern[index] == '.':
            return self.match(s[index + 1:], pattern[index + 1:])
        return False


string = 'aaa'
pattern = 'a*a'
pattern2 = 'aa.a'
s = Solution()
print(s.match(string, pattern))
print(s.match(string, pattern2))
```

## 54. 表示数值的字符串

Q: 请实现一个函数用来判断字符串是否表示数值（包括整数和小数）。例如，字符串"+100","5e2","-123","3.1416"和"-1E-16"都表示数值。但是"12e","1a3.14","1.2.3","+-5"和"12e+4.3"都不是。

A: 表示数值的字符串遵循模式 `A[.[B]][e∣EC]A[.[B]][e∣EC]` 或者 `.B[e∣EC].B[e∣EC]`，其中A为整数部分，B为小数部分，C为指数部分。首先尽可能多的扫描0-9的数位，也是就A的部分，如果遇到小数点，则开始扫描B部分，如果遇到e或者E，则开始扫描指数C部分。

```python
class Solution:
    def isNumeric(self, s):
        if s is None or len(s) <= 0:
            return False

        sList = [i.lower() for i in s]

        if 'e' in sList:
            index = sList.index('e')
            integral_digits = sList[:index]
            exponential_digits = sList[index + 1:]
            if '.' in exponential_digits or len(exponential_digits) == 0:
                return False

            isExponential = self.scanDigits(exponential_digits)
            isIntegral = self.scanDigits(integral_digits)
            return isExponential and isIntegral
        else:
            return self.scanDigits(sList)

    def scanDigits(self, s):
        dot_num = 0
        valid_list = '0123456789+-.e'
        for idx, item in enumerate(s):
            if item not in valid_list:
                return False
            if item == '.':
                if dot_num > 1:
                    return False
                dot_num += 1
            if item in '+-' and idx != 0:
                return False
        return True


string1 = '5e2'
string2 = '1a3.14'
s = Solution()
print(s.isNumeric(string1))
print(s.isNumeric(string2))
```

## 55. 字符流中第一个不重复的字符

Q: 请实现一个函数用来找出字符流中第一个只出现一次的字符。例如，当从字符流中只读出前两个字符"go"时，第一个只出现一次的字符是"g"。当从该字符流中读出前六个字符“google"时，第一个只出现一次的字符是"l"。如果当前字符流没有存在出现一次的字符，返回#字符。

A: 使用两个辅助空间，一个dict存储当前出现的字符以及字符出现的次数，list存储出现的字符，然后遍历list，第一个次数为1的就是所需要的字符。

```python
class Solution:
    def __init__(self):
        self.hashTable = {}
        self.charList = []

    def FirstAppearingOnce(self):
        for char in self.charList:
            if self.hashTable[char] == 1:
                return char
        return '#'

    def Insert(self, char):
        if char not in self.hashTable:
            self.hashTable[char] = 1
            self.charList.append(char)
        elif self.hashTable[char]:
            self.hashTable[char] = 2


s = Solution()
s.Insert('g')
print(s.FirstAppearingOnce())
s.Insert('o')
print(s.FirstAppearingOnce())
s.Insert('o')
print(s.FirstAppearingOnce())
s.Insert('g')
print(s.FirstAppearingOnce())
s.Insert('l')
print(s.FirstAppearingOnce())
```

## 56. 链表中环的入口结点

Q: 一个链表中包含环，如何找出环的入口结点？

A: 利用快慢指针，如果链表中的环有 n 个结点，快指针先在链表上移动 n 步，然后两个指针同时移动，当慢指针指向环的入口结点时，快指针正好围绕环走了一圈回到入口结点。

```python
class ListNode:
    def __init__(self, x):
        self.val = x
        self.next = None


class Solution:
    def MeetingNode(self, head):
        """找到快慢指针相遇的结点"""
        if not head:
            return

        slow = head.next
        if not slow:
            return
        fast = slow.next
        while fast:
            if slow == fast:
                return slow
            slow = slow.next
            fast = fast.next
            if fast:
                fast = fast.next

    def EntryNodeOfLoop(self, head):
        meeting_node = self.MeetingNode(head)
        if not meeting_node:
            return

        loop = 1
        start_node = meeting_node
        while start_node.next != meeting_node:
            loop += 1
            start_node = start_node.next

        fast = head
        for i in range(loop):
            fast = fast.next
        slow = head
        while fast != slow:
            fast = fast.next
            slow = slow.next
        return fast
```

## 57. 删除链表中重复的结点

Q: 在一个排序的链表中，如何删除重复的结点？例如，链表1->2->3->3->4->4->5 处理后为 1->2->5。

A: 第一步是确定删除的参数。当然这个函数需要输入待删除链表的头结点。头结点可能与后面的结点重复，也就是说头结点也可能被删除，所以在链表头添加一个结点。接下来我们从头遍历整个链表。如果当前结点的值与下一个结点的值相同，那么它们就是重复的结点，都可以被删除。为了保证删除之后的链表仍然是相连的而没有中间断开，我们要把当前的前一个结点和与当前结点的值不重复的结点相连。我们要确保前一个结点要始终与下一个没有重复的结点连接在一起。

```python
class ListNode:
    def __init__(self, x):
        self.val = x
        self.next = None


class Solution:
    
    def deleteDuplication(self, head):
        if not head:
            return
        prev = ListNode(-1)
        prev.next = head
        last = prev
        while head and head.next:
            if head.val == head.next.val:
                val = head.val
                while head and val == head.val:
                    head = head.next
                last.next = head
            else:
                last = head
                head = head.next
        return prev.next
```

## 58. 二叉树的下一个结点

Q: 给定一颗二叉树和其中的一个结点，如何找出中序遍历顺序的下一个结点？树中的结点除了有两个分别指向左右结点的指针以外，还有一个指向父结点的指针。

A: 三种情况：当前节点有右子树的话，当前节点的下一个结点是右子树中的最左子节点；当前节点无右子树但是是父节点的左子节点，下一个节点是当前结点的父节点；当前节点无右子树而且是父节点的右子节点，则一直向上遍历，直到找到最靠近的一个祖先节点pNode，pNode是其父节点的左子节点，那么输入节点的下一个结点就是pNode的父节点。

```python
class TreeLinkNode:
    def __init__(self, x):
        self.val = x
        self.left = None
        self.right = None
        self.next = None


class Solution:

    def GetNext(self, pNode):
        # 输入是一个空节点
        if not pNode:
            return
        # 注意当前节点是根节点的情况。所以在最开始设定pNext = None, 如果下列情况都不满足, 说明当前结点为根节点, 直接输出None
        pNext = None
        # 如果输入节点有右子树，则下一个结点是当前节点右子树中最左节点
        if pNode.right:
            pNode = pNode.right
            while pNode.left:
                pNode = pNode.left
            pNext = pNode
        else:
            # 如果当前节点有父节点且当前节点是父节点的左子节点, 下一个结点即为父节点
            if pNode.next and pNode.next.left == pNode:
                pNext = pNode.next
            # 如果当前节点有父节点且当前节点是父节点的右子节点, 那么向上遍历
            # 当遍历到当前节点为父节点的左子节点时, 输入节点的下一个结点为当前节点的父节点
            elif pNode.next and pNode.next.right == pNode:
                pNode = pNode.next
                while pNode.next and pNode.next.right == pNode:
                    pNode = pNode.next
                # 遍历终止时当前节点有父节点, 说明当前节点是父节点的左子节点, 输入节点的下一个结点为当前节点的父节点
                # 反之终止时当前节点没有父节点, 说明当前节点在位于根节点的右子树, 没有下一个结点
                if pNode.next:
                    pNext = pNode.next
        return pNext
```

## 59. 对称的二叉树

Q: 请实现一个函数，用来判断一颗二叉树是不是对称的。注意，如果一个二叉树同此二叉树的镜像是同样的，定义其为对称的。

A: 把叶子节点的 None 结点加入到遍历中，按照前序遍历的方式遍历二叉树，存入一个序列中，然后按照和前序遍历对应的先父节点，然后右节点，最后左结点遍历二叉树，存入一个序列中，如果两个序列相等，则对称。

```python
class TreeNode:
    def __init__(self, x):
        self.val = x
        self.left = None
        self.right = None


class Solution:

    def isSymmetrical(self, pRoot):
        return self._IsSymmetrical(pRoot, pRoot)

    def _IsSymmetrical(self, pRoot1, pRoot2):
        if pRoot1 is None and pRoot2 is None:
            return True
        if pRoot1 is None or pRoot2 is None:
            return False
        if pRoot1.val != pRoot2.val:
            return False
        return self._IsSymmetrical(pRoot1.left,
                                   pRoot2.right) and self._IsSymmetrical(
            pRoot1.right, pRoot2.left)
```

## 60. 把二叉树打印成多行

Q: 从上到下按层打印成多行，同一层的结点按从左到右的顺序打印，每一层打印到一行。

A: 两个队列，首先把当前层的节点存入队列1中，然后遍历队列1，遍历时，如果有左子树或者右子树，依次存入队列2，然后遍历队列2.

```python
class TreeNode:
    def __init__(self, x):
        self.val = x
        self.left = None
        self.right = None


class Solution:

    def print_level(self, root):
        if not root:
            return []

        nodes, result = [root], []
        while nodes:
            next_level = []
            for node in nodes:
                print(node.val, end=' ')
                if node.left:
                    next_level.append(node.left)
                if node.right:
                    next_level.append(node.right)
            nodes = next_level
            print()

        return result
```

## 61. 按之字形顺序打印二叉树

Q: 请实现一个函数按照之字形顺序打印二叉树，即第一行按照从左到右的顺序打印，第二层按照从右到左的顺序打印，第三行再按照从左到右的顺序打印，其他行以此类推。

A: 按之字形顺序打印二叉树需要两个栈。我们在打印某一行节点时，把下一层的子节点保存到相应的栈里。如果当前打印的奇数层，则先保存左子节点再保存右子节点到第一个栈里；如果当前打印的是偶数层，则先保存右子节点再保存左子节点到第二个栈里。

```python
class TreeNode:
    def __init__(self, x):
        self.val = x
        self.left = None
        self.right = None


class Solution:

    def Print(self, pRoot):
        if not pRoot:
            return []
        result, nodes = [], [pRoot]
        odd_level = True
        while nodes:
            curStack, nextStack = [], []
            if odd_level:
                for node in nodes:
                    curStack.append(node.val)
                    if node.left:
                        nextStack.append(node.left)
                    if node.right:
                        nextStack.append(node.right)
            else:
                for node in nodes:
                    curStack.append(node.val)
                    if node.right:
                        nextStack.append(node.right)
                    if node.left:
                        nextStack.append(node.left)
            nextStack.reverse()
            odd_level = not odd_level
            result.append(curStack)
            nodes = nextStack
        return result

    def zigzagLevelOrder(self, root):
        """存储的时候一直从左向右存储，打印的时候根据不同的层一次打印"""
        if not root:
            return []
        levels, result, leftToRight = [root], [], True
        while levels:
            curValues, nextLevel = [], []
            for node in levels:
                curValues.append(node.val)
                if node.left:
                    nextLevel.append(node.left)
                if node.right:
                    nextLevel.append(node.right)
            if not leftToRight:
                curValues.reverse()
            if curValues:
                result.append(curValues)
            levels = nextLevel
            leftToRight = not leftToRight
        return result
```

## 62. 序列化二叉树

Q: 实现两个函数，分别用来序列化和反序列化二叉树。

A: 二叉树的序列化，通过前序遍历二叉树输出节点，然后遇到左子结点和右子结点为None的时候，输出一个特殊字符。对于反序列化，就是通过输入的序列构建二叉树，针对前序遍历，可以先设置一个指针，先指向序列的开头，然后把指针位置的数字转化成结点，当遇到特殊字符时或者超出序列长度时，对应节点为none。然后继续遍历。

```python
class TreeNode:
    def __init__(self, x):
        self.val = x
        self.left = None
        self.right = None


class Solution:

    def serialize(self, root):
        if not root:
            return '$,'
        return str(root.val) + ',' + self.serialize(root.left) + self.serialize(root.right)

    def serialize2(self, root):
        serialize_res = ''
        if not root:
            return '$,'
        stack = []
        while root or stack:
            while root:
                serialize_res += str(root.val) + ','
                stack.append(root)
                root = root.left
            serialize_res += '$,'
            root = stack.pop()
            root = root.right
        return serialize_res

    def deserialize(self, s):
        nodes = s.split(',')
        root, _ = self.Deserialize(nodes, 0)
        return root

    def Deserialize(self, s, idx):
        if idx >= len(s) or s[idx] == '$':
            return None, idx + 1

        node = TreeNode(int(s[idx]))
        idx += 1
        node.left, idx = self.Deserialize(s, idx)
        node.right, idx = self.Deserialize(s, idx)
        return node, idx
```

## 63. 二叉搜索树的第 k 个结点

Q: 给定一个二叉搜索树，请找出其中的第 k 大的结点。

A: 使用中序遍历即可。

```python
class TreeNode:
    def __init__(self, x):
        self.val = x
        self.left = None
        self.right = None


class Solution:

    def KthNode(self, pRoot, k):
        if not pRoot or not k:
            return
        res = []

        def traverse(node):
            """中序遍历"""
            if len(res) >= k or not node:
                return
            traverse(node.left)
            res.append(node)
            traverse(node.right)

        traverse(pRoot)
        if len(res) < k:
            return
        return res[k - 1]
```

## 64. 数据流中的中位数

Q: 如何得到一个数据流的中位数？如果从数据流中读出奇数个数值，那么中位数就是所有数值排序后位于中间的数值。如果从数据流中读出偶数个数值，那么中位数就是所有数值排序之后中间两个数的平均值。

A: 将数据容器分隔为两个部分，如果能够保证数据容器左边的数据都小于右边的数据，这样即使没有排序，也可以根据左边最大的数和右边最小的数来算出中位数。可以使用最大堆和最小堆分别实现这两个容器。为了保证数据平均分配到两个堆中，因此两个堆中的数据数目之差不能超过 1。可以在数据的总数目为偶数时，将数据插入到最大堆中，否则就插入到最小堆中。

如果新的数据需要插入到最大堆中，但是它比最小堆中的一些数据还要大，那么可以先把数据插入到最小堆中，然后重排将最小堆中最小的数拿出来插入到最大堆中；如果新的数据需要插入到最小堆中，但是它比最大堆中的一些数据还要小，那么可以先把数据插入到最大堆中，然后重排将最大堆中最大的数拿出来插入到最小堆中。

插入时间复杂度为 O(logn)，得到中位数的时间复杂度为 O(1)。

```python
class Solution:
    def __init__(self):
        self.left = []
        self.right = []
        self.count = 0

    def Insert(self, num):
        # 如果总数目是偶数，就插入到左边最大堆，否则就插入到右边最小堆
        if self.count & 1 == 0:
            self.left.append(num)
        else:
            self.right.append(num)
        self.count += 1

    def GetMedian(self):
        if self.count == 1:
            return self.left[0]
        self.MaxHeap(self.left)
        self.MinHeap(self.right)
        if self.left[0] > self.right[0]:
            self.left[0], self.right[0] = self.right[0], self.left[0]
        self.MaxHeap(self.left)
        self.MinHeap(self.right)
        if self.count & 1 == 0:
            return (self.left[0] + self.right[0]) / 2.0
        else:
            return self.left[0]

    def MaxHeap(self, alist):
        """构造最大堆"""
        if not alist:
            return
        length = len(alist)
        if length == 1:
            return alist
        for i in range(length // 2 - 1, -1, -1):
            k = i
            heap = False
            while not heap and 2 * k < length - 1:
                index = 2 * k + 1
                if index < length - 1 and alist[index] < alist[index + 1]:
                    index += 1
                if alist[k] >= alist[index]:
                    heap = True
                else:
                    alist[k], alist[index] = alist[index], alist[k]
                    k = index

    def MinHeap(self, alist):
        """构建最小堆"""
        if not alist:
            return
        length = len(alist)
        if length == 1:
            return alist
        for i in range(length // 2 - 1, -1, -1):
            k = i
            heap = False
            while not heap and 2 * k < length - 1:
                index = 2 * k + 1
                if index < length - 1 and alist[index] > alist[index + 1]:
                    index += 1
                if alist[k] <= alist[index]:
                    heap = True
                else:
                    alist[k], alist[index] = alist[index], alist[k]
                    k = index
```

## 65. 滑动窗口的最大值

Q: 给定一个数组和滑动窗口的大小，找出所有滑动窗口里数值的最大值。例如，如果输入数组{2,3,4,2,6,2,5,1}及滑动窗口的大小3，那么一共存在6个滑动窗口，他们的最大值分别为{4,4,6,6,6,5}。针对数组{2,3,4,2,6,2,5,1}的滑动窗口有以下6个: {[2,3,4],2,6,2,5,1}，{2,[3,4,2],6,2,5,1}，{2,3,[4,2,6],2,5,1}，{2,3,4,[2,6,2],5,1}， {2,3,4,2,[6,2,5],1}，{2,3,4,2,6,[2,5,1]}。

A: 滑动窗口应当是队列，但为了得到滑动窗口的最大值，队列序可以从两端删除元素，因此使用双端队列。对新来的元素k，将其与双端队列中的元素相比较，前面比k小的，直接移出队列（因为不再可能成为后面滑动窗口的最大值了）,前面比k大的X，比较两者下标，判断X是否已不在窗口之内，不在了，直接移出队列，队列的第一个元素是滑动窗口中的最大值。

```python
class Solution:

    def maxInWindows(self, arr, size):
        if not arr or size <= 0:
            return []
        deque = []
        if len(arr) >= size:
            index = []
            for i in range(size):
                while len(index) > 0 and arr[i] > arr[index[-1]]:
                    index.pop()
                index.append(i)

            length = len(arr)
            for i in range(size, length):
                deque.append(arr[index[0]])
                while len(index) > 0 and arr[i] >= arr[index[-1]]:
                    index.pop()
                # 超出滑动窗口大小
                if len(index) > 0 and index[0] <= i - size:
                    index.pop(0)
                index.append(i)

            deque.append(arr[index[0]])
        return deque
```

## 66. 矩阵中的路径

Q: 请设计一个函数，用来判断在一个矩阵中是否存在一条包含某字符串所有字符的路径。路径可以从矩阵中的任意一个格子开始，每一步可以在矩阵中向左，向右，向上，向下移动一个格子。如果一条路径经过了矩阵中的某一个格子，则之后不能再次进入这个格子。 例如 [[a, b, c, e], [s, f, c, s], [a, d, e, e]] 这样的 3 X 4 矩阵中包含一条字符串”bcced”的路径，但是矩阵中不包含”abcb”路径，因为字符串的第一个字符b占据了矩阵中的第一行第二个格子之后，路径不能再次进入该格子。

A: 使用回溯法解决。任选一个格子作为路径的起点。假设矩阵中某个格子的字符为 ch 并且这个格子将对应于路径上的第 i 个字符。如果路径上的第 i 个字符不是 ch，那么这个格子不可能处在路径上的第i个位置。如果路径上的第 i 个字符正好是 ch，那么往相邻的格子寻找路径上的第 i+1 个字符。除在矩阵边界上的格子外，其他各自都有4个相邻的格子。重复这个过程直到路径上的所有字符都在矩阵中找到相应的位置。

```python
class Solution:

    def hasPath(self, matrix, rows, cols, path):
        if not matrix or rows < 1 or cols < 1 or not path:
            return False
        visited = [0] * (rows * cols)

        pathLength = 0
        for row in range(rows):
            for col in range(cols):
                if self.hasPathCore(matrix, rows, cols, row, col, path,
                                    pathLength, visited):
                    return True
        return False

    def hasPathCore(self, matrix, rows, cols, row, col, path, pathLength,
                    visited):
        if len(path) == pathLength:
            return True

        hasPath = False
        if 0 <= row < rows and 0 <= col < cols and \
                matrix[row * cols + col] == path[pathLength] and \
                not visited[row * cols + col]:

            pathLength += 1
            visited[row * cols + col] = True

            hasPath = self.hasPathCore(matrix, rows, cols, row, col - 1, path,
                                       pathLength, visited) or \
                      self.hasPathCore(matrix, rows, cols, row - 1, col, path,
                                       pathLength, visited) or \
                      self.hasPathCore(matrix, rows, cols, row, col + 1, path,
                                       pathLength, visited) or \
                      self.hasPathCore(matrix, rows, cols, row + 1, col, path,
                                       pathLength, visited)

            if not hasPath:
                pathLength -= 1
                visited[row * cols + col] = False

        return hasPath
```

## 67. 机器人的运动范围

Q: 地上有一个 m 行和 n 列的方格。一个机器人从坐标 (0,0) 的格子开始移动，每一次只能向左，右，上，下四个方向移动一格，但是不能进入行坐标和列坐标的数位之和大于k的格子。 例如，当k为18时，机器人能够进入方格 (35,37)，因为 3+5+3+7=18。但是，它不能进入方格 (35,38)，因为3+5+3+8=19。请问该机器人能够达到多少个格子？

A: 把方格看成一个 m*n 的矩阵，从 (0, 0) 开始移动。当准备进入坐标 (i, j) 时，通过检查坐标的数位来判断机器人能否进入。如果能进入的话，接着判断四个相邻的格子。

```python
class Solution:

    def movingCount(self, threshold, rows, cols):
        visited = [False] * (rows * cols)
        count = self.movingCountCore(threshold, rows, cols, 0, 0, visited)
        return count

    def movingCountCore(self, threshold, rows, cols, row, col, visited):
        count = 0
        if self.check(threshold, rows, cols, row, col, visited):
            visited[row * cols + col] = True
            count = 1 + self.movingCountCore(threshold, rows, cols, row - 1,
                                             col, visited) + \
                    self.movingCountCore(threshold, rows, cols, row + 1, col,
                                         visited) + \
                    self.movingCountCore(threshold, rows, cols, row, col - 1,
                                         visited) + \
                    self.movingCountCore(threshold, rows, cols, row, col + 1,
                                         visited)
        return count

    def check(self, threshold, rows, cols, row, col, visited):
        """判断是否可以进入(row, col)"""
        if 0 <= row < rows and 0 <= col < cols and \
                self.getDigitSum(row) + self.getDigitSum(col) <= threshold and \
                not visited[row * cols + col]:
            return True
        return False

    def getDigitSum(self, number):
        res = 0
        while number > 0:
            res += (number % 10)
            number = number // 10
        return res
```

## 参考

* [https://kaiyuanyokii2n.com/offer-python.html](https://kaiyuanyokii2n.com/offer-python.html)