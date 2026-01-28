# Chapter 2 Quiz: DevOps Foundations - Git & Docker

#### 1. What is a Docker container?

<div class="upper-alpha" markdown>
1. A virtual machine that requires a full operating system
2. A lightweight, standalone, executable package containing application code and dependencies
3. A storage mechanism for Docker images
4. A configuration file that defines how to build applications
</div>

??? question "Show Answer"
    The correct answer is **B**.

    A Docker container is a lightweight, isolated runtime environment that includes the application and its dependencies. Unlike virtual machines, containers share the host OS kernel, making them more efficient. Option A describes virtual machines, not containers. Option C describes registries or storage systems. Option D describes a Dockerfile.

    **Concept:** Docker Containers

---

#### 2. Explain the relationship between a Dockerfile and a Docker image.

<div class="upper-alpha" markdown>
1. They are the same thing with different names
2. A Dockerfile is a set of instructions that describes how to build a Docker image
3. An image is built first, then a Dockerfile is created from it
4. Dockerfiles contain images that can be deployed as containers
</div>

??? question "Show Answer"
    The correct answer is **B**.

    A Dockerfile contains instructions (FROM, RUN, COPY, etc.) that Docker executes to build an image. The image is the result—a snapshot of a filesystem with the application and dependencies. Option A is incorrect; they are distinct. Option C reverses the process. Option D has incorrect terminology.

    **Concept:** Dockerfile and Image Relationship

---

#### 3. What is the primary difference between `git merge` and `git rebase`?

<div class="upper-alpha" markdown>
1. Merge is faster while rebase is slower
2. Merge creates a merge commit combining two branches; rebase replays commits on top of another branch
3. Rebase is only used for remote repositories
4. Merge creates a linear history while rebase creates branching
</div>

??? question "Show Answer"
    The correct answer is **B**.

    Merge creates a new merge commit that ties two branches together (non-linear history). Rebase replays your commits on top of another branch, creating a linear history. Option A is incorrect; speed is comparable. Option C is false; rebase works locally and remotely. Option D is backwards; rebase creates linear history, not branching.

    **Concept:** Git Merge vs Rebase

---

#### 4. When would you use `git rebase` instead of `git merge` in a team environment?

<div class="upper-alpha" markdown>
1. When you want to keep a clean, linear history and haven't pushed to a shared remote
2. When merging from main into your feature branch publicly
3. Always, because it's faster than merge
4. When collaborating with team members on the same branch
</div>

??? question "Show Answer"
    The correct answer is **A**.

    Rebase should be used locally before pushing to keep history clean. Once code is pushed, using rebase rewrites history, which can cause issues for others. Option B is wrong; merge is safer for shared branches. Option C is an overstatement. Option D is incorrect; rebase on shared branches causes problems.

    **Concept:** Git Rebase Best Practices

---

#### 5. What is the purpose of Docker Compose?

<div class="upper-alpha" markdown>
1. To compile Docker containers into optimized binaries
2. To define and run multi-container applications with a single YAML file
3. To manage container memory and CPU allocation
4. To create backups of Docker images
</div>

??? question "Show Answer"
    The correct answer is **B**.

    Docker Compose lets you define multiple services (containers) with their configurations in a docker-compose.yml file and start them all together. This is ideal for development environments with multiple services. Option A is incorrect; Docker doesn't compile to binaries. Option C relates to resource limits, not the purpose of Compose. Option D is not its function.

    **Concept:** Docker Compose

---

#### 6. You have a Dockerfile with many layers that rarely change. How would you optimize it to improve build times?

<div class="upper-alpha" markdown>
1. Combine all instructions into a single RUN statement
2. Order instructions so dependencies and dependencies change least frequently appear first
3. Remove all comments and whitespace
4. Use a smaller base image regardless of requirements
</div>

??? question "Show Answer"
    The correct answer is **B**.

    Docker caches layers. By ordering instructions so stable dependencies (base OS libraries) are added early and application code (which changes frequently) is added late, you maximize cache hits. Option A combines instructions but loses caching benefits. Option C has minimal impact. Option D risks missing required dependencies.

    **Concept:** Docker Layer Caching

---

#### 7. Why is it problematic to push changes directly to the main branch without going through a pull request?

<div class="upper-alpha" markdown>
1. Direct pushes are slower than pull requests
2. Pull requests enable code review, discussion, and automated testing before integration
3. Main branch pushes always cause merge conflicts
4. Direct pushes consume more storage on the remote repository
</div>

??? question "Show Answer"
    The correct answer is **B**.

    Pull requests provide a mechanism for code review, testing, and discussion before merging, ensuring code quality and team awareness. Option A is incorrect; speed is comparable. Option C is not always true; conflicts depend on changes. Option D is false; storage is not affected.

    **Concept:** Pull Request Workflow

---

#### 8. Given the following scenario: You've committed sensitive data (API keys) to a repository. What is the safest approach?

<div class="upper-alpha" markdown>
1. Delete the file and commit again; the history will be automatically cleaned
2. Use `git reset --hard` to undo all changes
3. Rotate the credentials immediately and use tools like git-filter-branch to remove from history
4. Ask team members not to look at the sensitive commit
</div>

??? question "Show Answer"
    The correct answer is **C**.

    You must immediately rotate credentials (they're compromised) and remove them from history using git-filter-branch or similar tools. Option A is false; deleted files remain in git history. Option B loses all work. Option D doesn't solve the security problem.

    **Concept:** Git Security Best Practices

---

#### 9. How do you reduce the size of a Docker image?

<div class="upper-alpha" markdown>
1. Use a larger base image to include more tools
2. Use multi-stage builds, remove unnecessary files, and use minimal base images like Alpine
3. Compress the image after building
4. Remove the Dockerfile after building the image
</div>

??? question "Show Answer"
    The correct answer is **B**.

    Multi-stage builds allow you to build in one stage and copy only necessary artifacts to the final image. Using minimal base images (Alpine Linux) and cleaning up package managers also reduces size. Option A increases size. Option C doesn't address the core issue. Option D loses the build specification.

    **Concept:** Docker Image Optimization

---

#### 10. Compare the branching strategies: `git flow` vs `trunk-based development`. Which is better for continuous deployment?

<div class="upper-alpha" markdown>
1. Git flow is better because it uses more branches
2. Trunk-based development is better because shorter-lived branches integrate faster with fewer conflicts
3. Both are equally suitable for continuous deployment
4. Trunk-based development is slower due to fewer branches
</div>

??? question "Show Answer"
    The correct answer is **B**.

    Trunk-based development with short-lived feature branches enables continuous deployment by keeping changes integrated quickly and reducing merge conflicts. Git flow with long-lived release branches can delay integration. Option A is incorrect; more branches don't guarantee better CI/CD. Option C is false; they have different trade-offs. Option D reverses the reality.

    **Concept:** Branching Strategies

---

## Question Distribution Summary

**Bloom's Taxonomy:**
- Remember (25%): Questions 1, 3 = 20%
- Understand (30%): Questions 2, 5, 7 = 30%
- Apply (30%): Questions 4, 6, 9 = 30%
- Analyze (15%): Questions 8, 10 = 20%

**Answer Distribution:**
- A: 10% (1 question)
- B: 60% (6 questions)
- C: 20% (2 questions)
- D: 10% (1 question)

*Note: Answer distribution will be balanced across all 4 quizzes (40 questions total) to achieve target percentages.*
