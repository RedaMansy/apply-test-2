---
name: setup
description: First-run onboarding for this workspace. Checks the LaTeX install, imports the user's existing CV into cv/master.tex, inventories their skills, interviews them to seed cv/bullet-bank.md, and records their details and preferences in CLAUDE.md. Use when the workspace still has placeholder content, or when the user asks to get started, set up, or add to their bullet bank.
argument-hint: "[bullets | skills | cv]  (optional: run just one part)"
---

# Setup

Input: $ARGUMENTS

You're guiding someone through setting up this workspace for the first time. They may
not be technical and may never have used Claude Code before. Your job is to make this
feel easy and to get true, detailed material out of them, because the quality of every
tailored CV depends on what goes into the master CV and the bullet bank here.

## How to run this

- **Ask every question with the AskUserQuestion tool**, not as plain text the user has
  to reply to. It shows a clickable prompt, and it always adds an "Other" option where
  they can type their own answer, so open questions work too:
  - Turn open questions into suggested answers drawn from their CV and sources, e.g.
    "What roles are you targeting?" with options like "Backend engineer",
    "Full-stack engineer", "Data engineer" (multiSelect when several can apply).
  - Confirmations get clear options: "Looks right" / "Needs changes" (they type the
    change in Other).
  - Every question that might not apply gets a "Skip for now" or "I don't know" option.
  - Put the context they need to answer (what you found, a draft bullet, the PDF path)
    in the question or option text, because the prompt may hide earlier messages.
  - Ask up to 4 related questions in one call (the tool's limit), one topic per call.
  - Plain text only for things that don't fit a prompt: long pastes like CV text, or
    "tell me when you've done X" steps (and even those can be a "Done" / "Need help"
    prompt).
- **Explain why** you're asking in half a sentence in the question text.
- **"Skip" and "I don't know" are always fine.** Never fill a gap with a guess. No
  invented numbers, tools, team sizes or outcomes. If a bullet would be better with a
  metric they don't have, ask once, and if they don't know, write it without one.
- **Save as you go.** Write each piece to disk as soon as it's agreed, so stopping
  halfway loses nothing. Tell them they can stop at any time and run `/setup` again
  later to pick up where they left off.
- **Show progress.** Start each phase with a line like "Step 3 of 6: your CV".
- Never stage, commit or push.

## 0. Work out where they are

Check what's already done:

| Phase | Done when |
|---|---|
| 1. Tools | `pdflatex --version` succeeds and `cv/master.tex` compiles |
| 2. About you | `CLAUDE.md` has a CV file prefix other than `YourNameCV` |
| 3. Your CV | `cv/master.tex` no longer contains `Jane Doe` |
| 4. Skills | `cv/bullet-bank.md` has a filled-in `## Skills` section |
| 5. Bullet bank | `cv/bullet-bank.md` has no `*Example:*` entries left |
| 6. Finish | all of the above |

If the input names one part (`bullets`, `skills`, `cv`), run just that phase, even if
it's already done (to add more or redo it).

Otherwise, on a fresh workspace, welcome them in a few lines: what this workspace does,
what setup involves (the six steps above, about 20-40 minutes, most of it the bullet-bank
interview), and that they can stop at any time. Then ask them to put their files in
`cv/sources/` (create it if missing), with a prompt offering "Done, files are in
cv/sources", "I'll paste my CV text instead" and "Where is that folder?":

- **Their current CV** (required). PDF is best. For a Word file, ask them to save it as
  PDF, or paste the text into the chat.
- **Optional extras that make the bullet bank much better:** older CVs or versions
  tailored for other jobs, their LinkedIn profile as a PDF (on their profile: More >
  Save to PDF), cover letters, a portfolio or personal website link, and their GitHub
  username.

If some phases are already done, say which ones and continue from the first one that
isn't.

## 1. Tools

- Run `pdflatex --version`. If it's missing, detect the OS and tell them what to install:
  MiKTeX (https://miktex.org) on Windows, MacTeX (https://tug.org/mactex/) on macOS, or
  `texlive-latex-extra` (e.g. `sudo apt install texlive-latex-extra`) on Linux. They will
  need to restart the terminal / Claude Code afterwards so `pdflatex` is on the PATH.
  Then stop and wait for them to come back.
- For the Obsidian steps below, ask after each step with "Done" / "Stuck" options rather
  than listing all the steps and waiting.
- Compile the placeholder CV to check packages are present:
  `pdflatex -interaction=nonstopmode -halt-on-error -output-directory=cv/build cv/master.tex`
  On MiKTeX, if it fails over a missing package, run
  `initexmf --set-config-value=[MPM]AutoInstall=1` so packages install on demand, and
  retry.
- Ask whether they use Obsidian. If no, that's fine: the application notes are plain
  Markdown files that work anywhere. If yes, walk them through it one step at a time:
  1. Open this folder as a vault (Obsidian > Open folder as vault).
  2. Settings > Community plugins > Turn on community plugins.
  3. Browse, search "Job Application Tracker" (by cruisencode), Install, then Enable.
  4. Open it from the ribbon icon or the command palette to see the Kanban, Table and
     Sankey views. They'll fill up as they apply.
  Tell them to leave its settings at the defaults: this workspace is built around
  them. If they later change statuses, sources or folders in the plugin, they should tell
  Claude so it can update `CLAUDE.md` to match.

## 2. About you

Read everything in `cv/sources/` first, so you can confirm details rather than ask
for them.

- Confirm the contact details from their CV (name, location, phone, email, LinkedIn,
  GitHub or portfolio): show them in the question, with options "All correct",
  "Something's wrong" and "Leave something off".
- Then one prompt with these questions, pre-selecting the likely answer as the first
  option based on their location:
  - Page size: A4 (UK, Europe and most of the world) or US Letter (US, Canada).
  - Spelling: UK or US English.
  - CV file prefix, suggested from their name (e.g. `JaneDoeCV`). This becomes the
    start of every saved PDF's name, so recruiters see their name.
- Then a prompt about what they're targeting, with options suggested from their CV:
  job titles (multiSelect), seniority, work setup (Remote / Hybrid / On-site / Any), and
  locations. Follow up only on anything else that affects which jobs fit (e.g. needing
  visa sponsorship).
- Write to `CLAUDE.md`: set the prefix, and fill the `## About me` section with the
  target roles, page size and spelling.

## 3. Your CV

- Rewrite `cv/master.tex` with their CV in this template. Keep the preamble and custom
  commands exactly as they are, and only change the page size option in
  `\documentclass` if they chose US Letter.
- **Keep their wording.** This is their CV, not a rewrite. Move content into the
  template's structure (Education, Experience, Projects, Technical Skills; add sections
  like Publications or Certifications only if their CV has them) and escape LaTeX
  specials (`& % $ # _`).
- Then suggest improvements: weak openers ("Helped", "Responsible for"), vague bullets
  that need a number they might know, repeated verbs, bullets that wrap by one or two
  words, UK/US spelling. Offer them as a multiSelect prompt ("Which of these should I
  apply?"), each option showing the before and after, up to 4 per question.
- **It must be exactly one page, and it should fill the page.** Tailoring works by
  swapping content in and out of a fixed budget, so the master should be full.
  - If their CV is too long, keep the strongest and most general content in the
    master and move the rest **word for word** into `cv/bullet-bank.md` under the right
    role. Nothing true is thrown away. Tell them what moved.
  - If it's too short, note which roles need more and cover them in the bullet bank
    interview (phase 5), then come back and fill the page.
- Compile, read the PDF to check it looks right, and show them the path to
  `cv/build/master.pdf`. **Get approval before moving on** with a prompt: "Approve",
  "Changes needed" (they describe them in Other). Apply any feedback and ask again.

## 4. Skills

- List every skill you found in their sources, grouped (Languages, Frameworks, Tools,
  Cloud, Data, Domain knowledge, or whatever fits their field; not everyone is a
  software engineer).
- Ask what's missing with multiSelect prompts of common skills for their target roles
  that aren't in their sources, one category per question ("Have you used any of these
  databases?"), with Other for anything not listed. Tools from old jobs, university and
  side projects all count.
- For anything not already backed up by a bullet, ask how they used it, one skill per
  question: "Professionally", "In a project", "Briefly / at university". That decides
  whether it can only go on the skills line or can support a bullet.
- Write the result to the `## Skills` section of `cv/bullet-bank.md`, noting depth
  where it isn't obvious (e.g. "Terraform: used professionally at Example Corp,
  managed staging only").

## 5. Bullet bank interview

This is the most valuable step. Explain that the bullet bank lets each tailored CV pull
the most relevant true material for that job, and that the more is in it, the better
the tailoring.

Go through their roles from most recent, then their main projects. For a long career,
cover the last three roles and their best two projects, and offer to do older ones
later. For each one:

1. Show what's already on the CV and in the bank for it.
2. Ask up to three of these per prompt, picking the ones that will uncover the most.
   Give each one options you can infer from their sources (e.g. for tools, the ones
   their CV mentions for that job, multiSelect), plus "Skip"; their real detail will
   usually come through Other, and that's fine:
   - What did you build, own or change that isn't on your CV yet?
   - Any numbers: users, requests, money, time saved, team size, % improvement?
   - Who did you work with (team size, other teams, clients, senior leadership)?
   - Which tools and technologies did you use here?
   - What are you proudest of? What would a manager there say you were good at?
   - Did you lead, mentor, present, write documentation, or deal with production
     incidents?
3. From their answers and their sources, draft alternative bullets: different angles
   for different kinds of job (e.g. backend vs. data vs. leadership), and a short version
   of each long bullet. Mark which kind of job each angle suits.
4. **Ask them to confirm every drafted bullet is true** with a multiSelect prompt
   ("Which of these are accurate?"), one bullet per option, up to 4 per question.
   Unticked bullets don't go in the bank; ask in Other what needs fixing.
5. Write them to `cv/bullet-bank.md` under the role, and put facts that are true but
   don't make a bullet (e.g. "worked directly with the CEO") under `## Confirmed facts`.

After each role, prompt "Next role: <name>" / "Stop for now". They can run
`/setup bullets` any time to add more.

If the master CV was too short in phase 3, go back and fill it now from the new
material, recompile and get approval.

## 6. Finish

- Remove any `*Example:*` entries left in `cv/bullet-bank.md` and any placeholder text in
  `CLAUDE.md`.
- Ask whether there's anything they always want on their CV, or never want, offering
  suggestions from their CV as options (e.g. "Always keep <project>", "Never drop
  <role>") plus "Nothing for now". Add these to the
  `## CV preferences` section of `CLAUDE.md`.
- Compile `cv/master.tex` one last time to check it's still one page.
- Summarise what's set up in a few lines, then tell them how to use it:
  - `/apply <job link>` (or paste the job description) to tailor a CV and log the
    application.
  - Tell Claude about updates in plain words, e.g. "Acme rejected me" or "I've got an
    interview with Initech on Friday".
  - `/setup bullets` to add to the bullet bank later.
- If the folder is a git repo, suggest they commit their setup and remind them to keep
  the repo private, since it holds their CV and contact details. Don't commit for them.
