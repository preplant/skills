---

name: mcp-steroid
description: >
  Apply this skill when a task materially benefits from IntelliJ semantics or
  IDE capabilities: nontrivial Java/Kotlin refactoring, authoritative usage
  analysis and load-bearing audits, hierarchy/override analysis, inspections, debugger
  work, generated/light PSI, or IDE project/module models. Do not apply it to
  ordinary source reading, filename/literal search, simple navigation, or
  exploratory find-usages that normal Kilo filesystem/indexed text tools can
  answer reliably. Optimize MCP Steroid calls for low latency and minimal
  generated Kotlin without sacrificing required semantic correctness.
metadata:
  short-description: Efficient, semantic IntelliJ automation with MCP Steroid.

---

# MCP Steroid

Use MCP Steroid when IntelliJ semantics or IDE operations materially improve
correctness or capability, not as the default for Java/Kotlin work.

The goal is to obtain the strongest correctness guarantees necessary for the
task with the cheapest appropriate IntelliJ operation.

Prefer:

```
cheap discovery -> narrow candidates -> semantic verification -> mutation
```

over:

```
full-project semantic search -> inspect everything -> mutate
```

Do not use an expensive PSI operation merely because MCP Steroid makes it
possible.

MCP Steroid has fixed startup/tool overhead and generated-Kotlin latency. A
text search taking seconds is the correct choice when it reliably answers the
actual question; semantic IDE work taking longer is worthwhile when it prevents
an unsafe refactor or missed semantic reference.

## 1. Session preflight

Before the first IDE-backed operation in a session:

1. Reuse a current-conversation `project_name` when the project has not changed;
   otherwise call `steroid_list_projects`.
2. Match the current repository against the returned project paths.
3. Reuse the exact opaque `project_name` returned for that entry.
4. Do not repeatedly call `steroid_list_projects` unless project state may
   actually have changed.

Never derive, normalize, or guess `project_name` from a repository, directory,
workspace, or human-readable project name. Only a value returned by
`steroid_list_projects` is a valid routing key.

If the expected project is not open, do not silently operate against another
project.

If readiness is uncertain because the IDE is indexing, importing Gradle/Maven,
or otherwise busy, inspect IDE readiness before launching an expensive query.

Avoid repeatedly probing state that has already been established.

Reuse working imports/APIs, successful equivalent scripts, resolved
symbols/paths, and fetched resources from the current conversation. Do not
re-probe or refetch them without evidence that IDE/project state changed. Do
not call MCP Steroid when the requested answer is already reliably present in
conversation context.

## 2. Load MCP Steroid knowledge lazily

MCP Steroid exposes task-specific guides and recipes through
`steroid_fetch_resource`.

Fetch the relevant resource only when:

* the IntelliJ API is unfamiliar;
* threading/read/write-action requirements are uncertain;
* performing refactoring, debugger, inspection, test-runner, VCS, or other
  non-trivial IDE operations without an already-established working pattern;
* a previous attempt failed or was unexpectedly expensive.

Do not fetch every MCP Steroid resource at session start.

Do not repeatedly fetch the same resource during one task unless necessary.

Prefer an upstream MCP Steroid recipe as API guidance, but this skill's
performance and scoping rules remain authoritative. Do not copy a recipe's
project-wide search strategy when a narrower equivalent preserves correctness.

## 3. Choose the cheapest correct search

Classify the question before choosing a tool.

### Textual questions

Examples:

* find an error message;
* find a unique string literal;
* locate a filename;
* locate configuration text;
* search comments;
* inspect files outside the indexed IDE project;
* locate a known class/file/package;
* find literal method/class/field occurrences;
* read source or inspect nearby implementation;
* explore where a uniquely named symbol appears;
* make a straightforward edit with unambiguous symbol identity.

Use normal Kilo filesystem, file-index, and text-search tools first.

Do not invoke MCP Steroid merely because PSI could also answer the question.
If ordinary search reliably answers the requested question, stop there.

### Symbol questions

Examples:

* what calls this method?
* what overrides this method?
* what implements this interface?
* where is this field semantically referenced?
* which overload is called here?
* is this production method actually unused?
* what will this rename/signature change affect?

