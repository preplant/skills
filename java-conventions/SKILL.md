---

name: java-conventions
description: >
  Apply this skill when creating, modifying, refactoring, or reviewing Java
  source code. When +agents/java-code-style.md and
  +agents/java-design-patterns.md are available in the current repository,
  treat those files as authoritative and prefer their complete rules over
  this skill. Otherwise, enforce this skill strictly. Do not apply this skill
  to explanation-only or debugging analysis that does not generate or change
  Java source code.
metadata:
  short-description: Strict Java code style, naming, and design conventions.

---

# Java Conventions

This skill defines mandatory Java conventions when the repository's full
convention guides are unavailable.

## Authority

Before writing or changing Java, check whether these repository files exist:

```text
+agents/java-code-style.md
+agents/java-design-patterns.md
```

If available, read and follow them. They are authoritative and more complete
than this skill.

When either guide conflicts with, strengthens, narrows, or adds to a rule in
this skill, the repository guide wins.

Do not use this lighter skill to weaken, reinterpret, or omit a rule from an
available repository guide.

If the guides are unavailable, apply every rule in this skill as a hard
constraint. These are conventions, not suggestions.

Technical validity, generic Java idioms, personal preference, convenience,
brevity, or perceived readability are not reasons to violate them.

## Enforcement Scope

Enforce conventions forward.

Every declaration or code section introduced or intentionally changed by the
current task must comply.

Do not normalize unrelated legacy code merely because its file was touched.

Do not copy a nearby legacy inconsistency into new code.

When local code is inconsistent, infer the intended convention using this
priority:

1. Dominant or clearly intentional family in the same class.
2. Sibling classes in the same package.
3. Surrounding module.
4. These conventions.

New code should look native to the intended surrounding family.

# Code Style

## Final Locals

Use `final` for local variables wherever reassignment is not intentional.

```java
final Player player = event.getPlayer();
final UUID uuid = player.getUniqueId();
```

A non-final local communicates intentional reassignment.

## Getter Extraction

When the same getter is used more than once within a method, normally extract
its value into a local.

Prefer:

```java
final SourceArena.Kits kits = arena.getKits();

if (kits.hasStandard(kit)) {
    return;
}

kits.addStandard(kit);
```

over repeatedly calling:

```java
arena.getKits()
```

This applies to ordinary, fluent, and Lombok getters.

Static singleton-style getters such as:

```java
ServerNodeService.get()
```

should normally remain inline unless repeated excessively.

Do not extract a getter when separate reads are semantically necessary, such
as reading a value before and after mutation.

## Variable Grouping

Group variables by semantic relationship.

```java
final SourceArena arena = arguments.arena();
final SourceArena.Kits kits = arena.getKits();

final StandardKit kit = arguments.kit();
final String label = kit.getLabel();
```

Definition dependencies and correctness take precedence.

Keep declarations reasonably close to their use.

Do not combine unrelated declarations merely because they occur near each
other.

## Variable Spacing

Separate variable sections from executable logic with an empty line.

```java
final Profile profile = profiles.get(player);

profile.refresh();
```

If a declaration follows executable code, normally separate it above:

```java
player.sendMessage("loading");

final Profile profile = profiles.get(player);
```

If executable code follows a declaration or reassignment, separate it below:

```java
String message = "first";

player.sendMessage(message);

message = "second";

player.sendActionBar(message);
```

Related `final` declarations may remain together.

Separate non-final declarations from groups of final declarations.

## Guards

Prefer early `return` and `continue` over unnecessary nesting.

```java
if (!player.isOnline()) {
    return;
}

player.sendMessage("hello");
```

For loops:

```java
for (final Player player : players) {
    if (!player.isOnline()) {
        continue;
    }

    refresh(player);
}
```

If a method or loop contains guards, invert the final condition when necessary
so the main functional path remains outside the final conditional.

Prefer:

```java
if (!click.isRightClick()) {
    return;
}

open(player);
```

over:

```java
if (click.isRightClick()) {
    open(player);
}
```

when the conditional merely guards the final main path.

## Return And Continue Spacing

When executable code immediately precedes a `return` or `continue`, insert a
blank line.

```java
player.closeInventory();

return;
```

Do not insert one when the control statement immediately follows its guard:

```java
if (player == null) {
    return;
}
```

Final value returns follow the same rule:

```java
final boolean valid = player.isOnline();

return valid;
```

but trivial methods remain compact:

```java
public boolean enabled() {
    return true;
}
```

## Multiline Chains

Treat a multiline method chain as its own statement block.

When surrounded by other statements, separate it above and below:

```java
final String old = arena.getLabel();

MessagesConfig.get().arena().labelUpdated()
    .replace("{old}", old)
    .replace("{new}", label)
    .send(name);

arena.setLabel(label);
```

