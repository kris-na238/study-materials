# Top 25 NestJS Interview Questions & Answers (Senior / 7+ YOE)

Pitched at a level where the interviewer expects architectural reasoning, trade-offs, and internals knowledge — not just definitions.

---

### 1. What is NestJS and why choose it over plain Express/Fastify?

NestJS is an opinionated, TypeScript-first Node.js framework built on top of Express (default) or Fastify. It layers a modular architecture, dependency injection (DI), and decorators borrowed from Angular onto the underlying HTTP framework.

Why choose it: enforced structure at scale (modules/controllers/providers), first-class DI for testability, built-in support for microservices, GraphQL, WebSockets and CQRS, and a consistent way to implement cross-cutting concerns (guards, interceptors, pipes, filters). Plain Express gives freedom but no structure — on a large team that freedom becomes inconsistency. The trade-off is more boilerplate and a steeper learning curve.

---

### 2. Explain the request lifecycle and the order in which middleware, guards, interceptors, pipes, and filters execute.

Order on a normal (non-exception) request:

Middleware → Guards → Interceptors (pre-controller logic) → Pipes → Route Handler → Interceptors (post-controller logic, response transform) → Exception Filters (only if something throws).

Key nuance: middleware has no access to the ExecutionContext (route handler/class metadata) — it only sees req/res/next, like Express middleware. Guards run after middleware and decide access (return boolean or throw), and they *do* have ExecutionContext, so they can use `Reflector` to read custom metadata (e.g. `@Roles()`). Interceptors wrap the handler using RxJS (`tap`, `map`, `catchError`), so they can run logic both before and after execution, including transforming the response or catching errors. Pipes only apply to the arguments right before the handler runs — validation/transformation of `@Body()`, `@Param()`, etc. Exception filters catch anything thrown anywhere downstream and shape the HTTP response.

---

### 3. What's the difference between Guards, Interceptors, Pipes, and Middleware — and when would you use each?

- **Middleware**: cross-cutting request/response logic with no DI-aware context, e.g. logging raw requests, body parsing, CORS at a low level.
- **Guards**: authorization decisions — "can this request proceed?" (`@UseGuards(AuthGuard)`, `RolesGuard`).
- **Pipes**: validation and transformation of input data (`ValidationPipe`, `ParseIntPipe`).
- **Interceptors**: wrap the execution to add behavior around it — logging with timing, response mapping/serialization, caching, retrying, transforming exceptions into different shapes.

A common trap: candidates use a guard to do input validation, or a pipe to do auth. Keeping these separated is what interviewers are checking for.

---

### 4. Explain NestJS's Dependency Injection system and provider scopes.

Nest has its own IoC container. Providers (`@Injectable()`) are registered on a module and instantiated by the container, which resolves constructor-based dependencies by type (or by token for non-class dependencies).

Three scopes:
- **DEFAULT (singleton)** — one instance shared app-wide. Fastest, used almost everywhere.
- **REQUEST** — a new instance created per incoming request; needed when a provider must hold request-specific state (e.g. tenant ID from a header). Costly: it forces every provider in the injection chain that depends on it to also become request-scoped, which hurts performance because DI subtrees get rebuilt per request.
- **TRANSIENT** — a new instance every time it's injected, even within the same request.

Senior-level point to raise: avoid REQUEST scope unless truly necessary (e.g. multi-tenancy via CLS/AsyncLocalStorage is often a better alternative that avoids the DI performance penalty).

---

### 5. What are custom providers, and what's the difference between `useClass`, `useValue`, `useFactory`, and `useExisting`?

Custom providers let you decouple an injection token from a concrete implementation:

- `useClass` — bind a token to a class, useful for swapping implementations per environment (e.g. mock vs real service).
- `useValue` — bind a token to a static value/object, common for constants, config objects, or mocking in tests.
- `useFactory` — bind a token to a function's return value, which can itself inject other dependencies via `inject: [...]`. Used when construction requires async work or runtime logic (e.g. building a DB connection).
- `useExisting` — create an alias so two tokens resolve to the same underlying provider instance.

