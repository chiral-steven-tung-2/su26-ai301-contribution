# Contribution 7879: add documentation for WSL2

**Contribution Number:** 1
**Student:** Steven Tung
**Issue:** [GitHub issue link](https://github.com/kubernetes/minikube/issues/7879)
**Status:** Phase I In Progress

---

## Why I Chose This Issue

I have experience with WSL2 and understand the pain points of setting up Kubernetes locally on Windows. This issue is a great opportunity to contribute to a major open source project by improving the developer experience for Windows users and practicing writing clear, concise technical documentation.

---

## Understanding the Issue

### Problem Description

When running Minikube in WSL2 using the Docker driver, the IP address assigned to the cluster (e.g., `172.17.0.2`) is not accessible from the Windows browser. Users cannot reach their web apps or services using standard methods like `minikube service`, leading to connectivity timeouts.

### Expected Behavior

Users should be able to access locally deployed web apps from their Windows host browser using the Minikube cluster IP or via a documented workaround. The official documentation should clearly explain how to establish this networking.

### Current Behavior

Users follow standard Minikube commands under WSL2 with `--driver=docker` and receive an internal container IP. The browser simply spins and times out because Windows does not route traffic directly to the Docker subnet inside WSL2.

### Affected Components

Documentation (`site/content/en/docs/start/` and `site/content/en/docs/drivers/docker/`), specifically the sections regarding OS-specific networking limits, the Docker driver, and WSL2.

---

## Reproduction Process

### Environment Setup

I set up a fresh Ubuntu 20.04 WSL2 instance on Windows. I installed Docker natively inside WSL2 (no Docker Desktop) and launched Minikube using `--driver=docker`.

### Steps to Reproduce

1. Install WSL2, Docker engine, and Minikube.
2. Start Minikube: `minikube start --driver=docker`.
3. Create a test deployment: `kubectl create deployment hello-minikube --image=k8s.gcr.io/echoserver:1.4`.
4. Expose the deployment: `kubectl expose deployment hello-minikube --type=NodePort --port=8080`.
5. Run `minikube service hello-minikube` and attempt to navigate to the provided IP address in a Windows browser.
6. The browser will fail to connect.

### Reproduction Evidence

- **Commit showing reproduction:** Not applicable (documentation issue).
- **Screenshots/logs:** The browser says "Hmmmm.... can't reach this page" for `http://172.17.0.2:32498`.
- **My findings:** The WSL2 networking model prevents direct routing to the Docker bridge network. A workaround like `minikube tunnel` or `kubectl port-forward` is required.

---

## Solution Approach

### Analysis

The root cause is a well-known limitation of Docker networking on Windows and macOS. WSL2 operates on its own virtual switch and Docker provides an internal bridge network that the host OS cannot route to out-of-the-box. Many users lack guidance on utilizing tools like `minikube tunnel` to bridge the gap.

### Proposed Solution

Add clear, step-by-step instructions to the Minikube documentation (specifically the WSL2 and Docker driver pages) detailing how to access services using `minikube tunnel` and other workarounds.

### Implementation Plan

Using UMPIRE framework (adapted):

**Understand:** Users cannot reach services deployed on Minikube within WSL2 using the Docker driver because of networking boundaries.

**Match:** Similar documentation exists highlighting networking workarounds for Docker on macOS. We can adapt these notes for WSL2 users.

**Plan:**
1. Identify the key documentation pages that need updating.
2. Draft a new section explaining the behavior and offering `minikube tunnel` as a solution.
3. Submit a PR and request review from maintainers aware of WSL2 constraints.

**Implement:** Create a branch `wsl2-docs-update` and commit the new notes.

**Review:** Ensure the documentation is clear, accurate, and adheres to Minikube's styling guidelines.

**Evaluate:** Have a Windows user test the newly written documentation steps to verify they correctly resolve the connectivity issue.

---

## Testing Strategy

### Unit Tests

- [x] Documentation linting: Verify markdown syntax is valid and adheres to Minikube style guide
- [x] Link validation: Ensure all code examples and links in the documentation work correctly
- [x] Content structure: Confirm section headings and navigation structure are consistent

### Integration Tests

- [x] Full WSL2 setup integration: Document tested on fresh WSL2 instance with Docker engine
- [x] Cross-platform consistency: Verified similar documentation patterns exist on macOS and Linux docs

### Manual Testing

Tested on Windows 10 with WSL2 (Ubuntu 20.04) and Docker engine installed:
1. Verified `minikube tunnel` successfully bridges WSL2 networking gap
2. Confirmed `kubectl port-forward` provides alternative access method
3. Tested with sample deployment (echoserver) and confirmed accessibility from Windows browser
4. Results: Both workarounds successfully enable service access from host browser

---

## Implementation Notes

### Week 1 Progress

- Analyzed GitHub issue #7879 and identified affected documentation files
- Set up test environment with WSL2 and Docker to reproduce the connectivity issue
- Reviewed existing Minikube documentation structure and style guidelines
- Challenge: Understanding the networking architecture differences between WSL2 and other platforms
- Decision: Focus on `minikube tunnel` as primary solution with `kubectl port-forward` as backup

### Week 2 Progress

- Created documentation files under `site/content/en/docs/drivers/docker/wsl2-networking.md`
- Added troubleshooting section to existing Docker driver documentation
- Submitted PR for community review and maintainer feedback

### Code Changes

- **Files modified:** 
  - `site/content/en/docs/drivers/docker/docker.md` (updated with WSL2 networking section)
  - `site/content/en/docs/drivers/docker/wsl2-networking.md` (new file)
- **Key commits:** 
  - Add WSL2 networking guide for Docker driver
  - Update Docker driver docs with WSL2 specific troubleshooting
- **Approach decisions:** 
  - Chose to create separate WSL2-specific page for clarity while cross-linking from main Docker driver docs
  - Emphasized `minikube tunnel` approach as it directly addresses the root networking issue
  - Included multiple code examples to demonstrate both problematic and working scenarios

---

## Pull Request

**PR Description:**

> **Summary:** Add WSL2 networking documentation for the Docker driver
>
> This PR resolves issue #7879 by adding clear documentation for users running Minikube inside WSL2 with the Docker driver who cannot access their deployed services from the Windows host browser.
>
> **Changes:**
> - Added a new dedicated guide `site/content/en/docs/drivers/docker/wsl2-networking.md` covering:
>   - Explanation of why the Docker bridge IP is unreachable from Windows
>   - Step-by-step instructions for using `minikube tunnel` as the primary workaround
>   - Alternative instructions for `kubectl port-forward`
>   - Troubleshooting tips for common failure points
> - Updated `site/content/en/docs/drivers/docker/docker.md` to include a WSL2-specific callout and cross-link to the new guide
>
> **Testing:** Manually verified both workarounds on Windows 10 with WSL2 (Ubuntu 20.04) and Docker engine. Confirmed that both `minikube tunnel` and `kubectl port-forward` successfully expose services to the Windows host browser.

**Maintainer Feedback:**
- 2026-06-10: Maintainer requested moving the WSL2 callout higher on the Docker driver page so users see it before attempting service access
- 2026-06-10: Addressed by moving the WSL2 networking note to the "Known Limitations" section near the top of `docker.md`
- 2026-06-15: Maintainer asked to add a note clarifying that Docker Desktop users are unaffected, as it handles networking differently
- 2026-06-15: Added a brief clarifying sentence at the top of the WSL2 guide scoping it to native Docker engine installs inside WSL2

**Status:** Iterating on maintainer feedback

---

## Learnings & Reflections

### Technical Skills Gained

- Deepened understanding of WSL2 networking architecture: how WSL2 uses a Hyper-V virtual switch with NAT, and why Docker's internal bridge subnet (`172.17.x.x`) is not routable from the Windows host
- Learned how `minikube tunnel` works under the hood — it creates a network route on the host so that cluster IPs become reachable, effectively acting as a software-defined gateway
- Gained experience navigating a large open-source repository's documentation structure, style guide, and Hugo-based static site generation pipeline
- Practiced reading and contributing to GitHub issue threads to understand the full scope of a reported problem before writing a fix

### Challenges Overcome

- **Understanding the networking boundary:** At first it was not obvious why some users reported success and others did not. Researching Docker Desktop vs. native Docker engine installs clarified that Docker Desktop injects its own network bridge, which explains the different behavior.
- **Scoping the documentation:** The issue touched multiple pages and could have led to a sprawling PR. I chose to create one focused new page and make a minimal, clearly cross-linked edit to the existing Docker driver page to keep the change reviewable.
- **Matching the project's documentation style:** Minikube docs use specific callout shortcodes and Hugo front matter. Getting the formatting right required reading existing pages closely and running the site locally to preview changes.

### What I'd Do Differently Next Time

- **Reach out on Slack or Discussions earlier.** I spent extra time reproducing the exact environment conditions that trigger the issue before commenting on GitHub. In hindsight, asking a quick clarifying question in the `#minikube` channel would have helped me validate my understanding faster.
- **Open a draft PR sooner.** I waited until the documentation felt polished before opening a PR, but getting early maintainer eyes on the structure would have saved a revision cycle.
- **Write the test plan in the PR description upfront.** The maintainers asked how I tested the changes; including that detail from the start would have reduced back-and-forth.

---

## Resources Used

- [Minikube WSL2 documentation (existing)](https://minikube.sigs.k8s.io/docs/start/) — starting point for understanding existing coverage
- [Minikube Docker driver documentation](https://minikube.sigs.k8s.io/docs/drivers/docker/) — main page updated in this PR
- [WSL2 networking overview — Microsoft Docs](https://learn.microsoft.com/en-us/windows/wsl/networking) — authoritative explanation of the WSL2 NAT model and port forwarding
- [GitHub issue #7879 — original bug report and community discussion](https://github.com/kubernetes/minikube/issues/7879) — source of the problem statement and user-reported workarounds
- [minikube tunnel command reference](https://minikube.sigs.k8s.io/docs/commands/tunnel/) — used to verify how tunnel establishes routes
- [Minikube contributing guide](https://minikube.sigs.k8s.io/docs/contrib/) — documentation style guide and local preview instructions
- [Stack Overflow: Access WSL2 services from Windows](https://stackoverflow.com/questions/61002681/accessing-wsl2-services-from-windows) — community confirmation of the root cause and common workarounds
