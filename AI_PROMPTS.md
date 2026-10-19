# Generative AI Disclosure

Using AI tools for this challenge is allowed. Be honest here — an accurate
"I used AI heavily" is worth more to us than an inaccurate "I did not".

**Did you use AI tools?** (yes / no): yes

**Which ones?** (e.g. ChatGPT, Claude, Copilot, Cursor): ChatGPT

> If you answered **no**, write "N/A" in sections 1 and 2 and skip to
> section 3. That is a perfectly good answer.

---

## 1. Key prompts

List the main prompts you used. Paraphrasing is fine.



1. `Review the F5 DevOps Intern Challenge repository and help me create a step-by-step plan.`

2. `Analyze the Docker build failure using the terminal logs. Help me identify the root cause and understand how to fix it`

3. `Explain instructions in the proxy Dockerfile.`

4. `"Analyze the HTTP 502 failure and validate the Nginx upstream configuration.`

5. `Analyze the failing integration tests, identify possible root causes, and suggest alternative solutions with their trade-offs to help me choose the most appropriate approach.`

6. `Analyze the CI workflow configuration, identify possible execution issues, and compare alternative fixes`



---

## 2. Where the AI got it wrong

Describe **one concrete case** where the AI's answer was wrong, outdated, or
sub-optimal. Cover: what it suggested, how you noticed it was wrong, and what
you did instead. 3–5 sentences.

> The AI suggested using Docker Compose healthchecks to solve the startup race condition, describing them as adaptable to different startup times. However, I noticed that the suggested configuration had a limit of 15 consecutive failed checks, which the AI initially overlooked. After questioning this, we clarified that the service could be marked unhealthy if the startup time exceeded the configured retry threshold. I kept the healthcheck approach because it verifies actual service readiness, while recognizing that its retry settings must account for expected startup times.

---

## 3. Anything in your submission you could not fully explain?

We would rather know. Write "nothing" if that is the case.

> nothing
