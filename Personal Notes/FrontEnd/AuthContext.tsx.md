This file handles authentication system for the entire app. It has **AuthProvider** inside of it, which [[App.tsx]] wraps the entire app in. 

**AuthContext** = Storage bucket 
**AuthProvider** = Component that owns the state, runs the logic, and puts data into the bucket 
**useAuth** = How a component gets data from the bucket

Similar to AuthContext being a class, AuthProvider being an object, and useAuth being a static method that returns that instance from anywhere in the app. 

### AuthContextType: 
Interface that is the public API of the auth system. This is like the blueprint for the AuthProvider object that will be created. This stores: **User**, **loading** (true while app is validating a stored token), **login** (login function), **signup** (signup function), **logout** (logout function), **refreshUser** (re-fetches current user's data without requiring a full log out and log in). **isAuthenticated** (self explanatory), **backendAvailable** (false if backend can't be reached). 

### AuthContext: 
This is the context itself. This is what allows the state to be access throughout anywhere in the application. We initialize it to be undefined, it won't get any value until AuthProvider mounts to it. 
**useAuth** is a custom hook. This is what allows every other component to reach into the context and access it. If context is  called outside of the AuthProvider tree (where they shouldn't be calling it) they get a clear error message back. 

### AuthProvider: 
This is what holds all the auth state, and logic. It makes that available to the rest of the app by filling the AuthContext slot with real data. 

**[[useState]] declarations:** 
Without these the state would constantly be reset. We use it to store the user, whether loading or not, and if the backend is available. 

**checkAuthStatus:** 
What this does: 
* retrieves a token. If there is no token set user to null. 
* `authAPI.me()` sends the token in the header, and retrieves the current user's profile data from the backend. 
* Gets whether they are a instructor or not.
* if not a instructor, remove token, set user to null. 
* If they passed all of the checks, call the hook setUser to set their data. **end** the try block. 
* inside the catch block, resolve any errors, remove token, set the user to null. 

**useEffect - checkAuthStatus:** 
Because the [] are empty, we are not calling anything to make this ever re render. This will run ONCE on page render, to check auth status, and will not render again. (essentially means just run once on mount)

**login:** 
what this does: 
* Takes email and password. Outputs a promise (boolean). 
* **Open try block**
* Set loading to true 
* Call login (this is calling the API which goes to the backend) save it to response 
* if we have a [[JWT]], we set that token to local storage (**possible vulnerability worth changing**)
* **Open try block** 
* using the token get the user profile, and from that profile get their data to *userData* 
* Check if they are a instructor or have no canvas token. 
* If NOT instructor, remove token, set user null
* set the user with their data, return true. 
* **End try block** 
* Error handling, if something failed within the try block, set minimal profile required, and accept login while logging the error. **This is NOT good practice.** Theoretically a student could trigger an error, and get set to instructor. That would only be on the frontend, and they would likely access an empty / useless account, so **security risk is low.** 
* **End try block
* Return false - unable to get a token, login failed. 
* Error handling: log any error messages. set backendAvailable to false if there is a network error. 
* Finally setLoading false.

**signup** 
What this does: 
* Takes a signupRequest, outputs a promise(boolean)
* **Open try block** 
* setLoading to true 
* overwrite the data so it is type "instructor" - this seems **odd** - does NOT seem like a good practice. Likely not a big issue, but strange nonetheless. 
* Call signup API 
* **If we got a token:**
* Set the token to local storage. (**Again using localStorage instead of cookie**)
* **Open try block** 
* Get the user data from the API 
* call setUser to update the user data. 
* Return true (signup successful) 
* **End try block
* Error handling - failed to get user data, but signup was successful, sets base details, returns true. 
* **End try block** 
* Return false - we failed to signup. 
* Error handling - catch and log any errors that occurred. 
* If network error, setBackendAvailable false
* Finally setLoading false 

**logout** 
Simple removes the token from localStorage, set's the user to null. 

**refreshUser**
What this does: 
* **Open try block** 
* Sends the [[JWT]] to the API, gets user data. 
* Checks if the user is a instructor or has no canvas token
* IF user is NOT an instructor, calls logout. 
* Call setUser to update their data with the data we retrieved before. 
* **End Try block** 
* Error handling: log an error and call logout. 

**Value** 
This is essentially an instantiation of our AuthProvider - it is a object that has the values of AuthContextType. 

**Return** 
Now we return an AuthContext.Provider, with the value we just created passed in, and children rendered inside it. 
