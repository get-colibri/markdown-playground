import BlogPostHeader from '../src/components/blogPostHeader'
import YouTubeComponent from '../src/components/youtube'
import BigQuote from '../src/components/bigQuote'
import LoomComponent from '../src/components/loom'
import VideoComponent from '../src/components/video'

<BlogPostHeader 
  shortTitle="Legit" 
  longTitle="Legit - Git for AI Agents" 
  type="Project" 
  description="Productizing infrastructure: How we built a Git-style versioned SDK for AI agents, turned complex infrastructure into a usable developer product, shaped the experience from API to UI, and tested a real market hypothesis around safe AI collaboration." 
  imageSrc="/covers/legit-cover.png"
  imageAlt="Legit Schematic"
  imageWidth="1648"
  imageHeight="720"
  tableFirstColumn="Website"
  tableFirstColumnValue='legitcontrol.com'
  tableFirstColumnLink='https://legitcontrol.com'
  tableSecondColumn="Role"
  tableSecondColumnValue="Co-Founder & Product Engineer"
  tableThirdColumn="Year"
  tableThirdColumnValue="2025/2026"
/>

## 1. Context & Team

I met [Martin](https://www.linkedin.com/in/martin-lysk/) and [Jannes](https://www.linkedin.com/in/jannes-blobel-b00bb9142/) back at Opral. We were working on [inlang](https://inlang.com), a version control system for localization workflows. Basically applying Git concepts outside of traditional software engineering. That's where I first got pulled into thinking about structured change, diffs, and what *"safe edits"* actually mean in non-code environments.

Later at Decipad, I kept encountering a similar pattern: This time with AI. Agents that write or modify data feel magical for a few minutes. Then they quietly corrupt state, overwrite information, and leave you debugging behavior instead of reviewing structured change.

That déjà vu was hard to ignore.

We realized we were circling the same idea again: **If AI is going to touch real data, it needs proper versioning.** Not just logs. Not just “undo.” Something structural.

### Team

- [Martin (CTO)](https://www.linkedin.com/in/martin-lysk/) handled storage model and Git internals
- [Jannes](https://www.linkedin.com/in/jannes-blobel-b00bb9142/) took operations, positioning and distribution
- **Me**, product direction, frontend architecture and DX

---

## 2. The Problem

Today, most AI systems generate suggestions, not changes. They draft emails, rewrite paragraphs, summarize meetings. But a human still applies the result. We copy, paste, and manually commit the outcome back into the real system. Technically, AI could already act as a true collaborator. Thousands of automation frameworks and agent protocols exist. Yet adoption of autonomous workflows remains limited.

**Why?** Because real collaboration brings a cascade of problems that today’s infrastructure simply isn’t built to handle.

Real collaboration demands more than good models:

- **Accountability**: Every change must be auditable. (who did what, when, and why)
- **Safety**: Changes can be harmful. We need sandboxing, review, and rollback.
- **Boundaries**: Agents must respect privacy, security, and compliance constraints.
- **Coordination**: Multiple agents can conflict. Race conditions, duplicate work, and competing goals are inevitable.

Until these issues are solved, AI will remain on the sidelines -> **suggesting, not doing.**

---

## 3. The Core Approach

Our answer was simple in principle: treat agent changes like code changes.

Instead of allowing agents to directly mutate application state, we introduced a versioned layer in between. Every agent action became a structured change that could be inspected, reviewed, and reverted.

At the center of this was a Git-inspired mental model.

Agent actions behave like commits:
- Every change is recorded  
- Diffs can be inspected before and after execution
- Rollback is built in
- Execution happens in controlled environments, not directly on production state

<br/>
![Suggested changes](/projects/legit-experiment.png)
<br/>

Under the hood, this meant separating *“proposed change”* from *“applied state.”* Agents operate against a versioned workspace, not raw data. Nothing becomes canonical without passing through that layer.

**Roles:** Martin led the storage architecture and Git internals that made this possible. My focus was making sure these primitives could be understood and used without requiring developers to think in low-level version control mechanics.

---

## 4. Making It Usable (My Focus)

The underlying system was powerful. But power without clarity just creates hesitation. 
My responsibility was turning low-level versioning primitives into something developers could actually adopt and trust.

### API & Abstraction Design

Let's be honest: Git is still a mystery to most people. Advanced Git features? Forget about it. So just wrapping the CLI in an SDK doesn't magically make it easier. You actually have to make it easier to understand and use.

The key insight was the [file system](https://www.legitcontrol.com/docs/concepts/filesystem-api) design. **Most of the time, version control should be invisible.** It just silently intercepts reads and writes in the background. The only thing missing was a way to do meta operations, like "create a branch" or "show me the history", without making developers think about version control at all.

So we built an API that operates on virtual files *(Credits to Martin for making it possible)*. You just do normal Node.js-style file system operations, and everything else happens automatically. **Bonus:** since it's just a file system API, AI can work with it too. No weird custom protocols needed.

```typescript
// Creates new commit
await legitFs.promises.writeFile(
  '/my-document.txt',         // path
  'Updated content',          // content
)

// Access history
const historyJson = await legitFs.promises.readFile(
  '/.legit/history'
)

// Creates new branch
await legitFs.promises.mkdir('/.legit/branches/feature-xyz')
```

That was progress, but React developers needed something even simpler. They're a crucial group, and they don't want to deal with Node-style APIs. They want a hook that gives them reactive props. Done. I built that [abstraction](https://www.legitcontrol.com/docs/react-wrapper), and honestly? It ended up feeling more like a state library than a version control system. Which is probably a good sign.

```typescript
import { useLegitFile } from '@legit-sdk/react';
 
function Editor() {
  const { data, setData, history, loading } = useLegitFile('/document.txt');
  
  // data: current file content (string | null)
  // setData: save and commit function
  // history: array of commits
  // loading: loading state
}
```

### Frontend & Developer Experience

Infrastructure products live or die by their developer experience. ([Good read by Lee Robinson](https://leerob.com/dx)) Every roadblock is a missed opportunity. This was my main focus.

We started with the website and developer documentation. I wrote the docs with a simple goal: get developers from *"what is this?"* to *"I'm building something"* in **under five minutes**. No fluff, just clear concepts and working code examples. Turns out most people skip the marketing copy and go straight to docs anyway, so I made sure those docs actually delivered.

Then we went after early adoption through integrations. I built an [Assistant UI integration](https://www.legitcontrol.com/docs/integrations) ([demo here](https://www.linkedin.com/feed/update/urn:li:activity:7399391259376746496/)) that let developers using that library plug into our ecosystem with basically zero effort. The strategic play was getting links in their docs, which meant high-intent search traffic would find us. It's a playbook that worked really well at [inlang](https://inlang.com), so we applied it here.

Finally, we made onboarding frictionless with framework-specific starters that install via npx commands. *(Jannes suggested this, great idea!)* This also forced us to test more thoroughly, because if the starter didn't work, we'd hear about it immediately. We included the starters in our testing process to make sure they work as expected.

Most improvements came from watching where people got stuck. We spent a lot of time in Discord threads and GitHub issues, turning confusion into clarity.

#### We stayed close to our early community and got good first signals:
<div>
  <BigQuote text="That’s why I find Legit so interesting. It is a Git-compatible Version Control System where each branch is a folder in your filesystem. You can just cd into it & start working." author="Loris Sigrist" link="https://www.sigrist.dev/branching-in-opencode-with-legit" />
  <BigQuote text="I genuinely think Legit is addressing a critical pain point for developers building with AI agents. It brings Git-like guarantees (the ability to track, inspect, and revert changes) to the application level, where AI agents actually operate." author="Udoh Jeremiah - Engineer at Heartbeat AI" link="https://www.linkedin.com/feed/update/urn:li:activity:7410367682455261184" />
  <BigQuote text="A version‑controlled AI agent is a brilliant leap toward a future where intelligent code writes, edits, and safeguards itself without human blind spots. By making every shift explicit and reversible, we turn the 10 % uncertainty into a learning feed for the next generation of self‑improving systems. This isn’t just a safeguard-it’s a glimpse of the autonomous development pipelines that will power the coming intelligent era." author="Jonas P - CTO at Diedai Health" link="https://www.linkedin.com/feed/update/urn:li:activity:7398652066023407616?commentUrn=urn%3Ali%3Acomment%3A%28activity%3A7398652066023407616%2C7398693968814731265%29&dashCommentUrn=urn%3Ali%3Afsd_comment%3A%287398693968814731265%2Curn%3Ali%3Aactivity%3A7398652066023407616%29" />
</div>
    
---

## 5. Product Experiments

### Legit Chat

What is the most holy access to version control? **Your local files.** We built a chat assistant that could work with your local files via MCPs, and every conversation thread automatically created a new sandbox using the legit SDK. The idea was: let the AI go wild on files in a safe space, then review the changes before applying them to your actual documents.

I focused on the frontend and productization, with feedback from Jannes. Martin built the backend. It worked! You could see diffs, review changes, apply them to originals only when you were happy. It even worked in the browser and desktop apps, which was cool. The SDK made it all interoperable.


<LoomComponent id="f98c3b12a70747e88bc42130b9851ca8" />

### Legit Assistant UI

Interface for visualizing changes and rollback flows. Made agent actions transparent in an Assistant UI app. The aim was to have a plug in solution that makes the sandboxing effortless.

<VideoComponent url="/covers/git-for-assistant-ui.mp4" />

### WebMCP

Calendar demo built by Alex Nahas. Integrated legit SDK so agents could make changes in sandboxed branches, schedule meetings, create events, then review and accept/reject proposals. Had a time travel function to navigate timeline changes. Supported multi-agent scenarios with separate branches. Technically impressive, too complex for non-technical users.

<LoomComponent id="c0e34ef13fa940b4a1c7464ef1405398" />

### Prototype for Hoppscotch

PR to sync Hoppscotch collections with Git repos. Collections stored as JSON in Git, using legit SDK for simple file system interaction. Work happens in a collections branch on top of feature branches, update collections, commit, merge back when done. Keeps feature branches clean with one commit. Pretty seamless, not a lot of work to implement.

<LoomComponent id="9aa9edd9ad1b4f7191cc57038f7c9d1c" />

### Crypto Bro tried to steal our identity

Someone tried to steal our identity. Not a product experiment, but it happened.

---

## 6. Artifacts

- Legit Website ([legitcontrol.com](https://legitcontrol.com))
- Legit React Wrapper ([@legit-sdk/react](https://www.legitcontrol.com/docs/react-wrapper))
- Legit Assistant UI Integration ([@legit-sdk/assistant-ui](https://github.com/Legit-Control/monorepo/tree/main/packages/sdk-assistant-ui))
- Legit WebMCP Server ([webMCP-exploration](https://github.com/Legit-Control/webMCP-exploration))
- Legit Prototype for Hoppscotch ([Hoppscotch Legit Integration](https://www.loom.com/share/9aa9edd9ad1b4f7191cc57038f7c9d1c))
- Legit Concept Pages ([legitcontrol.com/concepts](https://www.legitcontrol.com/docs/concepts/problem))
- Legit Documentation ([legitcontrol.com/docs](https://www.legitcontrol.com/docs))

---

## 7. Market Timing & Personal Runway

Our assumption was that AI-native teams would adopt versioned agent infrastructure early as they moved toward autonomous workflows.

What we learned is that infrastructure adoption follows ecosystem gravity. SDKs like this scale fastest when embedded inside an existing platform or developer surface area. Building that leverage from scratch requires longer timelines and deeper capital reserves.

Given those dynamics, I decided to step away after reaching the end of my personal runway.

The underlying thesis remains compelling. As agents gain more autonomy, structural safeguards around change management will move from “nice to have” to mandatory.

---

## 8. Key Learnings

- **Powerful infrastructure does not automatically create urgency.** Even when developers agreed the problem was real, adoption required clear, immediate use cases.
- **SDK adoption depends heavily on ecosystem gravity.** Our integrations and starter templates drove significantly more engagement than standalone SDK messaging.
- **Trust in AI systems is less about model quality and more about change visibility.** Diffs, history inspection, and explicit rollback flows mattered more than “smartness.”
- **Abstraction is subtraction.** Every primitive we removed reduced hesitation. The final React hook felt more like a state library than version control, which increased approachability dramatically.

---

## Closing Reflection

Legit deepened my conviction that the future of AI is not just about smarter models, but about structured change. Building this project clarified where I do my best work: translating complex infrastructure into tools developers can trust and actually use.

I stepped away due to personal runway constraints. The team continues pushing the vision forward, and I’m proud of what we built together.

The core idea remains simple and powerful: **If AI is going to act, its actions must be inspectable, reversible, and accountable.**
