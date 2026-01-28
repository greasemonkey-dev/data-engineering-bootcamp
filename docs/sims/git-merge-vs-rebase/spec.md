# MicroSim: Git Merge vs Rebase Interactive

## Overview

### Concept Visualized
- **Concept:** Git branching strategies, merge vs rebase workflows, commit history management
- **Learning Goal:** Students will understand the fundamental difference between merge and rebase by manipulating a visual commit graph and observing how each operation creates different history structures
- **Difficulty:** Intermediate
- **Chapter:** Week 1-2 - Foundations (Git/GitHub workflows)

### The "Aha" Moment
When the student clicks "Rebase" after seeing the branched history, they watch commits literally lift off the old base and replay one-by-one onto the new base, creating a linear history. Then they try "Merge" and see it preserves the branch structure with a merge commit joining the two histories. This demonstrates: **rebase rewrites history linearly, merge preserves history truthfully**.

## Interface Design

### Layout
- **Left Panel (60% width):** Commit graph visualization showing branches, commits, and their relationships
- **Right Panel (40% width):** Interactive controls, scenario selector, and educational explanations

### Controls (Right Panel)

| Control | Type | Range/Options | Default | Effect on Visualization |
|---------|------|---|---------|------------------------|
| Scenario | dropdown | ["Simple Feature Branch", "Long-Running Feature", "Merge Conflict Preview", "Multiple Contributors"] | "Simple Feature Branch" | Loads different starting commit graphs |
| Action | button group | ["Merge", "Rebase", "Reset"] | - | Triggers animated transformation of commit graph |
| Show Command | toggle | on/off | on | Displays equivalent git commands below graph |
| Speed | slider | 0.5x-2x | 1x | Controls animation playback speed |
| Step Through | button | - | - | Pauses animation and allows frame-by-frame stepping |
| History View | tabs | ["Graph View", "Log View", "Timeline"] | "Graph View" | Changes visualization style |

**Additional UI Elements:**
- **Command Display:** Shows git commands like `git merge feature-branch` or `git rebase main`
- **Info Panel:** Explains what's happening at each animation step
- **Pros/Cons Table:** Updates to show trade-offs of current action

### Visualization Details (Left Panel)

**What is Displayed:**

**Graph View (Default):**
- **Commits:** Circles containing commit hash (first 7 chars: `a3f2c8b`)
- **Commit Metadata:** Author icon, timestamp, message snippet on hover
- **Branches:** Colored lines connecting commits
  - `main` branch: Blue line
  - `feature` branch: Green line
  - Other branches: Purple/orange for multi-contributor scenarios
- **Branch Labels:** Rounded rectangles with branch names at HEAD position
- **Commit Parents:** Arrows showing parent relationships (usually pointing left/down)
- **Merge Commits:** Diamond shape (two parents) vs regular circle (one parent)
- **Current HEAD:** Bold outline with "HEAD" label

**Color Coding:**
- Blue: main/master branch commits
- Green: feature branch commits
- Yellow: Commits being moved during rebase animation
- Red: Conflict indicators
- Gray: Commits that will be "abandoned" during rebase (old versions)

**Log View:**
- Text-based commit log similar to `git log --oneline --graph`
- ASCII art showing branch structure
- Updates in real-time as operations execute

**Timeline View:**
- Horizontal timeline with commits as events
- Shows chronological order (merge preserves, rebase changes)

**How it Updates:**

**When "Merge" button clicked:**
1. Pause (500ms) with highlight on both branch tips
2. Arrow animates from feature branch HEAD to main branch HEAD (500ms)
3. New merge commit (diamond shape) appears at convergence point (300ms)
4. Merge commit labeled with message: "Merge branch 'feature-branch' into main"
5. Both branches now point to merge commit
6. Fade previous branch pointers (300ms)
7. Final state: Y-shaped history preserved, feature commits retain original hashes

