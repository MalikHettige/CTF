# Hacker101 CTF — Micro-CMS v1

### Date: June 6 and July 26, 2026
### Time spent: ~6 hours across multiple days
### Flags found: 4/4


### Techniques Tried

| Time | Technique | URL/Payload | Result |
| --- | --- | --- | --- |
| Day 1 | IDOR | /page/1–9, /edit/1–9 | Found Flag #1 at /edit/6 |
| Day 1 | XSS stored | `<script>alert(1)</script>` in page body | Alert popped, Flag #2 found in page source |
| Day 1 | SQLi | Single quote `'` appended to /edit/2 → `/edit/2'` | Flag #3 found (SQL error triggered) |
| Day 1 | XSS variants | `<img src=x onerror=alert(1)>`, `<svg>`, `<iframe>` | No new flag |
| Day 1 | Parameter tampering | `?admin=true`, `?debug=1` | 200 OK but no effect |
| Day 1 | HTTP method change | GET→POST, DELETE on /page/10 | 400 or 404 |
| Day 1 | Header manipulation | X-Forwarded-For, X-Originating-IP | 403 |
| Day 1 | ID injection in body | POST /page/create with `id=3` in body | 302, server ignored parameter |
| Day 1 | Page reference in body | `parent=6`, `reference=6`, `page_id=6` | 302, ignored |
| Day 1 | Content-Type change | application/json on /page/6 | 403 |
| Day 2 | Path traversal in body | `../../../etc/passwd`, `{{7*7}}`, `${7*7}` | No result |
| Day 2 | Non-integer ID | `/edit/2a`, `/edit/2%00`, `/edit/2.0`, `/edit/02` | Various 400/404 |
| Day 2 | Delete endpoints | `/page/10/delete`, DELETE /page/10 | 404 |
| Day 2 | Restore endpoints | `/trash`, `/deleted`, `/restore/6`, `/api/restore/6` | 404 |
| Day 2 | Negative IDs | /page/0, /page/-1 to /page/-4 | 404 |
| Day 2 | Hidden endpoints | /api, /flag, /secret, /robots.txt | 404 |
| Jul 26 | XSS in title field | `<a href="/page/6">Click me</a>` in title | Flag #4 found on homepage render |

### Flag Breakdown

**Flag #1 — IDOR**
Enumerate `/edit/{id}` manually. Page 6 existed but was not linked. Different server responses matter: 403 = exists but blocked, more interesting than 404.

**Flag #2 — Stored XSS**
Paste `<script>alert(1)</script>` in page body → save → view page source. Flag was in the source, not in the alert.

**Flag #3 — SQL Error**
Append `'` to the edit URL: `/edit/2'`. Response changed — SQL error triggered. Flag in error output.

**Flag #4 — XSS in Title (Stored, Different Rendering Context)**
Put `<a href="/page/6">Click me</a>` in the **title** field. The homepage renders titles differently — as part of a hyperlink — creating a different XSS context. Flag appeared on the homepage after navigating back.

Key insight from the hint: "Sometimes a given input will affect more than one page" + "the bug doesn't exist in the most obvious place this input is shown." The title renders harmlessly on the edit page but dangerously on the homepage.

### What I Learned

- **USER INPUT ALMOST NEVER HAS ONLY ONE DESTINATION.** When you inject something, trace where it flows — edit page, homepage, API, logs — each is a different rendering context with different rules.
- 403 on IDOR is more interesting than 404. It means the resource exists but you're blocked — worth investigating further.
- XSS in a title field looks harmless until you check the homepage, profile pages, notification feeds — anywhere the title is reused.
- SQL injection doesn't need to produce a working attack. An error message alone can contain a flag or useful information.
- Decoding reference for future hunts:
  - `%3C%3E%22` → URL decode
  - `SGVsbG8=` → Base64 decode
  - `&lt;script&gt;` → HTML decode
  - `48656c6c6f` → Hex decode

### Future Hunting Rule

When a hint or observation suggests data is reused — stop focusing on payloads and start tracing where the data flows through the application. Different rendering contexts create different vulnerabilities.