This is the mechanism behind Nest's testing ergonomics — you override a real provider with a `useValue` mock in `Test.createTestingModule()`.

---

### 6. How does Nest resolve circular dependencies, and when do you need `forwardRef()`?

Circular dependencies happen when Module A imports Module B and B imports A, or two providers inject each other directly. Nest can't resolve the reference at declaration time because the class doesn't exist yet in that evaluation order.

`forwardRef(() => ModuleB)` (or the same for providers) tells Nest to lazily resolve the reference at runtime instead of at load time. It's a valid escape hatch, but circular dependencies are usually a sign the module boundaries are wrong — the senior-level answer is to mention that the preferred fix is refactoring shared logic into a third module both depend on, and reserving `forwardRef` for cases where the cycle is genuinely intrinsic to the domain (e.g. Users ↔ Posts modules needing each other for combined queries).

---

### 7. What are Dynamic Modules and when would you build one?

A dynamic module is a module whose providers/config are determined at runtime via a static method (conventionally `forRoot()`, `forRootAsync()`, `register()`, `registerAsync()`) that returns a `DynamicModule` object instead of using the static `@Module()` decorator alone.

Used for reusable, configurable modules — e.g. `TypeOrmModule.forRoot(config)` or a custom `CacheModule.forRootAsync({ useFactory, inject })` that needs to pull config from `ConfigService` asynchronously. The `Async` variants exist because config often depends on another provider that itself may be async (e.g. reading env vars, secrets manager).

---

### 8. How do you implement global exception handling, and how do you preserve type-safe, consistent API error responses?

Implement a class annotated `@Catch()` (optionally scoped to specific exception types, e.g. `@Catch(HttpException)`) implementing `ExceptionFilter`, overriding `catch(exception, host)`. Register it globally via `app.useGlobalFilters(new AllExceptionsFilter())` or, for DI-aware filters, via `APP_FILTER` in a module's providers.

Senior considerations to mention: distinguish `HttpException` (expected, has a status) from unexpected errors (log with full stack trace, return a generic 500 to avoid leaking internals), normalize the response shape (`{ statusCode, message, error, timestamp, path }`), and integrate with a logging/observability pipeline (correlate via request ID). Also mention that filters run outside the RxJS pipeline used by interceptors, so `catchError` in an interceptor can intercept before it reaches the filter if you need to transform errors earlier.

---

### 9. How does the `ValidationPipe` work with `class-validator` and `class-transformer`, and what performance/security considerations come with it?

`ValidationPipe` uses `class-transformer` to turn the plain incoming JSON into an instance of the DTO class, then `class-validator` runs the decorators (`@IsString()`, `@IsEmail()`, etc.) against that instance and throws a `BadRequestException` on failure.

Key options a senior engineer should know: `whitelist: true` strips properties not defined in the DTO (prevents mass-assignment attacks), `forbidNonWhitelisted: true` rejects the request instead of silently stripping, `transform: true` auto-converts primitives (e.g. path param strings to numbers) so handlers receive properly typed values, and `disableErrorMessages` in production to avoid leaking validation internals. This is also where object depth and validation cost matter for large payloads — worth caching compiled validators if using it at very high throughput.

---

### 10. Explain how NestJS supports microservices, and the difference between the transport strategies.

`@nestjs/microservices` lets a Nest app run as (or talk to) a microservice using a transport layer: TCP, Redis pub/sub, NATS, MQTT, gRPC, RabbitMQ (via `amqp-connection-manager`), or Kafka. Each transport is a `ClientProxy`/`Server` implementation behind the same abstraction, so business logic (`@MessagePattern` / `@EventPattern` handlers) stays transport-agnostic.

