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
  - Router is becoming simpler and more functional to use, especially with standalone components, search for functional guards (eg. https://dev.to/this-is-angular/how-to-use-functional-router-guards-in-angular-23kf)
  - Route query params can now be injected into components using `@Input()`

#### Example: Injected params
```ts
const routes: Routes = [
  {
    path: 'details/:id',
    component: Details,
    resolve: { profile: ProfileResolver }
  }
]

@Component({ ... })
export class Details {
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

## Other talks

### Setting up Enterprise Frontend for Success at Cisco CX and NgRx
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