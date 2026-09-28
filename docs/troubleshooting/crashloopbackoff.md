# CrashLoopBackOff Incident

## Intent
Deliberately run a container that exits with code 1.

## Symptom
Pod entered `CrashLoopBackOff` and restart count increased.

## Logs
The container printed:
- `Application starting...`
- `Application crashed!`

## Root cause
The process intentionally exited with status 1.

## Additional issue
An initial command was transformed by Git Bash on Windows into:
`C:/Program Files/Git/usr/bin/sh`

This caused `RunContainerError`.

## Recovery
The command was corrected to:
`/bin/sh`

The deployment then reached:
`1/1 Running`

## Interview lesson
For CrashLoopBackOff, inspect both `kubectl describe pod` and `kubectl logs`. The status is a symptom; the container logs/events reveal the cause.
