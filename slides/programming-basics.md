---
marp: true
theme: cdl-theme
math: katex
transition: fade 0.25s
author: Contextual Dynamics Lab
---

# Programming basics
### PSYC 81.09: Storytelling with Data

Jeremy R. Manning
Dartmouth College
Spring 2026

---

# Today's agenda

<div class="note-box" data-title="Plan for today">

1. **Quick refresher** — the Python fundamentals you already know
2. **Hands-on practice** — interesting problems in a companion notebook
3. **Version control with Git and GitHub** — what every data scientist uses
4. **The daily workflow** — clone, commit, push, pull, pull requests, issues

</div>

<div class="tip-box" data-title="Follow along">

Open the [companion notebook](programming_basics.ipynb) in [Google Colab](https://colab.research.google.com/github/ContextLab/storytelling-with-data/blob/master/slides/programming_basics.ipynb) so you can run every example yourself.

</div>

---

# You already know Python basics

<div class="note-box" data-title="Quick refresher">

You should be comfortable with:

- **Variables and types**: `int`, `float`, `str`, `bool`, `list`, `dict`
- **Operators**: `+`, `-`, `*`, `/`, `**`, `==`, `!=`, `<`, `>`, `and`, `or`, `not`
- **Control flow**: `if` / `elif` / `else`, `for` loops, `while` loops
- **Functions**: `def name(args):` ... `return value`
- **Imports**: `import math`, `from numpy import array`

If any of this feels unfamiliar, the [companion notebook](programming_basics.ipynb) has a quick refresher section — and you can always use AI tools to fill in gaps.

</div>

---

# Part 1: hands-on practice

<div class="important-box" data-title="Why practice matters">

Reading code is not the same as **writing** code. You only really learn programming by working through problems — getting stuck, making mistakes, and finding your way out.

Today's problems are **interesting on purpose**. They are not drills. Each one teaches a real programming pattern you'll use again and again.

</div>

---

# Problem 1: FizzBuzz

<div class="example-box" data-title="A classic warm-up">

Print numbers from 1 to 50, with these rules:

- Multiples of 3 → print `"Fizz"`
- Multiples of 5 → print `"Buzz"`
- Multiples of both → print `"FizzBuzz"`
- Otherwise → print the number

</div>

<div class="tip-box" data-title="Before you peek">

Try it yourself first in the [companion notebook](programming_basics.ipynb). The key operator is `%` (modulo) — it gives you the remainder after division.

</div>

---

# Problem 1: FizzBuzz — solution

```python
for n in range(1, 51):
    if n % 15 == 0:
        print('FizzBuzz')
    elif n % 3 == 0:
        print('Fizz')
    elif n % 5 == 0:
        print('Buzz')
    else:
        print(n)
```

---

# Problem 2: word frequency counter

<div class="example-box" data-title="A real data task">

Given a string of text, count how often each word appears. This is the foundation of tools like search engines, spam filters, and text analysis.

**Example input:**
> `"the quick brown fox jumps over the lazy dog the fox is quick"`

**Expected output:** a dictionary mapping each word to its count, sorted from most to least frequent.

</div>

---

# Problem 2: word frequency counter — solution

```python
text = "the quick brown fox jumps over the lazy dog the fox is quick"
words = text.lower().split()

counts = {}
for word in words:
    counts[word] = counts.get(word, 0) + 1

# Sort by frequency, descending
sorted_words = sorted(counts.items(), key=lambda x: -x[1])
print(sorted_words)
```

---

# Problem 3: reading a CSV without a library

<div class="example-box" data-title="Understanding what libraries hide">

Before we use Pandas next week, let's read a CSV "by hand" to understand what's happening under the hood.

**Your task:** Read `students.csv` and turn each row into a dictionary, where the keys are the column names.

</div>

---
<!-- _class: scale-90 -->

# Problem 3: reading a CSV — solution

```python
with open('students.csv', 'r') as f:
    lines = f.readlines()

header = lines[0].strip().split(',')
rows = []
for line in lines[1:]:
    values = line.strip().split(',')
    row = dict(zip(header, values))
    rows.append(row)

# Now rows is a list of dicts — one per record
print(rows[0])
```

---

# Problem 4: debugging practice

<div class="example-box" data-title="Find the bug">

This function is supposed to return the average of a list of numbers. It has a bug. Can you find it?

```python
def average(numbers):
    total = 0
    for i in range(len(numbers)):
        total = numbers[i]
    return total / len(numbers)
```

</div>

<div class="warning-box" data-title="Debugging tips">

- **Read the code out loud** — what does each line *actually* do?
- **Trace through with a small example** — try `average([1, 2, 3])` on paper
- **Use `print` statements** liberally — show intermediate values

</div>

---

# Problem 4: the bug

```python
total = numbers[i]      # BUG: replaces total each iteration
total += numbers[i]     # FIX: adds to total
```

The `=` operator **replaces**. The `+=` operator **accumulates**. After the loop, `total` was just the last number — not the sum.

Most bugs are like this: a single character in the wrong place. Reading code carefully and testing with small examples catches 90% of them.

---

# Part 2: GitHub in daily practice

<div class="important-box" data-title="Why version control matters">

Every data scientist uses Git and GitHub. Not optional. Not advanced. **Daily**.

- Track every change to your code
- Collaborate without overwriting each other's work
- Back up your projects automatically
- Show your work to employers, collaborators, and future you
- Recover from mistakes (including "I deleted the wrong folder")

</div>

---

# Git vs GitHub

<div class="definition-box" data-title="Two different things with similar names">

**Git** is a *tool* that runs on your computer. It tracks changes to files in a folder (called a **repository** or "repo").

**GitHub** is a *platform* (a website) that hosts Git repositories online, making it easy to share code, collaborate, and back up your work.

Git works without GitHub, but GitHub needs Git.

</div>

<div class="tip-box" data-title="For this course">

You need a free [GitHub account](https://github.com). Sign up with your Dartmouth email and pick a professional username — it'll be part of your coding identity.

</div>

---

# Key terms: the vocabulary

<div class="definition-box" data-title="The words you need to know">

- **Repository** ("repo") — a project folder tracked by Git
- **Commit** — a snapshot of your project at a point in time, with a message
- **Branch** — a parallel version of your project (usually `main` is the default)
- **Clone** — download a repo from GitHub to your computer
- **Push** — upload your commits from your computer to GitHub
- **Pull** — download the latest changes from GitHub to your computer
- **Pull request (PR)** — a proposal to merge your changes into another branch
- **Issue** — a tracked task, bug report, or question on GitHub
- **Merge conflict** — when Git can't automatically combine two sets of changes

</div>

---

# The daily workflow

<div class="example-box" data-title="What you'll do every time you work on a project">

1. **Pull** the latest changes: `git pull`
2. **Make your edits** — write code, add files, fix bugs
3. **Commit** your changes with a clear message: `git commit -m "..."`
4. **Push** your commits to GitHub: `git push`
5. **Open a pull request** when you want your changes merged

That's it. Five steps, every day.

</div>

<div class="tip-box" data-title="AI handles the commands">

Claude Code can run all of these for you — your job is to **review what it did** and make sure the commit messages are clear.

</div>

---
<!-- _class: scale-90 -->

# Writing good commit messages

<div class="tip-box" data-title="Clear messages save time">

A commit message tells the story of **what** you changed and **why**. Future you (and your collaborators) will thank you.

**Bad:**
```
update stuff
fixed it
asdf
```

**Good:**
```
add word frequency function to text_utils.py
fix off-by-one error in average() calculation
update README with installation instructions
```

**The rule:** if you had to tell a teammate what this commit does in one sentence, what would you say?

</div>

---

# Clone a repo

<div class="example-box" data-title="Getting a project onto your computer">

On GitHub, click the green **Code** button and copy the URL. Then in your terminal:

```bash
git clone https://github.com/ContextLab/storytelling-with-data.git
cd storytelling-with-data
```

You now have a full copy of the project on your computer, including its complete history.

</div>

<div class="note-box" data-title="Try it now">

Clone the course repository. You'll need it for Assignment 3!

</div>

---
<!-- _class: scale-90 -->

# Commit and push your changes

<div class="example-box" data-title="Saving your work">

After you edit a file:

```bash
git add my_file.py              # stage the file for commit
git commit -m "add new feature" # snapshot the staged changes
git push                        # upload commits to GitHub
```

Or, even better, use AI to handle the details:

> "Please commit these changes with a clear message and push to GitHub."

</div>

<div class="tip-box" data-title="Check your work">

After pushing, refresh your repository page on GitHub — you should see your changes appear, along with your commit message in the history.

</div>

---
<!-- _class: scale-90 -->

# Pull requests: proposing changes

<div class="definition-box" data-title="Not just for open source">

A **pull request** is a formal proposal to merge your branch into another branch (usually `main`). PRs let others:

- **Review** your changes before they're merged
- **Discuss** design decisions
- **Suggest improvements** and catch bugs
- **Keep a record** of how and why the project evolved

Even when you're working alone, PRs are a great way to document your thinking.

</div>

<div class="tip-box" data-title="The PR workflow">

1. Create a branch: `git checkout -b my-feature`
2. Make commits on that branch
3. Push the branch: `git push -u origin my-feature`
4. Open a PR on GitHub (the website will prompt you)
5. Review, discuss, and merge

</div>

---

# Issues: tracking work

<div class="note-box" data-title="A to-do list that collaborates">

GitHub **issues** track tasks, bugs, questions, and feature requests. Each issue has:

- A **title** and **description**
- **Labels** (`bug`, `enhancement`, `question`, ...)
- An **assignee** (who's working on it)
- **Comments** for discussion
- A **status** (open or closed)

Good issues describe the *what* and the *why*, not just the *how*.

</div>

<div class="tip-box" data-title="Use issues in your own work">

Issues aren't just for teams! When working solo, they're a great way to track "things I want to fix later" — and each one becomes a commit or PR when you get to it.

</div>

---
<!-- _class: scale-90 -->

# Merge conflicts: don't panic

<div class="warning-box" data-title="When Git can't automatically merge">

A **merge conflict** happens when two people (or you, from two branches) edit the *same lines* of the *same file*. Git asks you to choose which version to keep.

```
<<<<<<< HEAD
this is what's on main
=======
this is what's on my branch
>>>>>>> my-feature
```

**The steps:**
1. Open the file and find the conflict markers
2. Decide what the final version should look like
3. Delete the markers (`<<<<<<<`, `=======`, `>>>>>>>`)
4. Save, commit, and push

</div>

<div class="tip-box" data-title="Ask for help">

Claude Code is excellent at resolving merge conflicts — show it the conflict and ask "which version should I keep?"

</div>

---

# .gitignore: keep junk out of your repo

<div class="tip-box" data-title="What to ignore">

A `.gitignore` file lists files that Git should **skip** — never track, never commit. Common things to ignore:

- **Data files** (large CSVs, images, datasets)
- **Secrets** (API keys, passwords, `.env` files)
- **Build artifacts** (compiled code, caches, `__pycache__/`)
- **System junk** (`.DS_Store`, `Thumbs.db`)
- **Editor files** (`.vscode/`, `.idea/`)

</div>

<div class="warning-box" data-title="Before you commit">

**Never commit secrets or credentials.** Once something is in Git history, it's very hard to truly remove. When in doubt, add it to `.gitignore` first.

</div>

---
<!-- _class: scale-90 -->

# Summary

<div class="tip-box" data-title="Key takeaways">

**Programming practice:**
- You learn by doing — work through the [companion notebook](programming_basics.ipynb) exercises
- Read code out loud when debugging
- Test with small examples
- Use AI to explain, debug, and unstick yourself

**Git and GitHub:**
- Git tracks changes; GitHub hosts repos online
- The daily workflow: pull → edit → commit → push
- Write clear commit messages (future you will thank you)
- Use pull requests to propose changes and discuss them
- Use issues to track tasks, bugs, and questions
- Never commit secrets

</div>

---

# Questions? Want to chat more?

<div class="emoji-figure">
  <div class="emoji-col">
    <span class="emoji emoji-xl emoji-bg emoji-bg-navy">&#x1F4E7;</span>
    <span class="label"><a href="mailto:jeremy@dartmouth.edu">Email</a> me</span>
  </div>
  <div class="emoji-col">
    <span class="emoji emoji-xl emoji-bg emoji-bg-purple">&#x1F4AC;</span>
    <span class="label">Join our <a href="https://stories-about-data.slack.com">Slack</a></span>
  </div>
  <div class="emoji-col">
    <span class="emoji emoji-xl emoji-bg emoji-bg-green">&#x1F481;</span>
    <span class="label">Come to <a href="https://context-lab.com/scheduler">office hours</a></span>
  </div>
</div>

<div class="note-box" data-title="Up next...">

- **Thursday X-hour:** Introduction to vibe coding
- **Friday:** Assignment 3 brainstorm + release

</div>