**When "Rebase" button clicked:**
1. Highlight feature branch commits that will be moved (yellow glow, 500ms)
2. Show "ghost" versions of commits staying in place (fade to 30% opacity)
3. Animated "lifting" of commits off old base (800ms):
   - Commits float upward with slight rotation
   - Break parent connection arrows
4. Reposition commits horizontally to align with main branch tip (600ms)
5. Replay commits one-by-one onto new base (400ms each):
   - Each commit "lands" with small bounce animation
   - Hash changes (e.g., `a3f2c8b` → `f7e9d1a`) to show they're new commits
   - Parent arrow connects to previous commit
6. Ghost commits fade away completely (300ms)
7. Final state: Linear history, feature branch points to tip, no merge commit

**When "Reset" button clicked:**
- Smooth reverse animation back to initial state (1 second)

**Visual Feedback:**
- Hover states: Commits enlarge slightly and show detailed tooltip (message, author, date, full hash)
- Active states: Currently animating commits have pulsing yellow outline
- Conflict indicators: If "Merge Conflict Preview" scenario, red exclamation mark appears on conflicting commits

## Educational Flow

### Step 1: Default State
Student sees **Simple Feature Branch** scenario:
- `main` branch: 4 commits (A → B → C → D)
- `feature` branch: Split from C, has 2 commits (C → E → F)
- Current state: Two diverged branches

Info panel shows: "You've been working on a feature while main branch has advanced. Your branch is 2 commits ahead, 1 commit behind main. How should you integrate your changes?"

This shows the classic integration problem every developer faces.

### Step 2: First Interaction
**Prompt:** "Try clicking 'Merge' to see what happens. Watch how the history changes."

Student clicks Merge button.

**Result:**
- Animation shows merge commit (M) connecting F and D
- Final graph: A → B → C → D → M
                   └→ E → F ↗
- History preserves that feature development happened in parallel
- Command display shows: `git merge feature-branch`

**Info panel explains:** "Merge creates a new commit (M) with two parents, preserving the true history. Anyone can see that feature work happened separately. This is the 'safe' option—it never changes existing commits."

**Prompt:** "Click 'Reset' and then try 'Rebase' instead."

Student resets, then clicks Rebase.

**Result:**
- Animation shows commits E and F lifting off, moving right, landing after D
- Final graph: A → B → C → D → E' → F' (linear)
- Commits E' and F' have new hashes (rewritten)
- Command display shows: `git rebase main`

**Info panel explains:** "Rebase replays your commits on top of main's latest state. The history is linear—it looks like you started your feature work after commit D. Your original commits E and F are replaced with new versions E' and F'. Cleaner history, but history is rewritten."

**What it teaches:**
- Merge preserves true history, rebase creates clean linear history
- Rebase changes commit hashes (creates new commits)
- Both achieve integration, but create different history structures

### Step 3: Exploration
**Prompt:** "Switch scenarios to 'Long-Running Feature' and try both approaches. Notice the difference in how messy the merge looks vs how clean the rebase looks."

Student selects "Long-Running Feature" scenario:
- `main` branch: 10 commits
- `feature` branch: Split early, has 8 commits, with main advancing 6 commits since split
- Many interleaved development

Student tries merge: Sees complex graph with many crossing lines and merge commit
Student tries rebase: Sees clean linear progression

**Key insight:** Rebase is especially valuable for long-running branches—it makes history readable. But rebase rewrites more commits (all 8 feature commits get new hashes), which is risky if others are working on the same branch.

**Prompt:** "Try 'Multiple Contributors' scenario. What happens if two people are working on the same feature branch?"

Student selects scenario showing shared feature branch.

**Info panel warns:** "Rebase is dangerous on shared branches! If you rebase and force-push, anyone else who has the old commits will have a 'diverged branch' problem. Only rebase local branches or after coordinating with team."

**Key insight:** Rebase is safe for personal feature branches, dangerous for shared branches. This is the golden rule of rebasing.

