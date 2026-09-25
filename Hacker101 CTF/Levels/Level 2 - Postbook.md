# Hacker101 CTF — Micro-CMS v1

### Date: June 6 – July 26, 2026

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

# Documentations

> 7 points - Easy difficulty
> 

[The level](https://fad4a3d96f05d3c1dbb75fe486895d4f.ctf.hacker101.com)

# 8/6/2026

#### First of all. I should familiarize the app.

**Sign up, create an account,**

1. What URLs are there? (Home, posts, profile, etc.)
- Adds up `/index.php?` to the URL after the first click
1. What happens when I create a public post vs. private post?
2. Can I see other users' posts?
3. What does the post URL look like? (`/post/1`, `/post?id=1`, etc.)

`#1 Attempt`

Try to test IDOR

Steps taken:

1. After signing in, open a post
2. Notice the `id` parameter in the address bar - opportunity to test IDOR
3. Replace the `id` value with an arbitrary number
4. Flag exposed

!image.png

`#2 attempt`

Steps taken:

Try to find an endpoint that shows my account id so I can access someone else's 

1. Click my profile
2. Notice the id is shown in address bar
3. Change it to `b`

Accessed to admin’s account!

!image.png

Unfortunately there's no new flag - the flag there was already claimed on first attempt

`#3 attempt`

Test SQL

Steps taken :

1. Open a page
2. Type single quote `(’)` at the end of URL

Unfortunately the application sanitized it and replaced it with “`%27`”

`#4 attempt`

Try to post an XSS

Steps taken: 

1. Click “write a new post”
2. Paste an XSS : `<script>alert(1)</script>`
3. Click create post 

Unfortunately it didn't work. But got an Idea!

Try with another XSS

Steps taken

1. Do the same with `<script>alert('XSS')</script>`

then `<img src=x onerror=alert(1)>` , then `<img src=x onerror=alert('XSS')>`

#### Unfortunately none of them worked but at least I got a conclusion that the application sanitizes XSS payloads so I won't test XSS further.

`#5 attempt`

Try to do the same but IDOR to another one’s `/page/edit` and then place the XSS there. Since mine didn't work,

1. Click edit post
2. IDOR to `id=1` - which means the edit page of the page which `id` is 1
3. Flag exposed

`#6 attempt`

Try to intercept the `delete-post` request and do it on someone else's 

steps taken:

1. Go to home
2. Turn intercept on
3. Click a post
4. Replace the `page=view.php&id=3` in the starting line with  `page=home.php&message=Post%20deleted!`(its what appears on URL when a post is deleted)
5. Click forward, then turn intercept off

Unfortunately application doesn't let me delete posts. I did the same with repeater method it showed `200OK` but no flags nor post was deleted. 

#### Got another clue :

In the view page source of the post I placed `<script>alert(1)</script>` The XSS there was converted to a random string,

 I saw `&lt;script&gt;/alert(1)&lt;/script&gt;` (I had to type manually it didn't let me paste the exact thing) - This is **HTML encoding,** It's the the app converts `<` to `&lt;` and `>` to `&gt;`

I looked at some of the labs, There is a lab where I can un-sanitize the payload but I think only if the application sanitize it by adding string on the beginning, not mixed up like this. 

`#7 attempt`

Despite feeling ashamed I went to hints.

<aside>
💡

Flag3 -- Not Found
189 * 5

</aside>

So I figured the answer is 945 I 

- Just typed `/945` and nothing happened
- Noted the `index.php?page=view.php&id=` is the parameter to page IDs so I added 945

Flag exposed

`#8 attempt`

The hint saying *"The person with username 'user' has a very easy password..."* 

The common passwords that come to my mind are 

- `password`
- `123456`
- `user`
- `admin`

Tried first one. flag exposed

`#9 attempt`

Modified the `create-page` request by IDOR the `ID` parameter to admin’s 

(i think it works in by intercepting and modifying too)

Steps taken: 

1. Click create post
2. Locate the request in https history
3. Send to repeater
4. replace the value of `id` with 2 - it's admin’s post id number 
5. Click send and notice response shows `HTTP/2 302 Found` 
6. Search “Flag”’
7. Flag exposed

<img width="703" height="114" alt="image" src="https://github.com/user-attachments/assets/cbb837bb-03d5-4693-9cd8-595dc1c42ed4" />


# Completed CTF points that are enough to make me eligible for private programs
