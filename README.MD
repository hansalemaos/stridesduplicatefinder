# Calculate overlapping values between two arrays and return the results as a DataFrame

## Tested against Windows 10 / Python 3.10 / Anaconda

## pip install stridesduplicatefinder

#### Problem: you have to lists of different sizes and want to find the overlapping values. 

## Using pure Python - working, but slow

#### all indices / same values

```python
a1=[1,2,3,4,5,6,7]
a2=[0,0,3,1,5,6,8,1,32,]
res1=[(index1, index2, value1, value2) for index1, value1 in enumerate(a1) for index2, value2 in enumerate(a2) if value1 == value2]
print(res1)
# [(0, 3, 1, 1), (0, 7, 1, 1), (2, 2, 3, 3), (4, 4, 5, 5), (5, 5, 6, 6)]
```

#### same indices / same values

```python
res2=[(index1, index2, value1, value2) for index1, value1 in enumerate(a1) for index2, value2 in enumerate(a2) if value1 == value2 and index1==index2]
print(res2)
# [(2, 2, 3, 3), (4, 4, 5, 5), (5, 5, 6, 6)]
```

## Using stridesduplicatefinder - numpy or numexpr

```python
from stridesduplicatefinder import get_overlapping

def test_numexpr():
    start = perf_counter()

    _ = get_overlapping(
        fu="a==b", a=a1, b=a2, numpy_or_numexpr="numexpr", same_index_required=False
    )
    print(f"numexpr test: {perf_counter() - start}")
    print(_)


def test_numpy():
    start = perf_counter()
    _ = get_overlapping(
        fu=lambda a, b: a == b,
        a=a1,
        b=a2,
        numpy_or_numexpr="numpy",
        same_index_required=False,
    )
    print(f"numpy test: {perf_counter() - start}")
    print(_)


def python_test():
    start = perf_counter()
    _ = [(i1, i2, a, b) for i2, a in enumerate(a1) for i1, b in enumerate(a2) if a == b]
    print(f"python test: {perf_counter() - start}")
    print(_[:10])



a1 = np.random.randint(1, 100, size=(19000,),dtype=np.int64)
a2 = np.random.randint(1, 100, size=(7777,),dtype=np.int64)
from time import perf_counter

python_test()
# python test: 13.229658300006122

test_numpy()
# numpy test: 0.5666937999994843

test_numexpr()
# numexpr test: 0.48387080000247806


```


```python

Calculate overlapping values between two arrays and return the results as a DataFrame.

Parameters:
- fu: function or string to be evaluated as a condition for overlap.
- a: First input array.
- b: Second input array.
- numpy_or_numexpr: 'numpy' or 'numexpr' indicating the evaluation method.
- same_index_required: If True, only return rows where index1 == index2.

Returns:
- A DataFrame with columns 'index1', 'value1', 'index2', 'value2' containing
  information about overlapping values.

Example Usage:
- To find overlapping values between two NumPy arrays:
  
  a1 = np.random.randint(1, 10, size=(100000,))
  a2 = np.random.randint(1, 10, size=(100,))
  df1 = get_overlapping(
	  fu="a==b", a=a1, b=a2, numpy_or_numexpr="numexpr", same_index_required=True
  )
  print(df1)
  

- To find overlapping values using a custom function:
  
  a1 = np.random.randint(1, 10, size=(100000,))
  a2 = np.random.randint(1, 10, size=(100,))
  df2 = get_overlapping(
	  fu=lambda a, b: a == b,
	  a=a1,
	  b=a2,
	  numpy_or_numexpr="numpy",
	  same_index_required=False,
  )
  print(df2)
  

- To find overlapping values between two arrays of strings:
  
  a1 = np.array(["aa", "b", "c", "d", "ee11", "f", "gg", "h", "i", "j"])
  a1 = np.repeat(a1, 1000)
  a2 = np.array(["aa", "b", "c", "ee11", "f", "gg"])
  a2 = np.repeat(a2, 1000)
  np.random.shuffle(a1)
  np.random.shuffle(a2)
  df3 = get_overlapping(
	  fu="a == b",
	  a=np.char.array(a1).encode("utf-8"),
	  b=np.char.array(a2).encode("utf-8"),
	  numpy_or_numexpr="numexpr",
	  same_index_required=True,
  )
  print(df3)
  
		#     index1  value1  index2  value2
	# 0        5       1       5       1
	# 1       20       8      20       8
	# 2       33       5      33       5
	# 3       34       1      34       1
	# 4       41       5      41       5
	# 5       43       2      43       2
	# 6       51       7      51       7
	# 7       52       1      52       1
	# 8       55       7      55       7
	# 9       57       1      57       1
	# 10      70       2      70       2
	# 11      74       8      74       8


	#          index1  value1  index2  value2
	# 0             0       4       8       4
	# 1             0       4      12       4
	# 2             0       4      13       4
	# 3             0       4      26       4
	# 4             0       4      53       4
	#          ...     ...     ...     ...
	# 1112213   99999       9      47       9
	# 1112214   99999       9      62       9
	# 1112215   99999       9      72       9
	# 1112216   99999       9      81       9
	# 1112217   99999       9      96       9
	# [1112218 rows x 4 columns]


	#          index1 value1  index2 value2
	# 0             1     gg       4     gg
	# 1             1     gg       5     gg
	# 2             1     gg      10     gg
	# 3             1     gg      13     gg
	# 4             1     gg      17     gg
	#          ...    ...     ...    ...
	# 5999995    9999      c    5978      c
	# 5999996    9999      c    5979      c
	# 5999997    9999      c    5990      c
	# 5999998    9999      c    5992      c
	# 5999999    9999      c    5995      c
	# [6000000 rows x 4 columns]


	#      index1   value1  index2   value2
	# 0        31    b'aa'      31    b'aa'
	# 1        40     b'b'      40     b'b'
	# 2        46    b'aa'      46    b'aa'
	# 3        47    b'gg'      47    b'gg'
	# 4        65     b'b'      65     b'b'
	# ..      ...      ...     ...      ...
	# 626    5966    b'aa'    5966    b'aa'
	# 627    5982     b'f'    5982     b'f'
	# 628    5985  b'ee11'    5985  b'ee11'
	# 629    5995     b'c'    5995     b'c'
	# 630    5996    b'gg'    5996    b'gg'
	# [631 rows x 4 columns]

The function computes the overlapping values based on the specified condition (function or string)
and returns a DataFrame with the results. If `same_index_required` is set to True, it filters
the results to include only rows where the indices match.
```