# Changelog

## 0.2.0 - 2026-04-04
- documented the OpenClaw-side behaviors required for reliable Claude ACP persistence
- added a recovery matrix for warmup, stale identity, quota, and model-drift failures
- added sanitized self-hosted rollout notes for shipping Claude ACP fixes to a live OpenClaw install
- updated the public skill and README so operators can distinguish Telegram routing problems from Claude quota and ACP persistence problems

## 0.1.0 - 2026-04-03
- initial public release of `chip-claudecode-tg`
- added onboarding, runbook, preflight checks, model-selection notes, and manual review checklist
- documented restart warmup and stale persistent-session recovery pitfalls for Claude ACP via `acpx`
- clarified that backend-native model ids (`opus`, `sonnet`, `haiku`) are safer defaults for `acpx/claude`
