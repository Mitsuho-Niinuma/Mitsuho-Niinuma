[← Back to my profile](https://github.com/Mitsuho-Niinuma)

# 01 / From features to security questions

**A learning note from a student developer working toward security engineering.**

This note looks at my contribution to a team university application: input validation, registration and login, JSON persistence, and CLI/admin features. It is based on the current implementation and project documentation. It is a design reflection, not a penetration test or a claim of production readiness.

## Start with the flow

The application has command-line and graphical interfaces that call shared services. My work focuses on the account, persistence, and CLI/admin parts.

```text
User input → interface → shared service → record storage
                            ↓
                     result or clear error
```

Following this path makes it easier to see where data enters, where decisions belong, and where a failure must be handled.

## Three details worth examining

### 1. A saved file is still input

The storage layer checks the shape and types of records when loading them. Malformed data produces an error rather than being silently repaired or discarded.

**Why this matters:** a file can be edited, damaged, or created by a different version of the application. Reading from disk does not make its contents trustworthy.

**The limit:** validation checks expected structure and values. It does not prove that a record came from an authorized person or prevent someone with filesystem access from altering it.

### 2. Success belongs after the write

The save path validates records, writes a temporary file, and replaces the existing file after the write succeeds. Failures are surfaced to the caller.

**Why this matters:** an interface should not say that an account was created if the data could not be saved.

**The limit:** careful replacement is a reliability measure. It is not encryption, access control, a backup strategy, or a solution for concurrent writers. This coursework application assumes one modifying process at a time.

### 3. Confirmation is not authorization

The CLI asks for confirmation before destructive actions, and the service requires an explicit boolean confirmation for clearing records.

**Why this matters:** accidental deletion and unauthorized deletion are different problems. A confirmation prompt helps with the first; it does not establish who is allowed to act.

**The next question:** where should the application verify the caller's identity and permission before a change is made?

## What I would harden next

These are proposed learning and implementation steps, not completed features.

| Priority | Current limitation | Next step to explore | Evidence to look for |
| :--- | :--- | :--- | :--- |
| **Password storage** | The classroom implementation stores password values directly. | Use a maintained password-hashing library with an appropriate algorithm such as Argon2id. | Stored records contain hashes rather than passwords; correct and incorrect credentials are handled as expected. |
| **Authorization** | Admin navigation and confirmation do not enforce a role boundary. | Define roles and enforce permission checks in shared services, with denial as the default. | Unauthorized calls fail even when they bypass the UI; permitted actions still work. |
| **Failure behaviour** | File replacement does not cover every storage or deployment risk. | Extend failure scenarios and document the limits of the storage model. | A denied or failed operation leaves existing records intact and never reports success. |

The password-storage direction follows the [OWASP Password Storage Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Password_Storage_Cheat_Sheet.html). The authorization direction follows the [OWASP Authorization Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Authorization_Cheat_Sheet.html).

Any follow-up prototype should use synthetic records and a separate learning environment.

## What this adds to my learning

The useful shift is from “does the feature work?” to “what assumptions make it work, and what happens when those assumptions stop holding?”

That is the connection I want to build between my software development foundation and a future role in security engineering.

---

<sub>This public note summarizes design observations. Coursework source, assessment materials, teammate details, and runtime records are not included.</sub>
