# Write-up: [your name]

> Fill in each section below. Bullet points are fine. Clarity beats length.
> If we take your application forward, you'll talk someone from our team
> through this document, so write it as something you could explain without
> notes.

## 1. Getting started

The first error you hit, copied exactly as it appeared:

```
npm ERR! code EJSONPARSE
npm ERR! path /Users/abdirihim/Downloads/appvia-academy-challenge/app/package.json
npm ERR! JSON.parse Unexpected token "}" (0x7D) in JSON at position 297 while parsing near "...rgan\": \"^1.10.0\",\n  }\n}\n"
npm ERR! JSON.parse Failed to parse JSON data.
npm ERR! JSON.parse Note: package.json must be actual JSON, not just JavaScript.

npm ERR! A complete log of this run can be found in:
npm ERR!     /Users/abdirihim/.npm/_logs/2026-10-02T01_54_09_144Z-debug-0.log
```

What you did about it:

I opened the package.json file, i saw on the last dependency there was an extra comma after 'morgan' and then i removed it and checked that every other entry still has its comma. This is because JSON doesn't allow a trailing comma after the last item in an object or array but JavaScript objects do.


The second error i hit:

```
npm install

added 73 packages, and audited 74 packages in 3s

17 packages are looking for funding
  run `npm fund` for details

1 high severity vulnerability

To address all issues, run:
  npm audit fix --force

Run `npm audit` for details.
npm notice 
npm notice New major version of npm available! 9.5.0 -> 12.2.0
npm notice Changelog: https://github.com/npm/cli/releases/tag/v12.2.0
npm notice Run npm install -g npm@12.2.0 to update!
npm notice 

```
What you did about it:
I ran npm audit to get a better understanding of what the severity vulnerability may be and it seemed to be one of the dependencies called 'moment'. I was not sure why so i asked AI what is wrong with its version and it seemed to be running on a vulnerable version. So i used the internet to search for a version that will work and then used npm install moment@^2.31.0 which updated it to be a non-vulnerable version.

The third error i hit:

```
npm start

> taskboard@1.2.0 start
> node server.js

node:internal/modules/cjs/loader:1078
  throw err;
  ^

Error: Cannot find module './config.json'
Require stack:
- /Users/abdirihim/Downloads/appvia-academy-challenge/app/server.js
    at Module._resolveFilename (node:internal/modules/cjs/loader:1075:15)
    at Module._load (node:internal/modules/cjs/loader:920:27)
    at Module.require (node:internal/modules/cjs/loader:1141:19)
    at require (node:internal/modules/cjs/helpers:110:18)
    at Object.<anonymous> (/Users/abdirihim/Downloads/appvia-academy-challenge/app/server.js:7:16)
    at Module._compile (node:internal/modules/cjs/loader:1254:14)
    at Module._extensions..js (node:internal/modules/cjs/loader:1308:10)
    at Module.load (node:internal/modules/cjs/loader:1117:32)
    at Module._load (node:internal/modules/cjs/loader:958:12)
    at Function.executeUserEntryPoint [as runMain] (node:internal/modules/run_main:81:12) {
  code: 'MODULE_NOT_FOUND',
  requireStack: [
    '/Users/abdirihim/Downloads/appvia-academy-challenge/app/server.js'
  ]
}
```
When I ran npm start, the app crashed because server.js tried to load a file called config.json, and that file isn't in the repo. Only an example file is. The original developer must have had the real file on their own laptop, so the app worked for them but not for anyone else. The spec says the app has to start from a fresh clone with no config file.

What i did about it:

I made the app read its settings from environment variables instead, with sensible defaults. The port defaults to 3000 and the maximum text length to 200. The admin token has no default, so it has to be provided. I chose this because there's no file left that can go missing, and it's how apps are normally configured when deployed.

The fourth error i hit:

I was typing in empty characters and integers in the 'What needs doing" text bar on the app, and the spec shows that with these it should come up with certain errors. But it never showed no errors and allowed me to enter those values.

What I done to fix it:

```
for body in '{}' '{"text":""}' '{"text":"   "}' '{"text":123}' '{"text":["a"]}' '{"text":{}}' '{"text":null}'; do
  curl -s -o /dev/null -w "%{http_code}  $body\n" -X POST localhost:3000/api/todos \
    -H 'Content-Type: application/json' -d "$body"
done
400  {}
400  {"text":""}
201  {"text":"   "}
500  {"text":123}
500  {"text":["a"]}
500  {"text":{}}
400  {"text":null}
```
I asked AI how i could test each of those different cases and return its HTTP status code so i can compare it to what the spec expected. It gave me the code above, it sends seven different request bodies to POST /api/todos, one after another, and prints only the HTTP status code for each. It lets you compare what the API returns with what the spec says.

```
app.post('/api/todos', (req, res) => {
  if (!req.body.text) {
    return res.status(400).json({ error: 'text is required' });
  }

const text = req.body.text.trim().slice(0, settings.maxTextLength);
```
! means "not". This line says "if there's no text, reject it." In JavaScript, only a few values count as "nothing": a missing value, null, an empty string "", 0 and false. So an empty or missing text gets a 400, which is correct.

But everything else counts as "something", even if it's the wrong type. A number like 1004 passes. An array like ["a"] passes. An object {} passes. Even a string of just spaces " " passes, because it's not empty, it has spaces in it.

.trim() removes spaces from the start and end of a string. It only exists on strings.

