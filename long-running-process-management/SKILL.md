---
name: long-running-process-management
description: >
  Use this skill when starting, testing, monitoring, or stopping servers,
  development servers, watchers, port forwards, or other long-running processes.
  Prefer spawn over synchronous execution for processes that are expected to keep
  running, verify readiness separately, and always clean up processes started by
  the agent.
---

# Long-Running Process Management

## Purpose

Use this skill when working with commands that are expected to continue running
instead of terminating normally.

Typical examples include:

- Spring Boot servers
- `java -jar ...`
- `mvn spring-boot:run`
- `gradlew bootRun`
- Vite development servers
- `npm run dev`
- `npm start`
- Next.js development servers
- Python HTTP servers
- Uvicorn / Gunicorn
- Docker Compose foreground services
- `kubectl port-forward`
- file watchers
- log followers
- mock servers
- local API servers

The key distinction is:

```text
exec
  Use for commands that are expected to terminate.

spawn
  Use for commands that are expected to keep running.
```

Do not use synchronous execution and wait indefinitely for a server or other
long-running process to terminate.

---

# Core Rule

Before executing a command, classify it as one of the following:

1. **Finite command**
   - Expected to terminate after completing its work.
   - Use `exec`.

2. **Long-running process**
   - Expected to remain alive while providing a service, watching files,
     forwarding ports, or waiting for requests.
   - Use `spawn`.

If uncertain, inspect the command and its purpose before execution.

Do not assume that every successful command eventually exits.

For a server process:

```text
process remains running
```

is normally a success condition, not a reason to wait for completion.

---

# Finite Commands

Examples:

```text
mvn test
mvn package
gradlew test
npm install
npm test
curl http://localhost:8080/health
git status
docker compose ps
```

These should normally use `exec`.

A finite command must not be converted to `spawn` merely to avoid handling
timeouts or failures.

---

# Long-Running Commands

Examples:

```text
mvn spring-boot:run
./mvnw spring-boot:run
gradlew bootRun
java -jar app.jar
npm run dev
vite
next dev
python -m http.server
uvicorn app:app
docker compose up
kubectl port-forward ...
tail -f ...
```

These should normally use `spawn`.

Never execute them synchronously and wait for their natural termination as part
of a normal startup or verification workflow.

---

# Standard Lifecycle

Use the following lifecycle for long-running processes:

```text
spawn
  ↓
record processId
  ↓
verify process is still alive
  ↓
wait for readiness with a bounded timeout
  ↓
perform verification or tests
  ↓
collect relevant logs if necessary
  ↓
kill
  ↓
verify cleanup
```

Every process started by the agent should have a corresponding cleanup path.

---

# Starting a Process

When starting a long-running process:

1. Use `spawn`.
2. Record the returned `processId`.
3. Do not interpret successful `spawn` as proof that the application is ready.
4. Check whether the process exits unexpectedly during startup.
5. Verify readiness independently.

Example:

```text
spawn("./mvnw spring-boot:run")
→ processId = proc-123
```

After this point, use `proc-123` for status, logs, and cleanup.

---

# Spawn Success Is Not Readiness

There are three separate states:

```text
process created
application running
application ready
```

Do not treat them as equivalent.

For example:

```text
spawn succeeded
```

only means that process creation succeeded.

A Spring Boot process may still be:

- compiling
- loading configuration
- connecting to a database
- running migrations
- binding its HTTP port
- failing during application initialization

Always perform a readiness check.

---

# Readiness Checks

Prefer readiness checks in the following order.

## 1. Application Health Endpoint

If an explicit health endpoint exists, prefer it.

Examples:

```text
GET /actuator/health
GET /health
GET /ready
GET /healthz
```

A successful HTTP response is generally stronger evidence than checking only
whether the process exists.

---

## 2. Target Application Endpoint

If no health endpoint exists, call an endpoint that proves the application can
serve requests.

Example:

```text
GET http://localhost:8080/api/example
```

