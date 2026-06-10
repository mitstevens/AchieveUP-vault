---
tags: [concept, frontend]
---

# Tailwind CSS

Utility-first CSS: instead of writing CSS files, you compose tiny single-purpose classes directly in JSX.

```tsx
<div className="bg-white rounded-lg shadow-sm p-6 border border-gray-200">
//   white bg | rounded corners | subtle shadow | padding 1.5rem | 1px gray border
```

## Decoder ring (the classes you'll see most in this repo)
| Pattern | Meaning |
|---|---|
| `p-6 px-4 py-2 m-8 mb-4` | padding/margin (number × 0.25rem); `px` = horizontal only |
| `text-sm text-gray-500 font-medium` | font size / color / weight |
| `bg-blue-100 text-blue-800` | background/text color + shade (50–900) |
| `flex items-center justify-between` | flexbox row, vertically centered, spread apart |
| `grid grid-cols-3 gap-6` | 3-column grid |
| `w-8 h-8 min-h-screen max-w-7xl` | sizing |
| `rounded-full border-b-2` | fully round / bottom border |
| `hover:bg-gray-50 focus:ring-2 disabled:opacity-50` | state variants |
| `md:grid-cols-3 sm:px-6` | responsive breakpoints (mobile-first) |
| `animate-spin` | spinner animation (see every loading state) |

## Project specifics
- `tailwind.config.js` defines the custom **`ucf-gold`** color (used as `bg-ucf-gold`, `text-ucf-gold`, `focus:ring-ucf-gold`) — UCF branding.
- Both the frontend and the extension repo have Tailwind configs, but the extension's main views mostly use plain CSS files instead.
- `postcss.config.js` is just plumbing that lets the build understand `@tailwind` directives in `index.css`.

Related: [[React Concepts]]
