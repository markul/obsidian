---
tags:
  - alfa-bank/service
note-type: service
service: "[[work/alfa-bank/services/skp-product-change-workflow-service|skp-product-change-workflow-service]]"
---

# skp-product-change-workflow-service

## Repository

- Remote: [origin](ssh://git@git.moscow.alfaintra.net/ufrkpnsb/skp-product-change-workflow-service.git)
- Local path: `/home/marat/dev/git/alfa/skp-product-change-workflow-service`

## Local Development

- Main recent local branch: `integration-2`
- Local Skaffold work uses the repo-root `skaffold.yaml`; current notes distinguish dependencies-only runs from the `api` profile.
- Recent work includes local Skaffold setup, integration-test image changes, request logging, worker/schema work, and cron-job validation.

## Related Tickets

- [[work/alfa-bank/tickets/utk2-3274|UTK2-3274]]
- [[work/alfa-bank/tickets/utk2-3275|UTK2-3275]]
- [[work/alfa-bank/tickets/utk2-3276|UTK2-3276]]
- [[work/alfa-bank/tickets/utk2-3310|UTK2-3310]]
- [[work/alfa-bank/tickets/utk2-3329|UTK2-3329]]
- [[work/alfa-bank/tickets/utk2-3339|UTK2-3339]]
- [[work/alfa-bank/tickets/utk2-3340|UTK2-3340]]
- [[work/alfa-bank/tickets/utk2-3349|UTK2-3349]]
- [[work/alfa-bank/tickets/utk2-3365|UTK2-3365]]

## Recent Sessions

- [[agents/sessions/2026-04-28-skp-product-change-workflow-service-integration-tests|2026-04-28 integration tests]]

## Known Issues

- Docker default networking may resolve `binary.alfabank.ru` through the wrong resolver while host networking and minikube resolve through Alfa split DNS.
- Host-run integration tests depend on clean Skaffold and `kubectl port-forward` process state so expected local ports are not shifted.
