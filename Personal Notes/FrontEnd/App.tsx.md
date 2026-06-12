lthis is the *root* of the React app. It decides which page shows for which URL, and wraps everything with the auth system. It is mostly just wiring up other things, *NOT* unique logic. 

# Imports 
Standard imports at the top, necessary for the file. A lot of things from other pages or components that we need. 

# Protected Route Component 
Single sentence overview: 
*"Still checking? Spinner. Not logged in? Login page. Logged in? Show the page I'm wrapping."*

This solves the problem of something viewing a page they should not be able to view, when not logged in. So we wrap every private page with it in the routes section: 

``` HTML 
<ProtectedRoute>
	<Dashboard />
</ProtectedRoute>
```

This ensures that protected route decides whether it should be shown or not. 

Within the code, it uses ({ children }) - that children is the thing that gets wrapped. In the previous example, Dashboard is the children. 

In the line: 
``` JavaScript 
const { isAuthenticated, loading } = useAuth();
```
This asks the auth system if it is authenticated, and if it is still loading. 
After that, if(loading) it displays a animated spinner to visually show it is loading. If it is not authenticated, it redirects the user to the login page. If it is Authenticated **Return Children** which simply *returns the thing it was wrapped around* (Ex. Dashboard). 

# Student Progress Data
***THIS SHOULD NOT BE HERE !! THIS IS A PAGE, LIVING INSIDE app.tsx*** 

### This page should likely be moved from app.tsx to it's own page. 

*Copy pasted from claude code about the potential move it it's own page:*

  Gotcha 1 — the hidden dynamic imports. This is the one most likely to bite. Inside StudentProgress
  there are three mid-function imports (lines 79, 100, 221):

  const { instructorAPI } = await import('./services/api');

  That path is relative to App.tsx. From pages/StudentProgress.tsx it must become '../services/api'.
  Your editor will auto-fix the top-of-file imports when you move code, but it will not fix these
  buried ones — and because they're inside try/catch blocks, the mistake wouldn't crash the build; the
  page would just show "Failed to load courses" at runtime. After moving, grep the new file for
  ./services to be sure. (Same for BadgesDashboard: './components/...' → '../components/...'.)

  Gotcha 2 — a name collision already exists, dormant. There's also a TypeScript interface named
  StudentProgress in types/index.ts:99 (api.ts uses it). Today they coexist because App.tsx never
  imports that type. After the move they still won't conflict — the new page file doesn't use the type
  either — but if someone later adds import { StudentProgress } from '../types' to the page, the names
  will clash with the component. Naming the moved component StudentProgressPage would eliminate that
  trap permanently; keeping the name is also fine, just know it's there.

  Gotcha 3 — team coordination, not code. This is the only genuine downside: moving 600 lines is a
  giant diff on App.tsx. If a teammate has an unmerged branch that edits StudentProgress where it
  currently lives, your move guarantees them a painful merge conflict (git won't auto-resolve "edited
  here" vs. "moved there"). On a solo repo this is a non-issue; on your senior design team, say "I'm
  moving StudentProgress out of App.tsx" in the group chat first and do it when no one has open work in
  that file. Also do the move as its own commit with zero logic changes — that keeps it reviewable and
  makes git blame archaeology easy later.

The description of what it actually does: 
### State Variables:
Uses the [[useState]] method to allow things to have memory within the function components. 
Basic format: \[state, function to update the state] = useState (default value). 

### Effects: 
We have 2 effects, one loads the courses on component mount, the second loads student data when a course is selected. Explanation below: 

Because React re renders the entire function on every state change, if you called heavy operations directly in a function, it would overload your backend. To fix that, we want to call some things only once, or when needed. We use **useEffect** for that. 

``` JavaScript
  useEffect(() => {
    loadCourses();
  }, []);
```
useEffect simply: 
run this code when something in the list (\[]) changes. Because we gave it a empty list, nothing will change. So it runs *once* when the page is loaded. Then, when the page gets re rendered, react checks the list to see if anything changed, it didn't, so it does not get re rendered. This allows it to load ONLY on mount / initial use. 
Because the second effect has selectedCourse within the list, every time that selectedCourse is updated, React will run loadStudentData again, to update the student data. 

### Fetch functions: 
Many of these fetch functions use the [[async modifier]]. They are functions that the page uses, and update the state, thus why within the page directly. 

**Walk through:**
name = async (parameter the function takes : type of the parameter) => {the stuff the function does}
**Example:** 
``` JavaScript
const loadStudentData = async (courseId: string) =>
```

The code is also wrapped in a try block for error handling. 

In some cases, it uses destructuring. Example: 
``` JavaScript 
const { instructorAPI } = await import('./services/api');
const response = await instructorAPI.getCourseStudentAnalytics(courseId);
```
The { } means that when it gets sent back the entire object, it only wants to keep the variable with the exact name "instructorAPI", the rest get's discarded. 

### Conditional Rendering: 
A big block (~500 lines) of if shaped UI. Shows things like a spinner for loading, error boxes for errors, etc. React formatting - used condition && code as a if statement - common react practice. 

Note - There is some commented out code on lines 430-436 

### AppRoutes: 
Because react is a single-page application, we don't have actual pages like /dashboard anymore, but users would still expect that functionality to be the same. For that reason, we use a router, to ensure the right page is rendered. 
``` tsx
<protectedRoute>      {/* This makes sure that the user must be logged in to acces the route */}
	<Layout>          {/* This adds a navigation bar / page frame */}
		<Page />      {/* This is the page component itself - calls that component and renders what it returns */}
	</Layout> 
</ProtectedRoute> 
```

### App: 
App makes sure everything runs within Router, and AuthProvider to provide authentication to every component, as well as enable URL-based navigation. Within that, there is the AppRoutes, which was just covered and manages showing the right component, and Toaster, which manages pop-ups. 

Router connects React to the browser, watches the URL bar and browser history, and broadcasts the current URL to everything. From there, AppRoutes decides based on the URL Router broadcasts what page we should show. That is why AppRoutes is inside of Router. (Router knows where you are, makes no decisions, AppRoutes decides what to show based on Router's information). 

When a element changes, like the Router. It than re-renders along with all of its children components. (Re renders do NOT travel upward, so it doesn't cause AuthProvider to re-render). From there, it checks what changes, and only re draws things that changed (because of the virtual DOM). So if the page changed, but Toaster doesn't change, nothing changes with Toaster, but the new component will re draw. 