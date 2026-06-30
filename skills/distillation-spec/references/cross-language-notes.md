# Cross-Language Notes

A reference for `distillation-spec` (and the implementer in `distillation-implementation`) when source and target languages differ. Use to inform mode decisions and to guide adaptations in `port` and `learn-then-rewrite` chunks.

This is not a complete language-pair handbook. It captures the recurring decision points; specific translations belong in the spec's adaptation-notes column on a per-chunk basis.

## Type-system mappings

| Concept | TypeScript | Python | Go | Rust | Java |
|---------|------------|--------|-----|------|------|
| Nominal class with methods | `class Foo {}` | `class Foo:` | struct + receivers | `struct` + `impl` | `class Foo` |
| Interface / protocol | `interface Foo {}` | `typing.Protocol` | `interface` (structural) | `trait` | `interface` |
| Union types | `A \| B` | `Union[A, B]` / `A \| B` | (none — use interfaces) | `enum` | (sealed) classes |
| Discriminated union | tag field on object | `dataclass` w/ tag | tag field on struct | `enum` variants | sealed hierarchy |
| Optional / nullable | `T \| undefined`, `T?` | `Optional[T]` | `*T` or zero value | `Option<T>` | `Optional<T>` |
| Result / error | thrown exception | thrown exception | `(T, error)` tuple | `Result<T, E>` | thrown exception |
| Generics | `<T>` | `TypeVar` | type parameters (1.18+) | `<T>` | `<T>` |
| Async return | `Promise<T>` | `Awaitable[T]` / `Coroutine` | (channel pattern) | `Future<T>` | `CompletableFuture<T>` |

When porting:

- Map the concept first, then the syntax. A `Result<T, E>` in Rust ported to TS often becomes a discriminated union, not a thrown exception, if the original code branches on it explicitly.
- Where the target language has no equivalent (e.g., union types in Go), use the closest pattern (interfaces, tag fields) and note the loss in the adaptation-notes.

## Concurrency idioms

| Source | Target | Translation |
|--------|--------|-------------|
| JS Promise chain | Python `asyncio` | `await` per step; nest `try/except` where `.catch` appeared. |
| JS `async/await` | Go | one goroutine per concurrent step, channels for results, `sync.WaitGroup` for joins. |
| Python `asyncio.gather` | JS | `Promise.all`. |
| Go goroutine + channel | Rust | `tokio::spawn` + `mpsc::channel`, or rayon if CPU-bound. |
| Rust `tokio::select!` | JS | `Promise.race` if first-wins; otherwise refactor. |
| Thread + lock (Java) | Go | goroutine + channel; locks (`sync.Mutex`) only if state must be shared. |
| Callback-style (Node legacy) | async/await | `util.promisify` (Node) then `await`. For other targets, port to the target's async style. |

## Library equivalents (representative)

| Source library | Possible targets |
|----------------|------------------|
| `axios` (JS) | `fetch` (built-in JS), `requests` (Python), `net/http` (Go), `reqwest` (Rust) |
| `lodash` | stdlib where possible; otherwise `ramda` (functional), or just inline. |
| `requests` (Python) | `fetch`/`axios` (JS), `net/http` (Go), `reqwest` (Rust) |
| `Joi`/`zod` validation | `pydantic` (Python), `validator` package (Go), `serde` + custom checks (Rust) |
| `winston` logger | `loguru` (Python), `slog` (Go), `tracing` (Rust) |

When the user's project already pins a library for the function, prefer that pin over picking a new one — the spec records the choice.

## Idiom translations to watch for

- **Truthiness.** JS truthiness (`0`, `""`, `null`, `undefined`, `NaN` all falsy) is unlike Python's (`0`, `""`, `[]`, `{}`, `None`). Port explicit comparisons, not truthiness.
- **Equality.** JS `===` vs `==`; Python `==` vs `is`; Go has no operator overloading. Translate to the target's strict-equality default.
- **Mutation.** JS spreads (`{...obj}`) vs Python `dict(obj)` vs Go map copy vs Rust `.clone()`. Match target's idiomatic copy.
- **Default args.** Python's mutable default arg pitfall (`def f(x=[])`) doesn't exist in JS/Go/Rust. Don't copy the workaround if it's not needed.
- **Iteration.** JS `for...of`, Python `for x in`, Go `for i, v := range`, Rust `for x in ...`. Map to target's idiomatic loop.
- **String formatting.** Python f-strings, JS template literals, Go `fmt.Sprintf`, Rust `format!`. Don't transliterate `%s` from Python into JS.

## Things that usually do NOT port cleanly

When you see these in the source, lean toward `learn-then-rewrite`:

- Metaclasses, decorator-driven class registration (Python).
- Macro-heavy code (Rust, C, Lisp).
- Reflection-based runtime wiring (Java annotations, Spring `@Autowired`).
- C-style memory layout / pointer arithmetic.
- Heavy `goto`/`unsafe` usage.
- Reactive frameworks tied to a specific runtime (RxJS schedulers, akka actors).

## Things that usually DO port cleanly

- Pure functions with primitive inputs/outputs.
- Algorithms over standard data structures (arrays, maps, trees).
- Schema-driven validation and serialization (when an equivalent library exists).
- HTTP route handlers (with framework-equivalent shims).
- Stateless utilities.