### Hypothesis for Flag #4 (before finding it)

Title field input flows to at least two places: the edit page and the homepage listing. The homepage likely renders titles without the same sanitisation as the edit page. Injecting a link or script into the title and navigating to the homepage should trigger it in a different context.

**Confirmed correct.**

---

# Documentation

**Challenge:** A simple content management system with pages you can create and edit.

#### First flag

**Steps taken:**

1. Opened the app and noticed an Edit button on pages
2. Saw `edit/2` in the URL pattern
3. Recognized `edit/{id}` pattern — ID-based access
4. Tested nearby IDs manually (1 through 6)
5. Found flag hidden at `edit/6` — a page that existed but was not linked anywhere
6. Submitted flag — accepted

**Lesson:** When you see IDs in URLs (`/page/2`, `/edit/2`, `?id=2`), always enumerate nearby values. Different server responses reveal information — 403 means the page exists but you are blocked, which is more interesting than a 404.

---

- **Decoding Reference — use this when a value looks unreadable:**
    - Starts with `%3C%3E%22` → URL decode
    - Looks like `SGVsbG8=` → Base64 decode
    - Looks like `&lt;script&gt;` → HTML decode
    - Looks like `48656c6c6f` → Hex decode
    
    Rule: Ask yourself "does this look readable after decoding?" before moving on.
    

#### 2nd Flag

**Steps taken:**

1. Create or edit a page 
2. Paste an XSS `<script>alert(1)</script>`
3. Right click and view page source
4. Found flag 
5. Submitted flag — accepted

**Lesson:**  Paste XSS where possible, execute, look everywhere. View page source. All the time try to have the complete display of the application such as URL, inputs, page source.

---

#### 3rd Flag - Flag X – SQL Error Discovery

**Steps taken:**