Use MCP Steroid when PSI/indexes materially add needed semantic certainty or
IDE capability. Examples include overloaded or same-named symbols,
cross-language references, generated/light PSI, hierarchy and polymorphism,
project models, inspections, debugger work, and structural refactoring.

Text search may be used for candidate discovery, but must not be treated as
the authoritative answer when Java/Kotlin semantics matter.

The wording "find usages" does not itself select MCP Steroid. For exploratory
questions such as "Where is `run` used?", use normal indexed/text search when
literal occurrences are sufficient. Use semantic `ReferencesSearch` when the
intent is to find every semantic reference before rename/delete/API change, or
when overloads, same-named symbols, inheritance, Java/Kotlin syntax, generated
PSI, or ambiguous textual matches require resolution.

For ambiguous discovery prompts such as "Find a service class with a public
method that has multiple usages across the project", do not launch a broad PSI
scan to discover both target and answer. Use normal filename/text search to
identify a small set of plausible classes and methods, then use one narrow MCP
query only if public/source-declared status or semantic usage identity must be
verified. Return one valid match unless the user asks for every match.

### Load-bearing questions

If an answer will justify:

* deletion;
* rename;
* signature change;
* moving a symbol;
* changing inheritance;
* changing an interface or abstract contract;
* changing behavior where a missed caller could break production;

require semantic PSI/refactoring analysis before modifying code.

Never conclude that a symbol is unused solely because grep found no matches.

Bound authoritative audits to unrestricted IntelliJ semantic references plus
the hierarchy/override checks relevant to the symbol's contract. Do not
automatically audit reflection, arbitrary strings, configuration, serialization,
or framework registries. Investigate those dynamic mechanisms only when source
or project evidence indicates they are used, or the user explicitly requests a
dynamic-reference audit.

## 4. Performance hierarchy

Escalate through these levels and stop as soon as one reliably answers the
actual task:

1. Existing conversation or prior result
2. Normal Kilo file/index/text search and source reads
3. Narrow MCP Steroid PSI/index query
4. Scoped semantic IntelliJ search
5. Full project-wide semantic search or refactoring machinery

Do not pay a higher level's latency when a lower level is sufficient. Do not
remain at a lower level when load-bearing correctness requires semantic
certainty.

### Generated Kotlin budget

Treat every generated Kotlin character as an end-to-end latency cost even when
the IDE operation itself would execute equally fast. Optimize agent reasoning,
tool arguments, tool-call count, IDE execution, and returned output together.

For every `steroid_execute_code` call, generate the smallest program that
correctly answers the request:

* omit implicit imports and unused imports;
* minimize declarations, intermediate variables, collections, maps, output
  transformation, and formatting boilerplate;
* collect and print only fields the user requested;
* do not build reusable helpers, classes, wrappers, or generalized diagnostics
  inside a one-shot script;
* avoid repeated null/error handling when the established MCP/IDE contract and
  resolved target make it unnecessary;
* do not sort unless deterministic ordering materially helps the task;
* do not extract source lines, columns, snippets, or context unless requested
  or needed to disambiguate the answer;
* for load-bearing audits, emit only identity, usage locations, and relevant
  hierarchy results; omit signatures, source text, PSI class names, candidate
  counts, and other metadata unless needed;
* do not fetch a recipe when the current conversation or this skill already
  provides a verified pattern.

Correctness outranks brevity. Never replace semantic PSI/index operations with
unsafe textual guesses merely to shorten Kotlin.

Script declarations do not persist across executions, and the fixed context
has no reusable custom candidate-search helper. Keep the verified pipeline
compact, but emit its essential scope construction in each independent call.

### Known class/file lookup

Before MCP escalation, locate known filenames/classes with normal Kilo tools.
Inside an already-necessary MCP operation, use `FilenameIndex` for filenames
rather than scanning the VFS.

Use `PsiShortNamesCache` when exact Java class PSI is needed for the semantic
operation.

Do not recursively scan the repository to locate a known indexed class.

Example:

```kotlin
println(smartReadAction { PsiShortNamesCache.getInstance(project)
    .getClassesByName("MyService",projectScope()).single().containingFile.virtualFile.path })
```

