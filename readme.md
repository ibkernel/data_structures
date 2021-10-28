# Data Structures implemented in python


## Tree

```python
class TreeNode:

  def __init__(self, val=None, left=None, right=None):
    self.val = val
    self.left = left
    self.right = right


def printTree(node, level=0):
    if node != None:
        printTree(node.left, level + 1)
        print(' ' * 4 * level + '->', node.val)
        printTree(node.right, level + 1)

```


### Binary Tree

```python

def build_binary_search_tree(nums):
  def build(root, num):
    if not root:
      return TreeNode(num)
    if root.val >= num:
      root.left = build(root.left, num)
      return root
    else:
      root.right = build(root.right, num)
      return root
  root = None
  for n in nums:
    root = build(root, n)
  return root


def build_balanced_binary_search_tree(nums):
  def build_balanced_nums(start, end):
    # add middle into array on every iteration
    nonlocal nums, balanced_nums
    if start > end:
      return

    if start == end:
      balanced_nums.append(nums[start])
      return

    mid = end + (start - end) // 2
    build_balanced_nums(mid, mid) # current middle
    build_balanced_nums(start, mid-1) # left 
    build_balanced_nums(mid+1, end) # right    
  nums.sort()
  balanced_nums = []
  build_balanced_nums(0, len(nums) -1)
  return build_binary_search_tree(balanced_nums)
  
```


### Segment Tree (useful for finding min/max/sum value within any range in array
```python
import math


# Tree version
def build_segment_tree(arr, start, end):
  """return root of segment tree"""
  if start == end:
    return TreeNode(arr[start])
  mid = end + (start - end) // 2
  left = build_tree(arr, start, mid)
  right = build_tree(arr, mid+1, end)
  root = TreeNode(min(left.val, right.val), left, right)
  return root

def query_segment_tree(root, start, end, query_start, query_end):
  if query_start <= start and end <= query_end:
    return root.val
  elif query_start > end or query_end < start:
    return math.inf
  else:
    mid = end + (start - end) // 2
    return min(query_tree(root.left, start, mid, query_start, query_end),
               query_tree(root.right, mid+1, end, query_start, query_end))


# arr = [i for i in range(1, 6)]
# root = build_segment_tree(arr, 0, len(arr)-1)
# print( query_segment_tree(root, 0, len(arr)-1, 3, 4 ) )


# Array version
class SegmentTreeArrayHelper:

  @classmethod
  def init_trees(cls, arr, init_val=0):
    # if len(arr) is power of 2
    length = len(arr)
    power_of_two = cls.get_next_power_of_2(length)
    return [init_val] * (2 * power_of_two - 1)

  @classmethod
  def get_next_power_of_2(cls, num):
    assert num >= 0
    if num == 0:
      return 0

    # is power of 2
    if (num & (num - 1) == 0) and num != 0:
      return num
    else:
      # get next power of 2
      # bin() always return a string starting with '0b1 if num is greater than zero'
      leading_two_pos = len(bin(num)[2:])  
      return int(math.pow(2, leading_two_pos))

  @classmethod
  def build_min_segment_tree(cls, nums):
    trees = cls.init_trees(nums, math.inf)
    cls.build_segment_tree(nums, trees, 0, 0, len(nums)-1, 'min')
    return trees

  @classmethod
  def build_max_segment_tree(cls, nums):
    trees = cls.init_trees(nums, -math.inf)
    cls.build_segment_tree(nums, trees, 0, 0, len(nums)-1, 'max')
    return trees

  @classmethod
  def build_sum_segment_tree(cls, nums):
    trees = cls.init_trees(nums, 0)
    cls.build_segment_tree(nums, trees, 0, 0, len(nums)-1, 'sum')
    return trees

  @classmethod
  def build_segment_tree(cls, nums, tree, pos, start, end, kind='min'):
    if start == end:
      tree[pos] = nums[start]
      return

    mid = end + (start - end) // 2
    left_child_pos, right_child_pos = pos*2 + 1, pos*2 + 2
    cls.build_segment_tree(nums, tree, left_child_pos, start, mid, kind) 
    cls.build_segment_tree(nums, tree, right_child_pos, mid+1, end, kind)  

    if kind == 'min':
      tree[pos] = min(tree[left_child_pos], tree[right_child_pos])
    elif kind == 'max':
      tree[pos] = max(tree[left_child_pos], tree[right_child_pos])
    else:  # sum
      tree[pos] = tree[left_child_pos] + tree[right_child_pos]

  @classmethod
  def query_max(cls, nums, tree, query_range_start, query_range_end):
    return cls.query_segment_tree(tree, 0, query_range_start, query_range_end, 
                                         0, len(nums)-1, kind='max')

  @classmethod
  def query_min(cls, nums, tree, query_range_start, query_range_end):
    return cls.query_segment_tree(tree, 0, query_range_start, query_range_end, 
                                         0, len(nums)-1, kind='min')

  @classmethod
  def query_sum(cls, nums, tree, query_range_start, query_range_end):
    return cls.query_segment_tree(tree, 0, query_range_start, query_range_end, 
                                         0, len(nums)-1, kind='sum')

  @classmethod
  def query_segment_tree(cls, tree, pos, query_range_start, query_range_end, 
                         start, end, kind='min'):
    # full match, return node value
    if query_range_start <= start and end <= query_range_end:
      return tree[pos]
    
    # partial match
    elif start <= query_range_start <= end or start <= query_range_end <= end:
      mid = end + (start - end) // 2
      left = cls.query_segment_tree(tree, pos*2 + 1, query_range_start, query_range_end,
                                    start, mid, kind)
      right = cls.query_segment_tree(tree, pos*2 + 2, query_range_start, query_range_end,
                                     mid+1, end, kind)
      if kind == 'min':
        return min(left, right)
      elif kind == 'max':
        return max(left, right)
      else:
        return left + right

    # no match
    else:
      if kind == 'min':
        return math.inf
      elif kind == 'max':
        return -math.inf
      else:
        return 0


# arr = [i for i in range(1, 6)]
# tree = SegmentTreeArrayHelper.build_sum_segment_tree(arr)
# print(arr, '\n', tree)
# print( SegmentTreeArrayHelper.query_sum(arr, tree, 0, 2)  )

```

## Union Find (Disjoint Set)

### Quick find
```python
class UnionFind:
    def __init__(self, size):
        self.root = [i for i in range(size)]

    def find(self, x):
        return self.root[x]
		
    def union(self, x, y):
        rootX = self.find(x)
        rootY = self.find(y)
        if rootX != rootY:
            for i in range(len(self.root)):
                if self.root[i] == rootY:
                    self.root[i] = rootX

    def connected(self, x, y):
        return self.find(x) == self.find(y)

```

### Quick union by rank
```python
# UnionFind class
class UnionFind:
    def __init__(self, size):
        self.root = [i for i in range(size)]
        self.rank = [1] * size  # height of tree at each node

    def find(self, x):
        while x != self.root[x]:
            x = self.root[x]
        return x
		
    def union(self, x, y):
        root_x = self.find(x)
        root_y = self.find(y)
        if root_x != root_y:
            if self.rank[root_x] > self.rank[root_y]:
                # root_x is longer than root_y
                # add root_y into root_x
                self.root[root_y] = root_x
            elif self.rank[root_x] < self.rank[root_y]:
                # root_y is longer than root_x
                # add root_x into root_y
                self.root[root_x] = root_y
            else:  # equal
                # add root_y into root_x tree
                # update root_x height
                self.root[root_y] = root_x
                self.rank[root_x] += 1

    def connected(self, x, y):
        return self.find(x) == self.find(y)
```


