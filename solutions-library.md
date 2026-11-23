# Reusable Solutions Library

## Existing entries

## 2026-11-17 — LaunchProof AI additions

### 1. Chain-aware address validation — Verified

```js
function validAddress(chain, address) {
 if (chain === "solana") return /^[1-9A-HJ-NP-Za-km-z]{32,44}$/.test(address);
 return /^0x[a-fA-F0-9]{40}$/.test(address);
}
```

**QA note:** Live re-audit confirmed basic Solana and Base address-format validation works. This validates format only, not token existence or ownership.

### 2. Report schema and claim-status validation — Verified

```js
function validReport(report) {
 return report && report.token && Array.isArray(report.signals) && Array.isArray(report.claims) &&
 report.claims.every((claim) => ALLOWED_STATUS.has(claim.status));
}
```

**QA note:** Code audit found response-schema validation and an allowed claim-status set. It does not replace provider-data correctness checks.

### 3. Complete client API request and response validation — Verified

```js
async function requestReport(input) {
 var response = await fetch(API_URL, {
 method: "POST",
 headers: {"Content-Type": "application/json"},
 body: JSON.stringify(input)
 });
 if (!response.ok) throw new Error("API request failed");
 var data = await response.json();
 if (!data || !data.token || !Array.isArray(data.signals) || !Array.isArray(data.claims)) {
 throw new Error("Invalid report");
 }
 return data;
}
```

**QA note:** Code audit found a complete request body, content type, non-OK handling, JSON parsing, and required-field validation. Live API success was not verified because no Vercel deployment exists.

### 4. Accessible duplicate-submit guard with deterministic fallback — Verified

```js
if (busy) return;
busy = true;
button.disabled = true;
status.textContent = "Checking available evidence…";
try {
 latestReport = await requestReport(input);
 status.textContent = "Report ready.";
} catch (error) {
 latestReport = fallback(input);
 status.textContent = "The API was unavailable. A rules-only report is ready.";
} finally {
 render(latestReport);
 busy = false;
 button.disabled = false;
}
```

**QA note:** Live re-audit confirmed fallback behavior, status messaging, disabled submit control, and report rendering. It does not verify server-backed API behavior.

### 5. Server-side provider request with timeout and safe fallback — Unverified

```js
async function providerExplanation(report) {
 const name = process.env.AI_PROVIDER;
 const key = name === "gemini" ? process.env.GEMINI_API_KEY : name === "groq" ? process.env.GROQ_API_KEY : "";
 if (!key) return report;
const prompt = "Return JSON only with one string field summary. Summarize this evidence without advice, invented facts, prices, identities, or scam claims: " +
 JSON.stringify({signals: report.signals, claims: report.claims});
 const controller = new AbortController();
 const timer = setTimeout(() => controller.abort(), 8000);
 try {
 let response;
 if (name === "gemini") {
 response = await fetch("https://generativelanguage.googleapis.com/v1beta/models/gemini-2.0-flash:generateContent?key=" + encodeURIComponent(key), {
 method: "POST",
 headers: {"Content-Type": "application/json"},
 body: JSON.stringify({contents: [{parts: [{text: prompt}]}]}),
 signal: controller.signal
 });
 } else if (name === "groq") {
 response = await fetch("https://api.groq.com/openai/v1/chat/completions", {
 method: "POST",
 headers: {"Content-Type": "application/json", "Authorization": "Bearer " + key},
 body: JSON.stringify({model: process.env.GROQ_MODEL || "llama-3.1-8b-instant", temperature: 0, response_format: {type: "json_object"}, messages: [{role: "user", content: prompt}]}),
 signal: controller.signal
 });
 } else {
 return report;
 }
 if (!response.ok) return report;
 const data = await response.json();
 const text = name === "gemini" ? data.candidates?.[0]?.content?.parts?.[0]?.text : data.choices?.[0]?.message?.content;
 const parsed = JSON.parse(text || "{}");
 if (typeof parsed.summary === "string" && parsed.summary.length <= 900) report.summary = parsed.summary;
 report.provider = name;
 return report;
 } catch (error) {
 return report;
 } finally {
 clearTimeout(timer);
 }
}
```

**QA caveat:** Source audit found provider-specific request bodies, authorization, timeout, parsing, and fallback handling. The live re-audit did not test a deployed provider endpoint, so this remains Unverified for production runtime behavior.

