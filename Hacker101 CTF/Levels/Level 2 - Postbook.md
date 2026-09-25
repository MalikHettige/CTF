# Hacker101 CTF — Photo Gallery (Easy, 7 pts)

### Date: August 6, 2026

### Time spent: ~3 hours

### Flags found: 5/5 (enough to unlock private programs)

### Techniques Tried

| Time | Technique | URL/Payload | Result |
| --- | --- | --- | --- |
| Attempt 1 | IDOR | Changed `id` parameter in post URL | Flag #1 found |
| Attempt 2 | IDOR on profile | Changed profile `id` to `b` | Accessed admin account (no new flag — already claimed) |
| Attempt 3 | SQLi | Appended `'` to URL | App sanitized to `%27`, no result |
| Attempt 4 | XSS stored | `<script>alert(1)</script>` in post body | Sanitized — no result |
| Attempt 4 | XSS variants | `<img src=x onerror=alert(1)>`, `<img src=x onerror=alert('XSS')>` | All sanitized |
| Attempt 5 | IDOR on edit page | `/edit?id=1` | Flag #2 found |
| Attempt 6 | Delete-post IDOR | Replaced `page=view.php&id=3` with delete URL in intercept | 200 OK but no deletion, no flag |
| Attempt 7 | Hint-based math | `189 * 5 = 945` → `/index.php?page=view.php&id=945` | Flag #3 found |
| Attempt 8 | Default credentials | Username: `user`, Password: `password` | Flag #4 found |
| Attempt 9 | IDOR on create-post | Sent POST to create page, changed `id=2` (admin's) in Repeater | Flag #5 found in response |

### Flag Breakdown

**Flag #1 — IDOR on post view**
After signing in, opened a post and noticed `id` parameter in URL. Changed the value → flag exposed. Classic IDOR — always check ID parameters first.

**Flag #2 — IDOR on edit page**
XSS failed on my own post. Pivoted: IDORed the edit endpoint to `id=1` (someone else's post) instead. Flag exposed at the edit page of another user's content.

**Flag #3 — Hint math (obscure ID)**
Hint said "189 * 5" = 945. Used the existing post URL parameter format: `/index.php?page=view.php&id=945`. Flag exposed. The post existed but was never linked — enumeration would find it.

**Flag #4 — Default credentials**
Hint: "The person with username 'user' has a very easy password." Tried `user:password` first guess. Flag exposed. Always try `password`, `123456`, `user`, `admin` before moving on.

**Flag #5 — IDOR on create-post (write IDOR)**
Created a post, found the request in HTTP history, sent to Repeater. Changed the `id` parameter to `2` (admin's ID). Response returned `HTTP/2 302 Found` with flag in body.

### What I Learned

- **IDOR isn't only on read endpoints.** Edit and create endpoints are equally vulnerable — always test write operations too.
- **XSS being blocked doesn't mean pivot away.** When XSS fails on your own post, try IDORing to someone else's edit page and injecting there. Different access paths, different sanitisation.
- **HTML encoding clue:** Seeing `&lt;script&gt;` in page source means the app HTML-encodes angle brackets. Standard XSS won't work — need context-specific bypass or a different endpoint.
- **Profile IDOR with non-numeric values:** Changing ID to `b` accessed admin's account. Non-integer values (letters, null bytes, floats) can bypass ID validation that only checks for integer collision.
- **Obscure IDs still exist:** A post at ID 945 was accessible via the standard parameter — just never linked. Enumeration (or hints) finds these. In real hunting, tools like `ffuf` with a number range find these automatically.
- **Default credentials are always worth 30 seconds:** `user:password` is embarrassingly common in production apps, especially internal tools and staging environments.
- **302 in Repeater doesn't mean failure:** The redirect response body often contains the actual content. Always check the raw response body on 302s — don't just follow the redirect.

### Future Hunting Rules

- Check every `id`, `post_id`, `user_id`, `page` parameter on every endpoint — GET, POST, PUT, DELETE
- When XSS is blocked on your content, IDOR to another user's edit page and try there
- On any app with user accounts: try `admin:admin`, `user:password`, `admin:password` before moving to complex attacks
- A 302 response in Repeater — always read the body before following the redirect

---

### Milestone

Completing this CTF provided enough Hacker101 points to become eligible for private bug bounty programs on HackerOne.

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
