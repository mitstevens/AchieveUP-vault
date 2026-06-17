### Overview: 
This is the first page that a authenticated user sees. It will fetch their courses and skill data from the backend, render a personalized home screen, and shows different layouts depending on student or instructor. 
*Note:* It appears that this was built to start the student dashboard, but not built out. The student dashboard really should be it's own page, with any shared functions split into a utility. 

### Interfaces: 
There are five interfaces in this file. They are all fairly straight forward, and easy to read. I will note the interesting things: 
**title: string;** = TypeScript enforcing that the variable is a string type. 
**type: "badge" | "progress" ..etc** -- This means it can be any of the exact string matches. It has to match one of them. 
**status?: 'completed' | 'in-progress' | 'pending' ;** -- The **?** means it is optional. If it exists, it must be one of the three exact string matches. 
**icon: React.ComponentType<{ className?: string }>;** -- icon is a field name / variable, react.ComponentType<> - that means that TypeScript is saying that it will hold a React component (the function/component itself, instead of the output of the function/component). The {className?:string } part means that it has an optional prop, and if it is passed, it MUST be a string. 

**Activity:** 
*type, title, description, time, status* 
**QuickAction:**
*title, description, icon, href, color, priority* 
**WorkflowStep:**
*id, title, description, status, href, icon* 
**DashboardStats:**
*totalSkills, earnedBadges, averageScore, recentActivity* 
**InstructorDashboardStats:**
*totalCourses, totalStudents, averageProgress, recentActivity*

## Dashboard: 
This is the core of this file - it is a "React functional component" - a regular JavaScript/TypeScript function that returns JSX. The "React.FC" means it a function component. 

### State calls: 
**useAuth:** We call useAuth, and save the "user" property. 
**[[useState]]:** We call useState to keep state on courses, and only allow it to be updated using setCourses. The default value is an empty array, and the type that should be held in courses will be a array of CanvasCourse objects. 
**useState:** We call use state to keep variable loading, only allow updating uses setLoading, with a default value of true. 
**useState:** We set stats as the variable, and update it with setStats. It will hold the type "DashboardStats", and be initialized as an object holding 4 properties, 3 of which set to 0, and one of which is an empty array. 
**useState:** We set instructorStats and only update it using setInstructorStats. The type of instructorStats should be set to InstructorDashboardStats, and it is initialized to 4 properties, 3 with 0, one with an empty array. 
**useState:** We set skillMatricesCount, update it using setSkillMatricesCount, it's type should be a number, and it's default is 0. 

``` JavaScript
const isInstructor = user?.canvasTokenType === 'instructor';
```
Set isInstructor as a variable. If there is a user, check whether the canvasTokenType is exactly "instructor" (returns true/false). if there is no user, that returns undefined, in which case it does not === "instructor" and returns false. 

### generateMockActivity: 
This function is sort of initializing activity information. It is not pretending to show fake data as the name might suggest, but adding next-step suggestions based on where the instructor is in their setup process "Set up Canvas Integration" etc. More than being "generateMockActivity" is generating a contextual to-do list based on current state. 

We create **activities**, which is a Activity array initialized to an empty array. Activity was a interface defined above, and conditionally we **push** to it, which is a JavaScript way to add an item to the end of a array. 

**If you have 0 courses:** We set description to connect canvas account to load courses. 
**If you have !== 0 courses and we have === 0 matrices:** We set description to "You have x courses loaded. Create a skill matrix to get started" 
**Else:** Set description to "Successfully created x skill matrices"
**If matricesCount > 0:** Set description "Use AI to automatically map your quiz questions to the skills you've defined"
**Else:** Set description to "Once you create a skill matrix, you can automatically map questions to skills". 
**Push:** Regardless of everything else, always push "progress" type with description "Track student skill development as they complete Canvas assignments"

**NOTES:** There are **random time stamps** that don't **appear** to make **any sense**. It sets things to things like 2, 5, 10, 15, and 30 minutes before they were created, instead of the current time. 

**Claude code explanation of how to FIX:** 
![[Screenshot from 2026-06-17 14-03-24.png]]

### loadDashboardData: 
This is an async function, using [[useCallback]]. It goes from lines 135-302  and is quite lengthy. 