`@MessagePattern` = request/response (client waits for a reply). `@EventPattern` = fire-and-forget pub/sub (no reply expected). The interviewer is often checking whether you know **when** to reach for each transport: gRPC/TCP for low-latency internal RPC with strict contracts, Kafka for high-throughput event streaming and replay, RabbitMQ for reliable work queues with retry/DLQ semantics.

---

### 11. What is the difference between `@nestjs/event-emitter` based event-driven code and the CQRS module (`@nestjs/cqrs`)?

`EventEmitterModule` provides simple in-process pub/sub (`this.eventEmitter.emit('order.created', payload)` and `@OnEvent('order.created')` listeners) — useful for decoupling side effects (send email, update cache) from the main transaction without a separate infra dependency.

`@nestjs/cqrs` is a more structured pattern: **Commands** (intent to change state, handled by a single `CommandHandler`), **Queries** (read-only, handled by `QueryHandler`), and **Events** (facts that already happened, potentially handled by multiple `EventHandler`s, often via an event-sourced aggregate root). CQRS is worth reaching for when you have complex write-side business rules that benefit from being modeled as aggregates and want a clean separation between the write and read models — but it's meaningful added complexity that should be justified by the domain, not applied by default.

---

### 12. How would you handle authentication and authorization in a NestJS app (JWT, guards, Passport)?

Authentication typically uses `@nestjs/passport` with a strategy (`passport-jwt`, `passport-local`, `passport-google-oauth20`, etc.). You define a `Strategy` class implementing `validate()`, wrap it in an `AuthGuard('jwt')`, and apply it via `@UseGuards()`.

Authorization is layered on top with a custom `RolesGuard` reading metadata set by a `@Roles('admin')` decorator (via `Reflector.getAllAndOverride`), or attribute-based checks (CASL is a common pairing for fine-grained permissions). Senior-level detail worth mentioning: making guards global via `APP_GUARD` while using a `@Public()` decorator to opt specific routes out, refresh-token rotation strategy, and where token verification cost fits (JWT is stateless/fast; sessions require a store like Redis).

---

### 13. How do you test a NestJS application — unit tests vs. e2e tests — and how does `Test.createTestingModule` help?

Unit tests isolate a single provider/controller by building a minimal `TestingModule` and overriding its dependencies with mocks: `Test.createTestingModule({ providers: [UsersService, { provide: UsersRepository, useValue: mockRepo }] }).compile()`. This leverages Nest's own DI container so you get real constructor injection without needing a full app.

E2e tests spin up the full app (`app = moduleFixture.createNestApplication(); await app.init();`) and hit it with `supertest`, exercising real middleware/guards/pipes/filters and (usually) a real or containerized test database. The senior answer distinguishes: unit tests for business logic and edge cases (fast, many), e2e/integration tests for critical user flows and wiring correctness (slower, fewer), and mentions test doubles for external services (payment gateway, email) to keep e2e tests deterministic.

---

### 14. What are lifecycle hooks in Nest, and what's a real use case for each of the main ones?

- `OnModuleInit` — run setup logic after all providers in the module are resolved (e.g., warm a cache, verify a DB connection).
- `OnApplicationBootstrap` — after the entire app's modules are initialized, useful for logic that depends on other modules also being ready.
- `OnModuleDestroy` / `beforeApplicationShutdown` / `OnApplicationShutdown` — graceful shutdown: close DB pools, flush queues, deregister from service discovery, stop accepting new work while draining in-flight requests. Requires `app.enableShutdownHooks()` to actually fire on SIGTERM/SIGINT, which matters a lot in a Kubernetes environment doing rolling deploys — omitting this is a common production bug (pods get killed mid-request).

---

### 15. How does Nest handle configuration management for multiple environments, and how do you keep it type-safe?

