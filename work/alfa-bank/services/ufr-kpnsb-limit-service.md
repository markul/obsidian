---
tags:
  - alfa-bank/service
note-type: service
service: "[[work/alfa-bank/services/ufr-kpnsb-limit-service|ufr-kpnsb-limit-service]]"
status: active
---

# ufr-kpnsb-limit-service

## Repository

- Remote: [origin](ssh://git@git.moscow.alfaintra.net/ufrkpnsb/ufr-kpnsb-limit-service.git)
- Local path: `/home/marat/dev/git/alfa/ufr-kpnsb-limit-service`

## Local Development

- Recent work covered Skaffold migration/configuration, local config fixes, CRLF handling in `init-db.sh`, AppSec image-tag remediation, and Jenkins failure investigation.
- The active delivered runtime path for AppSec remediation is `devops/Dockerfile` with the Skaffold `api` profile.

## Related Tickets

- [[work/alfa-bank/tickets/utk2-3264|UTK2-3264]]
- [[work/alfa-bank/tickets/utk2-3290|UTK2-3290]]
- [[work/alfa-bank/tickets/utk2-3295|UTK2-3295]]
- [[work/alfa-bank/tickets/utk2-3326|UTK2-3326]]

## Related Projects

- [[agents/projects/utk2-3264|UTK2-3264 Agent Work]]
- [[agents/projects/utk2-3295|UTK2-3295 Agent Work]]
- [[agents/projects/utk2-3326|UTK2-3326 Agent Work]]

## Recent Sessions

- [[agents/sessions/2026-04-13-ufr-kpnsb-limit-service-flaky-test-stash|2026-04-13 flaky test stash]]
- [[agents/sessions/2026-04-13-ufr-kpnsb-limit-service-jenkins-1703|2026-04-13 Jenkins 1703]]
- [[agents/sessions/2026-04-14-ufr-kpnsb-limit-service-prelive-db-auth|2026-04-14 prelive DB auth]]
- [[agents/sessions/2026-04-24-ufr-kpnsb-limit-service-jenkins-1740|2026-04-24 Jenkins 1740]]
- [[agents/sessions/2026-04-27-ufr-kpnsb-limit-service-jenkins-1744-flaky-test|2026-04-27 Jenkins 1744 flaky test]]

## Known Issues

- Jenkins image publication has shown recurring temp-image locality failures after green build/test stages.
- `LimitChangeServiceTests.ProcessLimitGroupAsync_ShouldCreateChangeLimitRequest` had nondeterminism tied to random AutoFixture LM id collisions.