# Variable And Field Naming

## Locals

Prefer one-word local names.

```java
final Profile profile = session.profile();
final SourceArena arena = group.arena();
final StandardKit kit = request.kit();
```

Use two words only when one word cannot identify the value without ambiguity.

```java
final String kitLabel = kit.getLabel();
final String arenaLabel = arena.getLabel();
```

Two words are the normal hard maximum.

Three-word locals require genuine unavoidable domain ambiguity.

Do not add words merely for descriptive enthusiasm or to repeat surrounding
context.

## Naming Shape

Related variables must have a consistent word-count shape.

Good:

```java
final String oldLabel = arena.getLabel();
final String newLabel = input.label();
```

Bad:

```java
final String old = arena.getLabel();
final String newLabel = input.label();
```

Do not introduce a lone naming-shape deviation into an otherwise coherent
group.

Parameters form their own naming group and may use a different internally
consistent shape from locals.

## Fields

Fields follow the same concise naming philosophy, although two-word names are
more common because fields require longer-lived context.

Related fields must preserve the same vocabulary and naming shape.

Do not alternate terminology such as `server`, `node`, and `backend` for the
same concept.

# Method Naming

Method naming is a release-blocking convention.

Before introducing or intentionally renaming any explicit method, determine
the class's intended method-name parity.

## Mandatory Method Parity

Every class has one intended camel-case word-count parity:

* odd; or
* even.

Every explicitly declared method introduced or intentionally renamed by the
task must use that parity.

An even family may contain:

```text
findPlayer          2
saveProfile         2
removeEntry         2
publishServerState  4
```

It may not introduce:

```text
publishStateNow     3
```

An odd family may contain:

```text
create                   1
delete                   1
publishCurrentHeartbeat  3
```

It may not introduce a two- or four-word method.

Determine parity using:

1. Same-class intentional family.
2. Sibling classes.
3. Surrounding module.
4. Repository defaults.

For a new class, select the parity before naming its methods and preserve it.

Do not choose names independently and discover parity violations afterward.

Mixed legacy methods do not make parity optional and do not require unrelated
renaming.

## Method-Parity Exemptions

Only these are normally excluded:

* generated methods absent from source, including Lombok-generated methods;
* overridden methods whose names are imposed by an inherited API;
* annotated framework-controlled methods whose role/name is imposed by the
  framework.

For example:

```java
@Register
private void onRegister() {
}

@Unregister
private void onUnregister() {
}
```

The exemption applies only to the controlled method.

Unannotated lifecycle-looking methods are not automatically exempt.

Do not create wrappers, delegates, constructor-time setter calls, or other
structural hacks merely to avoid parity requirements for an imposed override.

## Accessor Style

Every class has one intended style for explicitly declared accessors:

JavaBean:

```java
getProfile()
setProfile(...)
isEnabled()
```

or fluent:

```java
profile()
profile(...)
enabled()
```

Infer the intended style using the same convention priority.

Do not introduce further mixing.

Generated Lombok accessors are excluded from this source-level visual rule.

Explicit accessors must satisfy both accessor style and method parity.

## Vocabulary

Equivalent operations should use equivalent verbs.

Prefer a coherent family:

```java
getByName(...)
getByUuid(...)
getByRole(...)
```

Do not arbitrarily mix:

```java
getByName(...)
findByUuid(...)
lookupRole(...)
```

`get`, `find`, `resolve`, `lookup`, `fetch`, and `load` are not interchangeable
merely because each can describe retrieval.

Preserve established repository vocabulary.

Within the required parity and vocabulary, use the shortest name that
accurately communicates the operation.

Do not repeat class context unnecessarily in method names.

# Design And Structure

## Plugin Modules

Application/plugin projects normally organize domain modules using established
roles such as:

```text
command
listener
service
core
data
meta
```

Additional semantic packages such as these are valid when they represent a
real domain role:

```text
event
task
provider
dialog
menu
query
annotation
resolver
router
schema
source
tokenizer
```

Do not invent vague categories such as `engine`, `processor`, `system`, or
`manager` when an established package already describes the responsibility.

### `core`

Use for important domain concepts, abstractions, central state objects,
constants, and conceptual foundations.

### `data`

Use for ordinary data structures, records, state holders, requests,
responses, and transport structures.

Do not put every object with fields into `data`.

### `meta`

Use primarily for top-level enums.

Top-level enums should ordinarily not live in `data`.

### `service`

Use for long-lived module behavior and services.

Preserve the sibling naming family, commonly three words:

```text
ChatFilterService
ChatDataService
ChatControlService
```

### `listener`

Use for listeners.

Preserve the sibling naming family, commonly three words:

```text
PlayerChatListener
PermissionRefreshListener
RootCommandListener
```

### `command`