If the text is a number, array or object, it has no .trim(). The code crashes, and the server sends back a 500 error. The spec says this should be a 400, never a 500.

If the text is spaces only, .trim() works and turns it into an empty string "". Nothing stops it from there, so an empty todo gets saved. The empty check already happened on line 1, before the trim, so it was too early to catch it.

So i changed it to this:

```
app.post('/api/todos', (req, res) => {
  const raw = req.body && req.body.text;
  if (typeof raw !== 'string' || raw.trim() === '') {
    return res.status(400).json({ error: 'text must be a non-empty string' });
  }
  // Long text is cut to the limit rather than rejected: the old mobile
  // app relies on this.
  const text = raw.trim().slice(0, settings.maxTextLength);
```

What I changed

Replaced the old check if (!req.body.text) with a stricter one: typeof raw !== 'string' || raw.trim() === ''.
Stored the incoming value in a variable called raw, so the code clearly treats it as unchecked input.
Changed the line that builds text to use raw.trim().slice(...), so it only runs after the checks pass.
Left the rest of the route (creating and saving the todo) unchanged.

Why I changed it

Wrong types caused a 500. A number, array or object passed the old check, then crashed on .trim(). The spec says bad text must get a 400, never a 500.
Blank text was saved. Spaces-only text passed the old check, got trimmed to nothing afterwards, and was saved as an empty todo. The spec says blank text must be rejected.
The old check only asked "is something there?" It never asked "is it a string?" or "is it blank once trimmed?"

Now the status looks like this :

```
for body in '{}' '{"text":""}' '{"text":"   "}' '{"text":123}' '{"text":["a"]}' '{"text":{}}' '{"text":null}'; do
  curl -s -o /dev/null -w "%{http_code}  $body\n" -X POST localhost:3000/api/todos \
    -H 'Content-Type: application/json' -d "$body"
done
400  {}
400  {"text":""}
400  {"text":"   "}
400  {"text":123}
400  {"text":["a"]}
400  {"text":{}}
400  {"text":null}
```

## 2. What was broken

For each fault you found: where it was, what the symptom was, how you found
it (what you ran or tried, and what you saw), the root cause, and what you
changed.

| # | Where (file) | Symptom | How I found it | Root cause | My fix |
|---|--------------|---------|----------------|------------|--------|
| 1 | package.json |npm install failed|Ran npm install and read the error. It named the file and quoted the text around the failure ("morgan": "^1.10.0", followed by }).|An extra comma found after the 'morgan' dependency JSON doesn't allow extra comma after the last entry and npm needs package.json to be strict JSON|Removed the comma after the morgan entry.|
| 2 |package.json|npm install reported 1 high severity vulnerability|Read the install output, then ran npm audit|moment pinned to exactly 2.29.1 (no ^), a version that was vulnerable, so npm couldn't update it|Upgraded to a non-vulnerable release|
| 3|server.js|npm start crashed with Cannot find module './config.json'|Ran npm start, the error pointed to line 7|Unconditional require of an untracked local file, violating the "no local config file" requirement|It no longer requires config.json and takes its settings from environment variables.|
| 4|app/server.js, POST /api/todos|Sending {"text":123}, {"text":["a"]} or {"text":{}} gave a 500 error. Sending {"text":" "} (spaces only) saved an empty todo and returned 201. The spec says all of these should get a 400.|Checking the HTTP source codes for when i was entering integers and arrays and empty strings|he route only checked if the text was falsy (!req.body.text). Numbers, arrays, objects and spaces are all truthy, so they got past it. Numbers, arrays and objects then crashed on .trim(), which gave the 500. Spaces-only text was trimmed to an empty string after the check had already run, so a blank todo was saved.|Replaced the check with typeof raw !== 'string' || raw.trim() === '', which returns 400 if the text isn't a string or is blank after trimming. Only then does the code trim and cut the text to maxTextLength. Long text is still shortened, not rejected, as the spec says. Re-ran the same curl loop and all seven bad bodies returned 400. Normal and long text still returned 201.|
| 5 ||         |                |            |        |

## 3. What I didn't fix

Anything you found but didn't fix, or suspected but couldn't pin down, and
why. Leave this blank if there's nothing.

## 4. Security

Which of the problems were security problems? For each one, what could
someone actually do with it?

## 5. One decision

Pick one fix where you considered more than one approach. What were the
options, and why did you choose the one you did?

## 6. What happened on Friday

- Timeline (times from the log, and what happened):
- How it was possible:
- Did everything users complained about that day have the same cause?
- What should happen now, beyond deploying the fixed code:
- Summary for Taskboard's owner (not technical, 150 words at most):

## 7. My top three improvements

Exactly three, in priority order, with your reasoning for both the choice and
the order.

1.
2.
3.

## 8. Optional: what I built

If you built one of your improvements: what you built, how far you got, and
what you would do next. Leave this blank if you didn't.

## 9. How to run my submission

- App: `cd app && npm install && npm start` (plus anything extra I've added:)
- Report script: `./report.sh <path-prefix> <path-to-log-file>` (written in [language])
- Anything else an engineer needs to know to run or test my work:

## 10. How I worked

Which resources and tools you used (documentation, search, AI assistants,
people, anything else), what you used them for, and how you checked what
they told you.

## 11. Reflections

- The hardest part of this exercise was:
- One thing I learned doing it:
- If I had another day, I would:
