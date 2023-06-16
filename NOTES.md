# ng-conf notes

## Core Angular: New features, developer experience and quality-of-life improvements

### Deferred loading
  - Uses new syntax
  - Kind of like lazy loading but does not depend on the router

### Improved control flow
  - New syntax, similar to deferred loading syntax
  - Built into the framework, instead of added using directives.
#### Example
```
{#if state === 'logged-in'}
  <user-profile />
{:else if state === 'error'}
  Loggin attempt failed ...
{:else}
  <login-screen />
{/if}
```

### Signals
  - Similar to react's `state`
  - `signal = value + change notification`
  - Intention is for it to play nicely with `rxjs`. See `@angular/core/rxjs-interop`
  - Work in: Components, Directives, Services, and everywhere else
  - Work with `OnPush`
  - Changing values sets a flag and does not immediately execute subscribers.
  - Lazy execution, if no-one uses the value then it won't be computed.
  - Memoized, only computed once per change.
  - Best-practice: Continue to use rxjs for async operations `http.get`. Signals can be used to replace Subjects.

#### Example: Basic
```ts
const x = signal(5);
console.log(x()); // 5
x.set(7);
console.log(x()); // 7
x.update(v => v + 1)
console.log(x()); // 8
```

#### Example: Mutate
```ts
const obj = signal({ hello: 'world' });
console.log(obj()); // { hello: world }

const temp1 = obj();
obj.set({ hello: 'foo' });
console.log(obj() === temp1); // false

const temp2 = obj();
obj.mutate(v => v.hello = 'bar');
console.log(obj() === temp2); // true
```

#### Example: Reactive computing
```ts
const x = signal(5);
const y = signal(3);
const z = computed(() => x() + y());
console.log(z()); // 8
x.set(10);
console.log(z()); // 13
```

#### Example: React to change
```ts
const x = signal(5);
effect(() => console.log(`The value x has changed: ${x()}`));
```

#### Example: rxjs interopability
```ts
export class UserService {
  http = inject(HttpClient);
  userUrl = 'https://jsonplaceholder.typicode.com/users';

  private users$ = this.http.get<User[]>(this.userUrl);

  users = toSignal(this.users$, {initialValue: [] as User[]}); // read-only signal
  // Can be converted back to observable
  // users$ = toObservable(this.users);
  selUserId = signal(0);

  setSelectedUser(id: number) {
    this.selUserId.set(id);
  }
}
```

### Standalone components
  - Intention is for standalone components to become the default in angular, replacing ng-module.
  - Editor can automatically auto import Components, Directives, and Pipes to standalone components.

### Directive composition
  - Components can finally define their own directives that will automatically be applied to the component.

### Required input
  - Compile-time enforced required template inputs.

#### Example: Required input
```ts
@Directive({ selector: '[tooltip]' })
export class Tooltip {
  @Input({ required: }) message: string;
}
```

