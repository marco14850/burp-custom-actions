# BurpSuite - Custom Actions

- 🇪🇸 [Leer en Español](README.es.md)

This repository contains a collection of custom BurpSuite actions I’ve developed to improve the efficiency of common testing tasks such as session token impersonation, anonymous usage, undocumented request testing, and discovery of common files.

## Script Descriptions

- **Common-Files.bambda**: Checks for common files relative to the current request directory. `/app/index.html?x=1` and `/app/` scan `/app/robots.txt`, etc.; `/app` scans the root directory. The original query is excluded, and the missing-path control uses the same directory.
- **Relevant-Cookies.bambda**: Tests cookie removal, then replaces retained cookie values with `a` (`b` if already `a`). Reports unnecessary, replacement-accepted, value-sensitive or inconclusive cookies. Validates and exports the reduced request with accepted replacement values to a new Repeater tab.
- **Try-Another-Token.bambda**: Re-sends the current request using another user's session token by modifying a specific header or cookie. Useful for testing token impersonation or session fixation.
- **Multi-Auth-Profiles.bambda**: Sends the original request once per authentication profile. Each profile combines any number of headers and cookies, with independent credentials. Reports status, body byte count and a short body preview in Output.
- **Selected-String-Analyzer.bambda**: Checks Base64 by decoding it, binary data and Luhn checksums. Passing Luhn does not establish an issued card. Use Hash Identifier for hashes.
- **DeleteAuth.bambda**: Preserves essential routing and body headers (including Host and Content-Type), removes other headers and all cookies, and preserves the original method, URL and binary body.
- ⚠️ **GET-POST-PUT-PATCH-DELETE.bambda**: Sends the request using different HTTP methods to identify available functionalities.  
  ⚠️ **Warning**: The DELETE method may trigger irreversible changes if accepted by the server.
- **OPTIONS-TRACE-HEAD-CONNECT.bambda**: Sends the request using less common HTTP methods to identify additional attack surfaces.
- **Relevant-Headers.bambda**: Tests removal of nonessential headers and cookies, then replaces required entries with `a` (`b` if already `a`) to distinguish presence requirements from value-sensitive behavior. Preserves protected headers and exports the validated reduced request, including accepted replacements, to a new Repeater tab.
- **HashIdentifier.bambda**: Recognizes formats compatible with MD5, SHA-1, SHA-256, SHA-512 and bcrypt; format alone does not confirm the algorithm.
- **Request-Smuggling-Tester.bambda**: Generates and sends four base probes (CL.TE, TE.CL, H2.CL and H2.TE) for inspection in Logger and further manual testing. Output identifies each variant and reports response status, body size and elapsed time, or transport failures.
- **Web-Cache-Deception.bambda**: Tests static-extension, directory and file-name normalization variants with anonymous controls before and after authenticated priming. Reports repeated private-content exposure and cache evidence without discarding restrictive Cache-Control responses. Does not automatically confirm vulnerabilities.

## Validation and limitations

### Multiple authentication profiles

Import `src/Multi-Auth-Profiles.bambda` and edit its configuration in Burp:

1. Replace the example credentials in `profiles`. Add as many `new AuthProfile(name, headers, cookies)` entries as needed. Each entry sends one request; profiles are not combined with each other.
2. Use `java.util.Map.of("Header-Name", "value", "Another-Header", "value")` for headers and `java.util.Map.of("cookieName", "value")` for cookies. Use `java.util.Map.of()` for an empty map. Cookie values are raw values, without the cookie name or a `Cookie:` prefix; values containing `=` are preserved. For more than ten entries, use `Map.ofEntries(Map.entry(...), ...)` with fully qualified `java.util.Map` names.
3. Add any other original credential header names to `credentialHeaders`, in lowercase. All original cookies and every instance of configured/profile header names are removed before applying each profile. Non-credential headers, method, path, query and binary body are preserved.
4. Set `configured = true` after replacing the examples. Results appear in Output; the action creates no Repeater tabs. Inspect Logger for full exchanges.

Example combined profile:

```java
new AuthProfile("User A", java.util.Map.of(
    "Authorization", "Bearer USER_A_TOKEN",
    "X-CSRF-Token", "USER_A_CSRF"
), java.util.Map.of(
    "session", "USER_A_SESSION",
    "csrf", "USER_A_CSRF"
))
```

Use two empty maps for an anonymous profile. A profile failure does not stop subsequent profiles. Output reports `[Profile name] status=200 | bytes=264659`, followed by a body preview of up to 50 characters (plus `...` when truncated). Line breaks and tabs are replaced with spaces; empty bodies are marked `(empty)`. Status and size are observations, not proof of an authorization flaw. Profile names should not contain secrets.

These profiles configure header/cookie authentication only. Remove credentials embedded in the original query/body separately and disable session handling rules or extensions that restore credentials. Each run repeats the original operation N times, including any server-side effects. All cookies must be explicitly included in each profile, including any non-authentication cookies required by the application.

The Relevant actions require two stable baseline responses. Configure `volatilePattern` for known dynamic fields; keep patterns specific so meaningful differences remain visible. Removals accumulate and the final request is validated again; a global minimum is not guaranteed. Original request bodies are preserved without text conversion.

Relevant Cookies compares HTTP status, Content-Type, Location and normalized body. A changed response after removal indicates a presence requirement in the current reduction; if the replacement preserves the baseline, the exported request uses that replacement. A changed response with the replacement is reported as value-sensitive, not proof of secure cookie validation. Accepting `a` does not establish that every value is accepted. Duplicate names are tested as a group, and consolidation of Cookie headers is checked first. Transport failures retain previous values and are inconclusive. Disable session rules/extensions that restore cookies. Other authentication headers remain unchanged, so results describe the current request context.

