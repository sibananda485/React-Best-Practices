# **Clean Coding and Best Practices**

# Container Components & Presentational Component

## Purpose
Container components are responsible for **data fetching, state management, and business logic**, while delegating UI rendering to presentational components. This separation improves readability, testability, and long-term maintainability.

---

## Rule
> **A container component should not contain complex JSX or UI styling logic.**  
> **A presentational component should not contain business logic or side effects.**

---

## What Is a Container Component?
A container component:
- Fetches or prepares data
- Owns state and side effects
- Handles business logic and orchestration
- Passes data and callbacks as props

A container component **does not care how things look**.

---

## What Is a Presentational Component?
A presentational component:
- Focuses purely on UI
- Receives data via props
- Emits events via callbacks
- Has minimal or no internal state

A presentational component **does not care where data comes from**.

---

## ❌ Anti-Pattern (Mixed - Container and Presentation )
```tsx
function UserList() {
  const [users, setUsers] = useState([])
  const [loading, setLoading] = useState(false)

  useEffect(() => {
    setLoading(true)
    fetchUsers().then((data) => {
      setUsers(data)
      setLoading(false)
    })
  }, [])

  return (
    <div className="p-4">
      {loading && <Spinner />}
      {users.map((user) => (
        <div key={user.id} className="border p-2">
          <p>{user.name}</p>
          <button onClick={() => deleteUser(user.id)}>Delete</button>
        </div>
      ))}
    </div>
  )
}
```

### Why This Is Bad
- Business logic + UI are tightly coupled
- Harder to test
- Harder to refactor UI or logic independently

---

## ✅ Correct Pattern (Container + View)

### `UserList.tsx`
```tsx
import { UserListView } from './UserList.view'
import { useUserList } from './useUserList'

export function UserListContainer() {
  const {
    users,
    isLoading,
    onDeleteUser,
  } = useUserList()

  return (
    <UserListView
      users={users}
      isLoading={isLoading}
      onDeleteUser={onDeleteUser}
    />
  )
}
```

### `UserList.tsx`
```tsx
export function UserListView({ users, isLoading, onDeleteUser }) {
  if (isLoading) return <Spinner />

  return (
    <div className="p-4">
      {users.map((user) => (
        <div key={user.id} className="border p-2">
          <p>{user.name}</p>
          <button onClick={() => onDeleteUser(user.id)}>Delete</button>
        </div>
      ))}
    </div>
  )
}
```

---

## Benefits for the Team
- UI and logic evolve independently
- Easier unit testing
- Predictable component responsibilities
- Cleaner pull requests

---

## When NOT to Create a Container
Do **not** introduce a container if:
- The component is trivial
- No business logic exists
- No side effects or state orchestration

> **Rule of thumb:** Extract a container when logic exceeds ~20–30 lines.

---

## Code Review Checklist
- [ ] Is business logic inside a container or hook?
- [ ] Is the view component free from side effects?
- [ ] Is JSX complexity kept out of containers?
- [ ] Are props clearly typed and minimal?

---

## Summary
Container components are **coordination layers**, not UI layers.  
They make the codebase scalable, testable, and team-friendly.

> **Think:** Containers think. Views render.

# UI / Primitive Components (Design System)

## Purpose
These components are very small atomic or molecule UI component for reusing UI building block. These are highly resuable, nestable and customizable. Doesn't contain any business logic. Barely contains state (That is also contollable).

## Team
> **Do not worry to maintain UI components we already have a design system - ShadCN + Radix Components which interally uses Radix primitive for API.**  
> **Feel free to customize them according to our need**

Examples:

- Button
- Input
- TextArea
- Tabs
- Select


# Page / Route Components

## Purpose
This is the only Component which is rendered on a specific route.
The primary resposibility should be composing multiple components (Mostly container components) by applying stying and layouting on them.
It should not contain any business logic as well and much jsx. Yes it may contain any effects related to routing, that is fine.


> **Rule of thumb:** Eventually Do not make a page component into a container component.



# Single Responsibility Principle (SRP)