Once the desired class is resolved, retain and operate on that PSI object
inside the same read operation where practical.

### Implicit script surface

In IntelliJ IDEA 2026.2.1 (`IU-262.9437.185`), scripts have these verified
context helpers without imports: `project`, `projectScope()`, `allScope()`,
`smartReadAction`, `readAction`, `writeAction`, `findProjectFile`,
`findProjectFiles`, `findProjectPsiFile`, `println`, `printJson`, `printToon`,
and `printCsv`.

The common classes used by the compact patterns below are also implicitly
imported: `PsiClass`, `PsiElement`, `PsiFile`, `PsiMethod`, `PsiModifier`,
`PsiReference`, `PsiDocumentManager`, `PsiManager`, `FilenameIndex`,
`GlobalSearchScope`, `PsiSearchHelper`, `PsiShortNamesCache`,
`TextOccurenceProcessor`, `UsageSearchContext`, `ReferencesSearch`, and
`VirtualFile`. Emit no imports for them in `steroid_execute_code` scripts.

If an explicit import is ever required outside that implicit surface, the
verified short-name cache package is
`com.intellij.psi.search.PsiShortNamesCache`.

Do not use the invalid package `com.intellij.psi.util.PsiSubstitutor`. If a
substitutor is genuinely required, its verified PSI package is
`com.intellij.psi.PsiSubstitutor`; omit it entirely when the task does not need
generic substitution.

When an API package or signature is uncertain, verify it with a minimal compile
or runtime probe before placing it in a larger execution. Do not use an
expensive operation as the compilation check.

### Candidate-first reference discovery

Candidate-first semantic verification is useful for exploratory named
Java/Kotlin references. It is authoritative only when every relevant semantic
reference is guaranteed to contain the exact indexed textual name.

When textual-name coverage is established, do not begin with:

```kotlin
ReferencesSearch.search(target, projectScope()).findAll()
```

For authoritative requests where that coverage cannot be established, skip
this prefilter and use the full semantic search described below.

First obtain code-occurrence candidate files using the verified IntelliJ word
index API:

```kotlin
smartReadAction {
    val c=PsiShortNamesCache.getInstance(project).getClassesByName("MyService",projectScope()).single()
    val target=c.methods.single { it.name=="run" }
    val f=java.util.Collections.synchronizedSet(mutableSetOf<VirtualFile>())
    check(PsiSearchHelper.getInstance(project).processElementsWithWord(
        { e,_ -> e.containingFile.virtualFile?.let(f::add);true },
        projectScope(),target.name,UsageSearchContext.IN_CODE,true))
    ReferencesSearch.search(target,GlobalSearchScope.filesScope(project,f)).findAll().forEach { r ->
        val e=r.element
        println("${e.containingFile.virtualFile.path}:${e.containingFile.viewProvider.document.getLineNumber(e.textOffset)+1}")
    }
}
```

`processElementsWithWord` is index-backed but its occurrences are not the
authoritative answer. Collect their files, then use `ReferencesSearch` to
semantically distinguish the exact target from same-named symbols.

Its processor may run concurrently. Never collect callback results in
`linkedSetOf` or another unsynchronized mutable collection; the verified
synchronized set above prevents nondeterministic candidate loss.

Kotlin SAM inference is verified here; do not spell out
`TextOccurenceProcessor { ... }` in this call.

The normal reference-search pipeline is:

```text
resolve target
-> obtain candidate files from indexes
-> construct candidate scope
-> semantically resolve references within candidate scope
-> return verified references
```

Candidate discovery is not the authoritative answer. It determines where
semantic verification needs to run.

For direct textual references known to contain the exact symbol name, the
`IN_CODE` result may be used as the complete candidate set and all candidates
must be semantically resolved.

For arbitrary Java/Kotlin references, exact textual-name coverage cannot be
assumed. Kotlin property syntax, implicit references, generated conventions,
framework behavior, and other language features may resolve to a symbol without
containing its textual name. Broader `UsageSearchContext` masks still cannot
find a word absent from source. For authoritative/load-bearing analysis, use
unrestricted semantic `ReferencesSearch` whenever textual-name completeness
cannot be established.

