
### Notes about hooks: 
Hooks can only be called within function, not within a class. 

Hooks must *ALWAYS* execute in the same order. For that reason, hooks can NOT be within a conditional statement. - For that reason, they are usually at the top level of your react function, with nothing around them. 