### Router improvements
  - Router is becoming simpler and more functional to use, especially with standalone components. search for functional guards (eg. https://dev.to/this-is-angular/how-to-use-functional-router-guards-in-angular-23kf)
  - Route query params can now be injected into components using `@Input()`

#### Example: Functional methods
```ts
const routes: Routes = [
  {
    path: 'details/:id',
    component: DetailsComponent,
    canActivate: [() => inject(AuthService).isAuthenticated()],
    resolve: { profile: () => inject(ProfileService).getCurrentUser() }
  }
]
```

#### Example: Injected params
```ts
const routes: Routes = [
  {
    path: 'details/:id',
    component: DetailsComponent,
    resolve: { profile: ProfileResolver }
  }
]

@Component({ ... })
export class DetailsComponent {
  @Input() query?: string; // query params
  @Input() id?: string; // path params
  @Input() profile?: Profile; // resolved data
}
```

### Inject function added to Angular v14
  - At a glance just a different way of writing dependencies
  - Much more powerful, if you look in depth
  - For more information see examples below or go to https://codereacter.medium.com/why-angular-14s-new-inject-function-is-so-amazing-ac281e7148d1

#### Example: Basic
```ts
export class TraditionalInject {
  constructor(private httpClient: HttpClient) {}
}

export class NewInject {
  protected httpClient = inject(HttpClient);
}
```

#### Example: Higher order inject functions
```ts
// user-details.fetch.ts
export const fetchUserDetails() = () => {
  const http = inject(HttpClient);

  return inject(ActivatedRoute).paramsMap.pipe(
    map((params) => params.get('id')),
    map((id) => +id),
    switchMap((id) => http.get(`/api/user/${id}/details`))
  );
}

// higher-order-inject.components.ts
export class HigherOrderInjectComponent {
  protected httpClient = fetchUserDetails();
}
```

### RXJS v8 improvements
  - Safer and easier to develop operators using `operate` function
    - Library is now implemented using the same syntax/conventions as the ones recommended in the documentation for library-users, meaning it's easier to copy source code from the rxjs repo

#### Example: Operator development in v7
```ts
export function map<In, Out>(
  fn: (value: In) => Out,
): OperatorFunction<In, Out> {
  return (source) => new Observable((destination) => {
    return source.subscribe({
      next: (value) => {
        let result;
        try {
          result = fn(value);
        } catch (err) {
          destination.error(err);
          return;
        }
        destination.next(result);
      },
      error: (err) => destination.error(err),
      complete: () => destination.complete(),
    })
  })
}
```

#### Example: Operator development in v8
```ts
export function map<In, Out>(
  fn: (value: In) => Out,
): OperatorFunction<In, Out> {
  return (source) => new Observable((destination) => {
    source.subscribe(operate({
      destination,
      next: (value) => destination.next(fn(value)),
    }));
  });
}
```

  - New `rx` function

#### Example: Piping using `rx`
```ts
// v7
from(someAsyncOrStream).pipe(
  filter(...),
  map(...)
).subscribe(console.log);

// v8
rx(
  someAsyncOrStream,
  filter(...),
  map(...)
).subscribe(console.log);
```

  - Really cool `for await` support in observables

#### Example: `for await`
```ts
async function test() {
  // source$ = observable
  // for await is basically a concatMap
  for await (const value of source$) {
    console.log(value);
    if (value > 100) {
      break; // unsubscribes
    }
  }
}
```

## Talks
Raw notes from talks. Information regarding new features is condesed and summarized in the section above. Does not contain notes from (sponsored) libraries that we don't use.

### Setting up Enterprise Frontend for Success at Cisco CX and NgRx
What was important from them to succeed?
  - People
    - Team communication
  - Tools
    - Monorepo
    - Feature flags
    - Automation
  - Processes
    - Trunk-based development
      - Less lead time - faster bug fixes and releases
      - Simpler to understand
    - Code style: "A shared understanding of excellence"
      - See: https://ts.dev/style
      - Angular style guide
      - Testing guide
      - Tools: Prettier, ESLint, Strict T, PR Pipeline
    - Testing
      - Important with engineering ownership
      - All should be written by developers: Unit, Integration, Visual, Acceptance
    - Planning
    - Daily releases

### Do More using GitHub, AI, and VS Code
  - 1m users of GitHub copilot
  - Good for regex, like really good
  - Prediction: Learning will be done using AI in the future
  - Aside: Pressing dot (`.`) in github opens the full codespace editor
  - Talk goes through some examples where coding with copilot speeds up development.
  - Copilot X, new features:
    - Chat: Ask dev questions. 
      - Similar to existing copilot, but with a chat window
      - Works better when working with entire files, where you want copilot to do more than just write a function somewhere
    - Voice: Same, ask dev questions
    - Docs: Auto generate docs in your repo
    - PR: Auto review and generate suggestions for your PR
    - CLI

### Superpowers with signals
  - Talk is a live code session, creating an application to generate superheroes

### Angular (a matter of opinion)
  - Why do we get frustrated with our code and tools?
  - Part 1. Too many choices can be bad
    - Angular provides a lot of tools already chosen for you
    - Makes code consistent and easier to read
  - Part 2. Information overload
    - Occurs when we come across novel ideas of information
    - Can trigger natural responses - causing stress
    - "Overwhelming" when you are prented with too many tools. Rxjs is another examples of this.
    - React on the other hand does only a few things, and does them well.
    - "Seeing all of angular and not making use of it can make you feel like you're doing something wrong."
  - Part 3. Balance
    - Concepts need to be introduced gradually - Progressive Disclosure
      - Hide complex information until it's needed
    - Standalone components are a better way to start learning angular
    - Anonymous function guards are also simpler
    - Signals are probably gonna be a great addition to angular, but are we gonna push them like rxjs as another thing people need to learn?
    - Need to be upfront about what problems some concept is intended to solve
    - "Angulars template syntax is angulars weak point - jsx is arguably better"
  - Summary:
    - We love having choice
    - We hate making choices
    - Reduce complexity!
    - Simple composable tools
    - Separate documentation for those tools

### Identity guardians
  - Quick talk on how OAuth 2.0 + OpenID Connect works
  - Handshake involve 3 "random" (incomprehensible) strings: the 3 guardians
    - Access token
    - ID Token
    - Refresh token
  - Access token
    - From OAauth Layer
    - Handles authorization
    - Usually with bearer-scheme
  - ID Token
    - From OIDC Layer
    - Handles authentication and identification
    - Contains metadata (claims in a JWT)
  - Refresh token
    - Used to extend the life of authentication and authorization

### MFE Production Gotchas, and thing you should prepare
  - MFE = Micro FrontEnd
  - Goes through lots of common errors in a quick 5 min talk

### Craig's Angular/Rust Spectacular (brought to you by Rust-eze)
  - Talk about getting started in Rust
  - Implementing fluid dynamics simulation in javascript is slow
  - Typescript vs Rust
    - Typescript
      - Superset of JS for Web development
      - Garbage collection (like JS)
      - Async/event-driven
      - Interpreted (but not really...)
    - Rust
      - Systems language emphasising safety and performance
      - Memory-safe without GC
      - Build-in concurrency and parallelism
      - Compiled an highly optimised
  - nvm - rustup
  - npm - cargo

### Lightweight Architectures with Angular's Latest Innovations
  - More is not always better. Under/over engineering is bad. What is the sweet spot?
  - 1. Standalone components
    - Code organization does not change:
      - Small and medium apps: Folder per feature
      - Medium and large apps: Folder per domain
        - Intra-domain imports are often prohibited
        - Can usually only import from "shared"
        - NX can be used for this
        - More fine-grained: `@softarc/eslint-plugin-sherrif`
  - 2. Custom standalone APIs
    - Allow packages (domains) to be imported into other modules
    - Like HttpClient, Router, Interceptors, etc
    - Golden rule: whenever possible, use `providedIn: 'root'`
    - `makeEnvironmentProviders` to disallow libraries from being loaded into standalone components (e.g. router shouldn't be used like that)
  - 3. "Simple angular mosaic"
    - Goes through all of the exciting features mentioned in "Core Angular"

### Delighting users with performant apps
  - The talk is a story of some specific performance issues during the implementation of a web-based music player
  - Box shadows are bad for performance, browser over-draws. Use blur and filters
  - Flip animations and view-transitions are a good way of moving things around
  - Web workers are good for processor-intensive functions
  - Moral of the story: you don't have to sacrifice nice looking ui to get good performance

### The Magic and Mystery of Angular Change Detection 
  - "Change detection is a sequence of steps that ensures the UI reflects the current state of the application"
  - Very in depth view of how change detection works by looking at compiled angular code.
  - Zone.js
    - Patches browser APIs
    - Tells angular when anything occurs that *could* impact the View
    - Does NOT tell angular *what* or *where an event* occurred
    - Angular has to go through each component in the whole tree to check if a template has changed
    - `OnPush` results in Angular only checking parts of the tree, still top-down
  - Zone-less (signals)
    - Change detection for a signal component is scheduled only when a signal is read in the template (essentially subscribing to it).
    - Enables local (per-view) change detection
    - Fine-grained info on model changes
    - Coexists seamlessly with zone components
    - Signals results in angular only cheking/updating singular nodes in the middle of the tree, no longer top-down

### What's new in RXJS 8 (alpha)
  - Smaller size, 30% smaller than v7, 61% smaller than v6
  - Safer and easier to develop operators using `operate` function
    - Library is now implemented using the same syntax/conventions as the ones recommended in the documentation for library-users, meaning it's easier to copy source code from the rxjs repo

#### Example: Operator development in v7
```ts
export function map<In, Out>(
  fn: (value: In) => Out,
): OperatorFunction<In, Out> {
  return (source) => new Observable((destination) => {
    return source.subscribe({
      next: (value) => {
        let result;
        try {
          result = fn(value);
        } catch (err) {
          destination.error(err);
          return;
        }
        destination.next(result);
      },
      error: (err) => destination.error(err),
      complete: () => destination.complete(),
    })
  })
}
```

#### Example: Operator development in v8
```ts
export function map<In, Out>(
  fn: (value: In) => Out,
): OperatorFunction<In, Out> {
  return (source) => new Observable((destination) => {
    source.subscribe(operate({
      destination,
      next: (value) => destination.next(fn(value)),
    }));
  });
}
```

  - Piping improvements

#### Example: Piping using `rx`
```ts
// v7
from(someAsyncOrStream).pipe(
  filter(...),
  map(...)
).subscribe(console.log);

// v8
rx(
  someAsyncOrStream,
  filter(...),
  map(...)
).subscribe(console.log);
```

  - Really cool `for await` support in observables

#### Example: `for await`
```ts
async function test() {
  // source$ = observable
  // for await is basically a concatMap
  for await (const value of source$) {
    console.log(value);
    if (value > 100) {
      break; // unsubscribes
    }
  }
}
```

### Reactive Patterns For Angular
  - Slides and examples: https://github.com/lara-newsom/reactive-patterns
  - Three ingredients for reactivity: Producers, Consumers, Side effects
  - Following good reactive patterns makes it easy to understand where changes should happen
  - Suggestion 1 - Input setters: Use setters with `@Input` instead of `ngOnChanges`. Can easily be adapter to signals (note: and rxjs)
  - `@Output` great tool passing data from child to parent, not great for passing data to other parts of the app which tightly couples the emitting component to the tree.
  - Suggestion 2 - Emit events where they happen: Use stateful services (note: like react contexts) using subjects or signals. Or use the router.
  - Suggestion 3 - Compose data before it reaches consumers

### Example: Updating state store using router events
(note: not sure I agree with this suggestion, I think router data providers would be a better fit)
```ts
export class PageComponent {
  private stateService = inject(StateService);
  @Input() set routerParam(value: string) {
    stateService.routerParam.set(value);
  }
}
```

  - Summary (note: great rules even if I don't agree with implementatio):
    - Have a clear flow of data through the application
    - Create and maintain single sources of truth
    - Compose data so all consumers get the same data

### Clean Up Your Old Angular Apps
  - Talk goes through the history of angular and all breaking changes that have happened throughout
  - Talk then goes through a process for how to upgrade a really angular version
  - "The nuclear option"
    - `npx @angular/cli@latest new new-beutiful-project`
    - Copy over `/src/app`
    - Copy over assets and config
    - Fix it

### Fine-Tuning Your Angular Workflow with Nx: Beyond the CLI
  - The talk goes through how to use generators and executors in Nx
  - Nx is compatible with angular schematics

### Angular Across the Stack with Analog
  - Analogjs is a fullstack Angular meta-framework
    - "A meta-framework is a system one level above that stitches multiple frameworks together"
    - Eg: Next.js, Sveltekit
  - Vite-powered
  - File-based routing and API routes
  - SSR and SSG (note: which is not really applicable to us)
  - Works with Nx

### Auxiliary Routes - the Ant-Man of Angular
  - The talk presents named outlets which is an existing angular feature
  - (note: This is what I've been experimenting with during BEEEP to add routable dialogs)