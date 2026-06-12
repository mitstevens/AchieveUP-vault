useState is a built in [[Hook]] within React, which allows components to maintain and update their own local memory. When a state variable changes, react automatically triggers a re-render of the component to update the UI. 

Every time react re draws a page, a normal variable would be wiped and reset every time. 
``` JavaScript
let selectedCourse = '';   // ❌ resets to '' every single re-draw — useless
```
**useState** allows you to declare a variable whose value survives re-draws. 

Looking at one line: 
``` JavaScript
const [selectedCourse, setSelectedCourse] = useState<string>('');
```
**selectedCourse** is the *current value* - like a variable 
**setSelectedCourse** is the *function to change the value*. The *ONLY* way to change it. 
**''** is the *starting value*, the default. 
So, when you call setSelectedCourse('abc') - react will update the value and re run the component, so the page re draws showing the new update. 

"So a good habit when reading any component: read the useState lines first. They're the complete list of everything the page can remember, which tells you everything the page can do before you read a single line of UI." - Claude Sonnet, June 11, 2026. 
  