# TaskFlow API: NTU AIE Module 4 end-of-module assignment

Owner: BigBoss (GitHub DAMI333N). Due Thu 29 Oct 2026. Target: full marks.
Scope: brief Tasks 1-4 + README, plus stretch Options 1, 2, 4 and 5 (not 3).
Started from my Module 3 repo DAMI333N/module3-taskflow-api (remote upstream, push disabled).
Option 2's frontend lives in DAMI333N/taskflow-web.
After grading this becomes a portfolio task manager: prefer production-grade choices, never at
the cost of the brief's literal requirements.

## Environment
- Edit here, on Windows (D333-PC). This PC has NO Java, Maven or Docker, by choice.
- Java builds, tests and Docker run on aiedev, through the PowerShell tool:
    ssh -T aiedev "cd ~/dev/aie/taskflow-api && git pull --ff-only && "
  Never ssh from the Bash tool: it cannot reach the Windows ssh agent.
- aiedev's clone is pull-only. Never edit files there. Push from here first.
- On aiedev, publish container ports on loopback only: -p 127.0.0.1:8080:8080 (Docker bypasses UFW).
- GitHub through gh (logged in as DAMI333N, repo + workflow scopes).

## Conventions
- Port 8080. API contract from Module 3 is fixed: 7 endpoints, status/completed rule, error body.
- Keep the brief's Dockerfile verbatim. No layertools (removed in Spring Boot 4.1).
- Spring Boot 4: @MockitoBean, @WebMvcTest from org.springframework.boot.webmvc.test.autoconfigure,
  Jackson 3. Check the pom before trusting any snippet.
- SemVer from git tags vX.Y.Z; pom version must equal the tag. Image DOCKER_USERNAME/taskflow-api.
  Main push -> sha-xxxxxxx, main. Tag -> X.Y.Z, X.Y, latest.
- CI: .github/workflows/ci.yml. Secrets DOCKER_USERNAME, DOCKER_TOKEN, NVD_API_KEY.
  Every action pinned to a full commit SHA (peeled ^{} line) with a version comment.
- Railway env: TASKFLOW_CORS_ALLOWEDORIGINS, optional OPENAI_API_KEY. Never in the repo.

## How to work with me
- I am learning, but do not stop to quiz me. Work through each ticket without waiting for
  predictions. When a ticket is done, give me a brief summary (what changed, what ran, the result)
  and up to five key learning points, each one line. If I skip tickets, do this per Part.
- One analogy frame for Docker and CI, kept throughout. Say where it breaks.
- On failure, show the real output and your diagnosis before fixing, and log it. Use
  java-build-resolver when the diagnosis is unclear. Never fake an error or knowingly write
  broken code.
- Stay inside the step. Show each diff and explain it before I accept.
- Ask before every git push, git tag, gh repo/release/secret command, sudo, or anything on
  aiedev, GitHub, Railway, Docker Hub or Netlify. Never ask me to paste a secret into chat.
- Never hand me a command block containing set -e. Write .md files directly to disk.

## Logging
- Append every real error (command, exact message, cause, fix) to BUILD-LOG.md (excluded via
  .git/info/exclude). It feeds the README Errors and failures section.
