# Scoped Access Token: Tested Behavior

**Student:** Miliano Blarasin  
**Course:** CIT 386: Cloud Network Design — Fall A 2026  
**Storage resource:** `$web/miliano-blarasin/index.html`
**Evidence status:** Tested from an unauthenticated browser before and after expiration. Never paste an unredacted SAS URL, signature, storage account key, or screenshot containing one into this file, the repository, the course shell, or an AI tool.

## 1. Initial read-only token parameters

I recorded these values from the Azure portal before generating the SAS. The storage account uses virtual blob folders, so the portal generated a blob-scoped service SAS for my known `index.html` object instead of a directory-scoped SAS.

| Parameter | Value used |
|---|---|
| Storage account | `cit386module04` |
| Container | `$web` |
| Scope | Blob `miliano-blarasin/index.html` |
| Signing method | Service SAS signed with the storage account key |
| Signing key selected | Key 1 |
| Stored access policy | None |
| Allowed permission | Read only (`r`) |
| Permissions not granted | List, write, create, add, delete, tag, move, and execute |
| Allowed protocol | HTTPS only |
| Start shown in the portal | October 25, 2026 at 8:33:08 PM, Eastern Time selector |
| Expiry shown in the portal | October 26, 2026 at 4:48:08 AM, Eastern Time selector |
| Allowed IP addresses | Not set |
| Signed resource | Blob (`sr=b`) |

Redacted test-URL format:

```text
https://cit386module04.blob.core.windows.net/$web/miliano-blarasin/index.html?<NONSECRET-SAS-PARAMETERS>&sig=REDACTED
```

The value after `sig=` is the credential and is intentionally omitted. Before committing, search the document for `sig=` and verify that the only value following it is `REDACTED`.

## 2. What the token permits

The token is intended to permit an anonymous recipient to read an existing blob in my own `miliano-blarasin/` directory over HTTPS. It does not sign the recipient into Azure and does not grant portal access.

### Permitted-operation evidence

- Test browser: Google Chrome Incognito, not signed into the Azure portal
- Test time (UTC): `2026-10-26 00:51 UTC` (screenshot captured at 8:51 PM Eastern on October 25)
- Requested object: `miliano-blarasin/index.html`
- Result shown by the browser: The résumé rendered and displayed **Miliano Blarasin**, the professional title, contact line, summary, and experience section.
- HTTP status: Not recorded; the browser rendered the complete HTML page.
- Conclusion: The existing `index.html` content opened successfully in an unauthenticated browser. A separate cropped address-bar screenshot confirms that the request used the `cit386module04.blob.core.windows.net` endpoint rather than the already-public static-website endpoint; the query string and credential were not exposed.

Together, the screenshots establish an unauthenticated Incognito browser, rendered content, and the Blob service endpoint without exposing the SAS query string or signature.

## 3. What the token forbids

The SAS deliberately omits list and all modification permissions. It therefore should not allow the recipient to enumerate the container, discover other names, upload a blob, overwrite the résumé, or delete content. The refused test must target only my own storage scope; I did not test another student's directory.

### Refused-operation evidence

I attempted to list the container with the same token by requesting the container endpoint with the list operation. The signature is redacted below.

```text
https://cit386module04.blob.core.windows.net/$web?restype=container&comp=list&<NONSECRET-SAS-PARAMETERS>&sig=REDACTED
```

- Test time (UTC): `2026-10-26 00:54 UTC` (screenshot captured at 8:54 PM Eastern on October 25)
- HTTP status: Not visible in the browser screenshot
- Azure error code: `PublicAccessNotPermitted`
- Error message: `Public access is not permitted on this storage account.`
- Interpretation: The attempted container operation was not authorized by the read-only blob SAS, and Azure did not allow the request to fall back to anonymous public access. The token that opened my known `index.html` object therefore did not authorize enumeration of the `$web` container.

