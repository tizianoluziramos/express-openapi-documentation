---
name: express-openapi-documentation
description: Create, update, and review OpenAPI YAML documentation based on actual API source code. Use this skill whenever the user asks to document endpoints, generate or edit *.openapi.yaml files, maintain Swagger contracts, or simplify their descriptions, even if they only say "document this feature." Produce verifiable contracts and concise descriptions focused on what happens.
---

# Feature-Based OpenAPI Documentation

Document observable API behavior using the style of `src/features/auth/auth.openapi.yaml`: YAML fragments per feature, reusable components, and concise, affirmative prose. Source code is the source of truth; existing documentation provides conventions rather than evidence of behavior.

## 1. Establish the Scope and Read the Contract

1. Read the repository instructions and review existing changes before editing.
2. Locate the requested file, the OpenAPI configuration, its version, the file-loading pattern, and a nearby example.
3. Trace router mounting through each path and method. Review the controllers, DTOs, services, models, and middleware affecting those operations.
4. Extract the following for each operation:
   - Full path relative to `servers.url`, HTTP method, parameters, and request body.
   - Effective authentication, roles, verification requirements, and middleware order.
   - HTTP status, JSON structure, messages, headers, and content type.
   - Constraints, required fields, transformations, defaults, and nullability.
   - Reachable errors and rate limits actually applied.

Associations and serialization options matter: document the attributes selected by the query and sent by the controller rather than every field in the model.

If apparent intent differs from the implementation, briefly flag the discrepancy to the user and document only what the code supports. Keep logic fixes outside the scope of a documentation task. Ask for clarification when a missing decision prevents an accurate contract description.

## 2. Build the YAML

- Use `src/features/<feature>/<feature>.openapi.yaml` unless another location is requested.
- Follow the existing architecture. With `swagger-jsdoc` and per-feature files, generate a fragment containing `tags`, `paths`, and the necessary `components`. Keep `openapi`, `info`, and `servers` in the central configuration. Standalone documents include their required root metadata.
- Use two-space indentation, quoted HTTP status codes, and double-quoted local `$ref` values.
- Follow the language of neighboring files; write OpenAPI content in English in this project.
- Include `tags`, a unique `operationId`, a short `summary`, inputs, and responses for each operation. Add a `description` when it contributes information beyond the summary.
- Model the actual response body, including its wrappers (`status`, `message`, `data`, `error`, `issues`) where present.
- Derive `required`, `additionalProperties`, formats, patterns, limits, enums, and defaults from the actual validator. Distinguish optional fields from values accepting `null`.
- Describe input transformations briefly and represent accepted input values; for example, a code processed with `.trim()` may accept whitespace around its six digits.
- Use syntax supported by the configured version: OpenAPI 3.1 supports `const` and `type: [string, "null"]`; use compatible equivalents for OpenAPI 3.0.
- Reuse schemas, responses, and headers when they share the same contract. Prefix feature-specific component names to prevent collisions between fragments; reuse existing global components where appropriate.
- Include small, valid examples with fictional data. Preserve response messages from the code verbatim.

### Security, Cookies, and Rate Limits

- Derive `security` from middleware: apply the effective scheme to protected operations and `security: []` to public operations where appropriate.
- For cookie-based sessions, define `type: apiKey`, `in: cookie`, and the actual cookie name. Document `Set-Cookie` on operations that set or clear it, including its effective attributes and lifetime.
- Describe role and verification requirements when they apply.
- Add `429` only to rate-limited routes. Document the window, limit, and actual content type; model a text response as text even when other errors use JSON.
- Derive errors from reachable execution paths, including the global error handler. This prevents copying HTTP statuses belonging to other routes.

## 3. Describe Only What Happens

Help readers quickly identify each operation's behavior. Describe concrete actions, requirements, and outcomes; remove commentary about absent functionality, unnecessary contrasts, and explanations that repeat the schema.

| Avoid in explanatory prose                                                              | Prefer                                                                |
| --------------------------------------------------------------------------------------- | --------------------------------------------------------------------- |
| `Creates a user without starting a session or sending a verification email.`            | `Creates a user.`                                                     |
| `Sets the accessToken cookie. The JWT is not returned in the JSON body.`                | `Sets the accessToken cookie.`                                        |
| `Requires a valid accessToken cookie. No separate refresh token is used.`               | `Requires a valid accessToken cookie and replaces it with a new JWT.` |
| `Clears the accessToken cookie. Does not revoke the JWT on the server.`                 | `Clears the accessToken cookie.`                                      |
| `Sends a verification code. No request body is required.`                               | `Sends a verification code.`                                          |
| `Requires a valid session, but does not require email verification or a specific role.` | `Requires a valid session.`                                           |

Apply this cleanup to explanatory prose rather than using a global word replacement. Actual errors such as `User not found` or `Could not send the verification email`, regular expressions, enums, and constraints are part of the contract and must be preserved. Retain useful information such as a code's expiration and single-use behavior.

When the user requests wording cleanup only, limit the diff to the relevant prose: preserve paths, methods, security, status codes, schemas, examples, and literal response values.

## 4. Validate and Deliver

1. Check that every path and method within scope has an operation and that `operationId` values are unique in the combined document.
2. Load the YAML using existing tools. In this project, import `src/docs/openapi.js` to check integration with `swagger-jsdoc` and its `failOnErrors` setting.
3. Check that internal `$ref` values resolve in the combined document; identify external references separately.
4. Run Prettier on the specific file using the project's package manager, for example `pnpm exec prettier src/features/auth/auth.openapi.yaml --check`. Apply formatting only to the task's file when needed.
5. Check that examples match their schemas and that the diff stays within the requested scope.

Validate documentation by calling the OpenAPI parser or assembler directly, avoiding server startup and its database or external service connections. Use validators already installed; if one is unavailable, report the scope of the checks performed rather than adding dependencies on your own initiative. Loading YAML and resolving references checks syntax and integration; it does not replace full specification validation or API testing.

Finish by naming the file created or updated and the checks performed. Briefly mention relevant blockers or discrepancies.
