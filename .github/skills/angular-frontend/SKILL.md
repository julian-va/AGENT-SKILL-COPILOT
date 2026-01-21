---
name: angular-frontend
version: 0.1.0
author: Your Name
contact: your.email@example.com
tags: [angular, frontend, typescript, signals, standalone, guidelines]
---

# Skill: Angular Frontend Development

## Purpose

This skill defines how the AI agent should write, refactor, and review Angular frontend code
in this repository. It follows modern Angular best practices, standalone components, signals,
and Clean Architecture principles for frontend applications.

---

## Language & Version

- Angular 17+ (prefer latest stable)
- TypeScript 5.x+ with strict mode enabled
- Node.js 20+ LTS
- Use modern Angular features: standalone components, signals, control flow syntax

## Build tools

- Angular CLI (latest version)
- Angular Material (Material Design 3 theme)
- npm or pnpm for package management
- Prefer pnpm for better performance and disk efficiency

---

## Metadata (machine-readable)

- Inputs: `request` (JSON) — action and payload; `config` — runtime configuration
- Outputs: `response` (JSON) — {status, data, errors}
- Automations: lint (ESLint), fmt (Prettier), unit-tests (Jest/Karma), e2e-tests (Playwright/Cypress)

Example input schema (JSON):

```json
{
  "action": "createUser",
  "payload": { "name": "Alice", "email": "alice@example.com" }
}
```

Example output schema (JSON):

```json
{
  "status": "ok|error",
  "data": {},
  "errors": []
}
```

---

## Code Style Rules

### TypeScript Best Practices

- Use strict type checking (`strict: true` in tsconfig.json)
- Prefer type inference when the type is obvious
- Avoid the `any` type; use `unknown` when type is uncertain
- Use `const` by default, `let` only when reassignment is needed
- Never use `var`

### Angular Best Practices

- **Always use standalone components** — NgModules are legacy
- **For Angular 19+**: Standalone is the default, do NOT set `standalone: true`
- **For Angular 17-18**: Must explicitly set `standalone: true` in component decorator
- Use signals for state management (`signal()`, `computed()`)
- Implement lazy loading for feature routes
- Use `NgOptimizedImage` for all static images (not for inline base64)
- **Do NOT use `@HostBinding` and `@HostListener`** — put host bindings inside the `host` object
- **Do NOT use `ngClass`** — use `class` bindings instead: `[class.active]="isActive()"`
- **Do NOT use `ngStyle`** — use `style` bindings instead: `[style.color]="color()"`

---

## Architecture Rules (Clean Architecture for Frontend)

- Follow Clean Architecture principles adapted for Angular frontend applications
- Clear separation between presentation, business logic, and data access

### Layer Structure

```
src/app/
├── core/
│   ├── models/          # Domain models and interfaces
│   ├── services/        # Business logic services
│   └── guards/          # Route guards
│
├── data/
│   ├── repositories/    # Data access layer (implements interfaces from core)
│   ├── api/             # HTTP clients and API services
│   └── mappers/         # DTO to domain model mappers
│
├── features/
│   ├── feature-name/
│   │   ├── components/  # Feature-specific components
│   │   ├── services/    # Feature-specific services
│   │   └── models/      # Feature-specific models
│   └── ...
│
├── shared/
│   ├── components/      # Reusable UI components
│   ├── directives/      # Shared directives
│   ├── pipes/           # Shared pipes
│   └── utils/           # Utility functions
│
└── layout/
    ├── header/
    ├── footer/
    └── sidebar/
```

### Dependency Rules

- **Core** should not depend on data or features (pure business logic)
- **Data** implements interfaces defined in core
- **Features** can depend on core and data
- **Shared** should be independent and reusable
- Dependencies point inward: features → core ← data

### Repository Pattern

Use the repository pattern to abstract data access:

```typescript
// core/models/user.interface.ts
export interface User {
  id: string;
  name: string;
  email: string;
}

// core/repositories/user-repository.interface.ts
export interface UserRepository {
  getAll(): Observable<User[]>;
  getById(id: string): Observable<User>;
  create(user: Omit<User, "id">): Observable<User>;
  update(id: string, user: Partial<User>): Observable<User>;
  delete(id: string): Observable<void>;
}

// data/repositories/user-repository.impl.ts
@Injectable({ providedIn: "root" })
export class UserRepositoryImpl implements UserRepository {
  private http = inject(HttpClient);
  private mapper = inject(UserMapper);

  getAll(): Observable<User[]> {
    return this.http
      .get<UserDto[]>("/api/users")
      .pipe(map((dtos) => dtos.map((dto) => this.mapper.toDomain(dto))));
  }

  // ... other methods
}
```

