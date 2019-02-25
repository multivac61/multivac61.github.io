+++
title = "Sorting algorithms in Python"
author = ["Olafur Bogason"]
date = 2015-10-03
lastmod = 2019-02-24T17:46:55+00:00
tags = ["python", "ai"]
categories = ["python"]
draft = false
+++

Recently I've been doing some coding in Python but I felt like I didn't really understand what was going on. So as a way to better understand python's syntax and semantics I implemented a couple of the better known sorting algorithms - for fun and the greater good!

This post showcases the code I wrote and talks about the hurdles I encountered while implementing the algorithms. This post is only meant to document my learning process of Python implementations and not really meant for learning about the mathematics behind the space- and time complexities of the different sort algorithms as I had already done that in my EE undergrad. All comments about implementation issues, bugs or pythonic styling are more than welcome!

<style>.org-center { margin-left: auto; margin-right: auto; text-align: center; }</style>

<div class="org-center">
  <div></div>

{{< figure src="https://upload.wikimedia.org/wikipedia/commons/thumb/c/c3/Python-logo-notext.svg/200px-Python-logo-notext.svg.png" >}}

</div>


## The simpler sorts {#the-simpler-sorts}

As a start I decided to go with the simplest looking sorts, insertion- and selection sort. They are both O(n^2) in time complexity and thus rarely ever used on large datasets. Their implementation is however straight forward. It is noteworthy to point out that I wanted each function to return a <strong>new list of numbers</strong> and that the functions shouldn't mutate the list passed in at all.


### Insertion sort {#insertion-sort}

<style>.org-center { margin-left: auto; margin-right: auto; text-align: center; }</style>

<div class="org-center">
  <div></div>

{{< figure src="https://upload.wikimedia.org/wikipedia/commons/0/0f/Insertion-sort-example-300px.gif" >}}

</div>

<blockquote>
  <p>Insertion sort is a simple sorting algorithm that builds the final sorted array (or list) one item at a time. It is much less efficient on large lists than more advanced algorithms such as quicksort, heapsort, or merge sort. -
  <a href="https://en.wikipedia.org/wiki/insertion_sort" title="Wikipedia: Insertion sort">Insertion sort</a></p>
</blockquote>

The search iterates through the unsorted array. Each time it passes on of its elements to an inner function insert<em>\_</em>item, which takes a list and element and inserts the element in the right place (as dictated by the passed in function sort\_by).

