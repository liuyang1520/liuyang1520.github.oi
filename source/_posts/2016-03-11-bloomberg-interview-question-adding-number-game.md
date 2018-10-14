---
layout: post
title: "Bloomberg Interview Question: Adding Number Game"
date: 2016-03-11 14:28:40
comments: true
description: "Bloomberg Interview Question: Adding Number Game"
keywords: ""
categories: Application

tags:
- Python
- Algorithm
- Recursive
- Bloomberg
---

Discovered a programming question from an interview in Bloomberg. The question is posted on a BSS. My analysis and solution are attached.

### Problem
> Number Adding Game:
> Given a list with 2 numbers. New numbers are appending to the end of the list during whole game, following 3 conditions:
> 1. All numbers in list are greater than 0;
> 2. All numbers in list are unique;
> 3. All new numbers must be the difference value among any 2 numbers in the list.
> New numbers are added until the conditions cannot be satisfied anymore.
> The program should return or output all possibilities of all processes.

### Analysis
With a given list, the time complexity of generating a new number for list is O(n^2) (calculate all difference values) and O(1) (check whether exist using hash map).

As for generating all possibilities, function recursive calls or stack both can solve this issue.

My solution is naive. Since the depth of recursive calls is limited, also, it is time consuming for calculating new numbers. I think there may exist better solutions, using the common divisor or common multiple with mathematical patterns.

### Solution (Python)
{% highlight python %}
inputData = [5, 30]
res = []

def checkDiffList(nums):
	diffNums = []
	numsSet = set(nums)
	length = len(nums)
	for i in range(length):
		for j in range(i + 1, length):
			absRes = abs(nums[i] - nums[j])
			if absRes not in numsSet and absRes not in diffNums:
				diffNums.append(absRes)
	return diffNums

def mathGame(nums):
	res.append(nums)
	diffNums = checkDiffList(nums)
	for i in diffNums:
		mathGame(nums + [i])

mathGame(inputData)
print res
{% endhighlight %}

Output:
```
[[5, 30], [5, 30, 25], [5, 30, 25, 20], [5, 30, 25, 20, 15], [5, 30, 25, 20, 15, 10], [5, 30, 25, 20, 10], [5, 30, 25, 20, 10, 15]]
```

Discussions are welcomed if you have a better solution.
