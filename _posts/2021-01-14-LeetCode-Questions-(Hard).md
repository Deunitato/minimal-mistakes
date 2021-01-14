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

## Populating Next Right Pointers in Each Node (116): [Link](https://leetcode.com/problems/populating-next-right-pointers-in-each-node/)

#### First attempt

```python
"""
# Definition for a Node.
class Node:
    def __init__(self, val: int = 0, left: 'Node' = None, right: 'Node' = None, next: 'Node' = None):
        self.val = val
        self.left = left
        self.right = right
        self.next = next
"""

class Solution:
    def connect(self, root: 'Node') -> 'Node':
        if root == None:
            return root
        queue = []
        queue.append(root)
        pointer = root
        root.next = None
        
        while len(queue) != 0:
            temp = []
            while len(queue) != 0: #Load the values of queue
                pointer = queue[0]
                del queue[0]
                if pointer.left == None:
                    return root
                temp.append(pointer.left)
                temp.append(pointer.right)
            
            queue = temp.copy()
            for x in range(0, len(queue)):
                curr = queue[x]
                if(x == (len(queue) - 1)):
                    curr.next = None
                else:
                    curr.next = queue[x+1]
                
                      
                
          
```


## Number of island (200): [Link](https://leetcode.com/problems/number-of-islands/)

Question statement:

Given a grid, find the number of islands denoted by a '1'


### First attempt

Idea:
- Using DFS/BFS to search for island
- Iterate through all grid space
- change the grid when an island is found

```python
def numIslands(self, grid: List[List[str]]) -> int:
        #gridspace = len(grid) * len(grid[0])
        m = len(grid)
        n = len(grid[0])
        stack = []
        count = 0
        for x in range(m):
            for y in range(n):
                if grid[x][y] == '1':
                    #Locate the island
                    count = count + 1
                    stack.append((x,y))
                    while len(stack) != 0:
                        coorX,coorY = stack[len(stack)-1]
                        del stack[len(stack)-1]
                        if coorX < 0 or coorY < 0 or coorX >= m  or coorY >= n:
                            continue
                        point = grid[coorX][coorY]
                        grid[coorX][coorY] = '#'
                        if point == '1':
                            stack.append((coorX + 1,coorY))
                            stack.append((coorX - 1,coorY))
                            stack.append((coorX ,coorY + 1))
                            stack.append((coorX,coorY - 1))
        return count
```