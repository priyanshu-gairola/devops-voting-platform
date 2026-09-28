# ImagePullBackOff Incident

## Intent
Deliberately deploy a nonexistent image to exercise failure diagnosis.

## Symptom
Pod entered `ImagePullBackOff`.

## Evidence
Pod Events reported failure to pull:
`priyanshu-failure-demo:does-not-exist-20260928`

The image/repository did not exist.

## Root cause
Invalid/nonexistent image reference.

## Recovery
Replace the image with a valid image and verify the Pod reaches `Running`.

## Interview lesson
`ImagePullBackOff` means Kubernetes is repeatedly unable to obtain the requested image. Always inspect Events for the exact registry/image error.
