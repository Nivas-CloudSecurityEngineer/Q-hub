<div align="center" markdown="1">

# 🔀 Git & GitHub
### 🎤 Interview Questions & Answers

![Category](https://img.shields.io/badge/Category-Git_%26_GitHub-blue?style=for-the-badge&logo=github&logoColor=white)
![Level](https://img.shields.io/badge/Level-Beginner_to_Advanced-yellow?style=for-the-badge)
![Focus](https://img.shields.io/badge/Focus-Real--Time%20Scenarios-success?style=for-the-badge)

</div>

---

> 💡 **Pro Tip:** Skim the 🧩 fundamentals first, then master the 🎯 real-time scenarios — that's where interviews are actually won or lost.

---

## 🧩 Fundamentals

<details markdown="1">
<summary>❓ <b>1. What is Git and how does it differ from a centralized VCS like SVN?</b></summary>
<br>

Git is a distributed version control system where every developer has a full copy of the repository history locally, enabling offline commits, faster operations, and branching/merging without contacting a central server - unlike SVN's centralized model where the history lives only on a central server.

</details>

<details markdown="1">
<summary>❓ <b>2. What is the difference between Git and GitHub?</b></summary>
<br>

Git is the version control tool/software itself (works locally, protocol-agnostic). GitHub is a cloud-hosted platform that hosts Git repositories and adds collaboration features (pull requests, issues, Actions CI/CD, code review, project management) on top of Git.

</details>

<details markdown="1">
<summary>❓ <b>3. Explain the three main areas in Git: Working Directory, Staging Area, and Repository.</b></summary>
<br>

Working Directory: your actual files as you edit them. Staging Area (Index): a snapshot of changes you've marked (`git add`) to be included in the next commit. Repository (`.git` folder): the committed history/objects stored permanently once you `git commit`.

</details>

<details markdown="1">
<summary>❓ <b>4. What is a commit and what does a commit hash represent?</b></summary>
<br>

A commit is a snapshot of the repository at a point in time, along with metadata (author, message, timestamp, parent commit(s)). The commit hash (SHA-1, or SHA-256 in newer Git) is a unique fingerprint of the commit's content and history, used to reference it uniquely.

</details>

## 🌿 Branching & Merging

<details markdown="1">
<summary>❓ <b>5. What is a branch in Git?</b></summary>
<br>

A lightweight, movable pointer to a specific commit, allowing parallel lines of development (e.g., feature branches) without affecting the main codebase until merged.

</details>

<details markdown="1">
<summary>❓ <b>6. Difference between `git merge` and `git rebase`.</b></summary>
<br>

Merge combines two branches by creating a new merge commit that has both branches' histories as parents, preserving the exact history (including all commits from both) but resulting in a non-linear history. Rebase replays your branch's commits on top of another branch's tip, creating a clean, linear history, but rewrites commit hashes - which is why you should never rebase commits already pushed/shared with others.

</details>

<details markdown="1">
<summary>❓ <b>7. What is a merge conflict and how do you resolve it?</b></summary>
<br>

Occurs when Git can't automatically reconcile changes to the same lines/file between two branches being merged. Resolve by manually editing the conflicted file(s) (marked with `<<<<<<<`, `=======`, `>>>>>>>` markers), choosing/combining the correct content, then `git add` the resolved file(s) and complete the merge/rebase with `git commit` or `git rebase --continue`.

</details>

<details markdown="1">
<summary>🎯 <b>8. Scenario: You accidentally committed to `main` directly instead of a feature branch. How do you fix this without losing work?</b></summary>
<br>

Create a new branch at the current commit (`git branch feature-x`), then reset `main` back to before your commit (`git reset --hard origin/main` or the last shared commit), and continue your work on `feature-x`, later opening a PR to merge it properly.

</details>

<details markdown="1">
<summary>❓ <b>9. What is `git cherry-pick` and when would you use it?</b></summary>
<br>

Applies a specific commit from one branch onto another without merging the entire branch - useful for backporting a critical bugfix to a release branch without pulling in unrelated changes from the source branch.

</details>

<details markdown="1">
<summary>❓ <b>10. Difference between Fast-Forward merge and a 3-way merge.</b></summary>
<br>

Fast-Forward merge happens when the target branch has no new commits since the feature branch diverged - Git simply moves the branch pointer forward, no merge commit created. 3-way merge occurs when both branches have diverged with new commits - Git uses the common ancestor plus both branch tips to create a new merge commit.

</details>

## ⏪ Undoing Changes

<details markdown="1">
<summary>❓ <b>11. Difference between `git reset`, `git revert`, and `git checkout` (for undoing changes).</b></summary>
<br>

`git reset` moves the branch pointer (and optionally staging/working directory) to a previous commit, rewriting history - dangerous on shared/pushed branches. `git revert` creates a NEW commit that undoes the changes of a previous commit, preserving history - safe for shared branches. `git checkout` (or `restore` in newer Git) can discard uncommitted changes in the working directory to match a specific commit/branch state.

</details>

<details markdown="1">
<summary>❓ <b>12. Difference between `git reset --soft`, `--mixed`, and `--hard`.</b></summary>
<br>

`--soft`: moves HEAD/branch pointer only, keeps changes staged. `--mixed` (default): moves HEAD, unstages changes but keeps them in the working directory. `--hard`: moves HEAD and discards all changes in staging AND working directory - unrecoverable unless referenced elsewhere (e.g., reflog).

</details>

<details markdown="1">
<summary>🎯 <b>13. Scenario: You need to undo a commit that's already been pushed and pulled by teammates. Which command do you use and why?</b></summary>
<br>

`git revert <commit>` - because it creates a new commit undoing the changes without rewriting history, which is safe for shared branches (unlike `git reset --hard` followed by a force push, which would rewrite history and break everyone else's local clones).

</details>

<details markdown="1">
<summary>❓ <b>14. What is `git reflog` and when would it save you?</b></summary>
<br>

A log of every place HEAD has pointed to (commits, resets, checkouts, rebases), even ones no longer reachable from any branch. It's a safety net to recover "lost" commits after a bad `reset --hard` or an accidentally deleted branch, as long as garbage collection hasn't run yet.

</details>

## 🧹 Stashing & Cleaning

<details markdown="1">
<summary>❓ <b>15. What is `git stash` used for?</b></summary>
<br>

Temporarily saves uncommitted changes (staged and/or unstaged) so you can switch branches/pull cleanly, then reapply them later with `git stash pop`/`git stash apply` - useful when you need to quickly switch context without committing incomplete work.

</details>

<details markdown="1">
<summary>🎯 <b>16. Scenario: You have uncommitted work but need to urgently switch branches to fix a production bug. What do you do?</b></summary>
<br>

`git stash` to save your current changes and get a clean working directory, switch branches (`git checkout main`), fix the bug, commit/push, switch back to your feature branch, then `git stash pop` to restore your work.

</details>

## 🤝 Remote Operations & Collaboration

<details markdown="1">
<summary>❓ <b>17. Difference between `git fetch` and `git pull`.</b></summary>
<br>

`git fetch` downloads new commits/branches from the remote without merging them into your current branch (safe, just updates your local view of the remote). `git pull` is `fetch` + `merge` (or `rebase` with `--rebase`) combined, automatically integrating the remote changes into your current branch.

</details>

<details markdown="1">
<summary>❓ <b>18. What is a Pull Request (PR) / Merge Request and its purpose in GitHub workflows?</b></summary>
<br>

A mechanism to propose merging changes from one branch (often a feature branch/fork) into another, enabling code review, automated CI checks, and discussion before the changes are integrated - central to collaborative Git workflows and code quality gates.

</details>

<details markdown="1">
<summary>❓ <b>19. What is the difference between Git Flow, GitHub Flow, and Trunk-Based Development?</b></summary>
<br>

Git Flow: multiple long-lived branches (main, develop, feature, release, hotfix) - structured but complex, suited for scheduled releases. GitHub Flow: simpler - `main` is always deployable, feature branches merge via PR directly to main, suited for continuous deployment. Trunk-Based Development: developers commit small changes directly to `main` (or very short-lived branches) frequently, relying on feature flags for incomplete work - suited for high-velocity CI/CD teams.

</details>

<details markdown="1">
<summary>🎯 <b>20. Scenario: Your team deploys to production multiple times a day. Which branching strategy fits best and why?</b></summary>
<br>

Trunk-Based Development (or GitHub Flow) - since Git Flow's long-lived branches and release cycles introduce merge overhead/delay that conflicts with high-frequency continuous deployment; trunk-based development with feature flags allows incomplete features to be merged safely without blocking releases.

</details>

## 🚀 Advanced Git

<details markdown="1">
<summary>❓ <b>21. What is `git rebase -i` (interactive rebase) used for?</b></summary>
<br>

Allows rewriting commit history before it's shared - squashing multiple commits into one, reordering commits, editing commit messages, or dropping commits entirely, commonly used to clean up a feature branch's history before opening a PR.

</details>

<details markdown="1">
<summary>🎯 <b>22. Scenario: You have 10 messy "WIP" commits on your feature branch and want a single clean commit before merging. How?</b></summary>
<br>

`git rebase -i HEAD~10` and mark the commits to `squash`/`fixup` into the first one, then edit the resulting commit message to be clear and descriptive before pushing (force-push if already pushed to your own feature branch).

</details>

<details markdown="1">
<summary>❓ <b>23. What is a detached HEAD state and how do you recover from it?</b></summary>
<br>

Occurs when you check out a specific commit (not a branch), so HEAD points directly to that commit rather than a branch reference - any new commits made here aren't attached to a branch and can be "lost" once you check out elsewhere. Recover by creating a new branch at that point (`git branch new-branch`) before switching away, to preserve any commits made.

</details>

<details markdown="1">
<summary>❓ <b>24. What are Git Hooks and give an example use case.</b></summary>
<br>

Scripts that Git executes automatically at certain points (`pre-commit`, `pre-push`, `commit-msg`, etc.). Example: a `pre-commit` hook running linters/tests before allowing a commit, or a `commit-msg` hook enforcing a commit message convention (e.g., Conventional Commits).

</details>

<details markdown="1">
<summary>❓ <b>25. What is `.gitignore` and why is it important?</b></summary>
<br>

A file specifying patterns of files/directories Git should not track (e.g., `node_modules/`, `.env`, build artifacts) - prevents accidentally committing sensitive files, build outputs, or OS/IDE-specific clutter into the repository.

</details>

<details markdown="1">
<summary>🎯 <b>26. Scenario: A sensitive credential file was accidentally committed and pushed to a shared repo. How do you fully remove it from history?</b></summary>
<br>

Simply deleting the file in a new commit isn't enough since it remains in history. Use `git filter-repo` (recommended, replacing the older `filter-branch`) or BFG Repo-Cleaner to rewrite history and strip the file from all commits, then force-push, and critically, rotate/invalidate the exposed credential immediately since it may already be compromised/cached elsewhere.

</details>

<details markdown="1">
<summary>❓ <b>27. What is the difference between a Git submodule and a monorepo approach?</b></summary>
<br>

A submodule embeds another Git repository as a subdirectory with its own independent history/commits, referenced by a specific commit pointer in the parent repo - allows sharing code across repos but adds complexity (developers must manage submodule updates explicitly). A monorepo keeps multiple projects/services in a single repository, simplifying cross-project changes/atomic commits but requiring tooling to manage build/CI scope efficiently at scale.

</details>

<details markdown="1">
<summary>❓ <b>28. What is `git bisect` used for?</b></summary>
<br>

A binary-search tool to find the exact commit that introduced a bug, by marking known good/bad commits and letting Git narrow down the range by checking out midpoints for you to test, dramatically speeding up root-cause identification in large histories.

</details>

## 🐙 GitHub-Specific

<details markdown="1">
<summary>❓ <b>29. What are GitHub Actions?</b></summary>
<br>

GitHub's native CI/CD automation platform, allowing you to define workflows (YAML files in `.github/workflows/`) triggered by repository events (push, PR, schedule) to build, test, and deploy code, using reusable "Actions" from the marketplace or custom scripts.

</details>

<details markdown="1">
<summary>❓ <b>30. What is branch protection in GitHub and why use it?</b></summary>
<br>

Rules applied to a branch (e.g., `main`) requiring PR reviews, passing status checks (CI), signed commits, or preventing force-pushes/direct pushes - enforcing code quality and review discipline before code can be merged into critical branches.

</details>

<details markdown="1">
<summary>🎯 <b>31. Scenario: You want to ensure no one can merge code to `main` without at least one approval and passing tests. How do you configure this in GitHub?</b></summary>
<br>

Enable Branch Protection Rules on `main`: require pull request reviews before merging (set minimum approvals), require status checks to pass (linked to your CI workflow), and optionally require branches to be up to date before merging and disallow force pushes.

</details>

<details markdown="1">
<summary>❓ <b>32. What is the difference between a Fork and a Branch?</b></summary>
<br>

A branch is a parallel line of development within the SAME repository. A fork is a full copy of an entire repository under a different owner/account, typically used for open-source contribution workflows where external contributors don't have direct write access to the original repo and submit changes via a PR from their fork.

</details>

<details markdown="1">
<summary>❓ <b>33. What are GitHub Environments and how are they used in deployment workflows?</b></summary>
<br>

Named deployment targets (e.g., `staging`, `production`) in GitHub Actions that can have protection rules (required reviewers before deploying, wait timers, restricted secrets/branches) - used to gate sensitive deployments (e.g., requiring manual approval before deploying to production).

</details>

<details markdown="1">
<summary>🎯 <b>34. Scenario: Multiple developers keep having merge conflicts because everyone works off long-lived feature branches for weeks. How do you address this at a process level?</b></summary>
<br>

Encourage smaller, more frequent PRs merged quickly (trunk-based development principles), use feature flags to merge incomplete work safely without exposing it to users, and regularly rebase/merge `main` into long-running branches to reduce drift and conflict size over time.

</details>

<details markdown="1">
<summary>❓ <b>35. What is the difference between SSH and HTTPS authentication when working with GitHub, and which is generally preferred for automation?</b></summary>
<br>

HTTPS uses a username/Personal Access Token (or OAuth) for authentication over standard HTTPS, easier to set up but requires token management. SSH uses key-pairs, avoiding the need to enter/store credentials on each push and is often preferred for developer workstations and CI/CD automation (deploy keys) since it integrates cleanly with existing SSH agent/key infrastructure.

</details>

<details markdown="1">
<summary>🎯 <b>36. Scenario: A CI/CD pipeline needs to clone a private GitHub repository securely without embedding long-lived credentials. How?</b></summary>
<br>

Use a GitHub App installation token or OIDC federation (e.g., GitHub Actions natively has repo access via its built-in `GITHUB_TOKEN`, scoped and short-lived per workflow run), or a fine-grained Personal Access Token with minimal scope stored as a CI secret, avoiding broad, long-lived tokens/passwords.

</details>

<details markdown="1">
<summary>❓ <b>37. What is the purpose of signed commits (GPG/SSH signing) in Git/GitHub?</b></summary>
<br>

Cryptographically verifies that a commit was authored by the claimed identity and hasn't been tampered with, displayed as "Verified" in GitHub - important for supply chain security, ensuring malicious actors can't impersonate a trusted developer in the commit history.

</details>

<details markdown="1">
<summary>🎯 <b>38. Scenario: How would you set up a Git workflow for a team practicing Continuous Deployment, ensuring code quality and fast feedback?</b></summary>
<br>

Adopt trunk-based development with short-lived feature branches, enforce branch protection requiring CI checks (lint/test/security scan) and at least one review before merge to `main`, use feature flags for incomplete features, and configure GitHub Actions to automatically deploy `main` to production (or staging first with automated smoke tests) upon merge.

</details>

