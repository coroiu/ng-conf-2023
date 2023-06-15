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