## 2026-11-17 — TrackProof additions

### 1. Complete client API request and response validation — Verified

```js
async function requestReport(input) {
 var response = await fetch(API_URL, {
 method: "POST",
 headers: { "Content-Type": "application/json" },
 body: JSON.stringify(input)
 });
 if (!response.ok) throw new Error("API request failed");
 var data = await response.json();
 if (!validReport(data)) throw new Error("Invalid report");
 return data;
}
```

**QA note:** Live source audit found a complete JSON request body, content type, non-OK handling, JSON parsing, and report-schema validation. The live Vercel API was not deployed, so endpoint success remains unverified.

### 2. Accessible duplicate-submit guard with deterministic fallback — Verified

```js
form.addEventListener("submit", async function (event) {
 event.preventDefault();
 if (busy) return;
 busy = true;
 submitButton.disabled = true;
 status.textContent = "Checking campaign structure…";
 var input = readInput();
 var local = fallback(input);
 render(local);
 try {
 var remote = await requestReport(input);
 render(remote);
 status.textContent = "Audit complete.";
 } catch (error) {
 status.textContent = "API unavailable. A rules-only audit is ready.";
 } finally {
 busy = false;
 submitButton.disabled = false;
 }
});
```

**QA note:** Live source audit found the duplicate-submit guard, disabled control, live status messaging, local rules-first rendering, and deterministic API-unavailable message. Real-browser regression remains outstanding.

### 3. Report schema validation — Verified

```js
function validReport(report) {
 return report && typeof report.finalUrl === "string" &&
 ["ready", "review", "blocked"].indexOf(report.status) !== -1 &&
 Array.isArray(report.issues);
}
```

**QA note:** Code audit found explicit status allow-listing, final URL type validation, and issues-array validation before remote rendering. Provider correctness is outside this schema check.

### 4. Provider request timeout and safe fallback — Unverified

```js
const controller = new AbortController();
const timer = setTimeout(() => controller.abort(), 8000);

try {
 let response;
 if (provider === "gemini") {
 response = await fetch("https://generativelanguage.googleapis.com/v1beta/models/gemini-2.0-flash:generateContent?key=" + encodeURIComponent(key), {
 method: "POST",
 headers: {"Content-Type": "application/json"},
 body: JSON.stringify({contents: [{parts: [{text: prompt}]}]}),
 signal: controller.signal
 });
 } else if (provider === "groq") {
 response = await fetch("https://api.groq.com/openai/v1/chat/completions", {
 method: "POST",
 headers: {"Content-Type": "application/json", Authorization: "Bearer " + key},
 body: JSON.stringify({
 model: "llama-3.1-8b-instant",
 temperature: 0,
 max_tokens: 180,
 response_format: {type: "json_object"},
 messages: [{role: "user", content: prompt}]
 }),
 signal: controller.signal
 });
 } else {
 return base;
 }
 if (!response.ok) return base;
} catch (error) {
 return base;
} finally {
 clearTimeout(timer);
}
```

**QA caveat:** Source audit found provider-specific bodies, authorization, timeout, and safe fallback. No live provider/API deployment was available, so runtime behavior is Unverified.

### 5. Sensitive-value and UTM normalization audit — Verified

```js
function hasSensitive(value) {
 return /[\\w.+-]+@[\\w-]+\\.[\\w.-]+/.test(value) ||
 /(?:\\+?\\d[\\d\\s().-]{7,}\\d)/.test(value) ||
 /(?:api[_-]?key|token|secret|password)[=:_-]/i.test(value) ||
 /^[A-Za-z0-9_-]{28,}$/.test(value);
}
```

**QA note:** Source audit found explicit email, phone-like, credential-label, and long-token checks applied to UTM fields. This is heuristic warning/blocking logic, not a complete privacy detector.

## 2026-11-17 — ClientShield additions

### 1. Response schema validation — Unverified

```javascript
function validReport(report) {
 return report && Number.isFinite(report.score) && report.score >= 0 && report.score <= 100 &&
 ["Low","Moderate","High","Critical"].indexOf(report.level) !== -1 &&
 typeof report.summary === "string" && Array.isArray(report.flags) &&
 Array.isArray(report.questions) && typeof report.reply === "string";
}
```

**QA caveat:** Code audit found explicit score bounds, level allow-listing, summary type validation, and array checks. The live API was not deployed or runtime-tested, so this remains Unverified.

