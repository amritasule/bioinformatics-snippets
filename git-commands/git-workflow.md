# Git Workflow Cheat Sheet

## Initial Setup (One-time)

### Configure Git
```bash
git config --global user.name "Your Name"
git config --global user.email "your.email@gmail.com"
```

### Check Configuration
```bash
git config --list
```

---

## Starting a New Project

### Clone an existing repository
```bash
git clone git@github.com:username/repo-name.git
cd repo-name
```

### Or initialize a new repository
```bash
mkdir my-project
cd my-project
git init
```

---

## Basic Workflow (Use Daily!)

### 1. Check status (see what changed)
```bash
git status
```

### 2. Add files to staging
```bash
git add filename.txt          # Add specific file
git add .                     # Add all changed files
```

### 3. Commit changes (save snapshot)
```bash
git commit -m "Description of what you changed"
```

### 4. Push to GitHub (upload)
```bash
git push
```

---

## Common Commands

### See commit history
```bash
git log
git log --oneline             # Condensed view
```

### Pull latest changes from GitHub
```bash
git pull
```

### See what changed in a file
```bash
git diff filename.txt
```

### Undo changes (before commit)
```bash
git checkout -- filename.txt   # Discard changes to a file
```

### Remove file from staging
```bash
git reset filename.txt
```

---

## Branch Management

### Create and switch to new branch
```bash
git checkout -b new-feature
```

### Switch between branches
```bash
git checkout main
git checkout other-branch
```

### List all branches
```bash
git branch
```

### Merge branch into current branch
```bash
git merge branch-name
```

---

## Quick Reference

| Command | What it does |
|---------|-------------|
| `git status` | Check what's changed |
| `git add .` | Stage all changes |
| `git commit -m "message"` | Save snapshot |
| `git push` | Upload to GitHub |
| `git pull` | Download from GitHub |
| `git log` | See history |

---

## Tips

- Commit often with clear messages
- Pull before you push (especially when collaborating)
- Use branches for new features
- Check `git status` frequently

---

**Created:** November 26, 2024
**Last Updated:** November 26, 2024
