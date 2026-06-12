A promise represents a value that you do not have yet. It is like a placeholder for the result of an operation still in progress. It has three stages: 
1. **Pending:** still waiting
2. **Fulfilled:** Finished, with the value 
3. **Rejected:** Failed, with the error 
In a way, it is like returning a pointer immediately, than filling in the value when it is ready. However, different from a pointer, you can NOT view it until it is finished. It notifies the code when it is ready, either with the returned value, or an error code. 