### 2. Complete client API request and response validation — Unverified

```javascript
async function requestReport(input) {
 var response = await fetch(window.API_URL, {method:"POST",headers:{"Content-Type":"application/json"},body:JSON.stringify(input)});
 if (!response.ok) throw new Error("API request failed");
 var data = await response.json();
 if (!validReport(data)) throw new Error("Invalid report");
 return data;
}
```

**QA caveat:** Source contains a complete JSON request body, content type, non-OK handling, JSON parsing, and schema validation. Live endpoint behavior was not tested because no runtime deployment exists.

### 3. Deterministic fallback with evidence-preserving AI merge — Unverified

```javascript
function cleanReport(report, base) {
 if (!validReport(report)) return base;
 report.score = Math.max(base.score, Math.min(100, Math.round(report.score)));
 report.level = report.score >= 75 ? "Critical" : report.score >= 50 ? "High" : report.score >= 25 ? "Moderate" : "Low";
 report.flags = base.flags;
 report.questions = report.questions.filter((item) => typeof item === "string").slice(0, 5);
 report.reply = report.reply.slice(0, 1200);
 report.summary = report.summary.slice(0, 900);
 return report;
}
```

**QA caveat:** Code audit found that provider output cannot lower the deterministic baseline score and that baseline flags are retained. Provider runtime and malformed-output behavior remain unverified.

### 4. Client IP extraction for lightweight rate limiting — Unverified

```javascript
function clientIp(req) { return String(req.headers["x-forwarded-for"] || req.socket?.remoteAddress || "unknown").split(",")[0].trim(); }
```

**QA caveat:** Source audit found proxy-header parsing used by the in-memory limiter. It is not durable across serverless instances and must not be treated as production-grade abuse prevention.

## 2026-11-18 — StreamGap additions

### 1. Output schema validation with bounded fields — Unverified

```js
function validOutput(item, input) {
 return item && typeof item.opening === "string" && item.opening.length <= 500 &&
 Array.isArray(item.prompts) && item.prompts.length === 3 &&
 item.prompts.every(function (x) { return typeof x === "string" && x.length <= 300; }) &&
 item.poll && typeof item.poll.question === "string" &&
 Array.isArray(item.poll.options) && item.poll.options.length >= 2 && item.poll.options.length <= 4 &&
 item.poll.options.every(function (x) { return typeof x === "string" && x.length <= 100; }) &&
 item.mini_game && typeof item.mini_game.name === "string" &&
 typeof item.mini_game.rules === "string" &&
 Number.isFinite(item.mini_game.duration_seconds) &&
 item.mini_game.duration_seconds <= input.duration &&
 typeof item.transition_back === "string" && item.transition_back.length <= 500 &&
 typeof item.chat_command_copy === "string" && item.chat_command_copy.length <= 500 &&
 typeof item.safety_note === "string" && item.safety_note.length <= 500;
}
```

**QA caveat:** Code audit found explicit array cardinality, type, length, and duration bounds. The live provider/API was not deployed or tested, so this is Unverified.

### 2. Complete client API request and response validation — Unverified

```js
async function requestOutput(input) {
 var response = await fetch(API_URL, {
 method: "POST",
 headers: {"Content-Type": "application/json"},
 body: JSON.stringify(input)
 });
 if (!response.ok) throw new Error("API request failed");
 var data = await response.json();
 if (!validOutput(data)) throw new Error("Invalid output");
 return data;
}
```

**QA caveat:** Source audit found the complete JSON request body, content type, non-OK handling, JSON parsing, and schema validation. The Vercel API was not deployed or live-tested.

### 3. Accessible duplicate-submit guard with deterministic fallback — Unverified

```js
if (busy) return;
busy = true;
button.disabled = true;
status.textContent = "Building a useful gap segment…";
var input = readInput();
var local = fallback(input);
render(local);
try {
 var remote = await requestOutput(input);
 render(remote);
 status.textContent = "Segment ready.";
} catch (error) {
 status.textContent = "The API was unavailable. A curated fallback is ready.";
} finally {
 busy = false;
 button.disabled = false;
}
```

**QA caveat:** Source audit found the duplicate-submit guard, disabled control, live status, immediate local fallback, and cleanup. A real deployed/browser acceptance test remains outstanding.

### 4. Provider timeout and environment-only API key handling — Unverified