Relevant Headers applies the same removal/value replacement checks to nonessential headers, then to individual cookie names. Duplicate header names are grouped case-insensitively; duplicate cookie names are grouped case-sensitively. It preserves `protectedNames`, including Host, Content-Type, Content-Encoding and protocol/framing headers; Cookie is handled separately. Output identifies each entry as Header or Cookie and summarizes removed, replacement-accepted, value-sensitive and inconclusive results. Accepted changes accumulate; the final request and original control must match the baseline before export to `Relevant Headers - reduced`. The same interpretation limits as Relevant Cookies apply.

Web Cache Deception requires GET and retains the original query parameters. Its default configuration tests 53 variants: 42 delimiter/extension combinations, nine directory normalization variants and two file-name variants. Edit `staticDirectories` and `staticFiles` to match the target. These are exploratory paths, not proof that the cache or origin normalizes them as intended. Output identifies each variant; full exchanges remain in Logger and no Repeater tabs are created.

Each variant is requested anonymously before authenticated priming, then anonymously twice. A positive sequence is checked against a fresh unprimed sibling and a fresh authenticated baseline. Content accessible before priming is reported separately; matching sibling content, unstable controls and transport failures are inconclusive. An anonymous pre-control may itself cache a rejection: if authenticated priming does not reproduce the baseline, the action retries once on a fresh URL with authenticated-first ordering and explicitly reports weaker attribution. Default processing can send up to seven requests per variant plus three initial baseline/control requests. No candidates does not exclude deception.

Configure `volatilePattern` to normalize specific dynamic fields. Alternatively, set `sensitivePattern` to a regex that identifies private, user-specific content: the first capture group (or full match if no group exists) is compared in occurrence order, alongside status, Content-Type and Location. An absent/empty match in the baseline is inconclusive; broad patterns can cause false positives. With both patterns empty, the full body must match exactly. If `sensitivePattern` is set, it takes precedence over `volatilePattern`.

The action records Cache-Control instead of excluding `private`, `no-store` or `no-cache` responses; a misconfigured shared cache may ignore restrictions, and `no-cache` permits storage with revalidation. Positive Age or HIT signals from X-Cache, CF-Cache-Status, X-Cache-Status or Cache-Status add supporting evidence, not confirmation. Candidates still require manual validation of shared-cache origin and persistence.

Extend `credentialHeaders` with every application credential header. Defaults remove all instances of Authorization, Proxy-Authorization, Cookie, X-Api-Key, Api-Key, X-Auth-Token, X-Access-Token, X-CSRF-Token and X-XSRF-Token, case-insensitively. Query/body credentials are preserved: remove them separately and disable session rules/extensions that restore credentials. By default, `removeCacheBypassHeaders` removes Cache-Control, Pragma, conditional and Range request headers before all tests. `addCacheBuster` appends a unique `__wcd` query parameter while preserving original parameters; file-name variants are skipped if this option is disabled. Unique paths and query parameters do not guarantee cache isolation, especially after normalization or when the query is excluded from the cache key. These tests can leave private content in a shared cache; use a controlled test account and environment.

Request Smuggling Tester provides starting requests for manual investigation, rather than automatically detecting vulnerabilities. Run it, inspect the four exchanges in Logger, and adapt the probes to the target. The HTTP/1 probes use conflicting Content-Length and Transfer-Encoding framing; the HTTP/2 probes target downgrade behavior. HTTP/2 conversion may normalize or reject the intended anomalies, so inspect the outgoing representation. The embedded `/KITO` requests are intentionally incomplete prefixes for follow-up testing. The probes retain the original service and path, but rebuild the headers and body without the original authentication. Status differences, delays and transport failures alone neither confirm nor exclude smuggling; validate behavior in an isolated test environment.

These actions replay requests: Relevant can repeat operations with side effects. Transport errors do not establish vulnerabilities. Verify final compatibility by importing the actions into your Burp version.

Method variants preserve the original body in GET/POST/PUT/PATCH/DELETE; CONNECT uses an HTTP/1 authority-form request without the original resource headers or body. Common Files excludes responses identical to a random missing-path control, but dynamic wildcard pages can still produce false candidates. Delete Auth keeps only the configurable essentialHeaders allowlist, matched case-insensitively: Host, Content-Type, Content-Length, Transfer-Encoding, Content-Encoding, Connection, TE, Trailer, Expect and HTTP/2 pseudo-headers. All other headers and cookies are removed. Query/body credentials remain, and session rules can restore authentication; inspect Logger for the actual outgoing request.

## Installation

1. Clone the repository:
   ```bash
   git clone https://github.com/marco14850/burp-custom-actions.git
   ```
2. Open BurpSuite and go to: Extensions > Bambda Library.
<p align="center"> 
    <img src="resources/Install-Guide-1.png" alt="Bambda Library location" width="500"/>
</p>

3. Click the Import button and select all files from the /src folder.
<p align="center">
    <img src="resources/Install-Guide-2.png" alt="Importing custom actions" width="500"/> 
</p>

4. In the Repeater tab, go to Custom Actions > Load, and import all the custom action files.

<p align="center">
    <img src="resources/Install-Guide-3.png" alt="Custom Actions in Repeater" width="500"/>
</p>
<p align="center">
    <img src="resources/Install-Guide-4.png" alt="Loading Custom Actions" width="500"/>
</p>

5. Execute the custom actions from any request within the Repeater tab.
<p align="center">
    <img src="resources/Install-Guide-5.png" alt="Running Custom Actions" width="500"/>
</p>

6. If a script requires customization (e.g., Try Another Token), open it in the editor and manually replace the token or cookie values as needed.
<p align="center">
    <img src="resources/Install-Guide-6.png" alt="Editing Custom Actions" width="500"/> 
</p>
