# Let's talk Data Structures, Algorithms, and Python (Stacks, Heaps)

Okay, I'm sure you've all heard of Leetcode before (obviously). So I've been doing some Leetcode for the past few months just to keep my mind sharp and 
to try and learn some of the intricacies of Python that I don't think about too often in my day-to-day as a Data Analyst. There's been a few problems
that, believe it or not, I've really enjoyed. I was already well aware of what heaps are and why they're useful, when to use a sliding window function, 
binary search, and how a linked list works. What I have forgotten since I took Data Structures and Algorithms in my second year of University, 
however, is how to implement these things in Python (or any language for that matter). So I've gathered a few of my absolute favourite leetcode problems and I'm
going to go through each one (not incredibly detailed, as I don't want to bore you).

### Palindrome Number:
  #### Given an integer x, return true if x is a palindrome, and false otherwise.

This one is very simple, so I won't waste too much time on it. We use a stack, but we don't even pop anything! 
  
  There's a few ways we can find if x is a valid palindrome. Let's go over 2 of them quickly.

  One way is to simply cast x as a string, and then iterate through it in reverse while appending each iterated value to a stack. This
  stack can then be compared to x (the stack is in reverse, x is not). If x is equal to the stack, then we found a palindrome!

  ```
        if x < 0:
            return False
        elif x == 0:
            return True
        stack = []
        x = str(x)
        for i in x[::-1]:
            stack.append(i)
        
        return (True if x == ''.join(stack) else False)
  ```

Our space complexity is alright, but our time complexity is poor. The if clauses at the very start are just to catch the two easy cases to avoid our function having to run through the loop for no reason. A negative number can never be a palindrome so return False, and 0 is automatically a palindrome, so return True. The return statement should be self-explanatory, but the ''.join(stack) simply converts the stack, which is an array, to one number so that we can compare it. Easy right?

Let's do it in an even easier way, and then we can take a look at a more interesting problem next. 

```
	    if x < 0:
	    	return False
    
	    return str(x) == str(x)[::-1]
```

Self explanatory and Pythonic as ever. Python will automatically return True if the comparison is true and False otherwise. I should mention,
using [::-1] simply tells Python we want the string in reverse. So what it does is check if the string x is equal to the reversed string x.

Now let's get to heaps!

### Top-K Elements: 
##### Given an integer array nums and an integer k, return the k most frequent elements. Return the answer in any order.
  
  In Python a heap can be (very succinctly) described as an array of key:value pairs. This will be useful for us as a key:value pair will allow us to keep track of the count of our elements. While a stack can be visualized as a stack of books with LIFO format (Last In First Out), a heap can be visualized as a binary tree where each node has a value, and the value of each node is either greater than or equal to (in a max heap) or less than or equal to (in a min heap) the values of its children. A heap is not always necessarily fast, but as far as inserting/updating items is concerned, a heap is the more efficient way to do it.
  
  First we want to initialize a heap (in Python this is simply a dictionary). Put simply, this is because heaps are inherently faster to traverse
  than a stack (array). In this case, we want our "key" to be the current integer and our "value" to be the count of that integer. We also want to make a simply for loop which iterates through the input array and checks if the current value is already in 
  our heap. If it is, we want to add one to the value of the "key". If it's not in the heap, then we add it to the heap and assign the key (integer)
  a value of 1, which is the first count.

  ```
  class Solution:
    def topKFrequent(self, nums: List[int], k: int) -> List[int]:
    dict = {}

      for i in nums:
        if i not in dict:
          dict[i] = 1
        else:
          dict[i] += 1
```

Simple, so far. If a value isn't in the dict add it and give it a value of 1, and if it is increment the value by one.

This is where it gets just a tad tricky. In Python, we can't make our function sort a heap by a value just by calling sorted(). Instead,
we must first convert our dictionary into a list, so that we may manipulate specifically the second element in the tuple (the frequency).
Then we can sort the list by the frequency using a simple lambda function and by setting reverse=True, which means descending order in 
Python (no DESC here).

```
k_elements = list(dict.items())
sorted_k_elements = sorted(k_elements, key=lambda x: x[1], reverse=True)

sorted_k_elements = sorted_k_elements[:k]   #Only pull up to the k element in the array.

sorted_k_elements = [item[0] for item in sorted_k_elements]
```

First things first, you'll notice we've used dict.items(). This just means we want the list to include everything in the dictionary, as .items()
is the combination of .values() and .keys() which both do exactly what you'd think they do - pull their respective element in the dictionary.
So by pulling dict.items(), we're working with the key-value pairs entirely. 

The next thing you'll probably notice is the lambda function in our sorted() function. While this may look daunting, what it does is simple, it's
considering x as the tuple itself, and assigning it to just x at index 1, which is the second element in the tuple. This is simply telling our
sorter that we want to sort by the second element. 

Let's go over the tricky part now.
```
sorted_k_elements = sorted_k_elements[:k]

sorted_k_elements = [item[0] for item in sorted_k_elements]
```
The first line here is incredibly simple and intuitive, and just means we only want to pull up to the k element. The second line in this block is 
the tricky part, and involves something called list comprehension. Essentially this just means creating a list based on the values of a list.

This in the most Pythonic way is expressed as [expression for member in iterable]. Each item in the iterable "for item in sorted_k_elements" is a tuple,
but by writing item[0] we are telling Python that we only want the elements, not their associated frequencies to be in our new list. Our new list
is being created by putting square brackets around the function.

And that, is Top-K Elements!


