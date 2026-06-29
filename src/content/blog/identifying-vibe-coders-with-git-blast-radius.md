---
title: Find vibe code with git blast radius
description: A tiny git helper for finding commits that touched suspiciously many files.
pubDate: 2026-06-29
---

One practical way to spot “vibe coding” in a repository is to look for commits with an unusually large blast radius: many files touched, often with a vague subject line.

This is not proof of bad work. Migrations, formatting passes, dependency updates, and generated files can all be legitimate. But it is a good first filter for review.

```bash
# Commits that touched the most files.
# Usage:
#   gblast
#   gblast "1 year ago" 25
function gblast() {
  local since="${1:-1 year ago}"
  local limit="${2:-20}"

  git log --since="$since" \
    --pretty=format:'COMMIT|%h|%ad|%an|%s' \
    --date=short \
    --shortstat \
    | awk -F'|' '
      /^COMMIT\|/ {
        hash=$2
        date=$3
        author=$4
        subject=$5
        next
      }

      /files? changed/ {
        split($1, a, " ")
        files=a[1]
        printf "%5d files | %s | %-10s | %-22s | %s\n", files, hash, date, author, subject
      }
    ' \
    | sort -nr \
    | head -"$limit"
}
```

Run it before a review:

```bash
gblast "6 months ago" 30
```
