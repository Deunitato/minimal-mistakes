---
published: true
---
# Leetcode (Tree Traversal)

## Inorder binary tree traversal (94) : [Link](https://leetcode.com/problems/binary-tree-inorder-traversal/)

#### First Attempt

```python
class Solution:
    def inorderTraversal(self, root: TreeNode) -> List[int]:
        #lEFT ROOT RIGHT
        if root == None:
            return []
        stack = []
        stack.append(root)
        ans = []
        visited = set()
        count = 0
        while len(stack) != 0:
            # count = count + 1
            # if count == 20:
            #     return ans
            pointer = stack[len(stack)-1]
            del stack[len(stack)-1]
            
            
            if pointer.left != None and pointer.left not in visited:
                if pointer not in visited:
                    stack.append(pointer)
                if pointer.left not in visited:    
                    stack.append(pointer.left)
                continue
                
            # Add
            ans.append(pointer.val)
            visited.add(pointer)   
            
            # Check right
            if pointer.right != None:
                #stack.append(pointer) # save for later
                stack.append(pointer.right)
        return ans
```


#### Answered Attempt

Notes: Its a much cleaner code compared to the previous, takes less space

```python
class Solution:
    def inorderTraversal(self, root: TreeNode) -> List[int]:
        stack = []
        pointer = root
        ans = []
        while len(stack)!=0 or pointer!= None:
            
            while pointer != None:
                stack.append(pointer)
                pointer = pointer.left
            
            # THis is the most left
            pointer = stack[len(stack)-1]
            del stack[len(stack)-1]
            ans.append(pointer.val)
            
            #Check for right
            if pointer.right == None:
                pointer = None
            else:
                pointer = pointer.right
                
        return ans
```
