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

1. **Two deep practice problems** — exploratory analysis you'll actually reuse
2. **Git and GitHub** — forking, the workflow you'll use for this course
3. **Everything you need** to submit Assignment 3 on Monday

</div>

<div class="tip-box" data-title="Follow along">

Open the [companion notebook in Colab](https://colab.research.google.com/github/ContextLab/storytelling-with-data/blob/master/slides/programming_basics.ipynb) so you can run every example yourself.

</div>

---

# Problem 1: exploring a real dataset

<div class="example-box" data-title="The task">

You have a list of songs, each a dict with `title`, `artist`, `genre`, and `plays`:

```python
songs = [
    {'title': 'Blinding Lights', 'artist': 'The Weeknd', 'genre': 'pop',  'plays': 4_200_000_000},
    {'title': 'Shape of You',    'artist': 'Ed Sheeran', 'genre': 'pop',  'plays': 3_800_000_000},
    {'title': 'Bohemian Rhapsody','artist': 'Queen',     'genre': 'rock', 'plays': 2_400_000_000},
    # ... many more ...
]
```

**Find the three genres with the highest *average* plays per song.**

</div>

<div class="tip-box" data-title="Think first">

This looks simple but it's a complete analysis pipeline: **group**, **aggregate**, **sort**, **slice**. You'll likely do some version of this in every Part II project.

</div>

---

# Problem 1: one reasonable solution

<style scoped>
section pre { font-size: 0.65em; line-height: 1.25; margin-bottom: 0.5em; }
</style>

```python
# Step 1: group plays by genre
plays_by_genre = {}
for song in songs:
    g = song['genre']
    plays_by_genre.setdefault(g, []).append(song['plays'])

# Step 2: compute the average plays per genre
avg_by_genre = {
    g: sum(plays) / len(plays)
    for g, plays in plays_by_genre.items()
}

# Step 3: sort genres by average plays, descending
ranked = sorted(avg_by_genre.items(), key=lambda item: -item[1])

# Step 4: take the top 3
top_three = ranked[:3]
print(top_three)
```

<div class="note-box" data-title="What to notice">

Every step is named and small. When something breaks, you can `print` any intermediate variable and see exactly what's happening. **This is the pattern of all data analysis.**

</div>

---

# Problem 2: find and fix the bug

<div class="example-box" data-title="What's wrong here?">

This function is supposed to return a dict mapping each student to their average grade. It runs without errors — but the numbers are wrong.

```python
def student_averages(grades):
    result = {}
    for student, scores in grades.items():
        total = 0
        for s in scores:
            total += s
        result[student] = total / len(grades)
    return result

grades = {'Ada': [90, 85, 92], 'Grace': [78, 88], 'Alan': [95, 91, 88, 84]}
print(student_averages(grades))
```

</div>

---

# Problem 2: the bug

```python
result[student] = total / len(grades)   # BUG: divides by # of students
result[student] = total / len(scores)   # FIX: divide by # of scores
```

<div class="warning-box" data-title="Why this is sneaky">

The code **runs** — no error, no crash. It just returns wrong numbers. These are the hardest bugs to catch. **Always test with a small example you can verify by hand.**

Ada has three scores (90, 85, 92) averaging **89**. The buggy function returns `267/3 = 89` by coincidence — but Grace's average comes out wrong (`166/3 = 55.3` instead of `166/2 = 83`).

</div>

---

# Part 2: Git and GitHub

<div class="important-box" data-title="Why it matters">

Every data scientist uses Git and GitHub — **daily**. For this course, you'll use GitHub to:

- Fork the course repo to your own account
- Submit every assignment via a pull request
- Collaborate on your Part II data stories
- Track issues, discussions, and feedback

</div>

---

# Git vs GitHub

<div class="definition-box" data-title="Two different things with similar names">

**Git** is a *tool* on your computer that tracks changes to files in a folder (a **repository**).

**GitHub** is a *website* that hosts Git repositories online — where you share, collaborate, and back up your work.

</div>

<div class="tip-box" data-title="Sign up">

You need a free [GitHub account](https://github.com). Use your Dartmouth email and pick a professional username.

</div>

---

# Key terms

<div class="definition-box" data-title="The vocabulary you'll use every day">

- **Repository** ("repo") — a project folder tracked by Git
- **Fork** — your own copy of someone else's repo on GitHub
- **Clone** — download a repo from GitHub to your computer
- **Commit** — a snapshot of your project with a message describing the change
- **Push** — upload your commits to GitHub
- **Pull** — download the latest changes from GitHub
- **Pull request (PR)** — a proposal to merge your fork's changes into the original repo
- **Issue** — a tracked task, bug, or question

</div>

---

# The forking workflow

<div class="example-box" data-title="How you'll work with the course repo">

1. **Fork** the course repo on GitHub — creates `yourname/storytelling-with-data`
2. **Clone** your fork to your computer: `git clone <your-fork-url>`
3. **Edit** files, add your assignment work
4. **Commit** with a clear message: `git commit -m "add assignment 3 demo"`
5. **Push** commits to your fork: `git push`
6. **Open a pull request** on GitHub, proposing your changes to the course repo

</div>

<div class="tip-box" data-title="AI handles the commands">

Claude Code runs all of these for you — your job is to **review what it did** and confirm the commit messages are clear.

</div>

---

# Good commit messages

<div class="tip-box" data-title="Clear messages save time">

**Bad:** `update stuff` · `fixed it` · `asdf`

**Good:**
- `add word frequency function to text_utils.py`
- `fix off-by-one error in average() calculation`
- `update README with installation instructions`

**The rule:** if you had to describe this commit to a teammate in one sentence, what would you say?

</div>

---

# Merge conflicts: don't panic

<div class="warning-box" data-title="When Git can't automatically merge">

A **merge conflict** happens when two edits touch the *same lines* of the *same file*. Git asks you to pick:

```
<<<<<<< HEAD
version from main
=======
version from your branch
>>>>>>> my-changes
```

**Steps:** open the file, decide what the final version should be, delete the `<<<`, `===`, `>>>` markers, save, commit, push.

</div>

<div class="tip-box" data-title="Ask for help">

Claude Code is excellent at resolving merge conflicts — show it the conflict and ask which version to keep.

</div>

---

# `.gitignore`: keep junk and secrets out

<div class="tip-box" data-title="What to ignore">

A `.gitignore` file lists files Git should **never track**:

- **Large data files** (CSVs, images, raw datasets)
- **Secrets** (API keys, passwords, `.env` files)
- **Build artifacts** and caches (`__pycache__/`, `.ipynb_checkpoints/`)
- **System junk** (`.DS_Store`, `.vscode/`)

</div>

<div class="warning-box" data-title="Never commit secrets">

Once something is in Git history, it's very hard to remove. **When in doubt, add it to `.gitignore` first.**

</div>

---

# Summary

<div class="tip-box" data-title="Key takeaways">

**Programming:**
- Name every step, test with small examples, read code out loud when debugging
- Use AI to explain, debug, and unstick yourself — but verify what it produces

**Git and GitHub:**
- **Fork** the course repo, clone your fork, commit, push, open a PR
- Write clear commit messages (future you will thank you)
- Never commit secrets — use `.gitignore`

**Get started now:** fork the [course repo](https://github.com/ContextLab/storytelling-with-data) and clone it before Friday.

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
