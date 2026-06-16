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
This function generates face activity. It is not real data pulled from the API or database. It is a placeholder. It should **NOT EXIST in the final production code**. Currently this only shows if there is no real data. However, it is built in later to show in any case no student data is present, which could be confusing. 


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
  