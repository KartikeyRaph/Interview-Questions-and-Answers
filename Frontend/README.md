# Frontend Interview Q&A

## Overview
This section covers React and Angular for interview preparation.

## Frameworks Covered

### React
- Components (Functional & Class)
- Hooks (useState, useEffect, useContext, etc.)
- State Management
- Props & Component Communication
- JSX & Rendering
- Life Cycle Methods
- Error Boundaries
- Performance Optimization
- Testing
- React Router

### Angular
- Components & Directives
- Services & Dependency Injection
- RxJS & Observables
- Forms (Template-driven & Reactive)
- HTTP Client
- Routing & Navigation
- Change Detection
- Pipes & Decorators
- Testing
- Performance Optimization

## Q&A

### Coming Soon...

---

## React - Basics

### Q1: What are React components and the difference between functional and class components?

**Answer:**
- Components are reusable UI building blocks.
- Functional components are plain JavaScript functions that return JSX.
- Class components extend `React.Component` and provide lifecycle methods and state (older style).

```jsx
// Functional
function Button({children, onClick}) {
  return <button onClick={onClick}>{children}</button>;
}

// Class
class Button extends React.Component {
  render() {
    return <button onClick={this.props.onClick}>{this.props.children}</button>;
  }
}
```

**Key Points:**
- Prefer functional components with hooks for new code
- Class components still supported but less common

---

### Q2: Explain JSX and how it is transformed.

**Answer:**
- JSX is a syntax extension that looks like HTML inside JavaScript.
- Babel transforms JSX to `React.createElement` calls.

```jsx
const el = <div className="card">Hello</div>;
// becomes
const el = React.createElement('div', { className: 'card' }, 'Hello');
```

**Key Points:**
- Use `className` instead of `class`
- Expressions inside `{}` are JavaScript
- JSX must return a single parent element (use fragments `<>`)

---

### Q3: Props vs State

**Answer:**
- **Props:** Read-only data passed from parent to child.
- **State:** Local mutable data managed by the component.

```jsx
function Counter() {
  const [count, setCount] = useState(0); // state
  return <Child value={count} />;         // pass via props
}
```

**Key Points:**
- Lift state up when needed by multiple components
- Avoid mutating state directly

---

### Q4: React Hooks - useState, useEffect, useRef, useMemo, useCallback

**Answer:**
- `useState` - manage state in functional components
- `useEffect` - side effects (replaces lifecycle methods)
- `useRef` - mutable ref container, access DOM
- `useMemo` - memoize expensive values
- `useCallback` - memoize functions

```jsx
const [count, setCount] = useState(0);
useEffect(() => {
  document.title = `Count: ${count}`;
  return () => { /* cleanup */ };
}, [count]);

const ref = useRef();

const memoized = useMemo(() => expensiveCalc(a, b), [a, b]);
const memoizedFn = useCallback(() => doThing(x), [x]);
```

**Key Points:**
- Always provide dependencies array to useEffect
- Avoid overusing useMemo/useCallback; measure performance

---

### Q5: Lifecycle equivalents in functional components

**Answer:**
- Mount: `useEffect(() => { ... }, [])`
- Update: `useEffect(() => { ... }, [dep])`
- Unmount: return cleanup function from effect

---

### Q6: Handling events and forms

**Answer:**
- Use `onClick`, `onChange`, etc. with camelCase
- Controlled components: form values in state
- Uncontrolled components: use refs

```jsx
function Form() {
  const [value, setValue] = useState('');
  return (
    <form onSubmit={e => { e.preventDefault(); submit(value); }}>
      <input value={value} onChange={e => setValue(e.target.value)} />
    </form>
  );
}
```

**Key Points:**
- Controlled forms give more control for validation
- Use libraries like Formik/React Hook Form for complex forms

---

### Q7: Routing basics (React Router)

**Answer:**
- Use `react-router-dom` to define routes

```jsx
import { BrowserRouter as Router, Routes, Route, Link } from 'react-router-dom';

function App() {
  return (
    <Router>
      <nav><Link to="/">Home</Link></nav>
      <Routes>
        <Route path="/" element={<Home />} />
        <Route path="/about" element={<About />} />
      </Routes>
    </Router>
  );
}
```

**Key Points:**
- `useNavigate`, `useParams`, `useLocation` hooks for navigation and route info

---

## React - Advanced

### Q8: Context API vs Prop Drilling

**Answer:**
- Context provides a way to pass data through the component tree without passing props manually at every level.
- Avoid excessive use of Context for highly dynamic data (use state management instead).