Use a harmless endpoint when possible.

---

## 3. TCP Port

If HTTP-level verification is unavailable, check whether the expected port is
accepting connections.

A listening port proves less than an application-level health check, so prefer
HTTP checks when available.

---

## 4. Startup Logs

Logs may be used when there is no better readiness signal.

For Spring Boot, a message similar to:

```text
Started ExampleApplication in ... seconds
```

is useful evidence of startup completion.

Do not rely on an exact message if the application or framework configuration
may change it.

---

# Bounded Waiting

Never wait indefinitely for readiness.

Readiness polling must have:

- a maximum duration
- a polling interval
- a failure condition

Conceptually:

```text
deadline = now + startup_timeout

while now < deadline:
    if process exited:
        fail and inspect logs

    if readiness_check succeeds:
        continue with verification

    wait briefly

fail because readiness timeout was exceeded
```

Do not implement an unbounded retry loop.

Do not retry the exact same failing operation forever.

---

# Process Status

While waiting for readiness, periodically verify that the process still exists.

If the process exits before becoming ready:

1. stop readiness polling
2. inspect its exit status
3. inspect recent stdout/stderr
4. report the actual startup failure

Do not continue polling an endpoint for a process that has already exited.

---

# Logs

Use process logs for diagnosis and startup confirmation.

Prefer retrieving only the relevant recent output rather than repeatedly
fetching the complete process log.

Typical uses:

- startup exception
- port conflict
- configuration error
- dependency connection failure
- Spring Boot startup completion
- runtime exception caused by a verification request

Do not poll logs in a tight loop.

When a process manager supports tailing a bounded number of lines, prefer that.

---

# Cleanup

A long-running process started by the agent must normally be stopped when it is
no longer needed.

Use `kill(processId)` rather than attempting to identify and terminate it using
an arbitrary OS PID.

Cleanup should happen whether verification:

- succeeds
- fails
- times out
- encounters an unrelated error

Conceptually, treat the lifecycle like:

```text
processId = spawn(...)

try:
    wait_until_ready(processId)
    run_verification()
finally:
    kill(processId)
```

Do not leave development servers or test servers running accidentally.

---

# Do Not Kill Unrelated Processes

Only stop processes that were started and are managed by the current agent
workflow.

Do not:

- guess a PID
- kill every `java` process
- kill every `node` process
- terminate a process merely because it uses the expected port
- terminate unrelated user processes

If the desired port is already in use before startup, diagnose the conflict
rather than indiscriminately killing the existing process.

---

# Existing Servers

Before starting a server, consider whether an instance is already intentionally
running.

If an existing service is clearly part of the user's environment, do not stop
or replace it without a reason.

When verification specifically requires a fresh instance, prefer:

- a different port
- an isolated test environment
- an explicit managed process

rather than interfering with unrelated services.

---

# Spring Boot Workflow

For Spring Boot applications, prefer this sequence.

```text
1. Run tests or build with exec.

   ./mvnw test

2. Start the application with spawn.

   ./mvnw spring-boot:run

3. Record processId.

4. Poll readiness with a bounded timeout.

   Preferred:
   /actuator/health

   Otherwise:
   an application endpoint
   TCP port
   startup log

5. If startup fails:
   inspect process status and recent logs.

6. Perform required HTTP/API verification.

7. Inspect logs if the verification exposes an error.

8. Kill the spawned process.

9. Verify that the managed process has stopped.
```

Do not run:

```text
./mvnw spring-boot:run
```

with synchronous `exec` and wait for command completion.

---

# Frontend Development Servers

For Vite, Next.js, or similar development servers:

```text
npm install
↓
spawn("npm run dev")
↓
wait for HTTP readiness
↓
perform verification
↓
kill(processId)
```

Remember that development servers may print a URL before all application-level
functionality is ready.

Prefer an actual HTTP request where possible.

---

# Docker Compose

Distinguish between foreground and detached operation.

If using:

