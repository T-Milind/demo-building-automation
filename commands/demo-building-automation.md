---
description: End-to-end pitch-demo pipeline — researches a local business, plans 3-4 distinct design directions, builds all of them in one repo with a single-link variation switcher, tests thoroughly, fixes issues, and deploys to Vercel.
argument-hint: Client/business name, plus every relevant link (Google Maps profile, current website, socials, review platforms)
---

# Demo Building Automation

You are orchestrating four specialized subagents — `business-logic-agent`, `planner-agent`, `coding-agent`, `testing-agent` — through a full client-pitch pipeline. This command is the trigger the user asked for: once invoked, run the pipeline through to a deployed link with minimal interruption. Stop and ask only when genuinely blocked (missing required input, a stuck fix loop, or a hard-to-reverse action).

Initial request: $ARGUMENTS

**Run every subagent call in this pipeline in the foreground (`run_in_background: false`)** — each phase depends on the previous phase's output file, so you need the result before continuing. Pass file paths between agents, not pasted file contents — that's the token-efficient handoff this pipeline is designed around.

---

## Phase 0: Setup

**Goal**: Establish the working directory and confirm required input

**Actions**:
1. If $ARGUMENTS doesn't include both a business/client name and at least one link (Google Maps, website, or social), ask the user for what's missing before proceeding — this genuinely blocks every downstream stage.
2. Derive a client slug (e.g. "Timmy's Mobile Mechanic" → `timmys-mobile-mechanic`) and confirm/create the working directory for this client if one doesn't already exist. Ask the user where it should live if not obvious from context.
3. Create `.demo-automation/` inside it for pipeline working files.
4. Create a TodoWrite list mirroring the phases below.

---

## Phase 1: Business Logic

**Goal**: Understand the business, its audience, and its competitive landscape

**Actions**:
1. Launch `business-logic-agent` with: all links from $ARGUMENTS, and the output path `.demo-automation/business-logic.md`.
2. Once it returns, briefly confirm the file was written (read the first ~30 lines to sanity check it's substantive, not a stub).

---

## Phase 2: Planning

**Goal**: Turn business understanding into 3-4 distinct, detailed build prompts

**Actions**:
1. Launch `planner-agent` with: the path to `business-logic.md`, and the output path `.demo-automation/demo-prompts.md`.
2. Once it returns, read the direction names/rationales (not the full prompts) and give the user a one-paragraph heads-up on the 3-4 directions chosen before moving on — this is informational, not a blocking approval gate, unless something looks clearly off (e.g. directions that don't actually differ, or a direction contradicted by the business-logic findings), in which case pause and flag it.

---

## Phase 3: Build

**Goal**: Build all demos plus the unified variation-switcher preview shell

**Actions**:
1. Launch `coding-agent` in build mode with: the path to `demo-prompts.md`, the target repo layout (`demo-1/` … `demo-N/` + `preview/` inside the client working directory), and an explicit reminder to follow the known gotchas (apostrophe-in-path metadata-route failure, next/image basePath prefix bug) documented in its own agent instructions.
2. Once it returns, confirm the repo structure exists as expected (`Glob` for `demo-*/package.json` and `preview/src/app/page.tsx`).

---

## Phase 4: Test

**Goal**: Verify everything works before anyone outside this pipeline sees it

**Actions**:
1. Launch `testing-agent` with: the repo path, the `demo-prompts.md` path, a local URL (have it start the preview shell itself), and output path `.demo-automation/test-report-round-1.md`.
2. Read the report's severity counts.

---

## Phase 5: Fix loop (local)

**Goal**: Drive Critical/Important issues to zero before deploying

**Actions**:
1. If there are zero Critical/Important issues, skip to Phase 6.
2. Otherwise, launch `coding-agent` in fix mode with the test report path, then launch `testing-agent` again for another round (increment the report filename).
3. Repeat up to **5 rounds total**. If Critical/Important issues remain after 5 rounds, stop and present the remaining issues to the user rather than looping further — this is a genuine escalation, not a phase to push through silently.

---

## Phase 6: Deploy

**Goal**: Get a live link

**Actions**:
1. Launch `coding-agent` in deploy mode: create/push the GitHub repo (confirm naming/visibility preference with the user first if not already established for this client — this is the one deploy-adjacent decision worth a quick check since repo visibility is a real, semi-permanent choice), then `vercel link` + `vercel deploy --prod` for `preview/`.
2. Confirm you have the final production URL back.

---

## Phase 7: Live verification

**Goal**: Confirm the deployed version actually works, not just the local build

**Actions**:
1. Launch `testing-agent` again, this time pointed at the live Vercel URL instead of localhost, output path `.demo-automation/test-report-live.md`.
2. If Critical/Important issues are found, launch `coding-agent` in fix mode, then redeploy (Phase 6 actions again), then re-test the live URL. Cap this loop at **3 rounds**; escalate to the user if unresolved after that.

---

## Phase 8: Final report

**Goal**: Hand the user a working link

**Actions**:
1. Mark all todos complete.
2. Give the user: the live Vercel URL, the number of distinct directions built and what makes each one different (pull this from the planner's direction names/rationales, don't re-derive it), and a one-line note on anything you had to make a judgment call on along the way.