```jsx
const ThemeContext = React.createContext('light');

function App() {
  return (
    <ThemeContext.Provider value={'dark'}>
      <Toolbar />
    </ThemeContext.Provider>
  )
}

function Toolbar() {
  return <ThemedButton />;
}

function ThemedButton() {
  const theme = useContext(ThemeContext);
  return <button className={theme}>Click</button>;
}
```

---

### Q9: State management - Redux / Redux Toolkit / Zustand

**Answer:**
- **Redux:** Predictable global state with actions and reducers.
- **Redux Toolkit (RTK):** Opinionated helper for Redux (createSlice, configureStore).
- **Zustand:** Minimal, simpler global state library using hooks.

```js
// RTK slice example
import { createSlice } from '@reduxjs/toolkit'

const counterSlice = createSlice({
  name: 'counter',
  initialState: 0,
  reducers: {
    increment: state => state + 1,
    decrement: state => state - 1
  }
})

export const { increment, decrement } = counterSlice.actions
export default counterSlice.reducer
```

**Key Points:**
- Use local state for UI-specific state; use global store for cross-cutting state
- Use selectors and memoization to avoid unnecessary renders

---

### Q10: Performance Optimization

**Answer:**
- Avoid unnecessary re-renders with `React.memo`, `useMemo`, `useCallback`
- Virtualize long lists (react-window, react-virtualized)
- Code-splitting with dynamic imports and React.lazy
- Lazy load images and assets
- Optimize bundle size (tree-shaking, remove unused libs)
- Use browser performance APIs and Lighthouse

```jsx
const Expensive = React.memo(function Expensive({ data }) {
  // render
});

const LazyComp = React.lazy(() => import('./Heavy'));

<Suspense fallback={<div>Loading...</div>}>
  <LazyComp />
</Suspense>
```

---

### Q11: React Suspense and Concurrent Features

**Answer:**
- `Suspense` allows components to wait for some asynchronous operation (data/code) with a fallback UI.
- Concurrent features (like startTransition) help keep UI responsive during heavy updates.

```jsx
import { startTransition } from 'react';

startTransition(() => {
  setValue(newValue);
});
```

**Key Points:**
- Most concurrent features are opt-in and experimental in some frameworks

---

### Q12: Server-Side Rendering (SSR) and Static Site Generation (SSG)

**Answer:**
- Use Next.js or Remix for SSR/SSG
- SSR: Render pages on the server per request
- SSG: Pre-render pages at build time

**Key Points:**
- SSR improves initial load and SEO
- SSG is great for static content with high performance

---

### Q13: Testing React apps

**Answer:**
- Use Jest for unit tests
- Use React Testing Library for component tests
- Use Cypress/Playwright for end-to-end tests

```js
// Example RTL test
import { render, screen } from '@testing-library/react'
import userEvent from '@testing-library/user-event'
import App from './App'

test('renders greeting', () => {
  render(<App />)
  expect(screen.getByText(/welcome/i)).toBeInTheDocument()
})
```

**Key Points:**
- Test behavior, not implementation details
- Use mocks for network calls

---

### Q14: Accessibility (a11y)

**Answer:**
- Semantic HTML, ARIA attributes where needed
- Keyboard navigation, focus management
- Use tools: axe, Lighthouse, screen readers

**Key Points:**
- Make forms and interactive elements accessible
- Ensure color contrast and readable fonts

---

## Angular - Basics

### Q1: What is Angular's architecture (NgModule, Components, Templates)?

**Answer:**
- Angular apps are modular, built from `NgModule`s that declare components, directives, pipes, and provide services.
- Components control views via templates and metadata.

```ts
// app.module.ts
@NgModule({
  declarations: [AppComponent, HomeComponent],
  imports: [BrowserModule, AppRoutingModule],
  providers: [],
  bootstrap: [AppComponent]
})
export class AppModule {}
```

---

### Q2: Data binding types in Angular

**Answer:**
- **Interpolation:** `{{ value }}`
- **Property binding:** `[src]="imageUrl"`
- **Event binding:** `(click)="onClick()"`
- **Two-way binding:** `[(ngModel)]="value"`

---

### Q3: Angular Directives (structural vs attribute)

**Answer:**
- **Structural directives:** change DOM layout (`*ngIf`, `*ngFor`)
- **Attribute directives:** change appearance/behavior (`ngClass`, `ngStyle`, custom directives)

```ts
@Directive({ selector: '[appHighlight]' })
export class HighlightDirective {
  constructor(private el: ElementRef) {
    el.nativeElement.style.backgroundColor = 'yellow';
  }
}
```

---

