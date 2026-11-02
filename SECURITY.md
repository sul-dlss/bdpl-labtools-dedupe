# Security policy

## Reporting a vulnerability

Please do not open a public issue for security problems. Use GitHub's
private vulnerability reporting instead: on the repository page choose
**Security → Report a vulnerability**. You will get an acknowledgement
within five working days.

## What is in place

- Secret scanning with push protection on every push.
- Dependabot alerts and automatic security updates.
- CI runs Bandit and Gitleaks on every pull request.
- The `main` branch accepts changes only through pull requests; force
  pushes and deletions are blocked for everyone, including admins.
