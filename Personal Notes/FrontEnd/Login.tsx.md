### Overview: 
Login is a [[page]] - it collects information from the user, validates it, then calls the login function from the auth system, then redirects to the dashboard on success.

### LoginFormInputs: 
A interface that just includes email and password. Not necessary, but a choice by the developer, makes it easy to see the format for a login form input. 

## Login:
This is the core of this file - it is a "React functional component" - a regular JavaScript/TypeScript function that returns JSX. The "React.FC" means it a function component. 

### Set up: 
We get **login** from useAuth() 
Get navigate from useNavigate() (used for changing URL / pages) 

call [[useState]] to update the variable showPassword from the function setShowPassword, starting with a default of false. 

call [[useForm]] and destructure it to get: **register**, **handleSubmit**, **errors**, and **isSubmitting**. We pass it `<LoginFormInputs>`, but that is only a TypeScript generic, informs it what the return type should be, it is a *purely a hint*, and it does *NOT change how the code runs*. 

### onSubmit: 
async function, takes in data in the type of LoginFormInputs. Calls login (from useAuth) with the email and password. Calls navigate with "/". Gives a popup message using toast that says "Welcome back, instructor!". If any errors, give a popup using toast giving the error message, or telling them that the login failed. 

### Return: 
This is all of the UI / Jsx code. This renders the UI and what you actually see on the page. 

Classname strings like "min-h-screen bg-gradient-to-br from-gray-50 to-gray-100 flex items-center justify-center p-4" are **Tailwind** for formatting / styling. 

Interesting react thing: 
``` tsx
                <input
                  {...register('email', { 
                    required: 'Email is required',
                    pattern: {
                      value: /^[A-Z0-9._%+-]+@[A-Z0-9.-]+\.[A-Z]{2,}$/i,
                      message: 'Please enter a valid email address'
                    }
                  })}
                  type="email"
                  className="w-full pl-10 pr-3 py-2 border border-gray-300 rounded-lg focus:outline-none focus:ring-2 focus:ring-primary-500 focus:border-transparent"
                  placeholder="Enter your email"
                />
```
This is an input element. There are three main things happening: 
1. The spread operator (...) essentially takes the things that are returned, and assigns them default properties (props) inside the input tag, instead of having to manually do it. The register line - register is called with two arguments. One is **email**, the second is the object with **required** (string), and **pattern** (object, containing two strings). 
2. type = "email": tells the browser that the field is an email, will do a basic format check on top of react-hook-form's. 
3. classname and placeholder: tailwind styling and placeholder text. 
Register does the main heavy lifting of this. This is standard code for a form anywhere throughout this react application. 