### Q4: Services and Dependency Injection

**Answer:**
- Services provide shared logic and are injectable via Angular's DI system.

```ts
@Injectable({ providedIn: 'root' })
export class ApiService {
  constructor(private http: HttpClient) {}
  getData() { return this.http.get('/api/data'); }
}

@Component({...})
export class HomeComponent {
  constructor(private api: ApiService) {}
}
```

---

### Q5: Routing basics

**Answer:**
- Use `RouterModule.forRoot(routes)` to configure routes

```ts
const routes: Routes = [
  { path: '', component: HomeComponent },
  { path: 'about', component: AboutComponent }
];

@NgModule({ imports: [RouterModule.forRoot(routes)], exports: [RouterModule] })
export class AppRoutingModule {}
```

**Key Points:**
- Use `routerLink`, `ActivatedRoute`, `Router` for navigation

---

### Q6: Template-driven vs Reactive forms

**Answer:**
- **Template-driven:** simpler, uses `ngModel`
- **Reactive:** more powerful, programmatic control with `FormGroup`, `FormControl`

```ts
// Reactive form example
this.form = new FormGroup({
  name: new FormControl('', [Validators.required])
});
```

---

## Angular - Advanced

### Q7: RxJS and reactive programming in Angular

**Answer:**
- RxJS provides Observables for async streams (HTTP, user events)
- Use `map`, `filter`, `switchMap`, `mergeMap`, `debounceTime`

```ts
this.http.get('/api/items')
  .pipe(
    debounceTime(300),
    switchMap(term => this.searchService.search(term))
  )
  .subscribe(results => this.results = results);
```

**Key Points:**
- Prefer `switchMap` for search to cancel previous requests
- Manage subscriptions with `takeUntil` or async pipe

---

### Q8: Change Detection strategies

**Answer:**
- Default: checks every component on change detection cycle
- OnPush: checks component only when inputs change or an event originates from it

```ts
@Component({ changeDetection: ChangeDetectionStrategy.OnPush })
export class MyComp {}
```

**Key Points:**
- Use immutable data with OnPush
- Use `ChangeDetectorRef` to trigger manual checks

---

### Q9: NgRx (Redux for Angular)

**Answer:**
- NgRx provides state management with store, actions, reducers, effects
- Use `createAction`, `createReducer`, `createEffect`

```ts
// action
export const loadItems = createAction('[Items] Load Items');

// reducer
const itemsReducer = createReducer(initialState, on(loadItems, state => ({ ...state, loading: true })));
```

**Key Points:**
- Use selectors to derive data
- Use effects for side effects (HTTP)

---

### Q10: Lazy loading and module splitting

**Answer:**
- Use lazy loading to split large apps into feature modules

```ts
const routes: Routes = [
  { path: 'admin', loadChildren: () => import('./admin/admin.module').then(m => m.AdminModule) }
];
```

**Key Points:**
- Reduces initial bundle size
- Combine with preloading strategies

---

### Q11: Performance tuning

**Answer:**
- Use OnPush change detection
- Avoid heavy computations in templates
- Use trackBy in ngFor
- Lazy load modules and components
- Use AOT and production mode builds

---

### Q12: Testing Angular apps

**Answer:**
- Unit tests with Jasmine/Karma or Jest
- Use TestBed for component testing
- E2E with Cypress or Protractor (deprecated)

```ts
beforeEach(async () => {
  await TestBed.configureTestingModule({ declarations: [MyComp], imports: [HttpClientTestingModule] }).compileComponents();
});
```

---

## Scenario-based Frontend Questions

### S1: The initial page load is very slow — how do you troubleshoot and fix it?

Answer:
- Measure with Lighthouse / WebPageTest / DevTools (Network & Performance).
- Check critical metrics: TTFB, Largest Contentful Paint (LCP), First Contentful Paint (FCP), Total Blocking Time (TBT).
- Common fixes:
  - Enable code-splitting and lazy-load non-critical routes/components (React.lazy or dynamic imports / Angular lazy modules).
  - Serve compressed assets (gzip/brotli) and enable HTTP/2.
  - Optimize critical CSS and inline above-the-fold CSS.
  - Defer or async non-critical scripts; move heavy work off the main thread (web workers).
  - Optimize images (responsive sizes, modern formats like WebP, lazy-loading).
  - Reduce bundle size (tree-shaking, remove polyfills not needed, analyze with webpack-bundle-analyzer).

Snippet (React lazy):

```jsx
const Heavy = React.lazy(() => import('./Heavy'));
<Suspense fallback={<Spinner/>}><Heavy/></Suspense>
```