`@nestjs/config` wraps `dotenv`, loading `.env` files into `ConfigService`. For senior-level rigor: use `ConfigModule.forRoot({ validationSchema })` with Joi (or a class-validator-based approach) to fail fast at boot if required env vars are missing/malformed, rather than discovering it via a runtime NPE. Namespaced config (`registerAs('database', () => ({...}))`) keeps config modular per-module. Also worth mentioning: never let secrets sit in `.env` files in production — inject them from a secrets manager (AWS Secrets Manager, Vault) at deploy time, with `ConfigService` just reading whatever is already in `process.env`.

---

### 16. How would you implement caching in NestJS, and what are the trade-offs between in-memory and distributed caches?

`CacheModule` (`@nestjs/cache-manager`) provides `CacheInterceptor` for automatic response caching per-route, or manual `cacheManager.get/set` for fine-grained control. In-memory (default store) is fast but doesn't share state across horizontally scaled instances and is lost on restart — fine for single-instance or per-pod caching of cheap-to-recompute data. A distributed store (Redis) is required once you scale horizontally and need cache coherence across instances, and it survives restarts.

Senior-level nuance: cache invalidation strategy matters more than the caching mechanism itself — TTL-based expiry is simplest but can serve stale data; explicit invalidation on writes is more correct but adds coupling between write paths and cache keys. Also worth flagging cache stampede protection for hot keys.

---

### 17. Explain custom decorators in Nest — how do you build one, and give an example beyond `@Roles()`.

Param decorators use `createParamDecorator((data, ctx: ExecutionContext) => {...})`, e.g. a `@CurrentUser()` decorator that pulls `request.user` off the request object (set earlier by a Passport guard) — cleaner than repeating `req.user` extraction logic in every controller. Method/class decorators use `SetMetadata(key, value)`, later read via `Reflector` in a guard/interceptor — this is exactly how `@Roles()` works under the hood. You can also compose multiple decorators into one with `applyDecorators()`, useful for bundling `@ApiOperation` + `@UseGuards` + `@ApiResponse` combos in Swagger-documented endpoints to reduce boilerplate.

---

### 18. How do interceptors handle response transformation and logging with timing, using RxJS?

An interceptor's `intercept(context, next)` returns an `Observable`. `next.handle()` is the call stream continuing to the route handler; you can `map()` the emitted value to reshape the response (e.g., wrap every response in `{ data, meta }`), `tap()` to log timing without altering the value (`const start = Date.now(); return next.handle().pipe(tap(() => console.log(Date.now() - start)))`), or `catchError()` to intercept exceptions before they reach filters. This RxJS-based design is what lets interceptors run logic both before *and* after the handler in a single class, unlike middleware.

---

### 19. What's the difference between `forRoot()` and `forFeature()` seen in modules like `TypeOrmModule`?

`forRoot()` configures the module once at the application (or a shared) level — e.g. the DB connection itself (host, credentials, pool size). `forFeature()` is called per-feature-module to register what that specific module needs from the already-configured root — e.g. `TypeOrmModule.forFeature([User])` registers the `User` repository for injection within the `UsersModule`, without re-specifying connection details. This pattern shows up across Nest's ecosystem (Mongoose, CQRS event bus registration, etc.) and is worth recognizing as a general "global setup vs. local registration" convention, not something TypeORM-specific.

---

### 20. How do you structure a large NestJS monorepo/application for maintainability at scale?

Feature-based modules (not layer-based — avoid a single giant `services/` folder), each module owning its controllers/services/DTOs/entities and exposing only what other modules need via `exports`. Nest CLI/Nx monorepo support for sharing code between multiple deployable apps (e.g. API + worker) via `libs/`. Barrel files per module for clean imports. Domain boundaries enforced by only injecting exported providers rather than reaching into another module's internals. Also worth mentioning: keeping DTOs/validation at the boundary, keeping entities/ORM concerns out of controllers, and using an application/domain/infrastructure layering within larger modules if the domain complexity warrants it (hexagonal-ish, without over-engineering a CRUD module).

---

### 21. How would you handle database transactions in NestJS with TypeORM or Prisma, especially across multiple service calls?