---

## Component Rules

- Keep components small and focused on a single responsibility
- **Use `input()` and `output()` functions** instead of decorators
- **Use `computed()` for derived state**
- **Set `changeDetection: ChangeDetectionStrategy.OnPush`** in all components
- Prefer inline templates for small components (< 10 lines)
- Extract complex templates to separate HTML files
- Use Reactive forms instead of Template-driven forms

Example component:

```typescript
import {
  Component,
  ChangeDetectionStrategy,
  input,
  output,
  computed,
} from "@angular/core";

@Component({
  selector: "app-user-card",
  standalone: true,
  changeDetection: ChangeDetectionStrategy.OnPush,
  template: `
    <div class="card" [class.selected]="isSelected()">
      <h3>{{ displayName() }}</h3>
      <p>{{ user().email }}</p>
      <button (click)="handleSelect()">Select</button>
    </div>
  `,
  styles: [
    `
      .card {
        padding: 1rem;
        border: 1px solid #ccc;
      }
      .card.selected {
        border-color: blue;
      }
    `,
  ],
})
export class UserCardComponent {
  user = input.required<User>();
  selected = input<boolean>(false);
  userSelect = output<User>();

  displayName = computed(() => this.user().name.toUpperCase());
  isSelected = computed(() => this.selected());

  handleSelect(): void {
    this.userSelect.emit(this.user());
  }
}
```

---

## State Management

- **Use signals for local component state**
- Use `computed()` for derived state
- Keep state transformations pure and predictable
- **Do NOT use `mutate` on signals** — use `update` or `set` instead
- For complex global state, consider using a state management library (NgRx with signals)

Example state management:

```typescript
@Component({...})
export class UserListComponent {
  private userRepo = inject(UserRepository);

  users = signal<User[]>([]);
  filter = signal<string>('');

  filteredUsers = computed(() => {
    const filterValue = this.filter().toLowerCase();
    return this.users().filter(user =>
      user.name.toLowerCase().includes(filterValue)
    );
  });

  addUser(user: User): void {
    this.users.update(current => [...current, user]);
  }
}
```

---

## Template Rules

- Keep templates simple and avoid complex logic
- **Use native control flow** (`@if`, `@for`, `@switch`) instead of `*ngIf`, `*ngFor`, `*ngSwitch`
- Use the `async` pipe to handle observables
- Use `trackBy` functions with `@for` for performance

Example modern template:

```html
<div class="user-list">
  @if (loading()) {
  <app-spinner />
  } @else if (error()) {
  <app-error [message]="error()" />
  } @else { @for (user of filteredUsers(); track user.id) {
  <app-user-card
    [user]="user"
    [selected]="selectedId() === user.id"
    (userSelect)="onUserSelect($event)"
  />
  } @empty {
  <p>No users found</p>
  } }
</div>
```

---

## Service Rules

- Design services around a single responsibility
- **Use `providedIn: 'root'`** for singleton services
- **Use the `inject()` function** instead of constructor injection
- Keep services framework-agnostic when possible
- Use RxJS operators for reactive programming

Example service:

```typescript
@Injectable({ providedIn: "root" })
export class UserService {
  private userRepo = inject(UserRepository);
  private toastr = inject(ToastrService);

  loadUsers(): Observable<User[]> {
    return this.userRepo.getAll().pipe(
      catchError((error) => {
        this.toastr.error("Failed to load users");
        return of([]);
      }),
    );
  }
}
```

---

## Routing & Lazy Loading

- Use lazy loading for all feature routes
- Implement route guards using functional guards
- Use `provideRouter` with standalone components

Example routing:

```typescript
// app.routes.ts
export const routes: Routes = [
  {
    path: "",
    redirectTo: "home",
    pathMatch: "full",
  },
  {
    path: "home",
    loadComponent: () =>
      import("./features/home/home.component").then((m) => m.HomeComponent),
  },
  {
    path: "users",
    loadComponent: () =>
      import("./features/users/users.component").then((m) => m.UsersComponent),
    canActivate: [authGuard],
  },
];

// guards/auth.guard.ts
export const authGuard: CanActivateFn = (route, state) => {
  const authService = inject(AuthService);
  return (
    authService.isAuthenticated() || inject(Router).createUrlTree(["/login"])
  );
};
```

