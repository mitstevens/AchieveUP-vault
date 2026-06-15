Within react a page is more of a **Coordinator**, not worker. They hold just enough logic to wire everything together, but the heavy lifting lives elsewhere. 

What is usually within a page: 
**[[useState]]:** calls for local UI state 
**useEffect:** calls to trigger data fetching when the page loads or something changes
**Calls to custom hooks:** like useAuth() or useCourses() 
**Handlers:** Like onSubmit, onClick, or onChange, short functions to respond to user actions. 
**Return block:** the block that returns the JSX layout. 

Sometimes within a page: 
**Small data transformations:** ex. Reshape the API response before storing it in state 
**Conditional rendering logic:** ex. show a spinner vs error vs content 

Rarely / Never in a well-organized codebase: 
**The actual API call code:** This belongs in a service layer like api.ts 
**Reusable UI pieces:** Something like a nav bar, these belong in components 
**Complex business logic:** Belongs in a hook or utility file 

Typically a page is a single functional component that gets exported - outside of that component you just have: Imports, Type/Interface definitions, constants that never change, helper functions that don't need access to state. 
If something needs access to state, it has to live inside the component. 