Commands follow the repository's established command-tree conventions rather
than ordinary module naming assumptions.

When a dedicated command guide exists, it is authoritative.

## Domain-Specific Packages

A specific package is valid when a concept genuinely does not fit an
established role.

A one-class package is acceptable when the concept itself warrants the
category.

Do not create packages solely to classify implementation details more finely.

## Plugin Class Naming

Ordinary top-level plugin-module classes use at least two words.

Annotations are the normal one-word exception.

Class names must match the exact word-count and vocabulary family of their
siblings.

Typical defaults include:

```text
data      -> 2 words
core      -> 2 words
meta      -> 2 words
listener  -> 3 words
service   -> 3 words
```

These defaults do not override a clearly established local family.

Do not independently shorten or lengthen a new class because another name
sounds better in isolation.

## Naming Symmetry

Prefer deliberate naming families:

```text
ChatFilterService
ChatDataService
ChatControlService
```

and:

```text
ChatMessage
ChatControls
ChatReply
```

A systematic alternative family is valid.

A lone misfit is not.

Existing isolated deviations are not templates.

## API And Library Projects

Do not force plugin-module structure onto API/library projects.

Capability-oriented packages are valid:

```text
listener
location
operation
reflect
sound
time
tuple
```

Deeper abstraction hierarchies such as:

```text
platform/config
platform/identity
platform/logger
platform/messaging
platform/scheduler

platform/config/impl
platform/identity/impl
```

are appropriate when the abstraction hierarchy itself defines the API.

## Top-Level Plugin Infrastructure

A plugin may contain project-level API/infrastructure packages outside its
domain modules when the concept genuinely belongs there.

Do not use top-level placement merely to escape module conventions.

Once an area legitimately belongs at project level, organize it like an API
rather than forcing artificial `core`, `data`, or `service` layers onto it.

# Java Patterns

## Nullable String Defaults

When selecting between a nullable `String` and fallback `String`, prefer:

```java
Objects.toString(value, fallback)
```

For example:

```java
return Objects.toString(System.getenv(key), "");
```

Do not use an equivalent ternary:

```java
value == null ? fallback : value
```

when the operation is simply nullable-string fallback selection.

Do not introduce a local solely to perform this selection.

This does not apply when the branches perform different computation or the
selected value is not a `String`.

## Lombok

Use Lombok when it cleanly replaces mechanical boilerplate.

Prefer narrow annotations such as:

```java
@RequiredArgsConstructor
@AllArgsConstructor
@Getter
@Setter
```

over manually writing equivalent trivial constructors or accessors.

Do not use broad annotations such as `@Data` when narrower annotations express
the intended semantics.

Explicit constructors/accessors remain appropriate when they contain real
behavior such as:

* validation;
* transformation;
* side effects;
* unusual visibility;
* non-trivial semantics.

## Singleton Services

Shared long-lived services, registries, managers, configurations, and similar
objects may expose a fluent singleton `get()`.

Use:

```java
@Getter
@Accessors(fluent = true)
private static final ServerNodeService get = new ServerNodeService();
```

Apply Lombok annotations directly to the field.

When constants coexist with this pattern, place the singleton field before
the constants.

Do not make every class a singleton.

Short-lived, event-driven, independently registered, or command classes
generally should not expose a global instance.

If behavior does not depend on an instance, prefer an appropriate static
operation instead of inventing a singleton.

# Validation

Before finishing Java-generating or Java-modifying work, review every
declaration and code section introduced or intentionally changed by the task.

Verify:

* repository convention files were checked first when available;
* repeated getters were extracted where appropriate;
* singleton getters were not unnecessarily extracted;
* locals use `final` wherever possible;
* variables are grouped by purpose;
* variable/executable sections use required spacing;
* guards use early `return` or `continue`;
* final main logic is not unnecessarily nested;
* `return` and `continue` spacing is correct;
* multiline chains are visually separated;
* locals use the shortest unambiguous names;
* related variable names have consistent shape;
* related fields preserve vocabulary and naming shape;
* every introduced or renamed explicit method satisfies class parity;
* no method introduces an odd/even parity break;
* every introduced or changed accessor matches the intended accessor style;
* generated, overridden, and framework-controlled methods are exempt only
  where actually applicable;
* equivalent methods preserve established verbs;
* new classes match sibling word-count and vocabulary families;
* plugin packages follow established semantic roles;
* API/library structure was not forced into plugin-module conventions;
* Lombok replaces appropriate mechanical boilerplate;
* nullable String defaults use `Objects.toString` where applicable;
* singleton patterns are used only for genuinely shared long-lived objects;
* unrelated legacy inconsistencies were left untouched.

If a current-work violation remains, the work is unfinished.

Do not waive a convention because the alternative compiles, is idiomatic Java,
looks reasonable in isolation, or would require less effort to correct.