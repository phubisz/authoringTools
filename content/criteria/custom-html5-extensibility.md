---
name: Custom HTML5 Extensibility
type: criteria
tags: [criteria, javascript, html5, custom-interactions, scorm-api, extensibility, games, three.js]
---

# Custom HTML5 Extensibility

## Definition

Custom HTML5 Extensibility measures a tool's ability to **embed arbitrary HTML5 content — custom JavaScript apps, three.js scenes, WebGL games, React components — and establish bidirectional communication between that embedded content and the course shell**, so that data produced by the custom interaction (game scores, completion status, learner choices) can:

1. Be read/written to course variables
2. Drive course navigation (jump to a slide, unlock next step)
3. Be submitted to an LMS via SCORM/xAPI

This is the "ceiling test" for power users: how far can you extend the tool beyond its native interactions?

**Why it matters**: No authoring tool can anticipate every interaction type. Organizations building immersive simulations, 3D product demos, gamified assessments, or AI-driven branching scenarios need a channel to plug custom code into the course tracking layer without rebuilding the entire SCORM shell from scratch.

---

## The Reference Architecture (Storyline)

Storyline 360 is the benchmark. The pattern it established:

```
┌─────────────────────────────────────┐
│         Storyline Player            │
│  ┌──────────────────────────┐       │
│  │   Web Object (iframe)    │       │
│  │   ← your three.js game → │       │
│  │                          │       │
│  │  window.parent           │       │
│  │    .GetPlayer()          │       │
│  │    .SetVar('score', 95)  │       │
│  └──────────────────────────┘       │
│                                     │
│  Storyline Variable 'score' = 95    │
│       ↓ (slide trigger)             │
│  SCORM cmi.score.raw = 95           │
└─────────────────────────────────────┘
```

### Storyline JavaScript API

```javascript
// From a Web Object (iframe) or Execute JavaScript trigger:

// Get the Storyline player reference
var player = window.parent.GetPlayer();

// Read a Storyline variable
var score = player.GetVar('GameScore');

// Write a Storyline variable (SCORM-tracked if configured)
player.SetVar('GameScore', 95);
player.SetVar('GameComplete', true);

// Navigate programmatically
player.SetVar('GoToNext', true); // combined with a trigger listening for this

// Access SCORM API directly (bypasses Storyline, advanced)
function findSCORMAPI(win) {
  if (win.API != null) return win.API;        // SCORM 1.2
  if (win.API_1484_11 != null) return win.API_1484_11; // SCORM 2004
  if (win.parent && win.parent !== win) return findSCORMAPI(win.parent);
  return null;
}
var scormAPI = findSCORMAPI(window);
scormAPI.LMSSetValue('cmi.score.raw', '95');
scormAPI.LMSSetValue('cmi.completion_status', 'completed');
scormAPI.LMSCommit('');
```

**Storyline score: ⭐⭐⭐⭐⭐ (5/5)**
- Documented, stable JavaScript API (`GetPlayer`, `GetVar`, `SetVar`)
- Web Object iframe embeds any HTML5 content
- Two-way: iframe → Storyline variables → SCORM
- Direct SCORM API access also possible from JavaScript
- Large community of developers who have solved every edge case
- postMessage pattern available for cross-origin iframes

---

## Rankings

| Tool | Score | API Quality | Mechanism |
|------|-------|-------------|-----------|
| [[tools/articulate-360\|Articulate Storyline 360]] | ⭐⭐⭐⭐⭐ (5) | Documented, stable | `GetPlayer()` / `GetVar` / `SetVar` + Web Object |
| [[tools/lectora\|Lectora]] | ⭐⭐⭐⭐ (4) | Functional, less polished | Reserved Variables + JS bridge + HTML extensions |
| [[tools/dominknow\|dominKnow ONE]] | ⭐⭐⭐⭐ (4) | Documented Content API | `parent.contentApi.*` — clean, purpose-built |
| [[tools/mindsmith\|Mindsmith]] | ❓ (TBD) | **Unknown** — Enterprise only | Custom Code tile (zip upload) confirmed; communication API not publicly documented |
| [[tools/ispring-suite\|iSpring Suite]] | ⭐⭐ (2) | Minimal | Web Object iframe; postMessage possible; no official widget API |
| [[tools/elucidat\|Elucidat]] | ⭐⭐ (2) | Limited | JavaScript widget support; API undocumented publicly |
| [[tools/parta-io\|Parta.io]] | ⭐ (1) | None | Custom CSS only; no iframe widget API |
| [[tools/easygenerator\|Easygenerator]] | ⭐ (1) | None | Not designed for custom code embedding |
| [[tools/gomo-learning\|Gomo Learning]] | ⭐ (1) | None | No custom widget API documented |