## Purpose
The Single Responsibility Principle ensures that **each unit of code has one clear reason to change**. In React, this applies to **components and hooks**.

Following SRP keeps the codebase:
- Easier to understand
- Easier to test
- Safer to change
- More predictable at scale

---

## Rule
> **A component, hook, or file must have exactly one primary responsibility.**  
> **If you need to explain it using “and”, it probably violates SRP.**

---

## SRP in React (Practical Interpretation)
In React, SRP means:
- Components do **one job** (render, compose, or coordinate)
- Hooks encapsulate **one slice of logic**

---

## ❌ SRP Violation Example
```tsx
function CompanyTable() {
  const [companies, setCompanies] = useState([])
  const [companyTickers, setCompanyTicker] = useState([])
  

  useEffect(() => {
    fetchCompanies().then(setCompanies)
  }, [])
  useEffect(() => {
    fetchCompanyTicker().then(setCompanyTicker)
  }, [])

  return (
   ///
  )
}
```

### Why This Violates SRP
This component:
- Multiple data Fetching
- Manages unrelated state in single component
- Applies business logic
- Renders UIs

That is **multiple reasons to change**.

---

## ✅ SRP-Compliant Refactor

### `useCompanies.js`
```ts
export function useCompanies() {
  const [companies, setCompanies] = useState([])
  

  useEffect(() => {
    fetchCompanies().then(setCompanies)
  }, [])

  return companies
}
```
### `useCompanyTicker.js`
```ts
export function useCompanyTicker() {
  const [companyTickers, setCompanyTicker] = useState([])
  
  useEffect(() => {
    fetchCompanyTicker().then(setCompanyTicker)
  }, [])

  return companyTickers
}
```

### `CompanyTable.tsx`
```tsx
export function CompanyTable() {
  const companies = useCompanies()

  return <Table rows={companies} />
}
```

### `CompanyTicker.tsx`
```tsx
export function CompanyTicker() {
  const companyTickers = useCompanyTicker()

  return <Tickers data={companyTickers} />
}
```

### `CompanyManagementPage.tsx`
```tsx
export function CompanyTicker() {

  return ( <div className="p-5 space-y-4">
            <CompanyTicker/>
            <CompanyTable/>
        </div>
  )
}
```
Responsibility splitting and separation of concern
- `useCompanies` -> Hooks that handles fetching companies
- `useCompanyTicker` -> Hooks that handles fetching company summary
- `CompanyTable` -> Container component that passes data and deligate responsibility to Table component (Presentation component) for ui
- `CompanyTicker` -> Container component that passes data and deligate responsibility to Tickers component (Presentation component) for ui
- `CompanyManagementPage` -> Does nothing just compose two componenent and ad 

Each unit now has **one reason to change**.


> **Think:** One unit. One job. One reason to change.

# Custom Hooks

## Purpose

Custom hooks are the **primary unit of business logic** in our React codebase. They exist to extract logic out of components so that:

* Containers stay thin
* Views stay purely presentational
* Logic becomes reusable, testable, and predictable

In our architecture:

* **Components compose UI**
* **Hooks own logic**

---

## Rule

> **If logic is stateful, effectful, reusable, or business‑driven — it MUST live in a custom hook.**
> **Components must consume hooks, not implement logic themselves.**

---

## What Belongs in a Custom Hook

A custom hook may:

* Manage local state
* Perform side effects (`useEffect`)
* Fetch server data (directly or via libraries like TanStack Query)
* Encapsulate business rules
* Coordinate multiple concerns into a single abstraction

### Examples

* `useUserList`
* `useCreateUserForm`
* `useDebouncedValue`

---

## What Must NOT Go in a Custom Hook

A custom hook must **never**:

* Return JSX
* Contain layout or styling logic
* Depend on specific UI components
* Perform navigation directly (unless injected)

> Hooks are **logic‑only** units.

---

## ❌ Anti‑Pattern: Logic Inside Component

```tsx
function UserListContainer() {
  const [users, setUsers] = useState([])

  useEffect(() => {
    fetchUsers().then(setUsers)
  }, [])

  return <UserListView users={users} />
}
```