The exact error code belongs here only after the request has actually been made. A predicted code is not evidence.

## 4. From where the token works

The SAS is bearer access: within its scope and time window, it can be used from any client that possesses the complete URL because I did not set an IP restriction. Requiring HTTPS protects the URL while it travels across the network, but it does not prevent a recipient from forwarding it.

My successful test came from a Windows computer using Google Chrome Incognito. The browser was not signed into the Azure portal, demonstrating that authorization came from the SAS rather than my Azure session.

## 5. Between which two moments the token works

The initial token used the portal window shown in Section 1. The second token supplied the precise UTC evidence: it was valid from **`2026-10-26 00:40:27 UTC`** through **`2026-10-26 00:57:27 UTC`**. Before the start time it would not authorize a request, and the test below proves that Azure rejected it after expiry.

### Short-expiry token parameters

| Parameter | Value used |
|---|---|
| Scope | Blob `miliano-blarasin/index.html` (the portal did not expose a directory SAS for the virtual folder) |
| Permission | Read only (`r`) |
| Protocol | HTTPS only |
| Start time (UTC) | `2026-10-26 00:40:27 UTC` (`8:40 PM` Eastern) |
| Expiry time (UTC) | `2026-10-26 00:57:27 UTC` (`8:57 PM` Eastern) |
| Other restrictions | No IP restriction; HTTPS only |

### Before-and-after-expiry evidence

| Attempt | Test time (UTC) | Result | HTTP status | Exact Azure error code |
|---|---|---|---|---|
| Before expiry | `2026-10-26 00:57 UTC` (8:57 PM Eastern) | The résumé rendered successfully in the private browser | `Not recorded` | `Not applicable` |
| After expiry | `2026-10-26 00:58 UTC` (8:58 PM Eastern) | Azure refused the same request | `Not recorded` | `AuthenticationFailed` |

After the second attempt, Azure returned `AuthenticationFailed` and the nonsecret detail `Signature not valid in the specified time frame: Start [Mon, 26 Oct 2026 00:40:27 GMT] - Expiry [Mon, 26 Oct 2026 00:57:27 GMT]`. This directly identifies the validity window and shows that the 00:58 UTC request occurred after expiration. I did not regenerate the token between the two attempts.

## 6. If a recruiter needed six months

I would not simply extend this short token without changing the control around it. The tested token was a service SAS signed with the account key, so a six-month expiry is technically possible, but I would keep HTTPS only, read only, and the single-blob scope and preferably associate the SAS with a stored access policy so it could be revoked without rotating the account key. Better still, I would use Microsoft Entra identity-based access when the recipient can authenticate. A six-month bearer URL has a much larger exposure window: it can be forwarded, copied from browser history, captured in logs, or recovered from an old message and remain useful for months. Shorter, replaceable access reduces the damage from the same leak.

## 7. Why the storage account key is the wrong handoff

The storage account key is not scoped to this résumé or directory. Handing it over could let a recipient authenticate broadly to the storage account and could expose, overwrite, or delete unrelated content depending on how it was used. Rotating a shared account key would also affect every legitimate workflow using that key. The SAS is better even for a trusted person because it applies least privilege: it grants only the required read operation, only to the intended resource, only over HTTPS, and only for a defined period. Trust does not remove the need to limit the impact of mistakes, compromised devices, or accidental forwarding.

## Final safety check before commit

- [x] Both tested tokens targeted only `miliano-blarasin/index.html`.
- [x] Only read permission was selected.
- [x] HTTPS only was selected.
- [x] The successful test used a browser not signed into Azure.
- [x] The refused list request was actually attempted and its exact error code recorded.
- [x] The short token was tested before and after its recorded expiry.
- [x] Every shown URL ends with `sig=REDACTED`; no real signature or account key is present.

The work was completed after the original assignment deadline. The initial SAS was limited to less than one day, but the instructor must decide whether that late validity window satisfies the course-term requirement.