### Full ReferencesSearch

Project-wide `ReferencesSearch` is not the default for exploratory find-usages,
but it is required for authoritative/load-bearing analysis when a textual
candidate prefilter cannot be proven complete.

Use:

```kotlin
ReferencesSearch.search(target, projectScope())
```

only when at least one of the following is true:

* index-assisted candidate discovery cannot soundly cover the possible
  references;
* the relevant reference form does not necessarily contain an indexable target
  name;
* a framework, language, generated construct, or indirect reference mechanism
  requires project-wide semantic search;
* candidate-scoped semantic verification is unavailable or demonstrably cannot
  preserve correctness.

Do not use project-wide `ReferencesSearch` merely because:

* the user asked for all usages;
* the target is a method, field, or class;
* the upstream find-references recipe demonstrates `ReferencesSearch`;
* project scope is easier to write.

Do use it when authoritative intent combines with any unresolved possibility
of non-textual, implicit, generated, cross-language, polymorphic, or
framework-provided references.

When falling back to project-wide `ReferencesSearch`, state internally why
candidate narrowing cannot preserve correctness before executing it.

If candidate files are available, do not perform both candidate-scoped
verification and a second project-wide `ReferencesSearch` merely for
confirmation.

A project-wide semantic search should be the exceptional expensive path.

## 5. Scope aggressively

Use the smallest scope that preserves correctness.

For project-model queries, resolve an exact module identity before reading its
roots or dependencies. Gradle source sets commonly produce similarly named
`main` and `test` modules; do not use `firstOrNull` with a substring match when
selecting a load-bearing module target. If the exact IDE module name is not
already known, list the small set of matching names first and disambiguate it.

Use each narrower scope below whenever it preserves the required result:

* one module over the whole project;
* production sources over production + tests;
* one package over all packages;
* candidate files over project scope;
* a known class hierarchy over unrelated project symbols;
* changed files over every project file for inspections.

Only use project-wide semantic scope when candidate discovery cannot soundly
preserve completeness. A project-wide question alone does not require a
project-wide semantic search.

Do not sacrifice correctness merely to make a search faster.

## 6. Java and Kotlin semantic operations

Use PSI when language semantics matter.

This includes:

* overload resolution;
* generated/synthetic members visible to IntelliJ;
* Lombok-generated Java members;
* inheritance;
* implementations;
* overrides;
* polymorphic call sites;
* generic dispatch;
* annotations;
* symbol identity despite equal names;
* cross-module usages;
* Javadoc/reference-aware refactoring.

Do not infer these relationships from text matches when IntelliJ can resolve
them directly.

When a request asks for explicitly source-declared methods or members, filter
generated/light PSI during the initial query rather than in a second execution.
In this build, `PsiElement.isPhysical` is verified and may be used, for example
`psiClass.methods.filter { it.isPhysical }`. Keep generated members only when
the request includes IDE-visible or generated API.

Compact class plus explicitly declared public methods:

```kotlin
println(smartReadAction { PsiShortNamesCache.getInstance(project)
    .getClassesByName("MyService",projectScope()).single().let { c ->
        c.name+"\n"+c.methods.filter { it.isPhysical&&it.hasModifierProperty(PsiModifier.PUBLIC) }
            .joinToString("\n") { it.name }
    } })
```

## 7. Hierarchy analysis

Do not emulate hierarchy searches using text.

Use appropriate IntelliJ search APIs/recipes for:

* class inheritors;
* interface implementations;
* overriding methods;
* overridden methods;
* call hierarchy.

For contract changes, inspect both the hierarchy and usages when either can
contain affected code.

A method having zero direct references does not prove an abstract/interface
contract is unused.

## 8. Refactoring policy

Meaningful cross-file or cross-module Java/Kotlin refactors are a primary MCP
Steroid use case. Strongly prefer IntelliJ's semantic/refactoring engine over a
manual sequence of textual replacements; the larger or more load-bearing the
change, the stronger this requirement becomes.

Examples:

* rename class/method/field/package;
* move class;
* change method signature;
* add/remove/reorder parameters;
* safe delete;
* extract;
* inline;
* pull up;
* push down;
* inheritance changes.