```js
const controller = new AbortController();
const timer = setTimeout(function () { controller.abort(); }, 8000);

try {
 let response;
 if (provider === "gemini") {
 response = await fetch("https://generativelanguage.googleapis.com/v1beta/models/gemini-2.0-flash:generateContent?key=" + encodeURIComponent(key), {
 method: "POST",
 headers: {"Content-Type": "application/json"},
 body: JSON.stringify({contents: [{parts: [{text: prompt}]}]}),
 signal: controller.signal
 });
 } else if (provider === "groq") {
 response = await fetch("https://api.groq.com/openai/v1/chat/completions", {
 method: "POST",
 headers: {"Content-Type": "application/json", "Authorization": "Bearer " + key},
 body: JSON.stringify({
 model: "llama-3.1-8b-instant",
 temperature: 0.4,
 max_tokens: 700,
 response_format: {type: "json_object"},
 messages: [{role: "user", content: prompt}]
 }),
 signal: controller.signal
 });
 }
 if (!response.ok) return null;
} catch (error) {
 return null;
} finally {
 clearTimeout(timer);
}
```

**QA caveat:** Source audit found provider allow-listing, environment-only keys, provider-specific request bodies, authorization, timeout, and null fallback. No live provider/API deployment was available, so runtime behavior remains Unverified.

## 2026-11-18 — RaidReceipt ABI additions

### 1. Output schema validation with bounded fields — Unverified

```javascript
function validRemote(data) {
 return data && typeof data.title === "string" && data.title.length <= 40 &&
 typeof data.verdict === "string" && data.verdict.length <= 220 &&
 typeof data.squadMessage === "string" && data.squadMessage.length <= 220;
}
```

**QA caveat:** Source audit found explicit type and length bounds for all three optional AI output fields. The Vercel API was not deployed or live-tested, so this remains Unverified.

### 2. Versioned encoded share URL — Unverified

```javascript
function encodedUrl(result) {
 var data = {
 v: 1, map: result.input.map, mode: result.input.mode,
 loadoutCost: result.input.loadoutCost, extractedLoot: result.input.extractedLoot,
 ammoCost: result.input.ammoCost, consumableCost: result.input.consumableCost,
 entryCost: result.input.entryCost, survived: result.input.survived,
 kills: result.input.kills, assists: result.input.assists, revives: result.input.revives,
 tone: result.input.tone
 };
 return SITE_URL + "/?d=" + encodeURIComponent(btoa(JSON.stringify(data)));
}
```

**QA caveat:** Source audit found a version marker and encoded user-entered calculation fields for project-path-safe sharing. Browser reload and privacy behavior still require manual verification.

### 3. Deterministic rules-first receipt with optional AI fallback — Unverified

```javascript
var local = localResult(input);
render(local);
try {
 var remote = await requestAI(input, local.calc);
 local.copy.title = remote.title;
 local.copy.verdict = remote.verdict;
 render(local);
} catch (error) {
 errorBox.textContent = "Rules-only receipt ready. Optional AI was unavailable.";
}
```

**QA caveat:** Source audit found immediate deterministic rendering, optional remote enhancement, and a user-visible fallback. No production API or browser acceptance run was available.

### 4. Client IP rate window — Unverified

```javascript
var previous = hits.get(ip) || [];
previous = previous.filter(function (time) { return now - time < 60000; });
if (previous.length >= 20) {
 res.status(429).json({ error: "Rate limit exceeded" });
 return;
}
previous.push(now);
hits.set(ip, previous);
```

**QA caveat:** Source audit found a 20-request/60-second in-memory limiter. It is best-effort only and is not durable across serverless instances.

## 2026-11-18 — AlbumClub Kit additions

### 1. Album discussion-kit schema validation — Unverified

```js
function validKit(item, input) {
 return item && typeof item.overview === "string" && item.overview.length <= 700 &&
 Array.isArray(item.questions) && item.questions.length === 10 &&
 item.questions.every(function (x) { return typeof x === "string" && x.length <= 300 && !/[<>]/.test(x); }) &&
 Array.isArray(item.prompts) && item.prompts.length === input.tracks.length &&
 item.prompts.every(function (x) { return x && typeof x.track === "string" && typeof x.prompt === "string" && x.prompt.length <= 300 && !/[<>]/.test(x.prompt); }) &&
 item.poll && typeof item.poll.question === "string" && Array.isArray(item.poll.options) &&
 item.poll.options.length >= 2 && item.poll.options.length <= 4 &&
 Array.isArray(item.agenda) && item.agenda.length >= 3 && item.agenda.length <= 5 &&
 typeof item.rating === "string";
}
```

