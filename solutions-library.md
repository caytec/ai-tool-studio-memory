# Solutions Library

## Lean dual-runtime generation pattern
- **When to use:** When a static frontend must remain useful even if a serverless AI endpoint is unavailable.
- **Working snippet:**
```js
const API_URL = window.APP_API_URL || "/api/generate";
async function generate(input) {
  try {
    const r = await fetch(API_URL, {method:"POST", headers:{"Content-Type":"application/json"}, body:JSON.stringify(input)});
    if (!r.ok) throw new Error("API unavailable");
    return (await r.json()).card;
  } catch { return deterministicFallback(input); }
}
```
- **QA caveats:** Keep browser and server fallbacks behaviorally aligned; use a timeout and visibly label fallback output. Static hosting cannot execute the serverless endpoint.

## Safe card rendering with textContent and Canvas
- **When to use:** When model-generated copy must be displayed and exported without allowing generated HTML/CSS execution.
- **Working snippet:**
```js
const headline = document.createElement("h3");
headline.textContent = modelCard.headline;
preview.replaceChildren(headline);
const canvas = document.createElement("canvas");
canvas.width = 1200; canvas.height = 675;
const ctx = canvas.getContext("2d");
ctx.fillStyle = "#101522"; ctx.fillRect(0, 0, 1200, 675);
ctx.fillStyle = "#fff"; ctx.font = "bold 58px system-ui";
ctx.fillText(String(modelCard.headline).slice(0, 100), 84, 250);
```
- **QA caveats:** Escape or avoid model HTML entirely; long text needs measured wrapping; preview and PNG layouts can diverge.

## Provider adapter with one stable card contract
- **When to use:** When supporting Gemini, Groq, OpenRouter, or future providers without changing the frontend.
- **Working snippet:**
```js
async function provider(input) {
  const provider = process.env.AI_PROVIDER || "gemini";
  const prompt = makeStructuredPrompt(input);
  const raw = provider === "gemini"
    ? await callGemini(prompt)
    : await callOpenAICompatible(provider, prompt);
  const card = JSON.parse(raw);
  const keys = ["headline","context","moment_label","reaction","x_caption","discord_caption"];
  if (!keys.every(k => typeof card[k] === "string")) throw Error("invalid output");
  return card;
}
```
- **QA caveats:** Provider response shapes differ; validate JSON and length limits; never expose API keys client-side; model names and quotas change.

## Static-friendly relative SEO metadata
- **When to use:** Before the production canonical domain is verified, while retaining deployability on GitHub Pages or Vercel.
- **Working snippet:**
```html
<link rel="canonical" href="/">
<meta property="og:url" content="/">
<link rel="sitemap" href="/sitemap.xml">
```
- **QA caveats:** Replace relative values with the verified absolute origin before launch; relative sitemap URLs may not satisfy all search-engine expectations; do not publish an unverified domain.

## Anonymous MVP protection with bounded IP rate limiting
- **When to use:** For a lightweight no-account serverless MVP needing basic abuse protection without persistent user storage.
- **Working snippet:**
```js
const hits = new Map(), WINDOW = 3600000, LIMIT = 20;
function allowed(ip) {
  const now = Date.now();
  const recent = (hits.get(ip) || []).filter(t => now - t < WINDOW);
  if (recent.length >= LIMIT) return false;
  recent.push(now); hits.set(ip, recent); return true;
}
```
- **QA caveats:** In-memory state is per instance and resets; it is not distributed protection. Avoid logging raw inputs or sensitive content, and add durable rate limiting as traffic grows.
