### Import axios - 
used for processing requests to the backend. It is a promise-based HTTP client for asynchronous network requests to REST endpoints. 
We create a const "api" for Axios, that is how we use it. 

### Interceptors - 
A request interceptor runs before every request leaves the browser, and a response interceptor runs before a response comes back. These run EVERY TIME a call is made. They are attached *to the API we created with axios*. 

There are two interceptors: 

The **first** reads the [[JWT]], and adds a Authorization header - Essentially attaching the JWT to the call. 

The **second** checks if we have a 401 response error, (and NOT a network error), then removes the token from local storage, and redirects the user to the login page. (This essentially deletes the token, and requires the user to login again). 

## API Groups: 
There is a long list of API groups - these are essentially just a table of API endpoints / routes. 

The group names (Ex. export const skillMatrixAPI) are essentially just objects, and each route within is a function of that object. You call it using a dot (Ex. skillMatrixAPI.create) 

Looking at a specific individual call, we have 4 parts: 
**Name** - Ex. "`create`" 
**Input** - Ex. "`(data: CreateSkillMatrixRequest)`" - Tells it that the input must be shaped like a CreateSkillMatrixRequest, which is defined in src/types/ 
**Output** - Ex. ": `Promise<AxiosResponse<SkillMatrix>>"` - The SkillMatrix at the end means that the backend will send back a SkillMatrix object. AxiosResponse<...> wrapping means axios will send the data plus the status code and headers - essentially just using axios to send it - the Promise<...> means it ensures the data will come, but it won't be immediate (since it has to go to the server and back) - thus why it is called with "await". 
**Work** - Ex. "`=> api.post('/achieveup/matrix/create', data)`" - "api" is our axios object we created. We are using axios to send a POST request, and it will be sent to the base URL + `/achieveup/matrix/create` - data is the body, which is what we just built. The interceptor adds the JWT automatically, so we don't have to worry about that. 

*Other types of requests*
GET requests have no body; Everything they need is in the URL. 
**GET**: Read something
**POST** Create/do Something
**PUT** update something
**DELETE** remove something 

***Note about a GET request*** - it only sends the URL and the body. If there are labels in the output section of the request, those are just what is expected, they do not actually do anything or change what gets sent back. It is more like documentation that actual effect, *they do NOT change the output*. 