### Why This Is Wrong

* Container becomes heavy
* Logic cannot be reused
* Harder to test
* Violates SRP

---

## ✅ Correct Pattern: Logic Inside Hook

### `useUserList.js`

```ts
export function useUserList() {
  const [users, setUsers] = useState([])

  useEffect(() => {
    fetchUsers().then(setUsers)
  }, [])

  return { users }
}
```

### `UserList.tsx`

```tsx
export function UserListContainer() {
  const { users } = useUserList()

  return <UserListView users={users} />
}
```

---

## One Hook = One Responsibility

A hook should answer **one question**.

❌ `useUserDashboard`

* fetches users
* filters users
* submits changes

✅ Split into:

* `useUserQuery`
* `useUserFilters`

---

## Return Shape Rules

Always return an **object**, never a tuple.

✅

```ts
return { data, isLoading, error }
```

❌

```ts
return [data, isLoading, error]
```

Objects are:

* Self‑documenting
* Easier to extend
* Safer to refactor

---

## Hook Composition (Encouraged)

Hooks may call other hooks.

```ts
export function useUserDashboard() {
  const users = useUsers()
  const stats = useUserStats(users)

  return { users, stats }
}
```

Prefer **small composable hooks** over large monolithic ones.

---

## SRP Evaluation for Hooks

Ask during development or review:

1. Can I describe this hook in **one sentence**?
2. Does it have **one reason to change**?
3. Could part of this logic be reused elsewhere?

If not — split the hook.


---

## Tooling & Linting (Mandatory)

These tools help **enforce to keep code clean, well formatted and redable and also applies rules automatically**.

### ESLint

- TREAT ESLINT WARNING AND ERRORS AS A COMPILATION ERROR [Link](https://react.dev/learn/removing-effect-dependencies#to-change-the-dependencies-change-the-code)
- DO NOT PUSH ANY CODE HAVING LINTING ISSUE

Recommended rules/plugins:

Use the default recommended configuration by react and other library.

---

### Prettier

Must needed for proper formatting for readability, proper speces and cross developer friendly. 

---

## Naming Conventions

Naming anything sould be clear, proper, concise, non ambiguous and self descriptive. It is completely fine if it becomes bit longer. Do not create your own convention. Follow the convention which has been created and followed by the community and industry.

`Camel Case`

- Variables
- Functions
- folders

`Pascal Case`
- Components
- Classes
- Component files


## Folder Structure


```


src/
│
├── components/
│   └── ui/
│       ├── Button.tsx
│       ├── Input.tsx
│       └── Select.tsx
│
│
├── hooks/
│   ├── useAuth.js
│   └── usePermission.js
│   └── useTheme.js
│
│
├── lib/
│   ├── queryClient.js
│   └── axios.js
│
│
├── utils/
│   ├── formatters.js
│   └── constants.js
│
│
├── features/
│   └── company/
│       ├── hooks/
│       │   ├── useCompanyQuery.js
│       │   └── useCompanyTicker.tsx
│       │
│       ├── components/
│       │   ├── CompanyTable.tsx
│       │   └── CompanyTicker.tsx
│       │   └── CompanyForm.tsx
│       │   └── ActionColumn.tsx
│       │
│       ├── pages/
│       │   ├── CompanyManagementPage.tsx
│       │   └── CompanyFormPage.tsx
│       │
│       ├── services.js
│       └── schemas.js
│       └── utils.js
│
│


```

## Alway refer official documentaion for right resource of learning
- [React js](https://react.dev/)
- [Tailwind CSS](https://tailwindcss.com/)
- [Tanstack Query](https://tanstack.com/query/latest)
- [MUI Datagrid](https://mui.com/x/api/data-grid/)
- [Zod](https://zod.dev/)
- [React Hook Form](https://react-hook-form.com/)



## By Following above best practices we will eventually achieve

- Super Clean Codebase
- High quality code
- Easy to understand
- Pin pointing issue
- Easy to debug
- Easy maintainance
- Seamless onboard a new developer
- Opportunity to become a good developer


Thanks for support 👍