Before executing a refactor:

1. resolve the exact PSI target;
2. inspect relevant usages/hierarchy if the impact is non-trivial;
3. fetch a refactoring recipe only if the required API/threading pattern is not
   already established;
4. use IntelliJ's refactoring APIs;
5. inspect the resulting changes.

Do not implement a semantic rename with global string replacement.

Do not manually rewrite every caller when IntelliJ has a safe structured
refactoring for the operation.

Escalate to IntelliJ refactoring machinery when a change requires searching
many callers, distinguishing symbol identity, updating imports or signatures,
preserving overrides/interfaces, moving symbols, or coordinating edits across
files/modules. Candidate discovery may narrow impact analysis, but it must not
replace semantic mutation machinery when missed references could break the
build or runtime behavior.

Refactoring processors must be invoked on the EDT in this environment. Run a
processor's `.run()` with `withContext(Dispatchers.EDT)`, not inside
`writeAction` or `writeIntentReadAction`; the processor manages its own actions
internally. Import `kotlinx.coroutines.Dispatchers`,
`kotlinx.coroutines.withContext`, and `com.intellij.openapi.application.EDT`.

Do not mechanically append a `writeAction` that calls
`PsiDocumentManager.commitAllDocuments()` after a refactoring processor. The
processor saves its changes, and this post-step can itself fail an EDT check
after the refactor has already succeeded. Commit only when the same script must
read the updated PSI, and perform that commit on the EDT before readback.

For temporary refactoring experiments, an inverse refactoring restores semantic
structure, not necessarily the original text. A change-signature reversal may
leave an equivalent lambda where the original used a method reference. Capture
the pre-iteration Git state, inspect the reverse diff, and restore only any
proven residual test edits before claiming byte-exact rollback.

Raw edits remain appropriate for genuinely textual changes where symbol
identity does not matter.

## 9. Read and write actions

Respect IntelliJ threading requirements.

Use the MCP Steroid-provided execution helpers and recipes rather than
inventing synchronization.

For read-only PSI/index operations, use the appropriate smart/read action
pattern.

For modifications, use the appropriate write/refactoring mechanism.

Do not mutate PSI from an arbitrary read action.

Do not bypass IDE APIs by editing IntelliJ configuration/state files when a
typed IntelliJ API exists, unless the task specifically requires a
pre-startup configuration mechanism.

## 10. Minimize `steroid_execute_code` calls

Each call should perform one coherent IDE operation and return only the
requested result. Structured output is not a default requirement.

Prefer:

```text
resolve target
+ gather required metadata
+ perform scoped analysis
+ print compact result
```

in one execution when those operations naturally share the same PSI state.

Use one coherent execution for exact-target resolution, physical/source-member
filtering, indexed candidate discovery, candidate-scoped semantic verification,
and compact output once every API in the script is known to compile.

Avoid:

```text
call 1: find class
call 2: get class methods
call 3: find method
call 4: search usages
call 5: get file paths
call 6: get line numbers
```

when one safe read operation can do all of them.

However, do not construct enormous scripts that perform unrelated tasks merely
to reduce tool-call count.

Optimize total generated Kotlin and total tool overhead, not each call in
isolation. Once APIs are known, keep a straightforward resolve + analyze + emit
flow in one compact execution rather than splitting it into smaller scripts.

Do not place an unfamiliar optimization API into a large combined execution.
Probe its import and signature cheaply first. If that probe fails, diagnose and
correct it; do not immediately replace it with project-wide
`ReferencesSearch`.

## 11. Keep output compact

Do not dump entire PSI trees, documents, source files, or huge reference
objects into model context.

Return exactly the information needed for the task. If asked for class name and
path, do not compute lines, columns, snippets, maps, sorting, or metadata. If
asked for methods and usages, emit only methods and usage locations.

Choose the output requiring the least Kotlin and useful output tokens:

* use `println(value)` for scalars, short lists, and simple newline-delimited
  results;
* use `printToon(records)` for an already-existing uniform record collection
  when structure is useful;
* use `printCsv(headers, rows, dictColumns)` for naturally tabular repeated data
  when it is shorter overall or path deduplication materially reduces output;