1. Found a parameter (end of URL)
2. Added a single quote (').
3. Response changed / error appeared.
4. Found flag
5. Retrieved flag.
- A reusable technique:
    
    Find input
    ↓
    Modify input slightly (')
    ↓
    Compare response
    ↓
    Look for unusual behavior
    

Lesson: Use small input changes to test assumptions. Different responses reveal how the application processes data. 

---

#### 3rd Flag -

Micro-CMS v1 Flag #2 Attempt Log — June 6, 2026
Time spent: ~4 hours across 2 days
Techniques tried:

> `icro-CMS v1 Flag #1 Attempts`
> 

Tried SQL, XSS, IDOR, Hidden endpoints (adding `/robots.txt` to URL) but none of them worked. Then I,

- Changed the **HTTP method change**
- I send a random request to repeater, added a header/parameter `admin='true'` and it response was 200OK which is surprising but when I try to access 6page with IDOR it still forbids

> Create page: `Micro-CMS v1 Flag #2 Attempts`
> 

<aside>
💡

> **New fact: we can add `id=3` to perform IDOR and it can be on the body or JSON in a request**
> 

- i added a para on bottom and added `id=3` and starting line was that post create. when i click send page created but it is post 10 and doesn't have data, everything i create is post 10th page

- Meaning Adding `id=3` to the POST body doesn't change the created page's ID
- The server ignores my `id` parameter and auto-assigns ID 10
- But the server still processes the request (302) — it just doesn't use my `id`
</aside>

> **Your exact next test:** `Micro-CMS v1 Flag #3 Attempt`
> 

When i create a new page (page 11), ill intercept the POST to `/page/create`. The body is `title=xxx&body=yyy`.

**Try adding a parameter that references another page:**

- `title=xxx&body=yyy&parent=6`
- `title=xxx&body=yyy&reference=6`
- `title=xxx&body=yyy&page_id=6`

Summery:

- IDOR: /page/3-9, /edit/3-9 — 403 or 404
- XSS: stored, reflected, variants — no flag
- SQLi: single quote — found different flag (#3)
- Method change: POST, PUT, DELETE — 400 or 403
- Admin headers: X-Forwarded-For, etc. — 403
- Parameter injection: id=3 in create body — 302 but ignored
- Page reference in body: /page/6 — rendered content but no flag
- Accept: application/json on /page/6 — 403
- /edit/6 loads empty form but save returns 403
Result: Flag not found
Next: Try /api endpoints, raw markdown, or fresh instance

### **One last attempt of the day**

**Hypothesis: Soft-deleted pages can be restored or overwritten**

**Step 1: Check for delete functionality**

- In the pages (10, 11, etc.)
- Is there a "Delete" button or link?
- Is there a `/page/10/delete` or `/delete/10` endpoint?
- Sending DELETE request to `/page/10` in Repeater

<aside>
💡

Shows `HTTP/2 404 Not Found` : Didnt worked

</aside>

**Step 2: If delete exists, delete pages**

- Delete page 10, then 11, then any others I created

**Step 3: Try to recreate with specific ID**

- Create a new page
- Intercept POST to `/page/create`
- Body: `title=test&body=test`
- Try adding: `id=6` or `page_id=6` or `overwrite=6`
- If server reuses IDs or accepts your ID, I might overwrite page 6

<aside>
💡

Became a Get request and /page/24 : didnt work

</aside>

**Step 4: Check for trash/restore endpoints**

- Try `/trash`, `/deleted`, `/restore`, `/admin`
- Try `/page/6/restore` or `/api/restore/6`

**Step 5: If none work, negative testing**

- Create page with `id=-1`, `id=0`, `id=99999`
- See if server behaves differently (error messages reveal logic)

8/6/2026

> `icro-CMS v1 Flag #3 Attempts`
> 

Steps:

1. **Go to `/edit/2`**
2. **Intercept the save request**
3. **The body is probably:** `title=xxx&body=yyy`
4. **Try changing the page ID in the URL or body to non-integer values:**
    - `/edit/2'` (with quote) — Found out a flag ( has been claimed before)
        
        > Skipped this for a while
        > 
    - `/edit/2a`
    - `/edit/2%00` (null byte)
    - `/edit/2.0` (float)
    - `/edit/02` (leading zero)
    - `/edit/2/1` (path injection)
    - `/edit/2%2f` (encoded slash)
5. **Or try in the POST body:**
    - `title=xxx&body=yyy&id=2'`
    - `title=xxx&body=yyy&id[]=2` (array)
    - `title=xxx&body=yyy&id=2&body=zzz` (duplicate parameter)
6. **Try changing Content-Type:**
    - Send as `application/json` instead of `application/x-www-form-urlencoded`
    - Body: `{"title":"xxx","body":"yyy","id":"2'"}`
7. **Try path injection in the page body:**
    - Create a page with body: `../../../etc/passwd`
    - See if it reads files
    - Or body: `{{7*7}}` — template injection
    - Or body: `${7*7}` — SSTI
  
      7. **Try path injection in the page body:**
    - Create a page with body: `../../../etc/passwd`
    - See if it reads files
    - Or body: `{{7*7}}` — template injection
    - Or body: `${7*7}` — SSTI
    
    ---
    
    # 26/07/2026
    
    ### 2nd flag - last one
    
    Hints : 
    
    - Sometimes a given input will affect more than one page
    - The bug you are looking for doesn't exist in the most obvious place this input is shown

“…input will affect more than one page” could mean one endpoint would lead to several results, like one command or click would open another page. This must be a mistake or but or technical issue if i make it crash with 2 page loads. 

# USER INPUT ALMOST NEVER HAS ONLY ONE DESTINATION!!

Attempt steps

1. Create or open a page
2. In the title box, insert this HTML script `<a href="/page/6">Click me</a>`
3. Click ‘save’

Ignore the title being bald, the script does not work in a title section anyway. Consider the fact that the homepage renders the title differently because it became part of a hyperlink

1. Click ‘home’
2. Flag found

   <img width="721" height="313" alt="image" src="https://github.com/user-attachments/assets/dcec646f-2c7a-4516-b2eb-c06bb34fbb3e" />


#### Future Hunting Rule

> When a hint or observation suggests that data is reused, instead of focusing on payloads — start tracing where the data flows through the application. Different rendering contexts often create different vulnerabilities.
> 

---
