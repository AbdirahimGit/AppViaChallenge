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


## 2. What was broken

For each fault you found: where it was, what the symptom was, how you found
it (what you ran or tried, and what you saw), the root cause, and what you
changed.

| # | Where (file) | Symptom | How I found it | Root cause | My fix |
|---|--------------|---------|----------------|------------|--------|
| 1 | package.json |npm install failed|Ran npm install and read the error. It named the file and quoted the text around the failure ("morgan": "^1.10.0", followed by }).|An extra comma found after the 'morgan' dependency JSON doesn't allow extra comma after the last entry and npm needs package.json to be strict JSON|Removed the comma after the morgan entry.|
| 2 |package.json|npm install reported 1 high severity vulnerability|Read the install output, then ran npm audit|moment pinned to exactly 2.29.1 (no ^), a version that was vulnerable, so npm couldn't update it|Upgraded to a non-vulnerable release|
| 3|server.js|npm start crashed with Cannot find module './config.json'|Ran npm start, the error pointed to line 7|Unconditional require of an untracked local file, violating the "no local config file" requirement|It no longer requires config.json and takes its settings from environment variables.|
| 4||         |                |            |        |
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