---

## Testing Rules

- Use Jest for unit testing (preferred over Karma)
- Use Playwright or Cypress for E2E testing
- Aim for > 80% code coverage
- Test components, services, and pipes
- Use signal testing utilities

Example component test:

```typescript
import { ComponentFixture, TestBed } from "@angular/core/testing";
import { UserCardComponent } from "./user-card.component";

describe("UserCardComponent", () => {
  let component: UserCardComponent;
  let fixture: ComponentFixture<UserCardComponent>;

  beforeEach(async () => {
    await TestBed.configureTestingModule({
      imports: [UserCardComponent],
    }).compileComponents();

    fixture = TestBed.createComponent(UserCardComponent);
    component = fixture.componentInstance;

    fixture.componentRef.setInput("user", {
      id: "1",
      name: "Alice",
      email: "alice@example.com",
    });

    fixture.detectChanges();
  });

  it("should display user name in uppercase", () => {
    expect(component.displayName()).toBe("ALICE");
  });

  it("should emit userSelect when button clicked", () => {
    let emittedUser: User | undefined;
    component.userSelect.subscribe((user) => (emittedUser = user));

    component.handleSelect();

    expect(emittedUser).toBeDefined();
    expect(emittedUser?.id).toBe("1");
  });
});
```

---

## UI Component Library: Angular Material

- **Use Angular Material** as the primary UI component library
- Follow **Material Design 3 (M3)** guidelines and theming
- Import only the specific Material modules needed (tree-shaking)
- Use Material's theming system for consistent colors, typography, and spacing

### Angular Material Setup

Install Angular Material with M3 theme:

```bash
ng add @angular/material
```

Select Material 3 theme when prompted (e.g., `azure-blue`, `rose-red`, `cyan-orange`, `magenta-violet`)

### Material Components Best Practices

- **Import components individually** for better tree-shaking:

  ```typescript
  import { MatButtonModule } from "@angular/material/button";
  import { MatCardModule } from "@angular/material/card";
  import { MatFormFieldModule } from "@angular/material/form-field";
  ```

- **Use Material's theming system** in your components:

  ```scss
  @use "@angular/material" as mat;

  .custom-card {
    background-color: mat.get-theme-color($theme, primary);
    color: mat.get-theme-color($theme, on-primary);
  }
  ```

- **Follow Material Design 3 spacing** (use multiples of 4px or 8px)
- **Use Material elevation and surfaces** for depth and hierarchy
- **Prefer Material icons** (`mat-icon` with Material Symbols)

### Material 3 Theme Configuration

Configure your theme in `styles.scss`:

```scss
@use "@angular/material" as mat;

// Define your color palette based on Material 3
$my-primary: mat.define-palette(mat.$azure-palette);
$my-accent: mat.define-palette(mat.$blue-palette);
$my-warn: mat.define-palette(mat.$red-palette);

// Create the theme
$my-theme: mat.define-theme(
  (
    color: (
      theme-type: light,
      primary: $my-primary,
      tertiary: $my-accent,
    ),
    typography: (
      brand-family: "Roboto, sans-serif",
      plain-family: "Roboto, sans-serif",
    ),
    density: (
      scale: 0,
    ),
  )
);

// Apply the theme
html {
  @include mat.all-component-themes($my-theme);
}

// Dark theme support
@media (prefers-color-scheme: dark) {
  html {
    $dark-theme: mat.define-theme(
      (
        color: (
          theme-type: dark,
          primary: $my-primary,
          tertiary: $my-accent,
        ),
      )
    );

    @include mat.all-component-colors($dark-theme);
  }
}
```

### Material Component Examples

Use Material components with signals and modern syntax:

```typescript
@Component({
  selector: "app-user-form",
  standalone: true,
  imports: [
    MatFormFieldModule,
    MatInputModule,
    MatButtonModule,
    ReactiveFormsModule,
  ],
  template: `
    <form [formGroup]="form()" (ngSubmit)="onSubmit()">
      <mat-form-field appearance="outline">
        <mat-label>Name</mat-label>
        <input matInput formControlName="name" />
        @if (form().get("name")?.hasError("required")) {
          <mat-error>Name is required</mat-error>
        }
      </mat-form-field>

      <mat-form-field appearance="outline">
        <mat-label>Email</mat-label>
        <input matInput type="email" formControlName="email" />
        @if (form().get("email")?.hasError("email")) {
          <mat-error>Please enter a valid email</mat-error>
        }
      </mat-form-field>

      <button mat-raised-button color="primary" type="submit">Submit</button>
    </form>
  `,
})
export class UserFormComponent {
  private fb = inject(FormBuilder);

  form = signal(
    this.fb.group({
      name: ["", Validators.required],
      email: ["", [Validators.required, Validators.email]],
    }),
  );

  onSubmit(): void {
    if (this.form().valid) {
      console.log(this.form().value);
    }
  }
}
```