* use `printJson(value)` only when JSON structure is actually useful.

Do not construct nested maps/lists solely to feed `printJson`, `printToon`, or
`printCsv`. Do not add counts, grouping, or sorting unless requested or useful
for interpreting a large result.

For ordinary usages, default to `path:line`. The shortest verified one-based
line expression is:

```kotlin
val e=reference.element
println("${e.containingFile.virtualFile.path}:${e.containingFile.viewProvider.document.getLineNumber(e.textOffset)+1}")
```

Do not use `PsiDocumentManager`, line start/end offsets, column calculations,
or source-text extraction unless the request requires them.

Token efficiency is part of query design.

## 12. Avoid redundant source reads

If PSI has already provided:

* qualified symbol identity;
* signature;
* containing file;
* line;
* annotations;
* hierarchy;
* usages;

do not immediately reread every entire source file through filesystem tools.

Read source bodies only when understanding implementation logic is necessary.

Use semantic metadata to decide which source actually needs inspection.

## 13. Debug runtime behavior instead of over-reading

When the question is fundamentally runtime-oriented, consider IntelliJ's
debugger instead of statically reading large amounts of code.

Examples:

* why does this branch execute?
* what implementation is selected at runtime?
* what value reaches this callback?
* how does an unfamiliar framework/plugin API behave?

Prefer:

```text
set targeted breakpoint
-> run/debug minimal reproduction
-> inspect stack/frame
-> evaluate necessary expressions
```

over reading dozens of potentially irrelevant implementations.

Fetch MCP Steroid's debugger skill before constructing unfamiliar debugger
automation.

Do not use the debugger when static PSI analysis already answers the question
cheaply.

## 14. Inspections

Use IntelliJ inspections when the IDE already has semantic knowledge capable
of validating the change.

Scope inspections to:

* changed files;
* affected modules;
* relevant source roots;

unless a project-wide inspection is explicitly required.

Do not launch every inspection over the entire repository after a tiny change.

Treat inspection findings as semantic diagnostics, then inspect only relevant
results.

## 15. Build and test routing

Do not assume IDE execution is always superior to normal build tooling.

Use the project's ordinary build/test commands when they are:

* already well-defined;
* long-running;
* CI-equivalent;
* integration/system tests;
* easier to execute reliably outside the IDE.

Prefer IDE-backed test execution when:

* structured test results materially help;
* rerunning a small failing test;
* navigating directly between failures and source;
* performing a tight compile/fix/test loop.

Avoid routing long full-suite operations through an MCP call merely because
the IDE can execute them.

## 16. Failure and latency handling

Treat unexpectedly long operations as a query-design signal.

If an operation that should be simple becomes slow:

1. do not immediately repeat it;
2. identify which API/search caused the latency;
3. check whether the IDE is indexing/configuring;
4. determine whether scope was unnecessarily broad;
5. determine whether an index-assisted candidate search can replace part of
   the operation;
6. fetch a recipe only if the failing API/pattern is not already established;
7. retry only with a materially improved strategy.

A transport or request timeout does not prove that the IDE-side script stopped.
Treat it as potentially still in flight and potentially mutating. Inspect Git
and filesystem state immediately. If accessible, inspect the IDE `idea.log` for
that execution's completion/session-removal entry before issuing another
`steroid_execute_code`; project/window listing proves routing/UI readiness, not
execution termination. Do not immediately submit a trivial readiness probe: a
wedged execution can retain MCP Steroid's serialized compilation lane, causing
the probe to queue behind it and add another stuck session. Use one short probe
only after logs or elapsed recovery indicate the prior execution ended.

MCP Steroid's body timeout uses cooperative coroutine cancellation, not a hard
JVM-thread interrupt. IntelliJ write/EDT operations or damaged lock state may
ignore or prevent timely cancellation and cleanup. If logs show no execution
completion and later calls report waiting for a previous compilation, stop all
execute-code retries. Preserve/inspect repository state, then recover by
restarting the IDE/backend only when doing so is safe for open user work; obtain
user approval when unsaved or interactive IDE state may exist. Never overlap a
retry with an execution whose termination is unknown.