---

## Mindsmith Custom Code Tile — Known + Unknown

> **Status: Partially confirmed. Needs hands-on testing.**

Mindsmith has a **Custom Code tile** (Enterprise plan only) that accepts a zip upload — the same architectural pattern as dominKnow's HTML Widget. The zip is almost certainly unpacked and embedded as an iframe.

**What is confirmed:**
- Custom Code tile exists as a block type in the editor
- Requires Enterprise plan
- Accepts a zip upload (observed in product UI)
- The zip likely follows the same convention as dominKnow: a `index.html` launcher + assets

**What is NOT publicly documented:**
- Whether Mindsmith exposes any JavaScript API to the embedded iframe (equivalent of `GetPlayer()` or `parent.contentApi.*`)
- Whether variables can be written from inside the custom code to the lesson
- Whether completion/score can be pushed to Mindsmith's SCORM layer from within the iframe
- Whether postMessage is a supported/documented communication pattern

**The critical test**: from inside your three.js game (the iframe), can you call anything on the parent that affects SCORM reporting?

```javascript
// Test these from inside your embedded zip's index.html:

// Does Mindsmith expose a player object?
console.log(window.parent.GetPlayer);          // Storyline-style?
console.log(window.parent.mindsmithAPI);       // proprietary?
console.log(window.parent.contentApi);         // dominKnow-style?

// Can you reach the SCORM API directly?
// (Mindsmith uses remote-hosted SCORM — the LMS SCORM API
//  may or may not be accessible from inside the custom code iframe)
```

**Likely outcome**: Given Mindsmith's AI-native positioning and the Enterprise gating of this feature, the Custom Code tile is most likely a **display-only embed** (iframe with your content rendered inside the lesson) without a bidirectional API. But this must be verified — if they've built even a simple `parent.mindsmith.setScore(n)` API, it changes the picture significantly.

> **Action item**: Contact Mindsmith enterprise sales or test on an Enterprise trial. The presence of the zip upload suggests the capability exists; the API question determines whether it's useful for your scenario.

---

## dominKnow Content API (Second-Best Confirmed Option)

dominKnow has a **purpose-built, documented Content API** for HTML Widgets. This is the most explicit competitor to Storyline's approach.

### HTML Widget Structure
Upload a `.zip` of your HTML5 app (three.js game, etc.). dominKnow unpacks it and embeds it as an iframe. Identify `index.html` as the launcher.

### Content API Functions

```javascript
// From within your HTML Widget iframe:

// Variables (read/write course variables)
parent.contentApi.getData('variableName');          // get
parent.contentApi.setData('variableName', value);   // set

// Learner info
parent.contentApi.getUserID();        // learner LMS ID
parent.contentApi.getStudentName();   // learner display name

// Navigation
parent.contentApi.contentGoNext();    // advance to next page

// Completion (maps to SCORM completion)
// setCompletion() is available per API docs
```

### Bidirectional Flow in dominKnow
```
three.js game ends
  → parent.contentApi.setData('GameScore', score)
  → dominKnow variable 'GameScore' = score
  → dominKnow conditional: if GameScore > 70 → mark complete
  → SCORM cmi.completion_status = 'completed'
  → LMS receives completion + score
```

**vs. Storyline**: dominKnow's API is cleaner and more explicit (purpose-built for widgets). Storyline's `GetPlayer()` approach requires knowing undocumented internals that Articulate never officially published as a formal API spec (though widely used).

---

## Lectora JavaScript Integration

Lectora supports custom JavaScript with access to **Reserved Variables** — special variables that map directly to SCORM cmi data elements.

```javascript
// Lectora Reserved Variables (map to SCORM):
// VAR_SCORE          → cmi.score.raw
// VAR_MASTERY_SCORE  → cmi.score.min / max
// VAR_LESSON_STATUS  → cmi.completion_status

// In an HTML extension or external HTML object:
// Access Lectora variables through the Lectora API bridge
// (syntax varies between Lectora versions)

// Direct SCORM API access also works (same findSCORMAPI pattern)
```

Lectora also supports embedding external HTML pages and can embed HTML Widgets in a manner similar to dominKnow, though the API is less cleanly documented. JavaScript access to LMS Reserved Variables is the key integration point.

---

## The Nuclear Option: Pure SCORM Shell