### Material Design 3 Guidelines

Follow Material Design 3 principles:

- **Color system**: Use theme colors (primary, secondary, tertiary, error)
- **Typography**: Use Material's type scale (display, headline, title, body, label)
- **Shape**: Use rounded corners (Material 3 uses more rounded shapes)
- **Elevation**: Use surface tint for elevation instead of shadows
- **Motion**: Use Material's animation curves and durations
- **Accessibility**: Follow WCAG 2.1 Level AA standards

Key M3 resources:

- Material Design 3: https://m3.material.io
- Angular Material Guide: https://material.angular.io
- Material Theme Builder: https://material-foundation.github.io/material-theme-builder

---

## Forbidden Patterns

- No NgModules (use standalone components only)
- No `@HostBinding` or `@HostListener` (use `host` object)
- No `ngClass` or `ngStyle` (use class/style bindings)
- No `*ngIf`, `*ngFor`, `*ngSwitch` (use `@if`, `@for`, `@switch`)
- No `@Input()` or `@Output()` decorators (use `input()` and `output()` functions)
- No constructor injection (use `inject()` function)
- No `mutate()` on signals (use `update()` or `set()`)
- No `any` type (use proper types or `unknown`)
- No manual subscriptions without cleanup (use `async` pipe or `takeUntilDestroyed()`)

---

## Reasoning Checklist (agent must return before code)

When asked to implement or modify code, the agent must return a short checklist containing:

1. Affected layer(s) (component/service/repository/model)
2. Component strategy (standalone, signals, OnPush)
3. Design validation vs. rules above (list any rule exceptions)
4. Dependencies and injection approach
5. Testing approach (unit/integration/e2e)

Example response the agent must produce before code:

```
Layer: feature component + repository
Strategy: Standalone component, signals for state, OnPush change detection
Validation: Uses inject() function, input()/output() functions, @for with track
Dependencies: UserRepository (injected), computed for derived state
Testing: Unit tests with Jest, component fixture testing
```

If a request violates the rules, explain why and propose an alternative.

---

## CI / Automation (suggested)

Add a GitHub Actions workflow `.github/workflows/angular-ci.yml` to run:

- Node.js 20 LTS
- Install dependencies with pnpm
- Run `ng lint`
- Run `ng test` (unit tests)
- Run `ng build --configuration production`
- Run E2E tests

Example workflow snippet:

```yaml
name: Angular CI
on: [push, pull_request]
jobs:
  build-and-test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: "20"

      - name: Setup pnpm
        uses: pnpm/action-setup@v2
        with:
          version: 8

      - name: Install dependencies
        run: pnpm install

      - name: Lint
        run: pnpm run lint

      - name: Test
        run: pnpm run test:ci

      - name: Build
        run: pnpm run build --configuration production
```

---

## Accessibility (a11y)

- Use semantic HTML elements
- **Use Angular Material's built-in accessibility features**
- Add ARIA labels where needed (Material components include many by default)
- Ensure keyboard navigation works (Material components are keyboard-accessible)
- Maintain proper color contrast (use Material 3 color system for WCAG compliance)
- Test with screen readers
- Follow WCAG 2.1 Level AA standards
- Use Material's `MatLiveAnnouncer` for dynamic content announcements

---

## Performance Best Practices

- Use `NgOptimizedImage` for images
- Implement `trackBy` with `@for` loops
- Use `OnPush` change detection everywhere
- Lazy load routes and components
- Use signals for efficient change detection
- Avoid unnecessary computations in templates
- Use `async` pipe to auto-unsubscribe from observables

---

## Notes & Exceptions

- Legacy code may still use NgModules; migrate incrementally to standalone
- `async` pipe is still valid and recommended for observables
- Document any exception to component-size or template-complexity rules in PR description

---

## Change log

- 0.1.0: Initial Angular frontend skill (standalone, signals, clean architecture, modern syntax, Angular Material with M3)
