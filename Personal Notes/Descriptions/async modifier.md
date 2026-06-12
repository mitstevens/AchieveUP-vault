async in a function is a keyword (a modifier). It changes how the function behaves. It does two main things: 
1. Allows await inside the function. Without using async, await would cause a syntax error. 
2. It makes the function always return a [[promise]]. If you returned 5, the caller would get a promise that resolves to 5. 