**QA caveat:** Code audit found bounded fields, exact question count, track-prompt cardinality, poll bounds, agenda bounds, and basic angle-bracket rejection. The live API was not deployed or runtime-tested, so this remains Unverified.

### 2. Input normalization with allow-lists — Unverified

```js
function normalize(input) {
 const tracks = Array.isArray(input.tracks) ? input.tracks.map(function (item) {
 return clean(item, 100);
 }).filter(Boolean).slice(0, 15) : [];
 return {
 album: clean(input.album, 100),
 artist: clean(input.artist, 100),
 tracks: tracks,
 context: ["Friends", "Classroom", "Podcast", "Fan group", "Solo reflection"].includes(input.context) ? input.context : "Friends",
 depth: ["Casual", "Thoughtful", "Deep dive"].includes(input.depth) ? input.depth : "Thoughtful",
 favorite: clean(input.favorite, 100),
 least: clean(input.least, 100),
 impression: clean(input.impression, 500),
 spoilers: Boolean(input.spoilers)
 };
}
```

**QA caveat:** Code audit found per-field cleaning, track limits, and allow-lists for context/depth. The deployed endpoint and malformed-body cases were not live-tested, so this remains Unverified.

### 3. Versioned encoded share URL — Unverified

```js
function encodedUrl(input, kit) {
 var data = {v: 1, input: input, kit: kit};
 return SITE_URL + "?d=" + encodeURIComponent(btoa(unescape(encodeURIComponent(JSON.stringify(data)))));
}
```

**QA caveat:** Code audit found a version marker and encoded input/output for project-path-safe sharing. Browser reload, URL-size behavior, and privacy implications require manual verification.

### 4. Deterministic rules-first kit with optional provider enhancement — Unverified

```js
var local = fallback(input);
render(local, input);
try {
 var remote = await requestKit(input);
 render(remote, input);
 status.textContent = "Kit ready. Edit any prompt before sharing.";
} catch (error) {
 status.textContent = "The API was unavailable. A rules-only kit is ready.";
}
```

**QA caveat:** Source audit found immediate fallback rendering, optional remote enhancement, user-visible fallback messaging, and duplicate-submit protection. No deployed API or real-browser acceptance run was available.

### 5. Client IP rate window — Unverified

```js
var previous = (hits.get(ip) || []).filter(function (time) { return now - time < 60000; });
if (previous.length >= 12) {
 res.status(429).json({error: "Rate limit exceeded"});
 return;
}
previous.push(now);
hits.set(ip, previous);
```

**QA caveat:** Source audit found a 12-request/60-second in-memory limiter. It is best-effort only and is not durable across serverless instances.

## 2026-11-23 — PromiseMatch additions

### 1. PromiseMatch report schema validation — Unverified

```js
function validReport(report) {
  var d = report && report.dimensions;
  return report && Number.isFinite(report.overall_score) &&
    report.overall_score >= 0 && report.overall_score <= 100 &&
    ["low", "medium", "high"].indexOf(String(report.confidence).toLowerCase()) >= 0 &&
    d && ["promise_alignment", "offer_alignment", "audience_alignment",
      "cta_alignment", "clarity", "trust_continuity"].every(function (key) {
        return Number.isFinite(d[key]) && d[key] >= 0 && d[key] <= 100;
      }) &&
    Array.isArray(report.shared_promises) &&
    Array.isArray(report.missing_confirmations) &&
    Array.isArray(report.contradictions) &&
    Array.isArray(report.claim_warnings) &&
    Array.isArray(report.priority_fixes) &&
    Array.isArray(report.revised_hero_variants);
}
```

**QA caveat:** Code/live re-audit found explicit score bounds, confidence allow-listing, six dimension bounds, and required array checks. The live static demo does not exercise a deployed API, so provider-runtime behavior remains Unverified.

### 2. PromiseMatch CTA continuity scoring — Verified

```js
function ctaScore(a, b) {
  var first = words(a).slice(0, 3).join(" ");
  var second = words(b).slice(0, 3).join(" ");
  if (!first || !second) return 45;
  if (first === second || first.indexOf(second) >= 0 ||
      second.indexOf(first) >= 0) return 92;
  if (hasTerm(first, ["start", "try", "get", "download", "buy", "book", "learn", "see"]) &&
      hasTerm(second, ["start", "try", "get", "download", "buy", "book", "learn", "see"])) return 68;
  return 38;
}
```