If an optimization fails to compile or run, diagnose the import, signature,
threading, index readiness, or scope and correct the optimization. Failure of
one attempted API is not justification for immediately reverting to the most
expensive project-wide operation.

For a known symbol in a warm project, locating the declaration itself should
normally be cheap. If a combined query is slow, distinguish declaration lookup
from expensive reference/hierarchy resolution before diagnosing the problem.

Never describe an expensive combined operation as "finding the class" when
the expensive portion is actually project-wide semantic analysis.

## 17. Learn from executions

Do not call `steroid_execute_feedback` mechanically after routine successful
read-only queries. Feedback is optional and has wall-clock cost. Use it when it
provides meaningful learning: failed executions, API incompatibilities,
surprising behavior or latency, or a genuinely novel reusable workflow.

When feedback is useful, rate the execution accurately and preserve patterns
that solve recurring work significantly better than generic recipes.

A successful operation is not automatically an efficient operation.

When rating or documenting it, distinguish:

* correctness;
* completeness;
* latency;
* scope efficiency;
* token efficiency.

A query that returns the correct answer after an unnecessarily expensive
project-wide search should not be treated as the ideal pattern merely because
it succeeded.

## 18. Mutation safety

Before modifying code through MCP Steroid:

1. establish the exact semantic target;
2. understand the affected scope;
3. use structured refactoring when appropriate;
4. keep the operation minimal;
5. inspect the resulting diff;
6. run targeted validation.

Do not combine unrelated refactors into one opaque IDE operation.

Do not make speculative project-wide changes when the task can be completed
locally.

If the requested mutation is destructive or unusually broad, inspect usages
and hierarchy before proceeding.

## 19. Practical routing rules

Use these defaults unless the task gives a reason not to.

| Task                                        | Preferred strategy                                                    |
| ------------------------------------------- | --------------------------------------------------------------------- |
| Find filename                               | Normal Kilo filename search                                           |
| Find unique text/name                       | Normal Kilo indexed/text search                                       |
| Find known Java class                       | Normal search; `PsiShortNamesCache` only when exact PSI is needed     |
| Read implementation                         | Normal file read                                                      |
| Exploratory named usages                    | Normal text search; candidate-scoped PSI only if resolution adds value|
| Authoritative usages, textual coverage proven | Thread-safe candidates → scoped `ReferencesSearch`                  |
| Authoritative usages, coverage not proven   | Project-wide semantic `ReferencesSearch`                              |
| Load-bearing API audit                      | Unrestricted semantic references + relevant hierarchy checks         |
| Explicitly source-declared members          | Resolve class and filter `isPhysical` in the initial query            |
| Find implementations                        | PSI hierarchy/search APIs                          |
| Find overrides                              | PSI hierarchy/search APIs                          |
| Determine overload target                   | PSI resolution                                     |
| Rename symbol                               | IntelliJ refactoring API                           |
| Change method signature                     | IntelliJ refactoring API                           |
| Delete symbol safely                        | Usage/hierarchy analysis + safe-delete/refactoring |
| Inspect generated Lombok members            | PSI                                                |
| Search error/log string                     | Text search                                        |
| Validate changed Java                       | Targeted IntelliJ inspections/build                |
| Investigate runtime path                    | Debugger when appropriate                          |
| Full CI validation                          | Normal project build/test tooling                  |

## 20. Core rule

MCP Steroid provides access to an enormous portion of the IntelliJ Platform.

That power does not justify using the most powerful API for every operation.

Always ask:

1. What correctness guarantee does this task require?
2. What is the cheapest IntelliJ or ordinary tool that provides that
   guarantee?
3. Can candidates be narrowed before semantic resolution?
4. Is a verified execution pattern already available in this conversation or skill?
5. Can the result be returned compactly?
6. If code will change, should IntelliJ's refactoring engine perform the
   change?

Use text for text.

Use normal Kilo tools for cheap exploration and source reading.

Use indexes for discovery.

Use PSI for semantics.

Use refactoring APIs for structural mutation.

Use the debugger for runtime truth.

Use project-wide semantic resolution only when completeness actually requires
it.