If instructor: Load and set the course information, and load and set the totalMatrices count. 
Set the actualStudentCount to 0 or accurate student count based on the dashboardresponse. 
**If students === 0 and data.length > 0:** THIS IS THE WORST THING I HAVE SEEN IN THIS FILE. We use a hash to create a random (but consistent amount the same courses) number of students, and show that number to the instructor. It is literally trying to create a random number to show them. CLEARLY AI WRITTEN SLOP. This **SHOULD BE REMOVED** 
IF the previous code errors, we run it all again (the hashing algorithm and fake student number) and submit that. **SO IF THE PREVIOUS BAD CODE ERRORS ANYWHERE, WE RUN MORE BAD CODE**. This is terrible, duplicating BAD code, it must be removed. 
We then have an catch for if the API fails, we set to 0's. 

If not an instructor, we now enter this else block, which is for loading student data. We call the instructor API though, which obviously won't work. We mostly set things to 0s. There is not much going on. **This should be removed:** We are calling a instructor API to set student data which makes no sense. Students should have their own dashboard page, it shouldn't be here within the instructor one. This makes no sense. 

Finish with some error handling, fallback sets to 0. Than a finally setLoading to false. 

**useEffect:** We call useEffect on loadDashboardData - Because loadDashboardData is using [[useCallback]], it only actually changes reference if one of it's dependencies changes, which is the only thing that triggers it to re update. 

### getGreeting: 
Gets a string greeting message based on time, Good morning/afternoon/evening. 

### getTimeAgo: 
Calculates based on a time given, and the current time, how long ago the time given was. It returns a string: if less than a minute ago "Just now", if less than 60 minutes ago "x min ago", if less than 1440 minutes ago "x hr ago", else "x day ago". 

### workflowSteps: 
This created a array of interface type WorkflowStep[] with 3 entries. It reads from skillMatricesCount which is kept in state, so though it does update every re render, it will show the proper information. This is like setup information that is being shown. 

### instructorQuickActions: 
A static array of 4 objects matching the QuickAction[] interface. Shows essentially navigation cards that show in the "essential tools" section of the instructor dashboard. Static, no dynamic values, no state, no API calls. A hardcoded list that gets looped over in the JSX to render the cards. **Note** Since this has no need for state or anything within the component, it could be put outside the component so it only renders once on load, and is not constantly being rebuilt. Overall, it is a minor impact, not big. 

### studentQuickActions: 
Pretty much same as instructor, but with only 2 instead of 4. This should **get removed from the instructor dashboard** and built on it's own page. 

### JSX code: 
Not spending much time here, pretty tedious to read through. Some issues Claude pointed out: 
1. On line 494 it uses `<a>` tag instead of `<Link` - this causes the page to essentially full reload and lose state. Bad practice for react, could cause bugs with state being lost. 
2. The instructorState.recentActivity is used for instructors and students. The student portal isn't really bad, so this isn't an issue, but should likely be removed. 
3. The stats cards are hardcoded. Instead of being driven by an array, like quickActions, the instructor cards and student cards are each written out manually as individual `<Card>` components. If you ever wanted to add or change a card, you would have to do it manually. Just bad practice, not a bug. 
4. It uses key={index} throughout - it is better practice to use key = {id} so that if the positioning of things got changed, it still renders correctly. Again, not a bug, but bad practice. 




### Issues presented by Claude Code to investigate: 

  ```
  The biggest problem: duplicated code (~30 identical lines appear twice)
  
  The hash-based student count calculation is copy-pasted almost verbatim at lines ~176–199 and again at lines ~213–243. The only
  difference is they sit inside different catch blocks. This should be one helper function called in both places. As-is, a future change
  to that logic (say, adjusting the "lab" threshold) would need to be made in two places — and someone will inevitably only change one.
  
---

  The second notable issue: mock data baked into production paths
  
  Two things in this file present themselves as real data but aren't:

  - The activity feed is always constructed by generateMockActivity(). There's no backend activity log being read. What the user sees is
  entirely synthetic, derived from "how many courses and matrices do you have right now."
  - The student count is a deterministic hash of the course name, not data from Canvas or the backend. A course called "Intro to
  Computing" will always show ~40–60 students regardless of how many are actually enrolled.

  This isn't inherently wrong for a demo/MVP, but it's not labeled as temporary anywhere in the code. Someone joining the team (hi)
  could easily think those numbers are real.

  ---
  The student view is a skeleton
  
  The else branch for students (line 264) fetches courses and then sets all stats to zero. totalSkills, earnedBadges, averageScore never
  get populated. The student-facing stat cards just show 0 for everything, permanently. There's no TODO comment flagging this — it
  reads like finished code that happens to not work.

  ---
  Minor things
  
  - Several console.log and console.error debug statements are sprinkled throughout and were left in (lines ~155, 166, 209). Fine for
  development, noise in production.
  - key={index} used in most .map() calls. Harmless here since none of those lists reorder, but it's the one React warning you'd expect
  to see in the browser console.
  ```
  