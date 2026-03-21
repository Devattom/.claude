# Clean Code — Robert C. Martin: Complete Reference (TypeScript)

> "Any fool can write code that a computer can understand. Good programmers write code that humans can understand." — Martin Fowler

---

## PRIME DIRECTIVE — READ THIS FIRST

**Readability and simplicity always win over strict rule application.**

Every rule in this document is a heuristic, not a law. Before applying any rule, ask:

> "Does this change make the code easier to understand for the next developer?"

If the answer is **NO** or **UNCERTAIN** → skip the change, explain why.

**When NOT to apply Clean Code rules:**
- Extracting a 3-line function called only once → adds indirection, no clarity gain
- Renaming well-understood abbreviations (`url`, `id`, `i`, `ctx`) → adds noise
- Splitting a naturally readable 25-line function into 5 fragments → hurts comprehension
- Applying polymorphism to a switch with 2 cases that will never grow → over-engineering
- Adding a named function for an obvious boolean (`user.age >= 18` doesn't need `isAdult()` if context is clear)
- Creating an interface for a dependency with only one implementation and no test doubles needed

**The goal is a codebase any developer can read fluently. Rules serve that goal — not the reverse.**

---

## 1. NAMING

### Intent-revealing names

```typescript
// BAD
const d = 86400;
function getThem(list: number[][]): number[][] {
  const r: number[][] = [];
  for (const x of list) if (x[0] === 4) r.push(x);
  return r;
}

// GOOD
const SECONDS_PER_DAY = 86400;
function getFlaggedCells(gameBoard: Cell[][]): Cell[][] {
  return gameBoard.filter(cell => cell.isFlagged());
}
```

### Pronounceable names
```typescript
// BAD
interface DtaRcrd { genymdhms: Date; modymdhms: Date; }

// GOOD
interface CustomerRecord { generationTimestamp: Date; modificationTimestamp: Date; }
```

### Searchable names — no magic numbers
```typescript
// BAD
setTimeout(tick, 86400000);

// GOOD
const MILLISECONDS_PER_DAY = 86_400_000;
setTimeout(tick, MILLISECONDS_PER_DAY);
```

### No encodings (no Hungarian notation)
```typescript
// BAD
interface IShapeFactory { }
const strName = "Alice";
class CUserService { }

// GOOD
interface ShapeFactory { }
const name = "Alice";
class UserService { }
```

### Classes = nouns / methods = verbs
```typescript
// BAD — classes with verbs
class ProcessUser { }
class DataManager { }   // "Manager" is vague
class UserInfo { }      // "Info" is vague

// GOOD
class UserAuthenticator { }
class UserRepository { }
class Account { }

// BAD — methods with nouns
class UserService {
  user(id: string): User { }
  password(): boolean { }
}

// GOOD
class UserService {
  findById(id: string): User { }
  isPasswordValid(password: string): boolean { }
}
```

### One word per concept — absolute consistency
```typescript
// BAD — three verbs for the same operation
class UserRepo { fetchById(id: string): User { } }
class OrderRepo { retrieveById(id: string): Order { } }
class ProductRepo { getById(id: string): Product { } }

// GOOD
class UserRepo { findById(id: string): User { } }
class OrderRepo { findById(id: string): Order { } }
class ProductRepo { findById(id: string): Product { } }
```

### No mental mapping — no opaque abbreviations

⚠️ **Exception**: Common abbreviations are fine (`url`, `id`, `i`, `ctx`, `req`, `res`, `err`). Only rename when the abbreviation genuinely causes confusion.

```typescript
// BAD — truly opaque
urls.forEach(u => {
  const m = u.match(/\/(\d+)$/);
  if (m) doSomething(m[1]);
});

// GOOD
urls.forEach(url => {
  const idMatch = url.match(/\/(\d+)$/);
  if (idMatch) processResourceId(idMatch[1]);
});
```

---

## 2. FUNCTIONS

### Small, one thing, one level of abstraction

**Target: 20 lines max. Ideally 5-10.**

⚠️ **Exception**: A function that reads naturally as a linear sequence of steps (even 30 lines) is often better left intact than fragmented into tiny pieces that break the reading flow.

```typescript
// BAD — mixes validation, hashing, DB, email at different abstraction levels
function processUserRegistration(data: unknown): void {
  if (typeof data !== 'object' || data === null) throw new Error('Invalid');
  const { email, password, name } = data as Record<string, unknown>;
  if (!String(email).includes('@')) throw new Error('Invalid email');
  const salt = crypto.randomBytes(16).toString('hex');
  const hash = crypto.createHmac('sha256', salt).update(String(password)).digest('hex');
  db.run(`INSERT INTO users VALUES (?, ?, ?)`, [email, `${salt}:${hash}`, name]);
  mailer.send({ to: String(email), subject: 'Welcome!' });
}

// GOOD — each function operates at one abstraction level
function processUserRegistration(rawData: unknown): void {
  const data = validateRegistrationData(rawData);
  const user = createUser(data);
  saveUser(user);
  sendWelcomeEmail(user);
}
```

### Step-down rule

Read code top to bottom — each function introduces the next level of detail.

```typescript
// GOOD — natural descending narrative
class ReportGenerator {
  generate(period: Period): Report {
    const rawData = this.fetchData(period);
    const processed = this.processData(rawData);
    return this.formatReport(processed);
  }

  private fetchData(period: Period): RawData { /* ... */ }
  private processData(data: RawData): ProcessedData { /* ... */ }
  private formatReport(data: ProcessedData): Report { /* ... */ }
}
```

### Arguments: 0-2 ideal, 3 limit, 4+ refactor

```typescript
// BAD — 6 arguments, arbitrary order, "true" is incomprehensible
createEvent("Conf", new Date(), "Paris", 100, true, "user-123");

// GOOD — argument object
interface EventOptions {
  title: string;
  date: Date;
  location: string;
  maxAttendees: number;
  isPublic: boolean;
  organizerId: string;
}
createEvent({ title: "Conf", date: new Date(), location: "Paris", ... });
```

### Flag arguments are evil — signals the function does two things

⚠️ **Exception**: If the two paths share 90% of the logic and the flag is clearly named, splitting may add more complexity than it removes. Use judgment.

```typescript
// BAD
function render(pageData: PageData, isSuite: boolean): string { }
render(data, true);  // what does "true" mean?

// GOOD
function renderForSuite(pageData: PageData): string { }
function renderForPage(pageData: PageData): string { }
```

### Avoid switch — prefer polymorphism

⚠️ **Exception**: A switch with 2-3 stable cases that will never grow is often clearer than introducing a class hierarchy. Polymorphism pays off when new types are regularly added.

```typescript
// BAD — switch duplicated everywhere for every new type
function calculatePay(employee: Employee): number {
  switch (employee.type) {
    case 'COMMISSIONED': return calculateCommissionedPay(employee);
    case 'HOURLY': return calculateHourlyPay(employee);
    default: throw new Error(`Unknown: ${employee.type}`);
  }
}

// GOOD — polymorphism (when types are expected to grow)
interface Employee { calculatePay(): number; }
class CommissionedEmployee implements Employee {
  calculatePay(): number { return this.basePay + this.commission; }
}
class HourlyEmployee implements Employee {
  calculatePay(): number { return this.hourlyRate * this.hoursWorked; }
}
```

### No hidden side effects

```typescript
// BAD — checkPassword secretly initializes the session
function checkPassword(userName: string, password: string): boolean {
  const user = UserGateway.findByName(userName);
  if (isValidPassword(user, password)) {
    Session.initialize(); // HIDDEN SIDE EFFECT!
    return true;
  }
  return false;
}

// GOOD — clear separation
function isPasswordValid(userName: string, password: string): boolean { /* ... */ }
function authenticateUser(userName: string, password: string): void {
  if (!isPasswordValid(userName, password)) throw new AuthError();
  Session.initialize();
}
```

### Command/Query Separation (CQS)

A function either **changes state** (command) or **returns a value** (query). Never both.

```typescript
// BAD — modifies AND returns
function set(attribute: string, value: string): boolean {
  if (!this.attributes.has(attribute)) return false;
  this.attributes.set(attribute, value);
  return true;
}

// GOOD
function hasAttribute(attribute: string): boolean {
  return this.attributes.has(attribute);
}
function setAttribute(attribute: string, value: string): void {
  if (!this.attributes.has(attribute)) throw new Error(`Unknown: ${attribute}`);
  this.attributes.set(attribute, value);
}
```

### DRY (Don't Repeat Yourself)

```typescript
// BAD — duplicated validation
function createUser(email: string): void {
  if (!email.includes('@')) throw new Error('Invalid email');
  // ...
}
function updateEmail(id: string, email: string): void {
  if (!email.includes('@')) throw new Error('Invalid email'); // duplicated!
  // ...
}

// GOOD
function assertValidEmail(email: string): void {
  if (!email.includes('@')) throw new ValidationError('Invalid email');
}
function createUser(email: string): void { assertValidEmail(email); /* ... */ }
function updateEmail(id: string, email: string): void { assertValidEmail(email); /* ... */ }
```

---

## 3. COMMENTS

**Core rule: code should be expressive enough to not need comments.**

> "Don't comment bad code — rewrite it." — Kernighan

⚠️ **Context matters**: In complex domain logic, business rules, or non-obvious algorithms, explanatory comments are valuable even if the code is clean. Comment the *why*, never the *what*.

### Bad comments (delete these)

```typescript
// BAD — redundant
i++; // Increment i

// BAD — change log (git's job)
// 2024-01-15 Alice — added validation
// 2024-02-20 Bob — fixed bug #1234

// BAD — commented-out code (delete it, git keeps history)
// const discount = calculateDiscount(order);
// order.applyDiscount(discount);

// BAD — noise with no information
constructor() {} // Default constructor
private name: string = ''; // The name
```

### Good comments (legitimate)

```typescript
// GOOD — intent (why, not what)
// Compare by UUID to guarantee deterministic ordering
// even when two elements share the same label.
return a.id.localeCompare(b.id);

// GOOD — clarification of a complex regex
// Format: ISO 8601 with mandatory timezone — e.g. "2024-01-15T14:30:00+02:00"
const ISO_DATE_WITH_TZ = /^\d{4}-\d{2}-\d{2}T\d{2}:\d{2}:\d{2}[+-]\d{2}:\d{2}$/;

// GOOD — warning of consequence
// WARNING: this test takes ~3 minutes (waits for JWT token expiry)
// Do not include in the fast test suite.

// GOOD — TODO with ticket
// TODO [JIRA-4521]: replace with Redis — currently re-reading from DB on every request
```

---

## 4. FORMATTING

### Vertical openness — separate concepts with blank lines

```typescript
// BAD — everything compressed
class UserService {
  private repo: UserRepository;
  private mailer: Mailer;
  constructor(repo: UserRepository, mailer: Mailer) {
    this.repo = repo;
    this.mailer = mailer;
  }
  async createUser(data: CreateUserData): Promise<User> {
    const user = new User(data);
    await this.repo.save(user);
    await this.mailer.sendWelcome(user.email);
    return user;
  }
}

// GOOD
class UserService {
  private readonly repo: UserRepository;
  private readonly mailer: Mailer;

  constructor(repo: UserRepository, mailer: Mailer) {
    this.repo = repo;
    this.mailer = mailer;
  }

  async createUser(data: CreateUserData): Promise<User> {
    const user = new User(data);

    await this.repo.save(user);
    await this.mailer.sendWelcome(user.email);

    return user;
  }
}
```

### Encapsulate complex conditionals

⚠️ **Exception**: If the condition is short and obvious, extracting it into a function adds a useless indirection. Only extract when the condition genuinely needs a name to be understood.

```typescript
// BAD — opaque condition
if (employee.age > 65 && employee.yearsOfService > 20 && !employee.isContractor) {
  grantEarlyRetirement(employee);
}

// GOOD — the condition has a meaningful name
if (isEligibleForEarlyRetirement(employee)) {
  grantEarlyRetirement(employee);
}
function isEligibleForEarlyRetirement(e: Employee): boolean {
  return e.age > 65 && e.yearsOfService > 20 && !e.isContractor;
}
```

### Avoid negative conditionals

```typescript
// BAD
if (!isNotAuthenticated(user)) { }

// GOOD
if (isAuthenticated(user)) { }
```

---

## 5. OBJECTS AND DATA STRUCTURES

### Abstraction — hide data, expose behavior

```typescript
// BAD — dangerous setter, no abstraction
class BankAccount {
  private _balance = 0;
  getBalance(): number { return this._balance; }
  setBalance(amount: number): void { this._balance = amount; } // anyone can set -1000!
}

// GOOD — domain behavior abstraction
class BankAccount {
  private balance = 0;

  deposit(amount: number): void {
    if (amount <= 0) throw new Error('Amount must be positive');
    this.balance += amount;
  }

  withdraw(amount: number): void {
    if (amount > this.balance) throw new InsufficientFundsError();
    this.balance -= amount;
  }

  getBalance(): number { return this.balance; }
}
```

### Law of Demeter — "Only talk to your immediate friends"

Do not chain method calls across multiple object layers.

```typescript
// BAD — "train wreck"
const streetName = order.getCustomer().getAddress().getStreet().getName();

// GOOD — each object delegates
class Order {
  getCustomerStreetName(): string { return this.customer.getStreetName(); }
}
class Customer {
  getStreetName(): string { return this.address.getStreetName(); }
}
```

### Feature Envy — move the method to where the data lives

```typescript
// BAD — Order uses Customer's data more than its own
class Order {
  calculateTotal(customer: Customer): number {
    return customer.getDiscount() * customer.getLoyaltyMultiplier() * this.subtotal;
  }
}

// GOOD — move to Customer or create a dedicated PricingService
class PricingService {
  calculateTotal(order: Order, customer: Customer): number { /* ... */ }
}
```

---

## 6. ERROR HANDLING

### Exceptions over return codes

```typescript
// BAD — return code forces immediate checking
function deleteUser(id: string): ErrorCode { }
if (deleteUser("123") !== ErrorCode.OK) { logger.log("failed"); }

// GOOD — nominal path is clean, errors are separate
function deleteUser(id: string): void {
  if (!UserRepository.exists(id)) throw new UserNotFoundError(id);
  UserRepository.delete(id);
}
try {
  deleteUser("123");
} catch (error) {
  logger.error("Failed to delete user", error);
}
```

### Domain-specific error hierarchy

```typescript
class AppError extends Error {
  constructor(message: string, public readonly code: string, options?: ErrorOptions) {
    super(message, options);
    this.name = this.constructor.name;
  }
}

class ValidationError extends AppError {
  constructor(message: string, public readonly field?: string) {
    super(message, 'VALIDATION_ERROR');
  }
}

class NotFoundError extends AppError {
  constructor(resource: string, id: string) {
    super(`${resource} with id "${id}" not found`, 'NOT_FOUND');
  }
}
```

### Never return null, never pass null

```typescript
// BAD
function findEmployee(name: string): Employee | null { }
const employee = findEmployee("Alice");
if (employee !== null) { /* forget this check → crash */ }

// GOOD — Option 1: throw
function findEmployee(name: string): Employee {
  const employee = this.employees.get(name);
  if (!employee) throw new NotFoundError('Employee', name);
  return employee;
}

// GOOD — Option 2: empty array instead of null
function getEmployees(): Employee[] {
  return this.employees.size === 0 ? [] : Array.from(this.employees.values());
}
```

---

## 7. SOLID PRINCIPLES

### S — Single Responsibility Principle

"A class should have only one reason to change."

```typescript
// BAD — Employee changes for HR, IT, and DBA reasons
class Employee {
  calculatePay(): number { }   // HR rule
  generateReport(): string { } // IT rule
  save(): void { }             // DB rule
}

// GOOD
class Employee { name: string; salary: number; }
class PayCalculator { calculate(e: Employee): number { } }
class EmployeeReporter { generate(e: Employee): string { } }
class EmployeeRepository { save(e: Employee): void { } }
```

### O — Open/Closed Principle

"Open for extension, closed for modification."

⚠️ **Apply when**: new types/behaviors are regularly added. Over-engineering a system for 2 stable cases is worse than a simple switch.

```typescript
// BAD — modify AreaCalculator for every new shape
class AreaCalculator {
  calculate(shapes: Shape[]): number {
    return shapes.reduce((total, shape) => {
      if (shape.type === 'circle') return total + Math.PI * (shape as Circle).radius ** 2;
      if (shape.type === 'rectangle') return total + (shape as Rectangle).width * (shape as Rectangle).height;
      return total;
    }, 0);
  }
}

// GOOD — extension without modification
interface Shape { area(): number; }
class Circle implements Shape { area(): number { return Math.PI * this.radius ** 2; } }
class Rectangle implements Shape { area(): number { return this.width * this.height; } }
class Triangle implements Shape { area(): number { return 0.5 * this.base * this.height; } }

class AreaCalculator {
  calculate(shapes: Shape[]): number {
    return shapes.reduce((total, s) => total + s.area(), 0);
  }
}
```

### L — Liskov Substitution Principle

"Subtypes must be substitutable for their base type without altering correct behavior."

```typescript
// BAD — classic Rectangle/Square violation
class Square extends Rectangle {
  setWidth(w: number): void { this.width = w; this.height = w; } // surprise modification!
}
// rect.setWidth(5); rect.setHeight(4); → area() returns 16 instead of 20

// GOOD — don't force inheritance when it violates LSP
interface Shape { area(): number; }
class Rectangle implements Shape { area(): number { return this.width * this.height; } }
class Square implements Shape { area(): number { return this.side ** 2; } }
```

### I — Interface Segregation Principle

"Clients should not be forced to depend on interfaces they don't use."

```typescript
// BAD — Robot forced to implement eat() and sleep()
interface Worker { work(): void; eat(): void; sleep(): void; }
class Robot implements Worker {
  work(): void { }
  eat(): void { throw new Error('Robots do not eat!'); }
  sleep(): void { throw new Error('Robots do not sleep!'); }
}

// GOOD — segregated interfaces
interface Workable { work(): void; }
interface Feedable { eat(): void; }
interface Restable { sleep(): void; }

class HumanWorker implements Workable, Feedable, Restable { /* all */ }
class Robot implements Workable { work(): void { } }
```

### D — Dependency Inversion Principle

"Depend on abstractions, not concrete implementations."

⚠️ **Apply when**: you need testability or expect to swap implementations. If a dependency has exactly one implementation and will never be mocked, an interface may be unnecessary complexity.

```typescript
// BAD — hard coupling on concrete EmailNotifier
class UserService {
  private notifier = new EmailNotifier(); // impossible to unit test!
  registerUser(email: string): void {
    this.notifier.send(email, 'Welcome!');
  }
}

// GOOD — dependency injection on an abstraction
interface Notifier {
  send(recipient: string, message: string): Promise<void>;
}

class UserService {
  constructor(private readonly notifier: Notifier) {}
  async registerUser(email: string): Promise<void> {
    await this.notifier.send(email, 'Welcome!');
  }
}

// Trivial test
const mock: Notifier = { send: vi.fn() };
const service = new UserService(mock);
```

---

## 8. CODE SMELLS

| Smell | Symptom | Fix |
|-------|---------|-----|
| **Rigidity** | One change triggers a cascade | Apply DIP, inject dependencies |
| **Fragility** | Changing A breaks B without apparent relation | Eliminate global state, pass context explicitly |
| **Immobility** | Code cannot be reused elsewhere | Decouple from frameworks/DB, extract as library |
| **Viscosity** | The wrong solution is easier than the right one | Improve tools and structure to make good choices easy |
| **Needless Complexity** | Infrastructure for hypothetical requirements (YAGNI) | Delete — only keep what is needed now |
| **Needless Repetition** | Same logic in multiple places | Extract to shared function, class or module |
| **Opacity** | Code is hard to understand | Rename, extract functions, clarify intent |
| **Feature Envy** | A method uses another class's data more than its own | Move the method or create a dedicated service |
| **Long Parameter List** | 4+ parameters | Introduce a parameter object |
| **Large Class** | Class doing too many things | Apply SRP, extract classes |
| **Long Method** | Method over 20 lines (without being naturally readable) | Extract sub-functions |
| **Comments as Deodorant** | Comments explaining bad code | Rewrite the code to be self-documenting |

---

## 9. REVIEW CHECKLIST

Use this list during any code review. **Skip any item where applying it would reduce clarity.**

### Naming
- [ ] Each name reveals its intent without ambiguity
- [ ] No magic numbers (named constants)
- [ ] Consistent verb choice (`findById` everywhere, not `get`/`fetch`/`retrieve` mixed)
- [ ] Classes = nouns, methods = verbs
- [ ] No opaque abbreviations (common ones like `id`, `url`, `ctx` are fine)

### Functions
- [ ] Each function does one thing
- [ ] No function over 20 lines (unless naturally readable as a single block)
- [ ] Max 3 arguments (otherwise parameter object)
- [ ] No flag arguments (boolean)
- [ ] No hidden side effects
- [ ] CQS respected (command OR query, not both)
- [ ] No duplication (DRY)

### Comments
- [ ] No redundant comments (that repeat the code)
- [ ] No commented-out code
- [ ] No change logs in comments

### Structure
- [ ] Classes with a single reason to change (SRP)
- [ ] Extension without modification where types grow (OCP)
- [ ] Inheritance doesn't violate LSP
- [ ] Specific interfaces, not "fat" ones (ISP)
- [ ] Dependencies injected, not instantiated (DIP) — where testability matters

### Error handling
- [ ] Semantic exceptions, not return codes
- [ ] Never return null / pass null
- [ ] Domain-specific error hierarchy

### Smells
- [ ] No repeated switch (→ polymorphism, if types grow)
- [ ] No Feature Envy (method using another class more than its own)
- [ ] No train wreck (`a.getB().getC().getD()`)
- [ ] No premature complexity (YAGNI)
