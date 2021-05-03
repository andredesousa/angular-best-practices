# Angular Best Practices

This is a guideline of best practices that we can apply to our Angular project.
These tips are based on Angular documentation, books, articles and professional experience.

## Table of Contents

1. [Follow conventions](#follow-conventions)
2. [Use Angular CLI](#use-angular-cli)
3. [Isolate API hacks](#isolate-api-hacks)
4. [Prefer Observables over Promises](#prefer-observables-over-promises)
5. [Subscribe in template](#subscribe-in-template)
6. [Avoid memory leaks](#avoid-memory-leaks)
7. [Avoid nested subscriptions](#avoid-nested-subscriptions)
8. [Do not pass streams to components directly](#do-not-pass-streams-to-components-directly)
9. [Do not pass streams to services](#do-not-pass-streams-to-services)
10. [Use pipeable operators](#use-pipeable-operators)
11. [Use pure functions](#use-pure-functions)
12. [Avoid manual subscribes](#avoid-manual-subscribes)
13. [Do not share all subscriptions](#do-not-share-all-subscriptions)
14. [When to Subscribe?](#when-to-subscribe)
15. [When to use Subjects?](#when-to-use-subjects)
16. [Do not expose subjects](#do-not-expose-subjects)
17. [Handle RxJS errors](#handle-rxjs-errors)
18. [RxJS clean-code practices](#rxjs-clean-code-practices)
19. [Use marble diagrams](#use-marble-diagrams)
20. [Use appropriate operators](#use-appropriate-operators)
21. [Use a state management library](#use-a-state-management-library)
22. [Avoid any. Type everything](#avoid-any-type-everything)
23. [Make use of lint rules](#make-use-of-lint-rules)
24. [LIFT Principle](#lift-principle)
25. [Do not Repeat Yourself](#do-not-repeat-yourself)
26. [Avoid long functions](#avoid-long-functions)
27. [Avoid change the DOM directly](#avoid-change-the-dom-directly)
28. [Avoid computing values in the template](#avoid-computing-values-in-the-template)
29. [Avoid logic in templates](#avoid-logic-in-templates)
30. [Use mandatory inputs](#use-mandatory-inputs)
31. [Provide private services](#provide-private-services)
32. [Use OnPush](#use-onpush)
33. [Use trackBy](#use-trackBy)
34. [Use virtual scrolling](#use-virtual-scrolling)
35. [Use lazy loading](#use-lazy-loading)
36. [Avoid poorly structured CSS](#avoid-poorly-structured-css)
37. [Lack of meaningful unit tests](#lack-of-meaningful-unit-tests)
38. [Add caching mechanisms](#add-caching-mechanisms)
39. [Avoid magic numbers](#avoid-magic-numbers)
40. [Avoid useless code comments](#avoid-useless-code-comments)
41. [Remove unused code](#remove-unused-code)
42. [Separation of concerns](#separation-of-concerns)
43. [Use aliases for imports](#use-aliases-for-imports)
44. [Develop in a modular way](#develop-in-a-modular-way)
45. [Components should only deal with display logic](#components-should-only-deal-with-display-logic)
46. [Small reusable components](#small-reusable-components)
47. [Smart and dummy components](#smart-and-dummy-components)
48. [Base component classes](#base-component-classes)
49. [Do not remove view encapsulation](#do-not-remove-view-encapsulation)
50. [Use reactive forms](#use-reactive-forms)

## Follow conventions

One of the core arguments to choose a framework over traditional way with [Vanilla JavaScript](http://vanilla-js.com/) is its clearly defined way of how things are supposed to be done.
That defined way enables the creation of a uniform code base across an organization, without worrying about defining certain rules.
Following them does not only increase the uniformity and therefore the quality of the code.
This strict approach also comes in handy across cooperation borders.
It enables new developers to integrate into a new team very quickly, because of the high familiarity with the code.
We should follow the [Angular Style Guide](https://angular.io/guide/styleguide) to get the most out of the Angular Framework.
It will make our life a lot easier, when coming into new projects and will increase the quality of our code almost automatically.

## Use Angular CLI

The [Angular CLI](https://cli.angular.io/) is the best way to build Angular Applications.
Angular CLI can be installed easily via terminal by typing in the following command:

```bash
npm install -g @angular/cli
```

The CLI has [scaffolding tools](https://angular.io/guide/schematics) for creating new projects and generating new code, such as, modules, services, components, directives and pipes.
However, this isn't the main benefit.
The main benefit of the CLI is the way it automates the build pipeline for both live development with `ng serve`, as well as for production code that we would ship down to browsers with `ng build --prod`.
Other common commands available are `ng lint`, `ng test` and `ng e2e`.

## Isolate API hacks

Sometimes, we need to add some logic in the code to make up for bugs in the APIs.
Instead of having the hacks in components where they are needed, it is better to isolate them in one place like in a function or a service and use them.
When fixing the bugs in the APIs, it is easier to look for them in one file rather than looking for the hacks that could be spread across the codebase.

## Prefer Observables over Promises

[Observables](https://rxjs-dev.firebaseapp.com/guide/observable) partly overlaps the standard JavaScript [Promise](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/Promise) functionality.
Both are meant to handle asynchronous code.
However, Observables can do so much more than Promises.
Observables can resolve to more than just one value, because they are a stream of values.
Observables are everywhere in Angular Framework.
We can see that, by looking at the Angular [HttpClient](https://angular.io/guide/http).
It returns Observables, even when it is clear, that a HTTP call can never result in more than one response.
Mixing Observables with Promises isn't a good solution.
That way, we have completely different implementations, that are hardly compatible with each other in different parts of our application.

## Subscribe in template

Avoid subscribing to observables from components and instead subscribe to the observables from the template.
To consume a stream, we need to subscribe that stream, that's simply how observables work.
The [async](https://angular.io/api/common/AsyncPipe) pipe unsubscribe themselves automatically and it makes the code simpler by eliminating the need to manually manage subscriptions.

```typescript
@Component({
  selector: 'my-component',
  template: `<child-component [data]="data$ | async"></child-component>`
})
export class MyComponent {
  data$: Observable<object>;
}
```

Using of the [subscribe() instead of the async pipe](https://medium.com/angular-in-depth/angular-question-rxjs-subscribe-vs-async-pipe-in-component-templates-c956c8c0c794) introduces complementary need to unsubscribe at the end of the component life-cycle to avoid memory leaks.
Subscribing to the observable manually in the `ngOnInit()` doesn't work with `OnPush` Change Detection Strategy.
The [OnPush](https://angular.io/api/core/ChangeDetectionStrategy) is great for performance, so, we should use `async` pipe as much as possible.

## Avoid memory leaks

When subscribing to observables, always unsubscribe from them appropriately by using operators like `take`, `takeUntil`, … or calling `unsubscribe()`.
To consume a stream, we need to subscribe to that stream.
That subscription will keep on living until the stream is completed or until we unsubscribe manually from that stream.
While Angular takes care of unsubscribing when using the `async` pipe, it quickly becomes a mess when we need to do it ourselves.
Failing to unsubscribe from observables will lead to unwanted [memory leaks](https://itnext.io/angular-rxjs-detecting-memory-leaks-bdd312a070a0) as the observable stream is left open, potentially even after a component has been destroyed or the user has navigated to another page.
This risk can also be mitigated by using a [lint rule](https://www.npmjs.com/package/rxjs-tslint-rules) to detect unsubscribed observables.

## Avoid nested subscriptions

Nesting subscribes is something that needs to be avoided as much as possible.
It makes the code unreadable, complex, and introduces side effects.
It basically forces us to not think reactively.
The next implementation is considered a bad practice:

```typescript
this.route.params
  .pipe(map(param => param.id))
  .subscribe(id => this.userService.fetchById(id)
    .subscribe(user => (this.user = user)));
}
```

## Do not pass streams to components directly

Passing streams to child components is a bad practice because it creates a very close link between the parent component and the child component.
A component should always receive an object or value and should not even care if that object or value comes from a stream or not.
It is better to handle the subscription in the parent component itself.
Angular has a feature called the [async](https://angular.io/api/common/AsyncPipe) pipe that can be used for this.

## Do not pass streams to services

By passing a stream to a service we don't know what's going to happen to it.
The stream could be subscribed to, or even combined with another stream that has a longer lifecycle, that could eventually determine the state of our application.
Subscriptions might trigger unwanted behavior.
It's recommended to use higher order streams in the components for these situations.

## Use pipeable operators

Use [pipeable operators](https://github.com/ReactiveX/rxjs/blob/master/doc/pipeable-operators.md) when using RxJS operators.
A pipeable operator is basically any function that returns a function with the signature: `<T, R>(source: Observable<T>) => Observable<R>`.
Pipeable operators are [tree-shakable](https://medium.com/@netxm/what-is-tree-shaking-de7c6be5cadd) meaning only the code we need to execute will be included when they are imported.
This also makes it easy to identify unused operators in the files.

## Use pure functions

RxJS follows the concepts of functional reactive programming which basically means that we will use pure functions to create our reactive flow.
In the beginning it might seem pragmatic to use side effects, but that mostly means we aren't fully thinking reactively.
Therefore, avoid side effects at much as possible.

## Avoid manual subscribes

If a component needs values from different streams, we need to subscribe all those streams and manually map all the values to unique properties.
Angular has a feature called the `async` pipe.
The `async` pipe unsubscribe themselves automatically and it makes the code simpler by eliminating the need to manually manage subscriptions.
It also reduces the risk of accidentally forgetting to unsubscribe a subscription in the component, which would cause a memory leak.
It triggers change detection automatically and cleans up the code a lot.

## Do not share all subscriptions

Since most streams are cold by default, every subscription will trigger the producer of these streams.
However, subscribing to Angular its `http.get()` multiple times will actually perform multiple XHR calls.
In some cases, we may want to share the subscriptions:

```typescript
users$ = this.http.get(...).pipe(share());
```

It works, but it is a common mistake to share everything.
Angular also provides an alternative that can reduce the sharing of streams to a minimum by using the `async as else` syntax.

```typescript
@Component({
  selector: 'my-component',
  template: `
    <div *ngIf="users$ | async as users; else loading">
        Number of users: 
        <users-grid [users]="users"></users-grid>
    </div>
    <ng-template #loading>Loading...</ng-template>
  `
})
class MyComponent {
  users$ = this.http.get(...);
}
```

## When to Subscribe?

The answer to that question is, "Only when we absolutely have to."
Because if we don't subscribe, we don't have to unsubscribe.
In Services, it's recommended never to subscribe to Observables.
In components, we can use the [async](https://angular.io/api/common/AsyncPipe) pipe.
So, we will subscribe when we absolutely have to, and we almost never have to.

## When to use Subjects?

A [Subject](https://rxjs-dev.firebaseapp.com/guide/subject) is both a hot observable and an observer at the same time.
This gives us the opportunity to next values into the stream ourselves.
Subjects tend to be overused by people that didn't make the mind switch towards reactive programming yet.
Only use them when really needed, for instance it's fine to use Subjects when we are mocking streams in tests, when we want to create streams from outputs in Angular or when handling circular references.
For most other cases an `operator` or `Observable.create` might be enough.

## Do not expose subjects

There is a pretty common practice to use Observable Data Services in Angular:

```typescript
export class DataService {
  private data: BehaviorSubject<number> = new BehaviorSubject<number>(0);

  readonly data$: Observable<number> = this.data.asObservable();

  increment(): void {
    this.data.next(this.data.getValue() + 1);
  }
}
```

Here we're exposing data stream as observable.
Just to make sure it can be changed only through a data service interface.

## Handle RxJS errors

Error handling is an essential part of RxJS.
By default, if something goes wrong with an Observable, it just dies.
If we don't deal with such errors, it will happen silently, and we won't know why we are not receiving data anymore.
[RxJS Error Handling - Complete Practical Guide](https://blog.angular-university.io/rxjs-error-handling/) provides the most common error handling strategies.

## RxJS clean-code practices

Consistent code indentation and formatting can improve the readability of complex streams:

- Align operators below each other;
- Extract into different streams when it becomes unreadable;
- Put complex functionalities in private methods;
- Avoid the use of brackets for readability.

## Use marble diagrams

[Marble Diagrams](https://rxmarbles.com/) provide a visual way for us to represent the behavior of an Observable.
We can use them to assert that a particular Observable behaves as expected, as well as to create [hot and cold observables](https://medium.com/@benlesh/hot-vs-cold-observables-f8094ed53339) we can use as mocks.
Marble diagram is a domain specific language for RxJS to help us to model the interactions and the values of one or more observable in our test.
Most of the documentation about [marble testing](https://rxjs-dev.firebaseapp.com/guide/testing/marble-testing) can be found on official documentation.

## Use appropriate operators

[RxJS Operators](https://rxjs-dev.firebaseapp.com/guide/operators) are the essential pieces that allow complex asynchronous code to be easily composed in a declarative manner.
There are operators for different purposes, and they may be categorized as: `creation`, `transformation`, `filtering`, `joining`, `multicasting`, `error handling`, `utility`, etc.
When using [flattening operators](https://medium.com/angular-in-depth/switchmap-bugs-b6de69155524) with our observables, use the appropriate operator for the situation.
Using a single operator when possible instead of chaining together multiple other operators to achieve the same effect can cause less code to be shipped to the user.
Using the wrong operators can lead to unwanted behavior, as different operators handle observables in different ways.

## Use a state management library

When building large and complex applications that has lots of information coming from and going to the database, and where state is shared across multiple components, we might start considering adding a [state management](https://medium.com/@2muchcoffee/angular-state-management-a-must-have-for-large-scale-angular-apps-8b98e5a761c7) library.
By using a state management library, we can keep the application state in one single place, which reduces the communication between components and keeps our app more predictable and easier to understand.
A [Store](https://ngrx.io/guide/store/) isolates all state related logic in one place and makes it consistent across the application.
It also has memorization mechanism in place when accessing the information in the Store leading to a more performant application.
A Store combined with the change detection strategy of Angular leads to a faster application.

## Avoid any. Type everything

Always declare variables or constants with a `type` other than `any`.
When declaring variables or constants in Typescript without a typing, the typing of the variable/constant will be deduced by the value that gets assigned to it.
This will cause unintended problems.
Another advantage of having good typings in our application is that it makes refactoring easier and safer.
The any type isn't necessarily a bad thing and, in fact, does still come in useful sometimes.
However, in most cases, there is a better alternative that leads to having better defined types overall.
In new projects, it is worth setting `strict:true` in the `tsconfig.json` file to enable all strict type checking options.

## Make use of lint rules

Having lint rules in place means that we will get a nice error when we are doing something that we should not be.
This will enforce consistency in our application and readability.
Besides checking style, linters are also excellent tools for finding certain classes of bugs, such as those related to variable scope.
We can combine [TSLint](https://palantir.github.io/tslint) with [Prettier](https://prettier.io/).
TSLint is an extensible static analysis tool that checks TypeScript code for readability, maintainability, and functionality errors.
Prettier is an amazing tool that enforces a consistent style by parsing our code and re-printing it, with its own rules in place.
Having Prettier setup with TSLint gives us a strong foundation for our applications, as we no longer need to maintain our code-style manually.
Combined with [husky](https://www.npmjs.com/package/husky) makes for an excellent workflow.

## LIFT Principle

The [LIFT Principle](https://angular.io/guide/styleguide#lift) helps us to find code quickly.
Structuring the app such that we can Locate code quickly, Identify the code immediately, keep the flattest structure we can, and Try to be DRY.
Keep a flat folder structure as long as possible.
It's very important to give good names for methods, variables, and parameters.
5 seconds rule says that if we can't understand in 5 seconds, we probably need a refactor.
We must organize the class code by putting the most important things first, properties followed by methods, grouped and sorted with consistent naming and spelling matter.

## Do not Repeat Yourself

Make sure we do not have the same code copied into different places in the codebase.
Extract the repeating code and use it in place of the repeated code.
Having the same code in multiple places means that if we want to make a change to the logic in that code, we must do it in multiple places. This makes it difficult to maintain and is prone to bugs where we could miss updating it in all occurrences.
It takes longer to make changes to the logic and testing it is a lengthy process as well.
In those cases, extract the repeating code and use it instead.
This means only one place to change and one thing to test. Having less duplicate code shipped to users means the application will be faster.

## Avoid long functions

Long functions generally indicate that they are doing too many things.
Small functions are better to read and faster to understand the purpose.
If our function has more than 10 lines, we need to ask ourselves if it would be better to break it into smaller functions.
The best functions or methods are from 5 to 10 lines of code.
Try to use the [Single Responsibility Principle](https://codeburst.io/understanding-solid-principles-single-responsibility-b7c7ec0bf80).
The method itself might be doing one thing, but inside it, there are a few other operations that could be happening.
We can extract those methods into their own method and make them do one thing each and use them instead.
This is sometimes measured as "cyclomatic complexity".
There are also some TSLint rules to detect [cyclomatic/cognitive complexity](https://palantir.github.io/tslint/rules/cyclomatic-complexity/), which we could use in your project to avoid bugs and detect code smells and maintainability issues.

## Avoid change the DOM directly

It's important to know that Angular uses Lifecycle Hooks that determine how and when components will be rendered and updated. Direct DOM access or manipulation can corrupt these lifecycle hooks, leading to unexpected behavior of the whole app.
Direct access to the DOM can make our application more vulnerable to [XSS attacks](https://owasp.org/www-community/attacks/xss/).
Use [ElementRef](https://angular.io/api/core/ElementRef) as the last resort when direct access to DOM is needed.
Use templating and data-binding provided by Angular instead.
Alternatively we can take a look at [Renderer2](https://angular.io/api/core/Renderer2) which provides API that can safely be used even when direct access to native elements is not supported.

## Avoid computing values in the template

Sometimes in Angular templates, we may be tempted to bind a method in the HTML template.
The problem is that the methods are constantly getting called during the [Angular Change Detection Cycle](https://blog.angular-university.io/how-does-angular-2-change-detection-really-work/).

```typescript
@Component({
  selector: 'my-component',
  template: `<h1>{{ getTitle() }}</h1>`
})
export class MyComponent {
  getTitle(): string {
    return 'Page Title';
  }
}
```

It's highly [recommended not to use methods calls](https://medium.com/showpad-engineering/why-you-should-never-use-function-calls-in-angular-template-expressions-e1a50f9c0496) in Angular template expressions.
While method calls in Angular templates are super convenient and technically valid, they may cause serious performance issues because the method is called every time change detection runs.
Instead, we can use pure [pipes](https://angular.io/guide/pipes) or manually calculate the values with [Lifecycle Hooks](https://angular.io/guide/lifecycle-hooks).

## Avoid logic in templates

If we have any kind of logic in our templates, even if it is a simple && clause, it is good to extract it out into its component.
Having logic in the template means that it is not possible to test with isolated unit tests it and therefore it is more prone to bugs when changing template code.
Logic should be contained in one place (the component class) instead of being spread in two places.
Keeping the component's presentation logic in the class instead of the template improves testability, maintainability, and reusability.

## Use mandatory inputs

To make the requirement explicit we can use the selector in the `@Component` decorator to require that the attribute on our component must exist.

```typescript
@Component({
  selector: 'my-component[items]',
})
export class MyComponent {
  @Input()
  items: object[];
}
```

Resulting an error, when we start the application or at compile time when the application is built Ahead of Time (AoT), if the `MyComponent` doesn't have a items attribute.
This approach improves the readability of the code because helps other developers to integrate this component into their projects, throwing errors, and we don't need to define explicit validations to check if it exists.

## Provide private services

Most [providers](https://angular.io/guide/providers) in angular are designed to act on a global scope.
They are then provided at an application level (AppModule).
This makes sense if the use of the [global-singleton-pattern](https://angular.io/guide/singleton-services) is required.
However, there are services that do not need to be provided globally.
Especially if they are used by just one component.
In that case, it can make sense to provide that service inside of the component, instead of globally.
That is, if the service is directly tied to that component, as shown below.

```typescript
@Component({
  selector: 'my-component',
  providers: [MyService]
})
export class MyComponent {}
```

Providers are [tree-shakable](https://angular.io/guide/dependency-injection-providers#tree-shakable-providers), the Angular compiler removes the associated services from the final output when it determines that our application doesn't use those services.
It also minimizes the risk of dead code and reduces the size of our bundles.

## Use OnPush

By default, Angular uses the `ChangeDetectionStrategy.Default` [change detection strategy](https://angular.io/api/core/ChangeDetectionStrategy).
The Default strategy doesn't assume anything about the application, therefore every time something changes in our application, as a result of various user events, timers, XHR, promises, etc., a change detection will run on all components.
This means anything from a click event to data received from an ajax call causes the change detection to be triggered.
Now, imagine a big application with thousands of expressions.
If we let Angular check every single one of them when a change detection cycle runs, we might encounter a performance problem.
We can set the `ChangeDetectionStrategy` of our component to `ChangeDetectionStrategy.OnPush`.
This tells Angular that the component only depends on its `@inputs()` and needs to be checked only in the following cases:

- Input reference of the component changes;
- An event originated from the component or one of its children;
- Emission of an observable event subscribed with `async` pipe;
- Change detection is manually run.

This practice is even more important for large and complex applications as the number of components skipped by the change detection is substantial.
We need to note that when we are working with observable subscriptions, there is no away for Angular to know that we are updating a component attribute.
In such cases, it is recommended to subscribe with `async` pipe.

## Use trackBy

When using `ngFor` to loop over an array in templates, use it with a `trackBy` function which will return an unique identifier for each item.
When an array changes, Angular re-renders the whole DOM tree.
But if we use `trackBy`, Angular will know which element has changed and will only make DOM changes for that element.

```typescript
@Component({
  selector: 'my-component',
  template: `<li *ngFor="let item of items; trackBy: trackByFn">{{ item.name }}</li>`
})
export class MyComponent {
  @Input()
  items: object[];

  trackByFn(index: number, item: object): number {
    return item.id;
  }
}
```

## Use virtual scrolling

In [virtual scrolling](https://dev.to/adamklein/build-your-own-virtual-scroll-part-i-11ib), we don't display the entire content on the screen, to reduce the amount of DOM node rendering and calculations.
We "fool" the user to think the entire content is rendered by always rendering just the part inside the window, and a bit more on the top and bottom to ensure smooth transitions.
We can use `<cdk-virtual-scroll-viewport>` directive from [Angular CDK](https://material.angular.io/cdk/scrolling/).

## Use lazy loading

When possible, try to lazy load the modules in Angular application.
[Lazy loading](https://angular.io/guide/lazy-loading-ngmodules) helps keep initial bundle sizes smaller, which in turn helps decrease load times.

```typescript
const routes: Routes = [{
  path: "customer-list",
  loadChildren: () => import("./customers/customers.module").then(m => m.CustomersModule)
}];
```

Lazy, or "on demand", loading is a great way to optimize our site or application.
This practice essentially involves splitting our code at logical breakpoints, and then loading it once the user has done something that requires, or will require, a new block of code.
This speed up the initial load of the application and lightens its overall weight as some blocks may never even be loaded.

## Avoid poorly structured CSS

Common mistakes are excessive use of deep selectors and inline styles.
[Inline styles](https://www.w3schools.com/css/css_howto.asp) are considered as bad practice due to poor scalability and maintainability.
As a rule of thumb, define all styles in the CSS files.
Usage of [::ng-deep](https://angular.io/guide/component-styles#deprecated-deep--and-ng-deep) to overwrite styles in other components is incredibly popular.
Despite being a working solution, it's marked as deprecated.
The main reason for that is that this mechanism for piercing the style isolation sandbox around a component can potentially encourage bad styling practices.
Though, it isn't going away until Angular implements `::part()` and `::theme()` from the [CSS Shadow Parts](https://drafts.csswg.org/css-shadow-parts/) spec, as there is no better alternative.

## Lack of meaningful unit tests

Angular CLI encourages to write unit tests by spanning out `*.spec.ts` files with every created component.
However, don't leave them empty or be satisfied by configuring the [TestBed](https://angular.io/api/core/testing/TestBed) with component initialization without actual tests.
If developers don't write tests, then absence of a test file would clearly indicate the state of affairs to other developers, rather than misleading them by giving a false sense of security with a rudimental `*.spec.ts` file.
We need to cover with tests the most fragile parts, rather than covering what's easier to test.

## Add caching mechanisms

When making API calls, responses from some of them do not change often.
In those cases, we can add a caching mechanism and store the value from the API.
When another request to the same API is made, we checked if there is a value for it in the cache and if so, use it.
Otherwise, make the API call and cache the result.
Having a caching mechanism means avoiding unwanted API calls.
By only making the API calls when required and avoiding duplication, the speed of the application improves as we do not have to wait for the network.
It also means we do not download the same information repeatedly.

## Avoid magic numbers

Magic numbers are values that appear in source code without any explanation of what they mean.
This makes the code difficult to understand and maintain.
Magic numbers should be avoided as they often lack documentation.
Forcing them to be stored in variables gives them implicit documentation.
With [no-magic-numbers](https://palantir.github.io/tslint/rules/no-magic-numbers/) lint rule, we make code more readable and refactoring easier by ensuring that special numbers are declared as constants to make their meaning explicit.

## Avoid useless code comments

Comments are considered a best practice, but if we are adding a comment, it's because it's not self-explanatory and we should choose a better way to implement it.

Good comments are informative comments, when be useful to provide basic information.
For example, a comment that contains legal information, or are a warning, when we are working with multiple developers on a project, we could use a comment to warn other developers about certain consequences, or are a to-do comments for tasks a developer thinks should be done, but for some reason can't be done at this moment.

Bad Comments are commented-out code is a common practice, but we shouldn't do it, because other developers will think the code is there for a reason and won't have the courage to delete it.
Just delete the code.
We have got version control, so the code isn't lost forever.
Another case is noise comments. Some comments that we see are just noise.
They restate the obvious and serve no real purpose.
Redundant comments are comments that are not more informative than the code.
These comments only clutter the code.

## Remove unused code

Unused code or [dead code](https://dev.to/apastuhov/dead-code-problem-3o2) is any code which will never be executed.
It may be some condition, loop or any file which was simply created but wasn't used in our project.
It is a problem because that code has no sense and we can drop it.
[Dead-code Elimination](https://webpack.js.org/guides/tree-shaking/) also reduces the size of our bundles and repositories. Less code also increases maintenance, IDE performance and makes it easier to understand.
Common mistakes in TypeScript projects are unused imports, variables, functions and private class members.
With [no-unused-variable](https://palantir.github.io/tslint/rules/no-unused-variable/) lint rule are automatically remove unused imports, variables, functions, and private class members, when using TSLint's --fix option.

## Separation of concerns

Angular is built around separation of concerns.
This is a [design-pattern](https://medium.com/@rkay301/programming-fundamentals-part-5-separation-of-concerns-software-architecture-f04a900a7c50) that makes our code easier to maintain and extend, and more reusable and testable.
It helps us encapsulate and limit the logic of components to satisfy what the template needs, and nothing more.
Separation of concerns is the core of writing clean code in Angular.
It's important follow the [Angular Style Guide](https://angular.io/guide/styleguide).
It will make our life a lot easier, when coming into new projects and will increase the quality of our code almost automatically.

## Use aliases for imports

Sometimes, we may use imports three folders deep or more, so the following import is not the ideal solution for the project.

```typescript
import { LoaderService } from '../../../loader/loader.service'
```

In TypeScript, we can avoid these “bad” looking imports with the help of [path aliases](https://www.typescriptlang.org/docs/handbook/module-resolution.html#path-mapping).
With path aliases we can declare aliases that map to a certain absolute path in our application.
Path aliases are defined in the `compilerOptions` section in the `tsconfig.json` file.

```typescript
import { LoaderService } from ' @app/loader/loader.service'
```

## Develop in a modular way

When we start developing in Angular, we may be tempted to disregard creation of modules for the sake of using components solely. That approach might be fine for smaller apps, but as our app starts to grow, development will become cumbersome.
That's when separation of concerns steps in, which is fueled by a modular Angular app.
Splitting our app into core, shared and multiple feature modules will make our life much easier.
Each module can have its own components, services, directives and pipes.
Modules help to organize our code into smaller bundles to make finding things easier.
With the help of lazy-loaded modules, we can also improve the user experience by only downloading the parts of the application, that are required at that moment.

## Components should only deal with display logic

We want our components to be as simple as possible.
Avoid having any logic other than the display logic in the components whenever possible and make the component deal only with the display logic.
Components are designed for presentational purposes and control what the view should do.
This means if our component needs to do some complex logic we need to decide if that logic belongs to the component or not.
Any business logic should be extracted into its own methods/services where appropriate, separating business logic from view logic.
Business logic is usually easier to unit test when extracted out to a service and can be reused by any other components that need the same business logic applied.

## Small reusable components

Ideally, a single component should render a specific bit of our page or modify a behavior.
It means, we should keep them small, so that one component corresponds to one function.
Make the component as dumb as possible, as this will make it work in more scenarios.
Making a component dumb means that the component does not have any special logic in it and operates purely based on the inputs and outputs provided to it.
Dumb components are simpler, so they are less likely to have bugs.
Dumb components make us think harder about the public component API and help sniff out mixed concerns.
Generally, the last child in the component tree will be the dumbest of all.
Reusable components reduce duplication of code therefore making it easier to maintain and make changes.

## Smart and dummy components

Most common use case of developing Angular's components is a separation of [smart and dummy components](https://blog.angular-university.io/angular-2-smart-components-vs-presentation-components-whats-the-difference-when-to-use-each-and-why/).
A dummy component is a component used for presentation purposes only, meaning that the component doesn't know where the data came from.
For that purpose, we can use one or more smart components that will inherit dummy's component presentation logic.
Making a component dumb means that the component does not have any special logic in it and operates purely based on the inputs and outputs provided to it.
Dumb components are simpler, so they are less likely to have bugs.
A smart component does not have to be a top-level router component only.
We can have other components further down the tree and don't necessarily get their data only from `@Input()`.

## Base component classes

Create a base class component may come in handy when we have lots of reused stuff and don't want to pollute each component with the same code all over.
Common situations are when we are creating form components, when we have pages with the same behavior, such as a list with the create, read, update and delete methods, or pages with HTML forms.
In these examples, having the same code in multiple places means that if we want to make a change to the logic in that code, we must do it in multiple places.
We can create a base class with the common data and methods.
Thus, we don't have the same duplicate code in different locations in the code base.

## Do not remove view encapsulation

In Angular, component CSS styles are encapsulated into the component's view and don't affect the rest of the application.
To control how this encapsulation happens on a per component basis, we can set the [view encapsulation](https://angular.io/guide/view-encapsulation) mode in the component metadata.
The default is Emulated and it  emulates the behavior of [Shadow DOM](https://developer.mozilla.org/en-US/docs/Web/Web_Components/Using_shadow_DOM) by preprocessing the CSS code to effectively scope the CSS to the component's view.
In the None mode, styles from the component propagate back to the main HTML and therefore are visible to all components on the page.
We can use this option, but we need to be careful and adopt other strategies like nested CSS or naming conventions like [BEM](http://getbem.com/introduction/).

## Use reactive forms

Angular presents two different methods for creating forms: [template-driven](https://angular.io/guide/forms) and [reactive forms](https://angular.io/guide/reactive-forms).
Reactive forms provide a model-driven approach to handling form inputs whose values change over time.
The Reactive approach removes validation logic from the template, keeping the templates clean of validation logic.
Reactive forms use an explicit and immutable approach to managing the state of a form at a given point in time.
Each change to the form state returns a new state, which maintains the integrity of the model between changes.

## Bibliography

- [Angular coding style guide](https://angular.io/guide/styleguide)
- [Angular Performance Advice](https://medium.com/slackernoon/angular-performance-enhancement-413c6f950ebb)
- [Angular Bad Practices](https://medium.com/angular-in-depth/angular-bad-practices-revisited-4f607fcb75da)
- [Angular Reactive Templates with ngIf and the Async Pipe](https://blog.angular-university.io/angular-reactive-templates/)
- [Angular Smart Components vs Presentational Components](https://blog.angular-university.io/angular-2-smart-components-vs-presentation-components-whats-the-difference-when-to-use-each-and-why/)
- [A Comprehensive Guide to Angular onPush Change Detection Strategy](https://netbasal.com/a-comprehensive-guide-to-angular-onpush-change-detection-strategy-5bac493074a4)
- [Best practices for a clean and performant Angular application](https://medium.com/free-code-camp/best-practices-for-a-clean-and-performant-angular-application-288e7b39eb6f)
- [Best Practices in Angular](https://itnext.io/best-practices-in-angular-a8926fa02ae2)
- [Clean Code Checklist in Angular](https://itnext.io/clean-code-checklist-in-angular-️-10d4db877f74)
- [Don't follow RxJS Best Practices](https://dev.to/nikpoltoratsky/don-t-follow-rxjs-best-practices-4893)
- [RxJS in Angular: When To Subscribe?](https://indepth.dev/posts/1279/rxjs-in-angular-when-to-subscribe-rarely)
- [5 Things that Improve your Angular Codebase Right Now](https://malcoded.com/posts/improve-your-angular-codebase/)
- [6 most common mistakes of Angular devs revealed on code reviews](https://alex-klaus.com/angular-code-review/)
