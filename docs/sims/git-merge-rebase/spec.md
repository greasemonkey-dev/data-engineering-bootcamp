# MicroSim: Git Merge vs Rebase Interactive

## Overview
### Concept Visualized
- **Concept:** Merge vs Rebase (merge-vs-rebase), Git Merge (git-merge), Git Rebase (git-rebase)
- **Learning Goal:** Students understand the fundamental difference between merge and rebase by manipulating branch histories and observing how commit graphs diverge, helping them choose the right integration strategy
- **Difficulty:** Intermediate
- **Chapter:** Week 1-2 (Foundations & Environment Setup)

### The "Aha" Moment
When students click "Merge" vs "Rebase" on the same starting scenario, they see two completely different commit graph outcomes side-by-side: merge preserves history with a merge commit, while rebase creates a linear history by replaying commits, demonstrating why rebase makes cleaner history but merge is safer for shared branches.

## Interface Design
### Layout
- Left Panel (60%): Split into two side-by-side commit graphs (Merge result | Rebase result)
- Right Panel (40%): Scenario controls and strategy comparison

### Controls (Right Panel)
| Control | Type | Range/Options | Default | Effect |
|---------|------|---------------|---------|--------|
| Scenario | dropdown | "Simple Feature", "Multiple Commits", "Conflicting Changes" | "Simple Feature" | Loads different branch divergence scenarios |
| Show Merge Result | button | N/A | N/A | Animates merge operation on left graph |
| Show Rebase Result | button | N/A | N/A | Animates rebase operation on right graph |
| Show Both | button | N/A | N/A | Animates both operations simultaneously |
| Reset | button | N/A | N/A | Clears to initial state |
| Speed | slider | 0.5x - 2x | 1x | Controls animation playback speed |

**Strategy Comparison Panel:**
- Displays side-by-side comparison table after operation:
  - Commit count
  - History linearity (Linear/Non-linear)
  - Preserves original timestamps (Yes/No)
  - Creates merge commit (Yes/No)
  - Safe for shared branches (Yes/No)
  - When to use (description)

### Visualization (Left Panel)
**What is Displayed:**
- Two identical starting commit graphs showing:
  - main branch (blue circles) with 3 commits
  - feature branch (green circles) diverged after commit 2, with 2 commits
- Commit nodes: circles with SHA (e.g., "a1b2c3"), timestamp, and commit message
- Branch labels: colored tags showing "main" and "feature"
- Arrows showing parent relationships

**How it Updates:**
- **Merge animation** (left side):
  1. Feature branch moves toward main (1s)
  2. New merge commit node appears combining both branches (0.5s)
  3. Graph shows diamond pattern with merge commit having two parents
  4. Branch pointers update

- **Rebase animation** (right side):
  1. Feature commits lift off the branch (0.5s)
  2. Commits fade to transparent showing they're being "replayed" (0.5s)
  3. New commits with different SHAs appear on top of main (1s, sequential)
  4. Original feature commits disappear
  5. Linear history is formed

- Color coding: Blue (main), Green (feature), Purple (merge commit), Yellow (rebased commits with new SHA)
- Transitions: Smooth bezier curve movements, fade in/out for commits

## Educational Flow
### Step 1: Default State
Student sees initial scenario: "Simple Feature"
- main branch: commits A → B → C
- feature branch: diverged at B, has commits D → E
- Both graphs show identical starting state
- No operations performed yet

### Step 2: First Interaction
Prompt: "Click 'Show Both' to see how merge and rebase handle this scenario differently"

Result:
- Left graph: Shows merge creating commit F with two parents (C and E), diamond pattern
- Right graph: Shows commits D' and E' replayed on top of C, linear history
- Comparison table populates:
  - Merge: 6 commits (A,B,C,D,E,F), Non-linear, Yes preserves timestamps, Creates merge commit
  - Rebase: 5 commits (A,B,C,D',E'), Linear, No (new timestamps), No merge commit

Students observe: Same end result, different history structure

### Step 3: Exploration
Prompt: "Try the 'Multiple Commits' scenario - feature branch has 5 commits instead of 2"

Pattern they should discover:
- Merge: Always creates just one merge commit regardless of feature branch size
- Rebase: Replays each commit individually, creating N new commits
- Merge history shows "what actually happened" (when branches diverged)
- Rebase history shows "what could have happened" (if feature was built on latest main)
- SHA hashes change during rebase, showing commits are rewritten

### Step 4: Challenge
Scenario: "Conflicting Changes - both main and feature modified the same file"
Question: "Your feature branch has already been pushed to GitHub and your teammate pulled it. Should you use merge or rebase to integrate with main?"

Expected learning:
- Students should choose MERGE
- Understand that rebase rewrites history (changes SHAs)
- Rewriting pushed commits causes problems for collaborators
- Merge is safe for shared branches, rebase for local cleanup
- "Golden rule": Never rebase commits that have been pushed to shared branches

## Technical Specification
- **Library:** p5.js
- **Canvas:** Responsive, min 700px width × 400px height (350px per graph)
- **Frame Rate:** 30fps
- **Data:** Static JSON scenarios with commit graph structures
  ```json
  {
    "scenarios": [
      {
        "name": "Simple Feature",
        "main": ["A", "B", "C"],
        "feature": {"base": "B", "commits": ["D", "E"]}
      },
      // ... more scenarios
    ]
  }
  ```

## Assessment Integration
After using this MicroSim, students should answer:

1. **Knowledge Check:** What is the main difference between git merge and git rebase?
   - a) Merge is faster than rebase
   - b) Merge preserves commit history, rebase rewrites it ✓
   - c) Rebase creates a merge commit, merge doesn't
   - d) There is no difference, they're aliases

2. **Application:** You have a local feature branch with 3 commits that you haven't pushed yet. Main branch has advanced. What should you do?
   - a) Always use merge to be safe
   - b) Use rebase to keep history clean ✓
   - c) Delete and recreate the branch
   - d) Never integrate local branches

3. **Scenario-Based:** Your teammate says "I can't pull the feature branch anymore, getting conflicts on commits that were already there." What likely happened?
   - a) Someone merged main into feature
   - b) Someone rebased a shared branch ✓
   - c) GitHub is experiencing issues
   - d) The branch was deleted

## Extension Ideas
- Add interactive conflict resolution step showing how conflicts differ between merge/rebase
- Include git log output panel showing actual Git command results
- Add "Interactive Rebase" mode showing commit squashing and reordering
- Show remote tracking branches and how push --force-with-lease works
- Include team collaboration scenario with multiple developers
- Add "Undo" operations showing git reset vs git revert
- Visualize reflog showing how to recover from rebase mistakes
