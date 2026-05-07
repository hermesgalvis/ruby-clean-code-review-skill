# Clean Code Principles — Ruby

Reference: https://github.com/uohzxela/clean-code-ruby

Read this file when executing the analysis step. Apply each principle
only to changed lines in the diff, never to untouched code.

## Table of Contents
- [Variables (V1–V6)](#variables)
- [Methods (M1–M9)](#methods)
- [Classes / SOLID (C1–C5)](#classes--solid)
- [Naming Conventions (N1–N4)](#naming-conventions)
- [Testing (T1–T3)](#testing)
- [Error Handling (E1–E2)](#error-handling)

---

## Variables

**V1 — Meaningful and pronounceable names**
Flag: single-letter variables (except loop index `i`, `j`, `k`), cryptic
abbreviations, names that require mental mapping to understand.
```ruby
# ❌
d = Date.today
yyyymmdstr = Time.now.strftime('%Y/%m/%d')

# ✅
current_date = Date.today
current_date_formatted = Time.now.strftime('%Y/%m/%d')
```

**V2 — Same vocabulary for the same concept**
Flag: using different names for the same concept in the same file/class
(`get_user_info`, `fetch_client_data`, `retrieve_person_record` for the same
idea). Pick one verb and use it consistently.

**V3 — Searchable names (no magic numbers/strings)**
Flag: literal numbers or strings with non-obvious meaning. Exceptions:
`0`, `1`, `true`, `false`, `nil`, empty string, and small loop indices.
```ruby
# ❌
(1..525_960).each { |i| work }

# ✅
MINUTES_IN_A_YEAR = 525_960
(1..MINUTES_IN_A_YEAR).each { |i| work }
```

**V4 — Explanatory variables**
Flag: complex boolean/conditional expressions used directly in `if` without
being assigned to a named variable first.
```ruby
# ❌
if city_zip.match?(/\d{5}/) && age > 18 && subscription == :active

# ✅
valid_zip = city_zip.match?(/\d{5}/)
is_adult = age > 18
is_subscribed = subscription == :active
if valid_zip && is_adult && is_subscribed
```

**V5 — No unnecessary context**
Flag: repeating the class/module name inside attribute or method names.
```ruby
# ❌  (inside class Car)
attr_accessor :car_make, :car_model, :car_color

# ✅
attr_accessor :make, :model, :color
```

**V6 — Default arguments instead of short-circuit**
Flag: `arg = arg || default` pattern when a default keyword argument suffices.
```ruby
# ❌
def create_account(name, type = nil)
  type = type || :individual

# ✅
def create_account(name, type: :individual)
```

---

## Methods

**M1 — Few arguments (≤ 2 ideally, ≤ 3 max)**
Flag: methods with 4+ positional arguments. Suggest keyword arguments or a
parameter object (Struct / dry-struct).
```ruby
# ❌
def create_menu(title, body, button_text, cancellable)

# ✅
def create_menu(title:, body:, button_text:, cancellable:)
```

**M2 — Do one thing**
Flag: methods containing multiple unrelated concerns; method names that include
"and", "or", or "also" to describe what they do.

**M3 — One level of abstraction**
Flag: methods that mix high-level orchestration with low-level detail in the
same body.
```ruby
# ❌
def process_order(order)
  data = JSON.parse(order)
  total = data['items'].sum { |i| i['price'] * i['quantity'] }
  tax = total * 0.19
  total + tax
end

# ✅
def process_order(raw_order)
  order = parse_order(raw_order)
  calculate_total_with_tax(order)
end
```

**M4 — No duplicate code (DRY)**
Flag: identical or near-identical logic blocks appearing more than once in the
diff. Extract to a shared private method or concern.

**M5 — No flag arguments**
Flag: boolean parameters that select a code path inside the method.
Each path should be its own named method.
```ruby
# ❌
def create_file(name, temp)
  temp ? create_temp_file(name) : create_permanent_file(name)
end

# ✅
def create_temp_file(name) = ...
def create_permanent_file(name) = ...
```

**M6 — Avoid side effects**
Flag: methods that mutate external state (global/class variables, modifying
caller's arguments) without making it clear from the name (e.g., no `!`
suffix, no verb like `save`, `update`, `reset`).

**M7 — Encapsulate conditionals**
Flag: multi-term `if` conditions that could be a named predicate method.
```ruby
# ❌
if subscription.plan == :freemium && subscription.days_since_trial > 30

# ✅
if subscription.trial_expired?
```

**M8 — Avoid negative conditionals**
Flag ONLY double-negation patterns: `if !condition?` or `unless` combined
with a negated predicate (e.g., `unless !valid?`). Do NOT flag plain
`unless condition` — that is idiomatic Ruby.
```ruby
# ❌ — double negation is confusing
if !is_dom_node_not_present?(node)
unless !contract.active?

# ✅
if dom_node_present?(node)
unless contract.inactive?
```

**M9 — No dead code**
Flag: commented-out code blocks, or methods clearly never called within the
scope visible in the diff (e.g., `private` method defined but not referenced).

---

## Classes / SOLID

**C1 — Single Responsibility Principle (SRP)**
Flag: classes that handle unrelated concerns in one body (e.g., persistence +
email delivery + PDF rendering in a single class).

**C2 — Open/Closed Principle**
Flag: `case`/`if` chains that branch on a type attribute — suggest moving
behavior into subclasses or strategy objects.
```ruby
# ❌
def calculate(contract)
  case contract.type
  when :employee then ...
  when :contractor then ...
  end
end

# ✅  — each type implements #calculate itself (polymorphism)
contract.calculate
```

**C3 — Liskov Substitution Principle**
Flag: overriding a parent method in a way that narrows its contract
(raises an exception the parent contract doesn't promise, returns a different
type, or ignores required arguments).

**C4 — Interface Segregation**
Flag: modules/concerns that force every including class to implement unrelated
methods (`raise NotImplementedError` stubs scattered across the same concern).

**C5 — Dependency Inversion**
Flag: high-level classes hard-instantiating their low-level collaborators with
`Foo.new` inside `initialize` instead of accepting them via injection.
```ruby
# ❌
class OrderProcessor
  def initialize
    @mailer = OrderMailer.new
  end
end

# ✅
class OrderProcessor
  def initialize(mailer: OrderMailer.new)
    @mailer = mailer
  end
end
```

---

## Naming Conventions

**N1 — Domain vocabulary**
If project conventions were loaded (CLAUDE.md / rules files), flag names that
use abbreviations explicitly banned by the project (e.g., `cc` instead of
`cost_center`, `ss` for `social_security`, `ep` for `electronic_payroll`).
Also flag names that don't match the domain model vocabulary.

**N2 — Rails naming conventions**
Flag: class names not PascalCase, methods/variables not snake_case, constants
not SCREAMING_SNAKE_CASE, files not matching their class name.

**N3 — Predicate methods end in `?`**
Flag: methods that return a boolean but don't end in `?`.
```ruby
# ❌
def active  # returns true/false

# ✅
def active?
```

**N4 — Bang methods must raise**
Flag: methods ending in `!` that don't raise on failure (misleads readers into
thinking the method will raise if something goes wrong).

---

## Testing

Only apply these when test files (files under `test/` or `spec/`) appear in
the diff.

**T1 — One behavior per test block**
Flag: test blocks that assert multiple independent behaviors. Each case should
be its own `test` or `it` block.

**T2 — Descriptive test names**
Flag: vague names like `test 'works'`, `test 'valid'`, `test 'invalid'`.
Good pattern: `'<action> <result> for <condition>'`.

**T3 — No logic inside tests**
Flag: `if`/`unless`/loops inside a test block — split into separate tests.

---

## Error Handling

**E1 — Don't swallow exceptions**
Flag: empty `rescue` blocks or `rescue => _e` / `rescue => e` where `e` is
never logged, re-raised, or returned.

**E2 — Rescue specific exceptions**
Flag: `rescue Exception` — catch `StandardError` or a more specific subclass.
```ruby
# ❌
rescue Exception => e

# ✅
rescue ActiveRecord::RecordNotFound => e
rescue StandardError => e
```
