# Data Structures implemented in python


## Segment Tree (useful for finding min/max/sum value within any range in array
```
import math


class SegmentTreeHelper:

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
    trees = TreeHelper.init_trees(nums, math.inf)
    cls.build_segment_tree(nums, trees, 0, 0, len(nums)-1, 'min')
    return trees

  @classmethod
  def build_max_segment_tree(cls, nums):
    trees = TreeHelper.init_trees(nums, -math.inf)
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


#arr = [i for i in range(1, 6)]
#tree = SegmentTreeHelper.build_sum_segment_tree(arr)
#print(arr, '\n', tree)
#print( SegmentTreeHelper.query_sum(arr, tree, 0, 2)  )

```
