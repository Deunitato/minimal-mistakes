---
published: true
---
## Valid Parentheses (20) : [Link](https://leetcode.com/problems/valid-parentheses/)

# Attempt one
```python
def isValid(self, s: str) -> bool:
        #back = {']', '}', ')'}
        pair = {'[' : ']', '{': '}' , '(' : ')'}
        # # if len(s) == 0:
        #     return True
        # elif len(s) == 1:
        #     return False
        
        queue = []
        #queue.append(s[0])
        index = 0
        while index < len(s):
            
            curr = s[index]
            
            if curr not in pair and len(queue) == 0:#Back
                return False
            
            if curr in pair and len(queue) == 0 :
                queue.append(curr)
                index = index + 1
                continue
                
            ne = curr
            find = queue[len(queue) - 1] #Latest front
            
            if pair[find] != ne: #Not matching bracket
                if ne not in pair: #this is back
                    return False
                queue.append(ne)
                
            else: #We found a matching bracket
                del queue[len(queue)- 1]
            
            index = index + 1
                
        if len(queue) > 0:
            return False   
            
        return True
```

My Explaination
- Using a stack, go through the string one index at a time
- If the index is a front brace, put it into the stack
- If its a back, peek the stack and see if its matches
	- No: False
    - Yes: Pop the stack and move on to the next index