```text
docker compose up
```

as a foreground long-running process, use `spawn`.

If using:

```text
docker compose up -d
```

the command itself terminates and may use `exec`.

However, detached containers then have their own lifecycle.

Do not confuse:

```text
docker compose up -d succeeded
```

with:

```text
all services are healthy
```

Check container or application health separately.

If the agent created temporary Compose services specifically for verification,
clean them up when appropriate.

---

# Port Forwarding

Commands such as:

```text
kubectl port-forward ...
```

are long-running processes.

Use:

```text
spawn
↓
verify local port
↓
perform operation
↓
kill
```

Do not synchronously wait for `kubectl port-forward` to finish.

---

# Watchers

Commands such as:

```text
npm run watch
tsc --watch
tail -f
```

are intentionally non-terminating.

Use `spawn` only when the watcher is actually required for the task.

Do not start a watcher when a one-shot equivalent is sufficient.

For example, prefer:

```text
tsc
```

over:

```text
tsc --watch
```

when only a single build is needed.

---

# Prefer Finite Alternatives

Before spawning a long-running process, ask whether a finite command can
accomplish the same task.

Examples:

```text
docker compose up -d
```

may be preferable to foreground Compose when lifecycle management is already
provided elsewhere.

```text
npm run build
```

may be preferable to starting a dev server when only compilation needs to be
verified.

Do not start servers unnecessarily.

---

# Failure Handling

## Spawn Failure

If process creation itself fails:

- report the spawn error
- do not attempt readiness polling
- do not fabricate a processId

---

## Early Process Exit

If the process exits before readiness:

- stop polling
- obtain exit status if available
- inspect recent stdout/stderr
- diagnose the failure

---

## Readiness Timeout

If the process remains alive but never becomes ready:

- stop waiting when the deadline is reached
- inspect logs
- report the timeout and relevant diagnostics
- kill the process

Do not silently increase the timeout repeatedly.

---

## Verification Failure

If the server starts successfully but an HTTP/API verification fails:

- distinguish application startup success from verification failure
- inspect relevant logs
- preserve useful diagnostics
- clean up the process

---

## Kill Failure

If normal process termination fails:

- check process status
- use the process manager's supported escalation mechanism if available
- do not attempt unrelated arbitrary process termination

Report cleanup failures explicitly.

---

# Multiple Processes

When several services are required, track every `processId` independently.

Example:

```text
backend  → proc-101
frontend → proc-102
mock     → proc-103
```

Do not mix process identifiers.

When cleanup is required, stop only the managed processes that were started for
the current task.

If startup of a later service fails, clean up already-started earlier services.

Example:

```text
spawn backend
spawn frontend  ← fails

cleanup backend
```

---

# Anti-Patterns

Never intentionally perform these patterns:

```text
exec("mvn spring-boot:run")
wait forever
```

```text
spawn(...)
assume application is immediately ready
```

```text
spawn(...)
poll forever until HTTP succeeds
```

```text
spawn(...)
run tests
finish task without kill
```

```text
kill arbitrary PID
```

```text
kill all java processes
```

```text
kill all node processes
```

```text
retry the same failed readiness check indefinitely
```

```text
wait for a server process to exit as proof of successful startup
```

---

# Decision Guide

Use the following decision rule before invoking the shell.

```text
Will this command normally terminate after completing its task?

YES
  → exec

NO
  → Is continued execution necessary for the current task?

      YES
        → spawn
        → readiness check
        → work
        → kill

      NO
        → choose a finite alternative or do not run it
```

---

# Completion Criteria

When a task requires starting a long-running process, consider the task
complete only when:

- the appropriate process was started with `spawn`
- its `processId` was recorded
- readiness was verified with a bounded wait
- the required verification was performed
- relevant failures were diagnosed using status/logs
- the managed process was stopped when no longer needed
- no agent-created background process was unintentionally left running

The goal is not merely to start a process.

The goal is to manage its complete lifecycle safely.
