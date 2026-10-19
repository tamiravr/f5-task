# Root Cause Analysis

Fill this in as you go. Bullet points are fine — be specific, not long.

---

## Defect 1: Proxy image fails to build

### Symptom / error output

proxy image fails during nginx installation because the config-file prompt cannot be answered in the non-interactive build.

the output:
Configuration file '/etc/nginx/nginx.conf'
*** nginx.conf (Y/I/N/O/D/Z) [default=N] ?
dpkg: error processing package nginx-common (--configure):
end of file on stdin at conffile prompt

### How I diagnosed it
I ran bash ./scripts/run-tests.sh and then saw the failure - the error above

I checked the Docker build logs and found that dpkg was asking which nginx.conf file to keep. Since Docker could not answer this question during the build, the installation failed.

Inspected proxy/Dockerfile and noticed that nginx.conf was copied before Nginx was installed

### Root cause

The Dockerfile copied the custom nginx.conf into /etc/nginx/nginx.conf before installing the Nginx package.

During installation, dpkg detected an existing configuration file and requested user input to resolve the conflict. Since Docker builds run automatically without an interactive terminal for answering such prompts, the installation failed.

### Fix

switched the COPY and the RUN commands so nginx installs its default configuration first, then our custom configuration replaces it without a prompt

---

## Defect 2: Proxy cannot reach the backend

### Symptom / error output

The /nginx-health test passed, but the /api/info request failed with HTTP 502 Bad Gateway.


the output:
test-runner-1  | Testing proxied backend /api/info...
proxy-1        | 2026/10/19 17:08:12 [error] 8#8: *2 connect() failed (111: Connection refused) while connecting to upstream, client: 172.19.0.4, server: localhost, request: "GET /api/info HTTP/1.1", upstream: "http://127.0.0.1:8080/api/info", host: "proxy:80"
proxy-1        | 172.19.0.4 - - [19/Oct/2026:17:08:12 +0000] "GET /api/info HTTP/1.1" 502 166 "-" "Python-urllib/3.11"
test-runner-1  | TEST RUNNER CRASHED: HTTP Error 502: Bad Gateway
test-runner-1 exited with code 1

the important thing: upstream: "http://127.0.0.1:8080/api/info", host: "proxy:80"


### How I diagnosed it


ran bash ./scripts/run-tests.sh and inspected the Nginx error logs.

examined proxy/nginx.conf and noticed that the upstream was configured to use 127.0.0.1:8080.

after changing the upstream address, verified that requests successfully reached the backend once it was ready.

### Root cause

The Nginx upstream was configured to use 127.0.0.1:8080, which refers to the proxy container itself rather than the backend container.

Since Docker Compose runs the services in separate containers, Nginx must use Docker's internal DNS to communicate with the backend using its service name.

### Fix

Changed the upstream address in proxy/nginx.conf from 127.0.0.1:8080 to backend:8080.

to verify this I ran curl -i http://localhost:8081/api/info after the backend finished starting. The request returned HTTP 200 with the expected JSON response.

The integration tests still failed during startup, indicating a separate readiness issue to investigate in Defect 3.
---

## Defect 3: Tests still fail after fixing defect 2

### Symptom / error output
After fixing the Nginx upstream configuration, the integration tests still failed with HTTP 502 Bad Gateway during startup.

However, requests through Nginx succeeded once the backend finished initializing.

### How I diagnosed it

Ran bash ./scripts/run-tests.sh and examined the container logs.

Observed that the backend loaded its configuration before starting the HTTP server.

Verified that /api/info returned HTTP 200 once the backend was ready.

Identified that the existing depends_on configuration did not wait for application readiness.

### Root cause
The backend takes approximately 8 seconds to initialize before listening on port 8080.

Docker Compose's default depends_on condition only ensures that containers have started, not that their applications are ready.

As a result, the integration tests started before the backend could accept connections.

### Fix, and why I chose it over the alternatives
Added a healthcheck to the proxy that verifies the /api/info endpoint and configured the test runner to depend on the proxy's service_healthy condition.

This ensures that the complete request path is operational before integration tests begin.

I chose a healthcheck over a fixed sleep delay because it verifies actual service readiness instead of assuming a fixed startup time, while allowing a configurable number of retries.

to verify it I noticed that The initial integration test run passed successfully and additional repeated runs will be used to confirm reliability.
---

## Defect 4: CI pipeline fails

### Symptom / error output
The CI workflow executes ./scripts/run-tests.sh directly, while the script was tracked by Git without executable permissions (100644).

### How I diagnosed it

Inspected .github/workflows/ci.yml and identified how the test script is executed.

Ran git ls-files -s scripts/run-tests.sh and confirmed the file mode was 100644.

Compared direct execution with running the script through Bash.

### Root cause

The test script was missing the executable bit in Git. Direct execution on Linux requires executable permissions, unlike running the script through bash.

### Fix
Used chmod +x scripts/run-tests.sh to make the script executable and staged the permission change in Git (100644 → 100755).

I chose this approach over modifying the CI command to use Bash because it also supports direct execution for anyone cloning the repository, as documented in the README.

Verification: Confirmed the Git mode change and successfully ran ./scripts/run-tests.sh locally with exit code 0.
GitHub Actions CI pipeline completed successfully on the fix/f5-challenge branch, confirming that the test script runs correctly in the GitHub-hosted Ubuntu environment.
---

## Bonus questions

Optional. Answer only the ones you got to — number them, skip the rest.

>1. I changed the proxy's base image from ubuntu:24.04 to nginx:stable-alpine to reduce its size. I also removed packages like git, vim, and htop because they are not needed to run Nginx. I kept curl since o ur healthcheck uses it.After making the changes, I ran ./scripts/run-tests.sh to make sure everything still worked. Both integration tests passed, and the script exited with code 0.


---

## Anything you would do differently with more time