The sort by function takes as a default an [lambda function](https://pythonconquerstheuniverse.wordpress.com/2011/08/29/lambda%5Ftutorial/). For those of you who haven't heard about the built-in lambda functions I encourage you to check them out. They can come in handy to build up simple functions in a lucid manner, when using def is simply too much.

```python
def insertion_sort(m, sort_by=(lambda a, b: a < b)):
    def insert_item(m, item0):
        for i, item in enumerate(m):
            if sort_by(item0, item):
                m.insert(i, item0)
                return
            m.append(item0)

        if len(l) <= 1:
            return l

        sorted_list = []
        for item in l:
            insert_item(sorted_list, item)
        return sorted_list
```


### Selection sort {#selection-sort}

<blockquote>
<p>The algorithm [selection sort] divides the input list into two parts: the sublist of items already sorted, which is built up from left to right at the front (left) of the list, and the sublist of items remaining to be sorted that occupy the rest of the list. - <a href="https://en.wikipedia.org/wiki/selection_sort" title="Wikipedia: Selection sort">Selection sort</a></p>
</blockquote>

My implementation  starts off with the passed-in list and copies all elements from the list into a new place in memory (very important so that we don't mutate the passed in list) and assigns that object to the variable initial<em>\_</em>list. Then we iterate through that list and at each passing we find the "lowest" element as dictated by the sort<em>\_</em>by function as before. We then append that element on the back of our sorted<em>\_</em>list object and voilá! after passing through the outer loop once we have sorted the list passed in. We then return.

```python
def selection_sort(m, sort_by=(lambda a, b: a < b)):
     def find_next(m):
          next_elem = m[0]
          for i in m:
               if sort_by(i, next_elem):
                    next_elem = i
          return next_elem

     if len(m) <= 1:	# if list less than one element return
          return m

     initial_list = list(m)	# deepcopy of passed in list
     sorted_list = []
     while initial_list:
          next_min = find_next(initial_list)	# find
          sorted_list.append(next_min)
          initial_list.remove(next_min)

     return sorted_list
```


## The more efficient sorts {#the-more-efficient-sorts}

Going on down the sorting algorithms difficulty-ladder we next encounter sorts with O(n log(n)) time-complexities. Here is where recursion comes in and the implementations begin to become more interesting and non trivial.


### Merge sort {#merge-sort}

<style>.org-center { margin-left: auto; margin-right: auto; text-align: center; }</style>

<div class="org-center">
  <div></div>

{{< figure src="https://upload.wikimedia.org/wikipedia/commons/c/cc/Merge-sort-example-300px.gif" >}}

</div>

Conceptually, a merge sort works as follows: ([Wikipedia](https://en.wikipedia.org/wiki/merge%5Fsort))

-   Divide the unsorted list into n sublists, each containing 1 element (a list of 1 element is considered sorted).
-   Repeatedly merge sublists to produce new sorted sublists until there is only 1 sublist remaining. This will be the sorted list.

Merge sort was invented in the time of the tape machines by the legendary mathematician Jon von Neumann.

<style>.org-center { margin-left: auto; margin-right: auto; text-align: center; }</style>

<div class="org-center">
  <div></div>

{{< figure src="https://upload.wikimedia.org/wikipedia/commons/thumb/7/78/HD.3F.191%5F%2811239892036%29.jpg/383px-HD.3F.191%5F%2811239892036%29.jpg" >}}

</div>

My implementation gets straight to the point. First we split the list into single element list and then merge those small lists recursively so that at each point the smaller sub-lists are individually sorted. The function that has the merge functionality is maintained within the merge\_sort function itself for brevity and as not to dirty our global name-space.

```python
def merge_sort(m, sort_by):
    def merge(left, right):
        result = []
        while left and right:
            # keep on merging/sorting from left/right
            # lists on an element basis
            if sort_by(left[0], right[0]):
                result.append(left[0])
                left.pop(0)
            else:
                result.append(right[0])
                right.pop(0)
            # there might be elements left in left/right list
        for i in left:
            result.append(i)
        for i in right:
            result.append(i)
        return result

    if len(m) <= 1: # if list less than one element return
        return m

    middle = len(m) / 2	# split list in half
    left = m[:middle]; right = m[middle:]

    left = merge_sort(left, max_order)		# sort left
    right = merge_sort(right, max_order)	# sort right

    return merge(left, right)	# merge together sorted left/right
```


### Heap sort {#heap-sort}

<style>.org-center { margin-left: auto; margin-right: auto; text-align: center; }</style>

<div class="org-center">
  <div></div>

{{< figure src="https://upload.wikimedia.org/wikipedia/commons/1/1b/Sorting%5Fheapsort%5Fanim.gif" >}}

</div>

Heapsort can be thought of as an improved selection sort: like that algorithm, it divides its input into a sorted and an unsorted region, and it iteratively shrinks the unsorted region by extracting the largest element and moving that to the sorted region. The improvement consists of the use of a heap data structure rather than a linear-time search to find the maximum. [Heap sort](https://en.wikipedia.org/wiki/heap%5Fsort)

The abstract description of heap sort may make it seem a little daunting but it really isn't much more complex than merge sort at all. The functionality I had to implement was _heapify_, take a list and make a [heap data structure](https://en.wikipedia.org/wiki/Heap%5F%28data%5Fstructure%29) out of it and sift<em>\_</em>down, a function that returns a heap into a legal state when the top element has been removed from it.

My implementation goes like this: First create a heap using the _heapify_ function, basically placing each item of the list at the bottom of the heap and then sift the elements up (incorrect elements down) until the inserted element is at a place so that the heap is valid. Next I remove the top node of the heap, which we know will be the lowest/highest/which ever way you want to sort your list, place it in a new sorted<em>\_</em>list and then we make the heap valid again. We continue this procedure until there are no leafs left in the heap.

```python
def heap_sort(m, sort_by):
    def sift_down(m, start, end):
        """Repair heap whose root element is at index start
          assuming the heaps rooted at its children are valid"""
        root = start

        while 2*root+1 <= end:
            child = 2*root + 1	# left child
            swap = root

            if sort_by(m[swap], m[child]):
                swap = child	# swap if left child is 'larger'

                if child+1 <= end and sort_by(m[swap], m[child+1]):
                    swap = child + 1 # swap if right child is 'larger'
            if swap == root:
                  return
            else:
                  m[root], m[swap] = m[swap], m[root]
                  root = swap

        def heapify(m):
            """If A is a parent node of B then the key of node A is
            ordered with respect to the key of node B with the same
            ordering applying across the heap."""
            start = (len(m) - 2) / 2

            while start >= 0:
                sift_down(m, start, len(m)-1)
                start -= 1

            m = list(m)	# deep copy
            heapify(m)

            end = len(m) - 1
            while end > 0:
                m[0], m[end] = m[end], m[0]
                end -= 1
                sift_down(m, 0, end)

            return m
```


### Quick sort {#quick-sort}

<style>.org-center { margin-left: auto; margin-right: auto; text-align: center; }</style>

<div class="org-center">
  <div></div>

{{< figure src="https://upload.wikimedia.org/wikipedia/commons/6/6a/Sorting%5Fquicksort%5Fanim.gif" >}}

</div>

Quicksort is a divide and conquer algorithm. Quicksort first divides a large array into two smaller sub-arrays: the low elements and the high elements. Quicksort can then recursively sort the sub-arrays. The steps are:

1.  Pick an element, called a pivot, from the array.
2.  Reorder the array so that all elements with values less than the pivot come before the pivot, while all elements with values greater than the pivot come after it (equal values can go either way). After this partitioning, the pivot is in its final position. This is called the partition operation.
3.  Recursively apply the above steps to the sub-array of elements with smaller values and separately to the sub-array of elements with greater values. [Wikipedia](https://en.wikipedia.org/wiki/quick%5Fsort)

I proceeded using a similar approach as I did in merge sort, the functions that do the heavy lifting are maintained within the quick\_sort function itself. The implementation follows directly from the description on [Wikipedia](https://en.wikipedia.org/wiki/quick%5Fsort) and I was lazy so I just chose the last element to always be the pivot in each sub-list that is to be sorted.

```python
def quick_sort(m, sort_by):
      def partition(m, lo, hi):
            p = m[hi]	# choose a pivot
            i = lo
            for j in range(lo, hi):
                  if sort_by(m[j], p):
                        m[i], m[j] = m[j], m[i]
                        i += 1

            m[i], m[hi] = m[hi], m[i]
            return i

      def _quick_sort(m, lo, hi):
            if lo < hi:
                  p = partition(m, lo, hi)
                  _quick_sort(m, lo, p-1)
                  _quick_sort(m, p+1, hi)

      m = list(m)	# deep copy
      _quick_sort(m, 0, len(m)-1)

      return m
```

One thing I noticed is that the python interpreter will throw the error `RuntimeError: maximum recursion depth exceeded in cmp` if the test cases I tried were too large. This doesn't mean that my implementation is buggy but simply that the interpreter has a default recursion depth which can be modified by the user. More about that [here.](http://stackoverflow.com/questions/25105541/python-quicksort-runtime-error-maximum-recursion-depth-exceeded-in-cmp)


## Take aways {#take-aways}

These implementations were much easier than I had imagined and I didn't really have any big problems.

Next up: code a Search class, implement the different algorithms there and do some testing! I also want to try and implement some of the AI search algorithms (A\*, minimax &amp; alpha-beta pruning).
