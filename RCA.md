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

### How I diagnosed it

### Root cause

### Fix

---

## Defect 3: Tests still fail after fixing defect 2

### Symptom / error output

### How I diagnosed it

### Root cause

### Fix, and why I chose it over the alternatives

---

## Defect 4: CI pipeline fails

### Symptom / error output

### How I diagnosed it

### Root cause

### Fix

---

## Bonus questions

Optional. Answer only the ones you got to — number them, skip the rest.

>

---

## Anything you would do differently with more time
