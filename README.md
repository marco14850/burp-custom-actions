# BurpSuite - Custom Actions

- 🇪🇸 [Leer en Español](README.es.md)

This repository contains a collection of custom BurpSuite actions I’ve developed to improve the efficiency of common testing tasks such as session token impersonation, anonymous usage, undocumented request testing, and discovery of common files.

## Script Descriptions

- **Common-Files.bambda**: Checks for the presence of common files such as `robots.txt` on the current domain.
- **Relevant-Cookies.bambda**: Sequentially reduces cookies, compares response status, body and selected headers, and validates the final request before exporting to Repeater.
- **Try-Another-Token.bambda**: Re-sends the current request using another user's session token by modifying a specific header or cookie. Useful for testing token impersonation or session fixation.
- **Selected-String-Analyzer.bambda**: Checks Base64 by decoding it, binary data and Luhn checksums. Passing Luhn does not establish an issued card. Use Hash Identifier for hashes.
- **DeleteAuth.bambda**: Sends the request without authentication tokens or cookies to verify whether access is possible without authentication.
- ⚠️ **GET-POST-PUT-PATCH-DELETE.bambda**: Sends the request using different HTTP methods to identify available functionalities.  
  ⚠️ **Warning**: The DELETE method may trigger irreversible changes if accepted by the server.
- **OPTIONS-TRACE-HEAD-CONNECT.bambda**: Sends the request using less common HTTP methods to identify additional attack surfaces.
- **Relevant-Headers.bambda**: Sequentially reduces headers and cookies, protects protocol headers case-insensitively, and validates the result before exporting to Repeater.
- **HashIdentifier.bambda**: Recognizes formats compatible with MD5, SHA-1, SHA-256, SHA-512 and bcrypt; format alone does not confirm the algorithm.
- **Request-Smuggling-Tester.bambda**: Logs status, size and elapsed time for CL.TE, TE.CL, H2.CL and H2.TE probes; transport failures are inconclusive and manual confirmation is required.
- **Web-Cache-Deception.bambda**: Finds candidates by comparing authenticated and anonymous content at unique paths, excluding non-cacheable responses. Does not automatically confirm vulnerabilities.

## Validation and limitations

The Relevant actions require two stable baseline responses. Configure `volatilePattern` for known dynamic fields; keep patterns specific so meaningful differences remain visible. Removals accumulate and the final request is validated again; a global minimum is not guaranteed. Original request bodies are preserved without text conversion.

Web Cache Deception requires a GET request and preserves its query. Anonymous controls remove only `Authorization` and `Cookie`: remove other credentials and disable session rules that reintroduce them. Matching content needs manual review to distinguish caching from authorization failures. Smuggling probes neither confirm nor exclude vulnerabilities and need validation in an isolated test environment.

These actions replay requests: Relevant can repeat operations with side effects. Transport errors do not establish vulnerabilities. Verify final compatibility by importing the actions into your Burp version.

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