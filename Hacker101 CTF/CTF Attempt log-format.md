## Date: [Month ,date], 2026

## Time spent: [] hours

## Flags found:

### Techniques Tried

| Time | Technique | URL/Payload | Result |
| --- | --- | --- | --- |
| [fill] | IDOR | /page/3-9, /page/6 | 404 or 403 |
| [fill] | XSS stored | [<img src=x onerror=alert(1)>] | Alert popped, same flag |
| [fill] | XSS variants | <script>, <svg>, <iframe>, markdown links | No new flag |
| [fill] | SQLi | ' in parameters | Found flag #3 |
| [fill] | Parameter tampering | ?admin=true, ?debug=1 | No result |
| [fill] | HTTP method change | GET→POST, DELETE | 400 Bad |
| [fill] | Header manipulation | X-Forwarded-For, X-Originating-IP, etc. | All 403 |
| [fill] | Page creation | Created pages 10, 11 | Found in source, no flag |
| [fill] | Delete attempts | Various methods | 400 Bad |
| [fill] | Hidden endpoints | /edit, /api, /flag, /secret | 404 |
| [fill] | Negative/zero IDs | /page/0, /page/-1 to -4 | 404 |

### What I Learned

- [Your lesson here]

### Hypothesis for Flag #4

- [Your guess here]
