# Real Task Benchmarks

## Method

- Warm-cache runs
- `dotnet restore` executed once before timed steps
- Timed steps:
  - `dotnet clean <solution> -c Debug`
  - `dotnet build <solution> -c Debug --no-restore`
  - `dotnet test <unit-test-project> -c Debug --no-restore`

## Ufr.Kpnsb.LimitService

- Repository:
  - `optiplex`: `~/dev/git/alfa/ufr-kpnsb-limit-service`
  - `alfa`: `~/dev/git/ufr-kpnsb-limit-service`
  - `MacBook`: `/Users/marat/dev/git/alfa/ufr-kpnsb-limit-service`
- Unit test project: `test/Ufr.Kpnsb.LimitService.Tests.Unit`

### Results

| Task | MacBook | optiplex | alfa |
| --- | ---: | ---: | ---: |
| `dotnet clean` | `1.32s` | `0.96s` | `2.17s` |
| `dotnet build` | `16.45s` | `21.95s` | `37.25s` |
| `dotnet test` unit | `16.59s` | `12.36s` | `30.84s` |

### Notes

- `optiplex` and `alfa` were measured on commit `89f292a004540cccf9b76aa06fddc1c2a2640cc9`
- MacBook was measured on a different commit: `d6d6b9d26dbfb5a2f112dc42e7bb8a955971d521`
- Because of that, the MacBook result is useful but not strictly apples-to-apples for this repository
- Unit tests on all three completed successfully for the measured runs:
  - Passed: `625`
  - Skipped: `5`
  - Failed: `0`

## Ufr.Kpnsb.ProductRequestWorkflowService

- Repository:
  - `optiplex`: `~/dev/git/alfa/ufr-kpnsb-product-request-workflow-service`
  - `alfa`: `~/dev/git/ufr-kpnsb-product-request-workflow-service`
  - `MacBook`: `/Users/marat/dev/git/alfa/ufr-kpnsb-product-request-workflow-service`
- Unit test project: `test/Ufr.Kpnsb.ProductRequestWorkflowService.Tests.Unit`

### Results

| Task | MacBook | optiplex | alfa |
| --- | ---: | ---: | ---: |
| `dotnet clean` | `1.03s` | `1.02s` | `2.39s` |
| `dotnet build` | `67.68s` | `42.07s` | `143.85s` |
| `dotnet test` unit | `22.31s` | `19.01s` | `33.64s` |

### Notes

- All three were measured on commit `9d702c933f206b3ef6f97c18766e0e2981ac978d`
- `optiplex` and `alfa` unit tests passed:
  - Passed: `1695`
  - Skipped: `2`
  - Failed: `0`
- MacBook unit tests did not fully pass:
  - Passed: `1690`
  - Skipped: `2`
  - Failed: `5`
- MacBook failures were in `LoanManagerClientProviderTests`

## Summary

- `optiplex` is the most consistently strong machine across the two measured .NET workloads
- `MacBook` is materially faster than `alfa`, but not consistently faster than `optiplex`
- `alfa` is the slowest machine in every full real-task comparison that used matching commits