**QA note:** Source audit found explicit exact/substring CTA continuity scoring and a deterministic fallback score for different action families. The deployed static page was confirmed; semantic scoring remains heuristic.

### 3. PromiseMatch client duplicate-submit guard and rules-only fallback — Verified

```js
form.addEventListener("submit", function (event) {
  event.preventDefault();
  if (busy) return;
  if (!form.reportValidity()) return;
  busy = true;
  submitButton.disabled = true;
  status.textContent = "Checking promise, offer, audience, and CTA continuity…";
  var input = readInput();
  var local = fallback(input);
  render(local);
  requestReport(input).then(function (remote) {
    render(remote);
    status.textContent = "Audit complete.";
  }).catch(function () {
    status.textContent = "The API was unavailable. A rules-only audit is ready.";
  }).finally(function () {
    busy = false;
    submitButton.disabled = false;
  });
});
```

**QA note:** Live static re-audit confirmed the form workflow, required-field validation, immediate deterministic rendering, disabled submit control, and API-unavailable fallback messaging. Real-browser interaction and deployed API behavior remain untested.

### 4. PromiseMatch provider timeout and safe fallback — Unverified

```js
async function providerReport(input, base) {
  var provider = process.env.AI_PROVIDER;
  var key = provider === "gemini" ? process.env.GEMINI_API_KEY :
    provider === "groq" ? process.env.GROQ_API_KEY : "";
  if (!key) return base;
  var controller = new AbortController();
  var timer = setTimeout(function () { controller.abort(); }, 8000);
  try {
    var response;
    if (provider === "gemini") {
      response = await fetch("https://generativelanguage.googleapis.com/v1beta/models/gemini-2.0-flash:generateContent?key=" + encodeURIComponent(key), {
        method: "POST",
        headers: {"Content-Type": "application/json"},
        body: JSON.stringify({contents: [{parts: [{text: promptFor(input)}]}]}),
        signal: controller.signal
      });
    } else {
      response = await fetch("https://api.groq.com/openai/v1/chat/completions", {
        method: "POST",
        headers: {"Content-Type": "application/json", "Authorization": "Bearer " + key},
        body: JSON.stringify({model: "llama-3.1-8b-instant", temperature: 0,
          max_tokens: 1200, response_format: {type: "json_object"},
          messages: [{role: "user", content: promptFor(input)}]}),
        signal: controller.signal
      });
    }
    if (!response.ok) return base;
    var data = await response.json();
    var text = provider === "gemini" ? data.candidates && data.candidates[0] &&
      data.candidates[0].content && data.candidates[0].content.parts[0].text :
      data.choices && data.choices[0] && data.choices[0].message &&
      data.choices[0].message.content;
    var parsed = JSON.parse(text || "{}");
    if (!validReport(parsed)) return base;
    parsed.overall_score = Math.max(base.overall_score,
      Math.min(100, Math.round(parsed.overall_score)));
    parsed.priority_fixes = parsed.priority_fixes.slice(0, 5);
    parsed.revised_hero_variants = parsed.revised_hero_variants.slice(0, 3);
    return parsed;
  } catch (error) {
    return base;
  } finally {
    clearTimeout(timer);
  }
}
```

**QA caveat:** Code audit found provider-specific request bodies, environment-only keys, timeout, JSON parsing, schema validation, and deterministic fallback. No deployed provider endpoint was live-tested; this remains Unverified for production runtime behavior.

### 5. PromiseMatch lightweight client-IP rate window — Unverified

```js
function clientIp(req) {
  return String((req.headers &&
    (req.headers["x-forwarded-for"] || req.headers["x-real-ip"])) ||
    "unknown").split(",")[0].trim();
}

function limited(req) {
  var now = Date.now();
  var ip = clientIp(req);
  var prior = (hits.get(ip) || []).filter(function (time) {
    return now - time < 3600000;
  });
  if (prior.length >= 5) return true;
  prior.push(now);
  hits.set(ip, prior);
  return false;
}
```

**QA caveat:** Source audit found a five-requests-per-hour in-memory limiter and proxy-header extraction. It is best-effort only, not durable across serverless instances, and requires a durable limiter before public traffic.
