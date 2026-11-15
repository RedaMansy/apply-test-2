---
name: apply
description: Tailor the LaTeX CV to a job description, save the PDF and create the Obsidian tracker entry. Use when given a job link or pasted job description to apply for.
argument-hint: <job URL or pasted job description>
disable-model-invocation: true
---

# Apply workflow

Input: $ARGUMENTS

Work through the steps in order. There are two approval gates. At each one, show the
details in a message, then ask with the AskUserQuestion tool: "Approve" / "Changes
needed" (I describe them in Other). Only "Approve" moves on; anything else is feedback:
apply it and ask again. Ask any other question in this workflow the same way, with
options rather than waiting for a typed reply.

`<CVPrefix>` below is the CV file prefix set in `CLAUDE.md` (e.g. `JaneDoeCV`).

Before starting: if the prefix is still `YourNameCV` or `cv/master.tex` still contains
`Jane Doe`, the workspace isn't set up. Tell me to run `/setup` first and stop.

Follow the About me and CV preferences sections of `CLAUDE.md` throughout.

## 1. Get the job description

- If the input is a URL, fetch it. JS-rendered job sites (Apple, Workday, many ATSs)
  often return only the title. If so, download the raw HTML with curl and look for the
  job data embedded in a script tag (e.g. `__staticRouterHydrationData`, `__NEXT_DATA__`,
  or `application/ld+json` with `"@type": "JobPosting"`). If that also fails, or the page
  needs a login (LinkedIn), ask me to paste the text instead.
- Extract: company, role, location, workplace type (Remote, Hybrid or On-site),
  employment type (Full-time, Contract, Part-time or Internship), salary (blank if not
  stated), job URL, source. Source is one of the sources listed in `CLAUDE.md`; use
  "Other" only if none fit.
- Check whether `Job Applications/<Company> - <Role>.md` already exists. If it does,
  tell me and ask before continuing ("Continue as a new application" / "Stop").
- Give me a 3-5 line read of the role: what they mostly want, the must-have skills, and
  where my CV is strong or weak against it.

## 2. Tailor the CV

- Pick a short CV suffix: the company name with no spaces (e.g. `Acme`, `Initech`).
  If `Job Applications/Attachments/<CVPrefix>_<suffix>.pdf` already exists, add a role hint
  (e.g. `Acme_DevOps`).
- Copy `cv/master.tex` to `cv/tailored/<suffix>.tex` and edit only the copy.
  Never edit `cv/master.tex` during this workflow.
- Tailoring rules:
  - Never invent experience, technologies, metrics or dates. Only reword, reorder,
    emphasise, trim, or pull material from `cv/bullet-bank.md`.
  - If the JD asks for something that might be true but isn't in the master CV or the
    bullet bank, ask me rather than guessing. If I confirm it, add it to the
    "Confirmed facts" section of `cv/bullet-bank.md` so it's available next time.
  - Mirror the JD's terminology where it honestly describes what I did.
  - Reorder bullets within each role so the most relevant come first; drop the weakest
    bullets if space is needed.
  - Open every bullet with a strong action verb (Built, Designed, Led, Reduced...),
    never "Helped" or "Responsible for", and don't start consecutive bullets with the
    same verb.
  - Every project keeps at least two bullets; if one can only justify a single bullet,
    drop the project instead.
  - Reorder and regroup the Technical Skills lines to lead with what the JD asks for.
  - Keep the formatting and preamble untouched, and escape LaTeX specials (`& % $ # _`).
  - **It must be exactly one page, no exceptions,** and it should fill the page: no
    large white space at the bottom. Treat line count as a fixed budget: every line
    added (longer bullet, extra skill line, bank bullet) must be paid for by a line
    removed elsewhere. Pay for it first with the least relevant content for this JD:
    shorter wordings from the bullet bank, then dropping the weakest bullets. If space
    is left over, fill it with true, relevant material from the bullet bank.
  - Avoid widows: a bullet or skills line that wraps onto a new line by only one or two
    words wastes a line; shorten it so it fits.
- Compile from the repo root:
  `pdflatex -interaction=nonstopmode -halt-on-error -output-directory=cv/build cv/tailored/<suffix>.tex`
  If it fails, read the error from `cv/build/<suffix>.log`, fix it and recompile.
- Check the log for `Output written on ... (1 page`. If it is more than one page, trim
  and recompile. Never show me a CV at Gate 1 that isn't one page.
- Read the compiled PDF to check it looks right.
- **Gate 1:** show me a short changelog (bullets reworded, moved, added from the bank,
  removed; skills changes) and the path to `cv/build/<suffix>.pdf`, then ask for
  approval.

## 3. Save the CV

Once approved, copy `cv/build/<suffix>.pdf` to
`Job Applications/Attachments/<CVPrefix>_<suffix>.pdf`.
If I asked for a reworded bullet during Gate 1 that's worth reusing, add it to
`cv/bullet-bank.md`.

## 4. Create the tracker entry

- Fill `.claude/skills/apply/entry-template.md` and write it to
  `Job Applications/<Company> - <Role>.md`. Strip characters that are invalid in filenames (`/ \ : * ? " < > |`)
  from the name.
- `{{today}}` is today's date (YYYY-MM-DD). `{{cvFile}}` is `<CVPrefix>_<suffix>.pdf`.
  `{{tailoringSummary}}` is a one-line summary of the changelog.
  `{{jobDescription}}` is the full JD text, cleaned of navigation/cookie junk.
- Leave unknown fields as `""` in the frontmatter and drop the matching Overview line.
- **Gate 2:** show me the frontmatter and Overview section, and list this application's
  files (the entry `.md`, the attached PDF and `cv/tailored/<suffix>.tex`), then ask
  for approval.
- That's the end of the workflow. I commit and push myself: never stage, commit or push.
