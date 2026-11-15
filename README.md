# apply-skill

A [Claude Code](https://claude.com/claude-code) skill that tailors a LaTeX CV to a job
description and logs the application in an [Obsidian](https://obsidian.md) vault.

Give it a job link and it will:

1. Fetch the job description (including from JS-heavy ATS pages like Workday) and give
   you a quick read on how your CV matches it.
2. Copy your master CV, tailor it to the role and compile it. It reorders and rewords
   bullets, pulls alternatives from your bullet bank and regroups skills, all while
   keeping the CV to **exactly one page**.
3. Stop for your approval, then save the PDF to `Job Applications/Attachments/`.
4. Create an Obsidian note for the application with status, source, salary, the full
   job description and a link to the exact CV you sent, then stop for approval again.

It never invents experience. Everything on a tailored CV comes from your master CV or
from your bullet bank, and when a job asks for something that isn't in either, it asks
you instead of guessing.

## Requirements

- [Claude Code](https://claude.com/claude-code)
- A LaTeX distribution with `pdflatex` on your PATH:
  [MiKTeX](https://miktex.org) (Windows), [MacTeX](https://tug.org/mactex/) (macOS) or
  TeX Live (Linux)
- [Obsidian](https://obsidian.md) with the
  [Job Application Tracker](https://github.com/cruisencode) community plugin (optional,
  but it gives you Kanban, Table and Sankey views of your applications). The notes use
  the plugin's own format and default folders, statuses and sources, so it works with
  no configuration.

## Getting started

1. **Get your own copy.** Click **Use this template** on GitHub and **make it
   private**: it will hold your CV, contact details and every application you make.
   Then clone it. (No git? Use **Code > Download ZIP** and unzip it instead.)
2. **Open Claude Code in the folder:** `cd apply-skill` then `claude`, or open the
   folder in VS Code with the Claude Code extension.
3. **Run `/setup`.** Claude notices the workspace is new and walks you through it in
   about 20-40 minutes:
   1. Checks LaTeX is installed and tells you how to install it if not.
   2. Asks for your current CV (PDF is best) and anything else useful, like old CVs or
      your LinkedIn profile saved as PDF. Put these in `cv/sources/`.
   3. Confirms your details, target roles, page size (A4 / US Letter) and spelling.
   4. Converts your CV into the template, keeping your wording, and fits it to one
      full page. Anything that doesn't fit goes into your bullet bank, not the bin.
   5. Goes through your skills and asks what's missing.
   6. Interviews you about each role to build up your bullet bank: what you built,
      numbers, who you worked with. It drafts alternative bullets and you confirm each
      one is true.

   You can stop at any point and run `/setup` again later to carry on. Run
   `/setup bullets` any time to add more to your bullet bank.

## Usage

In Claude Code, from the repo root:

```
/apply https://jobs.example.com/software-engineer-123
```

You can also paste the job description text instead of a link (needed for LinkedIn,
which requires a login).

Any reply other than an approval counts as feedback: Claude applies it and shows you
the result again.

To update an application later, just tell Claude, e.g. "Acme rejected me" or "got a
screening call with Initech on Friday". `CLAUDE.md` tells it how to update the
status, history and activity log.

## Layout

```
CLAUDE.md                         your details, CV preferences and rules for Claude
.claude/skills/setup/SKILL.md     the /setup onboarding
.claude/skills/apply/SKILL.md     the /apply workflow
.claude/skills/apply/entry-template.md   frontmatter + layout for application notes
cv/sources/                       your original CVs, LinkedIn PDF, etc.
cv/master.tex                     your base CV (never edited by /apply)
cv/bullet-bank.md                 true alternative bullets, skills and confirmed facts
cv/tailored/<Company>.tex         one tailored CV per application
cv/build/                         LaTeX output (gitignored)
Job Applications/<Company> - <Role>.md            one note per application
Job Applications/Attachments/<Prefix>_<Company>.pdf  the exact PDF you sent
Job Applications/Interviews/                      interview prep notes
```

The notes are in the Job Application Tracker plugin's format, plus one extra field,
`CV used`, which links each application to the exact PDF you sent. If you change the
plugin's statuses, sources or folders, tell Claude so it updates `CLAUDE.md` to match.

## Tips

- Tell Claude your standing preferences ("always keep my dissertation project",
  "never drop the second bullet at X") and it records them under CV preferences in
  `CLAUDE.md`. The skills are plain English too, so you can edit them directly.
- When you reword something at the approval step and like the result, ask Claude to
  add it to the bullet bank.
- Keep the PDFs you send in git. Rebuilding one later won't always give the same file
  if the template or your LaTeX install has changed, and the PDF is the record of what
  the recruiter actually saw.

## Credits

The CV template is [Jake Gutierrez's resume](https://github.com/jakegut/resume), based on
[sb2nov/resume](https://github.com/sb2nov/resume), both MIT licensed.