For maximum control — especially for a three.js game or complex WebGL simulation — skip the authoring tool entirely for the custom piece:

```javascript
// scorm-shell/index.html (your three.js game IS the course)
import { Scorm12API } from 'scorm-again'; // npm package

const scorm = new Scorm12API({});
scorm.loadFromJSON({});

// Start the SCORM session
scorm.lmsInitialize();

// Game runs...
// On game end:
scorm.lmsSetValue('cmi.score.raw', playerScore.toString());
scorm.lmsSetValue('cmi.lesson_status', 'passed');
scorm.lmsCommit('');
scorm.lmsFinish('');
```

Or use **pipwerks SCORM wrapper** (the community standard):
```javascript
pipwerks.SCORM.version = "1.2";
pipwerks.SCORM.init();
pipwerks.SCORM.set("cmi.core.score.raw", score);
pipwerks.SCORM.set("cmi.core.lesson_status", "passed");
pipwerks.SCORM.save();
pipwerks.SCORM.quit();
```

This approach is best when the custom experience IS the course (full-screen game, VR scene). Embed your existing SCORM course as an iframe within the shell for pre/post content, or use xAPI for richer data.

---

## xAPI as the Better Channel for Game Data

For game-like interactions, **xAPI (Tin Can) is architecturally superior to SCORM** for reporting complex data:

```javascript
// xAPI statement from a game:
var statement = {
  actor: { name: "Learner Name", mbox: "mailto:learner@org.com" },
  verb: { id: "http://adlnet.gov/expapi/verbs/completed", display: { "en-US": "completed" } },
  object: { id: "http://example.com/game/level-3", definition: { name: { "en-US": "Level 3" } } },
  result: {
    score: { raw: 95, min: 0, max: 100, scaled: 0.95 },
    completion: true,
    success: true,
    extensions: {
      "http://example.com/game/kills": 42,
      "http://example.com/game/time_seconds": 187
    }
  }
};
```

xAPI can send arbitrary data (kill counts, paths taken, time per level) — SCORM can only send a score and pass/fail. Most modern LMS platforms support both.

---

## Recommended Approach by Scenario

| Scenario | Recommended Path |
|----------|-----------------|
| three.js game inside existing Storyline course | Storyline Web Object + `GetPlayer().SetVar()` |
| three.js game inside dominKnow course | HTML Widget + `parent.contentApi.setData()` |
| three.js game as standalone SCORM package | Pure HTML5 + pipwerks or scorm-again |
| Complex game data (many metrics) | xAPI statements from within the game |
| Game + pre/post course content | SCORM shell wrapping both; or separate SCORM objects in LMS |
| Need to display game score on a later slide | Storyline variable → Result Slide; or dominKnow variable → condition |

---

## The postMessage Pattern (Cross-Origin Fallback)

When the iframe is cross-origin or sandboxed, direct parent API access is blocked. Use postMessage:

```javascript
// In your three.js game (iframe):
window.parent.postMessage({
  type: 'GAME_COMPLETE',
  score: 95,
  level: 3,
  timeSeconds: 187
}, '*');

// In Storyline (Execute JavaScript trigger on a slide):
window.addEventListener('message', function(event) {
  if (event.data.type === 'GAME_COMPLETE') {
    var player = GetPlayer();
    player.SetVar('GameScore', event.data.score);
    player.SetVar('GameLevel', event.data.level);
  }
});

// In dominKnow (Execute JavaScript panel):
window.addEventListener('message', function(event) {
  if (event.data.type === 'GAME_COMPLETE') {
    contentApi.setData('GameScore', event.data.score);
  }
});
```

---

## Summary Assessment

For the exact scenario described (three.js game, bidirectional communication, display score on later slide, submit to SCORM):

1. **Storyline 360**: The community benchmark. Widest documentation, most examples, deepest community knowledge. `GetPlayer().SetVar()` is the standard pattern.
2. **dominKnow ONE**: Best-documented competitor API. `parent.contentApi.*` is cleaner by design. Viable alternative especially if already in the dominKnow ecosystem.
3. **Lectora**: Capable but less ergonomic. Reserved Variables are the right bridge to SCORM, but the JavaScript integration feels less purpose-built.
4. **Pure HTML5 SCORM shell**: The right choice when the game IS the course — no authoring tool overhead, full control, best performance.
5. **xAPI**: Upgrade from SCORM for any game/simulation that produces richer data than a single score.

The tools not listed above (Mindsmith, Parta.io, Easygenerator, Gomo, iSpring) are **not viable paths for this use case**. They are authoring-first platforms, not extensible development environments.