---

### S2: A page is interactive but the UI freezes when a user types or scrolls. What's happening and how to fix it?

Answer:
- Likely long-running JavaScript on the main thread or expensive synchronous rendering.
- Fixes:
  - Identify long tasks in Chrome DevTools Performance trace.
  - Break up work using requestIdleCallback or setTimeout batching, or use web workers for CPU-heavy tasks.
  - Prevent expensive re-renders (use memoization, avoid creating new props/functions each render).
  - Virtualize long lists (react-window) and use pagination for huge datasets.

---

### S3: Users report memory leaks — how do you find and resolve them?

Answer:
- Reproduce with performance/memory snapshot in DevTools.
- Look for detached DOM nodes, growing retained size, unbounded event listeners, subscriptions not unsubscribed.
- Fixes:
  - Remove event listeners and cancel timers on unmount (useEffect cleanup/useEffect return or ngOnDestroy).
  - Unsubscribe RxJS streams or use async pipe.
  - Avoid storing large caches in memory without eviction.

---

### S4: Accessibility bug — screen reader can't focus a modal. How do you fix it?

Answer:
- Ensure modal has correct ARIA roles (role="dialog"), label (aria-labelledby) and focus management.
- Trap focus inside the modal, return focus to trigger on close.
- Ensure keyboard accessibility (Esc to close, Tab/Shift+Tab navigation).
- Use tools: axe, Lighthouse, NVDA/VoiceOver to validate.

---

### S5: How to prevent XSS in a frontend app that renders user-generated HTML?

Answer:
- Avoid dangerouslySetInnerHTML or sanitize input before rendering.
- Use libraries like DOMPurify to sanitize HTML.
- Apply Content Security Policy (CSP) headers from the server to restrict script sources.

---

### S6: CORS issues when calling APIs from the browser — what causes this and how to solve it?

Answer:
- CORS is enforced by browsers; server must return appropriate Access-Control-Allow-* headers.
- Fixes:
  - Configure the API server to allow the origin or use a restrictive allowlist.
  - For development, use a reverse-proxy (dev server proxy) to avoid cross-origin requests.
  - Avoid relying on CORS as a security mechanism; secure APIs with authentication.

---

### S7: Bundle size keeps growing, what are strategies to control it?

Answer:
- Analyze with bundle visualizers (source-map-explorer, webpack-bundle-analyzer).
- Techniques:
  - Tree-shake, remove unused dependencies, replace heavy libs with lighter alternatives.
  - Dynamic imports and route-based code splitting.
  - Use component-level lazy loading and import only needed locales for i18n.
  - Enable compression and caching (long-term caching with content-hashed filenames).

---

### S8: Users see stale content after deployment — what's wrong and how to fix it?

Answer:
- Likely caching issue (CDN or browser cache serving older assets).
- Use cache-busting: content-hashed filenames and set appropriate Cache-Control headers.
- Invalidate CDN cache or use short TTLs during rollouts.
- Consider atomic deploys and feature flags to avoid partial states.

---

### S9: How do you set up CI/CD for a frontend app (basic requirements)?

Answer:
- Steps:
  - Linting and type checks (ESLint / TypeScript) on pull requests.
  - Unit tests and small integration tests (Jest, RTL).
  - Build step creating production artifacts (minified bundles, source maps).
  - Deploy artifacts to CDN / S3 or container registry.
  - Optional: run end-to-end tests on a staging environment before production.
  - Rollback/blue-green or canary deployments for safer releases.

---

### S10: Production errors are happening but stack traces are minified. How do you debug?

Answer:
- Ensure source maps are generated at build time and uploaded to your error tracking service (Sentry, Rollbar).
- Capture contextual data (user session, breadcrumbs) and reproduce with that data.
- Use feature flags to toggle problematic features for a subset of users.

---

### S11: Progressive Web App (PWA) issues — service worker not updating content. What's typical cause?

Answer:
- Service worker lifecycle can cause old SW to serve cached content until clients are closed.
- Best practices:
  - Use a cache versioning strategy and prompt users to refresh when a new SW is available.
  - Call skipWaiting() and clients.claim() strategically, but ensure you don't break in-flight requests.
  - Keep runtime caches small and use stale-while-revalidate patterns.

---

### S12: How do you design testing strategies for frontend features?

Answer:
- Pyramid approach: many unit tests (logic), fewer integration/component tests, and a small number of E2E tests for critical flows.
- Use mocks/stubs for network and third-party services in unit tests.
- Run linters and type checks in CI to catch issues early.

---

**Last Updated:** November 12, 2025
