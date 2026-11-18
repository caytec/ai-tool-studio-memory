# Ideas Log

## 2026-11-17 — LootLens ABI
- **Chosen niche:** Arena Breakout: Infinite contextual loot decisions.
- **Tool name:** LootLens ABI.
- **Description:** A no-signup, rules-first tool recommending Keep, Sell, Use, Extract, or Leave using item value, inventory space, mission relevance, raid position, and risk.
- **Keywords:** arena breakout infinite loot calculator; ABI keep or sell; ABI loot value per slot; arena breakout infinite mission items; ABI loot decision tool.
- **Runner-up niches considered:** ABI budget-aware raid/loadout planning (recommended in the research report as RaidBudget ABI); ABI mission preparation; ABI extraction route risk planning; ABI weapon/ammunition optimization.
- **Repo/preview URL:** https://github.com/caytec/lootlens-abi-mvp-20261117; planned Pages URL https://caytec.github.io/lootlens-abi-mvp-20261117/.
- **Release status:** **NO-GO / deployment blocked** — live re-audit found the planned GitHub Pages URL returns GitHub Pages 404; the repository was not created, no files were deployed, and Vercel was not imported or configured.

## Prior ideas

- **StreamMoment Cards** — streaming audience engagement / shareable stream moment cards.

## 2026-11-17 — LaunchProof AI
- **Chosen niche:** Cross-chain meme coin launch due diligence.
- **Tool name:** LaunchProof AI.
- **Description:** A no-signup evidence-reconciliation tool that compares meme coin promotion claims with available token, market, holder, permission, and limitation evidence, then produces a timestamped screening report and share card.
- **Keywords:** meme coin due diligence; token launch checker; verify meme coin contract; crypto promotion fact checker; liquidity lock checker; meme coin risk report.
- **Runner-up niches considered:** WalletCluster Meme / holder-wallet cluster explainer; meme coin narrative-to-data checker; token comparison battle cards; evidence-backed meme coin social cards.
- **Repo/preview URL:** https://github.com/caytec/launchproof-ai-mvp-20261117 ; https://caytec.github.io/launchproof-ai-mvp-20261117/
- **Release status:** **Live static MVP; not public-launch ready.** GitHub Pages build is healthy and the frontend loads, but `/api/generate.js` is not executable on GitHub Pages; no Vercel API deployment is live or connected, so server-backed analysis is unavailable and the demo uses deterministic fallback output.

## 2026-11-17 — TrackProof
- **Chosen niche:** Paid-campaign tracking QA and attribution handoff.
- **Tool name:** TrackProof.
- **Description:** A rules-first UTM checker that generates clean campaign URLs, catches tracking mistakes, suggests campaign/event naming, and creates a launch-ready tracking contract.
- **Keywords:** UTM checker; UTM audit tool; campaign tracking QA; tracking contract; campaign tracking checklist; UTM link checker.
- **Runner-up niches considered:** local review-recovery cards; content freshness checking; creator-deal scoping; ad-to-landing-page message matching.
- **Repo/preview URL:** https://github.com/caytec/trackproof-mvp-20261117/ ; https://caytec.github.io/trackproof-mvp-20261117/
- **Release status:** **Conditional preview / not production-ready.** GitHub Pages static frontend is accessible and the rules-only UTM audit flow is present. Vercel API has not been imported, deployed, or live-tested; browser regression, escaping, Pages build completion, PNG/JPEG OG support, automated checks, and stronger rate limiting remain release blockers.

## 2026-11-17 — ClientShield
- **Chosen niche:** Freelance client, job-post, and payment-risk screening.
- **Tool name:** ClientShield — a no-signup freelance scam checker.
- **Description:** Paste a freelance job post or client message to receive an explainable risk report, verification checklist, safer reply, and privacy-safe share card.
- **Keywords:** freelance scam checker; fake client message checker; freelance job scam detector; freelance payment scam; freelance contract red flags.
- **Runner-up niches considered:** side-hustle opportunity legitimacy checker; template profitability calculator; scope-creep and payment-term checker; resale authenticity and scam-risk explainer.
- **Repo/preview URL:** https://github.com/caytec/clientshield-mvp-20261117 ; https://caytec.github.io/clientshield-mvp-20261117/
- **Release status:** **NO-GO for production / partial deployment only.** Live re-audit found the homepage and supporting pages load, but live `js/app.js` is corrupted, `assets/og.png` returns 404, the AI API is not deployed to a runtime, and the privacy page lacks a real monitored contact. Rules-only experience can proceed after JavaScript repair; production approval is blocked.

## 2026-11-18 — StreamGap
- **Chosen niche:** Ad-break and dead-air rescue prompts for streamers.
- **Tool name:** StreamGap.
- **Description:** A no-signup utility that turns a game, downtime reason, remaining minutes, and channel tone into a ready-to-run mini-segment with prompts, a poll, a mini-game, transition copy, chatbot-ready text, and a downloadable card.
- **Keywords:** stream filler generator; Twitch ad break ideas generator; stream dead air ideas; loading screen stream prompts; quiet Twitch chat ideas; Twitch chat prompt generator.
- **Runner-up niches considered:** Gaming challenge/ruleset generator (ChallengeForge); raid handoff assistant (RaidReady); viewer question-to-stream segment planner (SegmentSpark); live audience prompt deck (PromptDeck Live).
- **Repo/preview URL:** https://github.com/caytec/streamgap-mvp-20261118 ; https://caytec.github.io/streamgap-mvp-20261118/
- **Release status:** **Deployment incomplete / blocked by GitHub Pages propagation or build failure.** The public repository and root files exist and Pages was configured from `main` root, but the latest build remained `building` and the preview returned HTTP 404 / “There isn't a GitHub Pages site here.” The Vercel API is also not deployed or live-tested, so the end-to-end release is blocked.