### Step 4: Challenge
**Present scenario:** "You're on a team following trunk-based development. The team policy is: main must have linear history (no merge commits). You've finished your feature branch with 5 commits. But main has advanced 3 commits since you branched. How do you integrate?"

Student must:
1. Select appropriate scenario (or use custom state)
2. Choose Rebase (only option that creates linear history)
3. Observe that rebase makes it look like feature work happened after main's advances

**Follow-up question shown:** "What command would you run before creating a pull request to make sure your branch is up-to-date?"

Student should answer: `git rebase main` (or click Rebase button)

**Advanced challenge:** "After rebasing, you try to push but get 'rejected' error. Why? What flag do you need?"

**Expected learning:**
- After rebase, must force-push because history is rewritten: `git push --force-with-lease`
- `--force-with-lease` is safer than `--force` (checks that remote hasn't changed)
- This is why rebasing shared branches is dangerous—force-pushing affects others

## Technical Specification

### Technology
- **Library:** p5.js for graph rendering and animations
- **Canvas:** Responsive, min 600px width × 400px height, scales to viewport
- **Frame Rate:** 60fps for smooth commit movement animations
- **Data:** Commit graph stored as directed acyclic graph (DAG) in JSON format

### Implementation Notes

**Mobile Considerations:**
- Touch-friendly buttons (min 44px × 44px)
- On screens < 768px: Stack layout (graph above controls)
- Pinch-to-zoom on graph for detailed inspection
- Simplified animations on mobile (reduce particle effects, simpler transitions)

**Accessibility:**
- Keyboard navigation:
  - Tab through buttons and controls
  - Arrow keys to navigate commits in graph
  - Enter/Space to activate buttons
  - Escape to pause/reset animation
- ARIA live regions announce animation progress: "Replaying commit E onto main..."
- Screen reader describes graph structure: "Main branch has 4 commits. Feature branch diverged at commit C with 2 additional commits."
- High contrast mode option (removes subtle gradients, increases stroke width)
- Text alternatives for all visual elements

**Performance:**
- For graphs with >20 commits, use virtualization (render only visible portion + buffer)
- Use requestAnimationFrame for smooth animations
- Separate animation loop from interaction handlers
- Memoize graph layout calculations (only recalculate on structural changes)

### Data Requirements

**Commit Graph Format (JSON):**

```json
{
  "scenarios": {
    "simple_feature": {
      "name": "Simple Feature Branch",
      "description": "Basic case: feature branch and main have diverged",
      "commits": [
        {"id": "a3f2c8b", "message": "Initial commit", "author": "Alice", "timestamp": "2024-01-15T10:00:00Z", "parents": []},
        {"id": "b7e9d3a", "message": "Add database schema", "author": "Bob", "timestamp": "2024-01-16T11:30:00Z", "parents": ["a3f2c8b"]},
        {"id": "c4f8e2d", "message": "Add user model", "author": "Alice", "timestamp": "2024-01-17T09:15:00Z", "parents": ["b7e9d3a"]},
        {"id": "d2a7f9c", "message": "Update README", "author": "Charlie", "timestamp": "2024-01-18T14:20:00Z", "parents": ["c4f8e2d"]},
        {"id": "e9b3d1f", "message": "Start auth feature", "author": "Alice", "timestamp": "2024-01-17T15:00:00Z", "parents": ["c4f8e2d"]},
        {"id": "f1c5e7b", "message": "Complete auth feature", "author": "Alice", "timestamp": "2024-01-18T16:45:00Z", "parents": ["e9b3d1f"]}
      ],
      "branches": {
        "main": {"current_commit": "d2a7f9c", "color": "#4A90E2"},
        "feature-auth": {"current_commit": "f1c5e7b", "color": "#7ED321"}
      },
      "head": "feature-auth"
    }
  },
  "merge_algorithm": {
    "create_merge_commit": true,
    "merge_commit_message": "Merge branch '{branch}' into {target}",
    "merge_commit_parents": ["target_head", "branch_head"]
  },
  "rebase_algorithm": {
    "identify_commits_to_replay": "commits reachable from branch HEAD but not from target",
    "replay_order": "chronological by original commit timestamp",
    "generate_new_hash": "simulate with hash(parent_hash + commit_message + timestamp_new)"
  }
}
```

**Animation Timeline Definition:**
```json
{
  "merge_animation": [
    {"step": 1, "duration": 500, "action": "highlight_both_heads"},
    {"step": 2, "duration": 500, "action": "animate_arrow", "from": "branch_head", "to": "target_head"},
    {"step": 3, "duration": 300, "action": "create_merge_commit", "shape": "diamond"},
    {"step": 4, "duration": 300, "action": "update_branch_pointer"}
  ],
  "rebase_animation": [
    {"step": 1, "duration": 500, "action": "highlight_commits_to_move", "color": "yellow"},
    {"step": 2, "duration": 300, "action": "create_ghost_commits", "opacity": 0.3},
    {"step": 3, "duration": 800, "action": "lift_commits", "direction": "up", "distance": 50},
    {"step": 4, "duration": 600, "action": "reposition_commits", "align_with": "target_head"},
    {"step": 5, "duration": 400, "action": "replay_commit", "effect": "bounce", "repeat_for_each": true},
    {"step": 6, "duration": 300, "action": "fade_ghost_commits"}
  ]
}
```

## Assessment Integration

After using this MicroSim, students should be able to answer:

1. **Quiz Question:** "Your team uses pull requests and shared feature branches. You want to clean up your branch before merging to main. Should you use merge or rebase?"
   - A) Merge—always safe ✓
   - B) Rebase—cleaner history
   - C) Either one works equally well
   - D) Force push to main directly

   **Answer: A** (Rebase on shared branches requires force-push, affecting team members)

