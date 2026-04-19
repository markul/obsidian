# UTK2-3275 In Two Phases: Inbox, Then Client Negative Handler

## Summary

- Supporting note for the canonical project: [[agents/projects/utk2-3275|UTK2-3275]].
- Create the agent project note `agents/projects/utk2-3275.md` in the vault with `status: active` in properties, linked from the existing ticket note, and record the phased rollout below.
- Deliver `UTK2-3275` in two implementation phases:
  - Phase 1: add inbox infrastructure to `skp-product-change-workflow-service`.
  - Phase 2: add the `DwhClientResultEventHandler` business logic that stores client negatives.
- Use the business allowlist `NB302`, `GP302`, `EI302`, `CD148`, `CR101`.

## Repo Gap Assessment As Of 2026-04-11

- Already present in the repo:
  - inbox tables, migration, and seeded topic `ufr-kpnsb-dwh-client-result`
  - background inbox registration with `AddOpenApiConsumer<DwhClientResultEvent, DwhClientResultEventHandler>`
  - local Kafka topic config and Skaffold support
  - `DwhClientResultEvent` transport contract with `AppNegatives[]`
  - `client_negatives` entity, EF mapping, DbSet, and generic repository registration
- Still missing in the repo:
  - `CommonOptions.ClientNegativeAllowedCodes`
  - `ILoanManagerClientProvider` lookup by `PinEq`
  - OData allowlist entry for `RelationObjectAbses?$filter=Pin eq '{pinEq}' and AbsId eq 4188&$select=ObjectId&$top=100`
  - real `DwhClientResultEventHandler` business logic
  - unit tests for the new provider method and handler branches
- Open specification point:
  - define how to handle multiple allowed codes inside `AppNegatives[]`: persist only the first match or persist one row per allowed code

## Phase 1: Inbox Infrastructure

- Add package support for inbox aligned with the repo’s Kafka stack:
  - `Skp.Kafka.Inbox 12.0.13`
  - `Skp.Kafka.Inbox.MessagePack` is not needed
  - `Skp.Kafka.Inbox.Redis` is not needed unless deployment already requires Redis result publishing
- Update `ProductChangeDbContext` to implement `IInboxDbContext` in addition to `IOutboxDbContext`.
- Add inbox DbSets required by the package: `Inbox`, `InboxPayload`, `InboxTopic`.
- Call `modelBuilder.ConfigureInbox()` in `OnModelCreating`.
- Generate a dedicated EF migration and matching forward or revert manual SQL scripts for inbox tables only.
- Add inbox registration in API or background wiring:
  - `services.AddInbox<ProductChangeDbContext>(builder => ...)`
  - `builder.AddDefaultSerializer()`
  - `builder.ConfigureHostedService(...)`
  - `builder.RegisterConsumers(...)`
- Add Kafka config sections needed by inbox:
  - consumer connection settings
  - inbox hosted-service settings
  - topic entry for `DwhClientResult`
- Register the consumer subscription for `DM :: ufr-kpnsb-dwh-client-result`, but the handler in this phase may be a minimal stub that accepts the message and returns success without business side effects.
- Phase 1 acceptance:
  - service builds with inbox dependencies
  - DB model and migration are valid
  - inbox hosted service is registered
  - the consumer is wired to the target topic

## Phase 2: Client Negative Handler

- Step 1: configuration
  - Add `CommonOptions.ClientNegativeAllowedCodes`.
  - Set local defaults to `NB302`, `GP302`, `EI302`, `CD148`, `CR101`.
  - If runtime values are controlled outside the repo, verify the matching `infra-helm-values` settings before closure.
- Step 2: LM lookup provider
  - Extend `ILoanManagerClientProvider` with a dedicated `PinEq` lookup method.
  - Implement the Confluence OData query:
    - `RelationObjectAbses?$filter=Pin eq '{pinEq}' and AbsId eq 4188&$select=ObjectId&$top=100`
  - Add the matching allowed OData pattern to `appsettings.OData.yaml`.
  - Add provider tests for empty, single, and multiple returned clients, and verify the exact URI with `VerifyUriIsAllowed`.
- Step 3: handler implementation
  - Replace the phase-1 stub with `DwhClientResultEventHandler : IInboxHandler<DwhClientResultEvent>`.
  - Inject logger, `ILoanManagerClientProvider`, `IBaseRepository<ClientNegative>`, and `IOptions<CommonOptions>`.
  - Resolve the allowed negative code(s) from `AppNegatives[]`.
  - If no allowed `AppNegativeCodeSb` is present, return success with no side effects.
  - If `PinEq` is empty, log `Для id={id} не задан Pin клиента` and stop.
  - If more than one client is found, log `Для id={id} найдено несколько клиентов по PinEQ клиента: {pinEq}` and stop.
  - If zero clients are found, log `Для id={id} не найден клиент по Pin клиента: {pinEq}` and stop.
  - If exactly one client is found, log `Для id={id} найден единственный клиент {clientId} по Pin клиента: {pinEq}` and persist into `client_negatives`.
  - If LM lookup fails, return a failed inbox result so retry handling stays active.
- Step 4: persistence rule
  - Confirm whether `AppNegatives[]` should produce only one `ClientNegative` row or one row per allowed negative code.
  - Persist `negative_code`, `client_id`, and `negative_date = message.CheckDate` according to the agreed rule.
- Step 5: validation
  - Add handler tests for allowlist skip, missing `PinEq`, zero clients, multiple clients, successful insert, and LM failure.
  - Run:
    - `dotnet build`
    - `dotnet test test/Skp.ProductChangeWorkflowService.Tests.Unit/Skp.ProductChangeWorkflowService.Tests.Unit.csproj`

## Test Plan

- Phase 1:
  - build passes
  - migration is generated and manual scripts are present
  - inbox registration can be covered by startup or service-registration tests if the repo already has that pattern
- Phase 2:
  - provider tests for exact OData URI and zero or one or many results
  - handler tests for allowlist skip, missing `PinEq`, zero clients, many clients, one client insert, and LM failure
- Verification commands:
  - `dotnet build`
  - `dotnet test test/Skp.ProductChangeWorkflowService.Tests.Unit/Skp.ProductChangeWorkflowService.Tests.Unit.csproj`

## Assumptions

- Inbox must be implemented before business logic, and Phase 1 is allowed to land with a stub or no-op handler.
- Payload format is the inbox default serializer, not MessagePack.
- The verified Kafka producer shape is `AppNegatives[]`, not a singular `AppNegative` field.
- Redis-backed inbox result publishing is out of scope unless existing deployment config explicitly requires it.