With TypeORM: either `QueryRunner` manually (`connection.createQueryRunner()`, `startTransaction()`, `commit()/rollback()` in try/catch/finally) for full control, or the `@Transaction()` / repository-per-transaction pattern. A cleaner pattern for multi-service transactions is wrapping the unit of work in a service method that takes an `EntityManager` and passes it down, so all repository calls in that flow share the same transaction context — this avoids the common bug of each service independently grabbing its own repository/connection and not being part of the same transaction. With Prisma, `prisma.$transaction([...])` for simple batched operations, or the interactive transaction API for logic requiring conditionals between steps. Mention the trade-off: long-lived transactions holding row locks can hurt throughput under load — keep transaction scope as narrow as possible.

---

### 22. How do you secure a NestJS API against common vulnerabilities (rate limiting, CORS, headers, injection)?

- **Rate limiting/throttling**: `@nestjs/throttler` with `ThrottlerGuard`, ideally backed by Redis in a multi-instance deployment so limits are enforced globally, not per-pod.
- **CORS**: `app.enableCors({ origin, credentials })`, explicit allow-list rather than `*` when credentials are involved.
- **Security headers**: `helmet()` middleware for sane defaults (CSP, HSTS, X-Frame-Options).
- **Injection**: parameterized queries/ORM query builders instead of raw string concatenation; `whitelist`/`forbidNonWhitelisted` on `ValidationPipe` to prevent mass assignment.
- **Auth**: short-lived JWTs with refresh rotation, `bcrypt`/`argon2` for password hashing, never returning password hashes even accidentally via `class-transformer`'s `@Exclude()` on entities.

A senior candidate should also mention keeping dependencies patched (`npm audit`) and not trusting client-supplied IDs for authorization checks (always re-verify ownership server-side).

---

### 23. How would you scale a NestJS application horizontally, and what stateful pitfalls should you watch for?

Run multiple stateless instances behind a load balancer; move anything stateful out of process memory — sessions/cache to Redis, rate-limit counters to Redis, pub/sub for cross-instance events (e.g. WebSocket gateways need a Redis adapter — `@nestjs/platform-socket.io` with `socket.io-redis` — otherwise a message emitted from one instance won't reach clients connected to another). Avoid REQUEST-scoped providers where possible (performance cost per earlier answer). Use the cluster module or a process manager (PM2) / container orchestrator (k8s HPA) for scaling actual Node processes, since Node is single-threaded per process. Graceful shutdown hooks (question 14) matter here too, for zero-downtime rolling deploys.

---

### 24. What's the difference between `@Injectable()` services and using plain utility functions/classes — when is DI actually worth it?

DI is worth it when a dependency needs to be swapped (mocked in tests, swapped per environment, decorated/wrapped), when it has its own lifecycle (connections, lazy init), or when multiple consumers share a single instance (singleton caching, connection pools). Stateless pure functions (formatters, small pure calculations) gain nothing from being wrapped in an injectable class — that's DI cargo-culting and adds indirection without benefit. A senior engineer should be able to draw this line rather than defaulting every piece of logic into a service class.

---

### 25. How do you handle versioning of a NestJS REST API?

Nest has built-in support: URI versioning (`/v1/users`), header versioning, media-type versioning, or custom versioning, enabled via `app.enableVersioning({ type: VersioningType.URI })` and `@Controller({ version: '1' })` / `@Version('2')` on individual routes so v1 and v2 can coexist in the same controller during a migration window. Beyond the mechanism, the interview is usually probing for judgment: prefer additive, backward-compatible changes where possible to avoid version proliferation; when a breaking change is unavoidable, version at the resource/controller level rather than the whole API; deprecate old versions with clear sunset communication (headers, docs) rather than silently dropping them.

---

*Tip for the interview: senior-level answers should consistently pair the "what" with a trade-off or a "when NOT to use this" — that's usually what separates a mid-level answer from a senior one on these questions.*