2. **Conceptual Question:** "After rebasing your feature branch onto main, you try `git push` but get rejected. Why? What command should you use?"
   - Expected answer: Rebase rewrites history (changes commit hashes), so remote and local have diverged. Must use `git push --force-with-lease` to update remote with rewritten history.

3. **Trade-off Question:** "What is the main advantage of merge over rebase? What is the main advantage of rebase over merge?"
   - Expected answer:
     - Merge advantage: Preserves true history, safe for shared branches, never changes existing commits
     - Rebase advantage: Creates clean linear history, easier to understand project evolution, avoids "merge commit soup"

4. **Scenario Question:** "You just rebased your branch and realized you made a mistake. Can you undo a rebase? How?"
   - Expected answer: Yes, use `git reflog` to find commit before rebase, then `git reset --hard <commit>`. Reflog tracks HEAD movements even after rebase.

## Extension Ideas (Optional)

- **Interactive Rebase Simulation:** Show `git rebase -i` functionality where students can reorder commits, squash commits together, or edit commit messages during rebase
- **Conflict Resolution Preview:** Animate what happens when rebase encounters conflicts (pauses at conflicting commit, shows resolution options)
- **Cherry-Pick Visualization:** Show how `git cherry-pick` works (similar to rebase but for individual commits)
- **Fork Point Detection:** Highlight the "merge base" (last common ancestor) between branches—the point where rebase starts replaying
- **Team Workflow Comparison:** Show side-by-side comparison of different team strategies:
  - "Merge everything" workflow (preserves all history)
  - "Rebase before PR" workflow (clean feature branches, merge to main)
  - "Rebase always" workflow (fully linear history)
- **Force-Push Disaster Scenario:** Demonstrate what happens when someone rebases a shared branch without coordination (show teammate's broken state)
- **Reflog Explorer:** Visualization of Git reflog showing how to recover from mistakes
- **Bisect Integration:** Show how linear history (from rebase) makes `git bisect` more effective for finding bugs

---

**Target Learning Outcome:** Students understand that merge and rebase are different tools for integration with different trade-offs, and can choose the appropriate strategy based on branch sharing status and team workflow preferences. Students recognize that rebase rewrites history and requires force-pushing, making it unsuitable for shared branches.
