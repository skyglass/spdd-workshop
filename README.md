# SPDD Token Billing Workshop

This workshop replays the local `../token-billing` reference repository, checked
out from [gszhangwei/token-billing](https://github.com/gszhangwei/token-billing),
using the SPDD workflow from Martin Fowler's
[Structured-Prompt-Driven Development](https://martinfowler.com/articles/structured-prompt-driven/)
article. The goal is to create your own empty repository, generate the prompt
and code artifacts through OpenSPDD, and end with a commit history that is very
close to the reference history in `../token-billing`.

The article focuses on the enhancement workflow. This workshop also replays the
initial billing-service history so your local repo has the same starting point.
The reference implementation in `../token-billing` is useful only as a checkpoint.
Do not copy files from it during the exercise.

## Start With A New Empty Repository

Create a new repository with any name you want.
The commands below assume your created repository at `../spdd-workshop-snapshot`.

```bash
cd ../spdd-workshop-snapshot
```

You are now in `../spdd-workshop-snapshot`.
Run the remaining commands from here.

## Prerequisites

- Java 21
- Docker, for PostgreSQL via `compose.yaml`
- Gradle installed locally for generating the initial wrapper, or another way to
  create a Gradle wrapper before the first commit
- OpenSPDD installed as `openspdd`
- Codex installed and authenticated
- Your new empty workshop repository checked out at `../spdd-workshop-snapshot`
- The reference repository checked out at `../token-billing` from
  [gszhangwei/token-billing](https://github.com/gszhangwei/token-billing)
- Commands that include `../token-billing` are read-only reference checks

Check OpenSPDD:

```bash
openspdd -v
openspdd list --all
```

OpenSPDD commands used in this guide:

| Command | Project-local Codex skill file | Purpose |
| --- | --- | --- |
| `$spdd-story` | `./.agents/skills/spdd-story/SKILL.md` | Split or refine enhancement ideas into focused user stories. |
| `$spdd-analysis` | `./.agents/skills/spdd-analysis/SKILL.md` | Analyze a story and produce domain concepts, risks, and design direction. |
| `$spdd-reasons-canvas` | `./.agents/skills/spdd-reasons-canvas/SKILL.md` | Generate the REASONS Canvas used as the implementation blueprint. |
| `$spdd-generate` | `./.agents/skills/spdd-generate/SKILL.md` | Generate or update code from the current structured prompt. |
| `$spdd-api-test` | `./.agents/skills/spdd-api-test/SKILL.md` | Generate a cURL-based functional API test script. |
| `$spdd-prompt-update` | `./.agents/skills/spdd-prompt-update/SKILL.md` | Update the structured prompt first when requirements or behavior change. |
| `$spdd-sync` | `./.agents/skills/spdd-sync/SKILL.md` | Sync code-side refactors or fixes back into the structured prompt. |

OpenSPDD writes Codex skills under `./.agents/skills/<skill-id>/SKILL.md`.
The leading dot in `./.agents` is part of the generated directory name.
For optional commands, generate the skill before invoking it.
The first time you run `codex` in this repo, approve the project trust prompt if
Codex shows one. Some Codex versions need that before project skills appear.

The article's enhancement flow is: create or refine the story, clarify scope,
run `spdd-analysis`, run `spdd-reasons-canvas`, run `spdd-generate`, keep prompt
and code synchronized with `spdd-prompt-update` or `spdd-sync`, then add tests.
Part 1 below is setup for recreating the article's starting system locally.

## Expected Revisions

Use this as a contents-style checklist for the workshop. The goal is not to
recreate the exact reference commit history, but to use the SPDD workflow and
arrive at a similar result.

| Step | Expected revision |
| --- | --- |
| 0 | Copy the prepared initial files |
| 1 | Verify the prepared starting point |
| 2 | Init the repo |
| 3 | Initialize OpenSPDD Codex skills |
| 4 | Generate the initial analysis document |
| 5 | Generate the initial REASONS Canvas |
| 6 | Add architecture intent to the Canvas |
| 7 | Generate the initial product code |
| 8 | Correct active subscription lookup through prompt-first update |
| 9 | Refactor `calculateBill` and sync the Canvas |
| 10 | Extract magic numbers in `Bill` and sync the Canvas |
| 11 | Remove unused methods and sync the Canvas |
| 12 | Add initial tests |
| 13 | Add the broad enhancement story |
| 14 | Split the broad story into focused stories |
| 15 | Generate enhancement analysis |
| 16 | Generate enhancement REASONS Canvas |
| 17 | Generate enhancement code |
| 18 | Generate the functional API test script |
| 19 | Refactor remaining magic numbers and sync the Canvas |
| 20 | Generate the enhancement test prompt |
| 21 | Generate enhancement unit tests |
| 22 | Update the structured prompt for required `model_id` |
| 23 | Generate code from the updated prompt |
| 24 | Update the API test command output |
| 25 | Add the story command and first initial story |
| 26 | Add the initial enhancement idea |
| 27 | Generate the second initial story |
| 28 | Rename initial story files |

## Part 1: Replay The Initial Billing Service

The article focuses on the enhancement, but the reference history begins from an
existing billing service. Copy the prepared initial files first, then start the
OpenSPDD workflow from that baseline.

### Step 0: Copy The Prepared Initial Files

Current workshop repository includes `./init_files/` with the initial Spring Boot
billing service and normalized plan/subscription environment. Assuming you are in
`../spdd-workshop-snapshot`, copy those files into the new repository root:

```bash
cp -R ../spdd-workshop/init_files/. .
```
These files are based on the initial setup commits from the reference
`../token-billing` repository.

### Step 1: Verify The Prepared Starting Point

The starter application is configured to use PostgreSQL at
`localhost:54323`, and the Spring context test runs Flyway against that
database. Start the bundled Compose database before running tests:

```bash
docker compose up -d
./gradlew test
```

### Step 2: Commit The Starting Point

Commit the copied starting point with the initial reference commit message:

```bash
git add .
git commit -m "[000] feat: init the repo"
```

### Step 3: Initialize OpenSPDD Skills

Generate the project-scoped SPDD skills for Codex. You run these commands once
from the terminal; OpenSPDD writes Codex skill files under `./.agents/skills/`.

This initialization is needed for the Codex-based workshop flow. Later steps ask
Codex to execute prompts such as `$spdd-analysis`, `$spdd-reasons-canvas`, and
`$spdd-generate`. Codex can only understand those project commands after the
corresponding skill files have been generated. Committing the generated files
also keeps the workflow instructions and agent configuration tied to this
repository, instead of depending on whichever OpenSPDD setup happens to be
installed on a user machine.

```bash
openspdd --tool codex init
openspdd --tool codex generate --all
```

Expected OpenSPDD skill artifacts:

```text
./.agents/skills/spdd-analysis/SKILL.md
./.agents/skills/spdd-analysis/agents/openai.yaml
./.agents/skills/spdd-generate/SKILL.md
./.agents/skills/spdd-generate/agents/openai.yaml
./.agents/skills/spdd-prompt-update/SKILL.md
./.agents/skills/spdd-prompt-update/agents/openai.yaml
./.agents/skills/spdd-reasons-canvas/SKILL.md
./.agents/skills/spdd-reasons-canvas/agents/openai.yaml
./.agents/skills/spdd-sync/SKILL.md
./.agents/skills/spdd-sync/agents/openai.yaml
```

Commit the generated `./.agents/skills/` files.

Checkpoint:

```bash
git add .
git commit -m "[000] feat: initialize the commands for the SPDD workflow"
```

### Step 4: Generate The Initial Analysis

From this point forward, each `Prompt:` block is text to run in Codex. Start
Codex from your new repository, `../spdd-workshop-snapshot`, paste the prompt,
wait until Codex completes the requested file changes successfully, then exit
Codex before continuing with the next terminal commands in the guide.

The `$spdd-*` commands derive artifact filenames from the requirement context
and current time. In the filenames below, `{timestamp}` represents the timestamp
generated by the command, for example
`./spdd/analysis/GGQPA-XXX-{timestamp}-[Analysis]-token-usage-billing.md`.
The generated SPDD skill treats this position as the
Jira or ticket key: `GGQPA-XXX` means default ticket, and `GGQPA-001`
corresponds to the first enhancement story in the reference history.

Prompt:

```text
$spdd-analysis @./requirements/token-usage-billing-story.md
```

Generated analysis file:
`./spdd/analysis/GGQPA-XXX-{timestamp}-[Analysis]-token-usage-billing.md`

Review for:

- Existing concepts: Customer, PricingPlan, CustomerSubscription, Bill
- New API concepts: UsageRequest, BillResponse
- Current-month usage aggregation from existing bills
- Quota-first billing rules
- Risks around current month boundaries, rounding, and active subscription lookup

Checkpoint:

```bash
git add .
git commit -m "[000] feat: generate an analysis document, first round"
```

### Step 5: Generate The Initial REASONS Canvas

Prompt:

```text
$spdd-reasons-canvas @./spdd/analysis/GGQPA-XXX-{timestamp}-[Analysis]-token-usage-billing.md
```

Generated structured prompt file:
`./spdd/prompt/GGQPA-XXX-{timestamp}-[Feat]-api-token-usage-billing.md`

Review the canvas before code generation. It should cover:

- Requirements for `POST /api/usage`
- Domain entities and DTOs
- Repository, service, controller, exception, and persistence structure
- Operations detailed enough for implementation
- Norms and safeguards for validation, rounding, and error responses

Checkpoint:

```bash
git add .
git commit -m "[000] feat: generate a structured prompt based on the analysis doc, first round"
```

### Step 6: Add Architecture Intent To The Canvas

Before generating code, refine the canvas with the architecture rules that the
reference repo used.

Prompt:

```text
$spdd-prompt-update @./spdd/prompt/GGQPA-XXX-{timestamp}-[Feat]-api-token-usage-billing.md

Update the structured prompt before code generation with these architectural decisions:

1. Use a three-layer architecture: controller -> service -> repository.
2. Separate domain models from persistence objects.
3. Domain classes must be pure Java objects with no JPA or Spring annotations.
4. Persistence objects must live under infrastructure/persistence/entity and use the PO suffix.
5. Repository interfaces must live under repository and be depended on by services.
6. JPA/Spring Data implementations must live under infrastructure/persistence as adapters.
7. Controllers must depend on service interfaces, not service implementations.
8. Services must depend on repository interfaces, not JPA repositories.
9. Add mappers under infrastructure/persistence/mapper for Domain <-> PO conversion.
10. Capture these as Structure, Operations, Norms, and Safeguards updates. Preserve the existing business requirements.
```

Expected canvas changes:

- Adds design principles for three-layer architecture, dependency inversion, and domain-persistence separation
- Introduces `service/impl`, `repository`, `domain`, `infrastructure/persistence`, `entity`, and `mapper` packages
- Adds adapter and mapper operations
- Adds safeguards preventing PO usage outside infrastructure

Checkpoint:

```bash
git add .
git commit -m "[000] feat: communicate additional intent to the AI -- the architecture and some design practices"
```

### Step 7: Generate The Initial Product Code

Prompt:

```text
$spdd-generate @./spdd/prompt/GGQPA-XXX-{timestamp}-[Feat]-api-token-usage-billing.md

Generate the product code task by task from the Operations section.
Do not add unit tests yet.
Keep the implementation inside the packages and boundaries defined by the canvas.
```

Expected production artifacts:

- `./src/main/java/org/tw/token_billing/controller/UsageController.java`
- domain classes under `./src/main/java/org/tw/token_billing/domain/`
- DTOs under `./src/main/java/org/tw/token_billing/dto/`
- exceptions under `./src/main/java/org/tw/token_billing/exception/`
- repository interfaces under `./src/main/java/org/tw/token_billing/repository/`
- JPA adapters, Spring Data repositories, POs, and mappers under `./src/main/java/org/tw/token_billing/infrastructure/persistence/`
- `./src/main/java/org/tw/token_billing/service/BillingService.java`
- `./src/main/java/org/tw/token_billing/service/impl/BillingServiceImpl.java`
- optional generated smoke script `./scripts/api-test.sh`

Compile before committing:

```bash
./gradlew test
```

Checkpoint:

```bash
git add .
git commit -m "[000] feat: the first round of code generated based on the structured prompt"
```

### Step 8: Correct Active Subscription Lookup

Review result: the reference changed the subscription lookup from a list of active
subscriptions to one optional active subscription. This is a behavior/design
correction, so update the prompt first, then the code.

Prompt:

```text
$spdd-prompt-update @./spdd/prompt/GGQPA-XXX-{timestamp}-[Feat]-api-token-usage-billing.md

Business and design correction:
Only one active subscription is allowed for a customer at a given date.
The repository contract should return Optional<CustomerSubscription>, not List<CustomerSubscription>.

Update the relevant Operations sections:
- CustomerSubscriptionRepository method should be findActiveSubscription(String customerId, LocalDate date): Optional<CustomerSubscription>.
- JpaCustomerSubscriptionRepositoryAdapter should map one PO to one domain object.
- SpringDataCustomerSubscriptionRepository should return Optional<CustomerSubscriptionPO>.
- BillingServiceImpl should call findActiveSubscription(...).orElseThrow(...).
```

Follow-up prompt:

```text
$spdd-generate @./spdd/prompt/GGQPA-XXX-{timestamp}-[Feat]-api-token-usage-billing.md

Apply only the code changes needed for the active-subscription lookup correction.
```

Checkpoint:

```bash
git add .
git commit -m "[000] feat: update the logic of querying customer subscription"
```

### Step 9: Refactor `calculateBill`

This is a non-behavioral refactor, so refactor code first and sync the canvas
afterwards.

Prompt:

```text
@./src/main/java/org/tw/token_billing/service/impl/BillingServiceImpl.java

Refactor calculateBill into small private methods without changing observable behavior:
- validateCustomerExists(String customerId)
- resolveActivePricingPlan(String customerId)
- calculateRemainingQuota(String customerId, PricingPlan plan)
Keep calculateBill as the orchestration method.
```

Follow-up prompt:

```text
$spdd-sync @./spdd/prompt/GGQPA-XXX-{timestamp}-[Feat]-api-token-usage-billing.md

Synchronize the BillingServiceImpl refactor back into the Operations section.
```

Checkpoint:

```bash
git add .
git commit -m "[000] feat: refactoring for calculateBill"
```

### Step 10: Extract Magic Numbers In `Bill`

Prompt:

```text
@./src/main/java/org/tw/token_billing/domain/Bill.java

Extract billing calculation magic numbers into named constants:
- TOKENS_PER_PRICING_UNIT = 1000
- CALCULATION_PRECISION_SCALE = 10
- CURRENCY_SCALE = 2
Do not change billing behavior.
```

Follow-up prompt:

```text
$spdd-sync @./spdd/prompt/GGQPA-XXX-{timestamp}-[Feat]-api-token-usage-billing.md

Sync the Bill constant refactor into the structured prompt.
```

Checkpoint:

```bash
git add .
git commit -m "[000] feat: refactoring for magic numbers in the Bill"
```

### Step 11: Remove Unused Methods

Prompt:

```text
Review the generated code for unused domain, DTO, and mapper methods.
Remove only methods that are not called by production code or tests.
Do not change observable behavior.
Expected removals include:
- CustomerSubscription.isActiveOn(LocalDate)
- UsageRequest.getTotalTokens()
- unused toPO methods from read-only mappers where the adapter does not use them
```

Follow-up prompt:

```text
$spdd-sync @./spdd/prompt/GGQPA-XXX-{timestamp}-[Feat]-api-token-usage-billing.md

Sync the removed unused methods back into the structured prompt.
```

Checkpoint:

```bash
git add .
git commit -m "[000] feat: refactoring for all the unused methods"
```

### Step 12: Add Initial Tests

Prompt:

```text
Create ./spdd/template/TEST-SCENARIOS-TEMPLATE.md.
It should define sections for controller, service, repository, DAO, model class,
and integration test scenarios. It should require test names in the format:
should_return_[expected_output]_when_[action]_given_[input]
```

Follow-up prompt:

```text
Based on the implementation details prompt
@./spdd/prompt/GGQPA-XXX-{timestamp}-[Feat]-api-token-usage-billing.md
combined with the template @./spdd/template/TEST-SCENARIOS-TEMPLATE.md,
generate the test prompt file:
./spdd/prompt/GGQPA-XXX-{timestamp}-[Test]-api-token-usage-billing.md
```

Follow-up prompt:

```text
Based on @./spdd/prompt/GGQPA-XXX-{timestamp}-[Test]-api-token-usage-billing.md,
generate the corresponding unit and integration tests for the current billing implementation.
```

Expected tests:

- `./src/test/java/org/tw/token_billing/UsageControllerIntegrationTest.java`
- `./src/test/java/org/tw/token_billing/controller/UsageControllerTest.java`
- `./src/test/java/org/tw/token_billing/domain/BillTest.java`
- `./src/test/java/org/tw/token_billing/dto/BillResponseTest.java`
- `./src/test/java/org/tw/token_billing/infrastructure/persistence/JpaBillRepositoryAdapterTest.java`
- `./src/test/java/org/tw/token_billing/service/impl/BillingServiceImplTest.java`

Verify:

```bash
./gradlew test
```

Checkpoint:

```bash
git add .
git commit -m "[000] feat: add tests for all the generated codes"
```

## Part 2: Replay The Article Enhancement

Now implement the multi-plan, model-aware billing enhancement described in the
SPDD article.

### Step 13: Add The Broad Enhancement Story

Create `./requirements/enhancement-token-usage-billing-story.md`.

Use this content as the requirement to be refined by SPDD:

```text
## Background
As our LLM API platform scales, a single pricing model is no longer sufficient.
We are transitioning to a multi-plan, model-aware billing system. We need to
refactor our existing billing engine to support different subscription strategies,
variable pricing based on the AI model invoked, and complex volume-based tiered
discounts.

## Business Value
1. Flexible Monetization: Support diverse billing strategies (Standard, Premium,
   Enterprise) to capture different market segments.
2. Model-Aware Pricing: Charge different rates based on the specific AI model used.
3. Architecture Scalability: Implement an extensible design to isolate complex
   tiering logic and easily add future pricing models.

## Scope In
* Enhance the existing POST /api/usage endpoint.
* New Request Field: Add modelId (required, string, e.g. "fast-model",
  "reasoning-model").
* Introduce dynamic billing calculation based on three Plan Types:
  1. Standard Plan: Has a monthly global quota. Overage rates now depend on modelId.
  2. Premium Plan: No quota. Prompt and completion tokens are billed separately,
     and rates vary by modelId.
  3. Enterprise Plan: Volume-based tiered discount. The price drops as the
     customer's total monthly usage crosses specific thresholds.
* Implement a Strategy/Factory routing mechanism.

## Scope Out
* Subscription plan creation and assignment CRUD.
* Invoice generation.
* Changing plan status mid-billing cycle.

## Acceptance Criteria
1. Invalid request, such as missing modelId or negative tokens, returns HTTP 400.
2. Standard customer with 100,000 monthly quota and 90,000 used submits 30,000
   fast-model tokens at $0.01/1K. Bill shows 10,000 from quota, 20,000 overage,
   and $0.20 charge.
3. Premium customer submits 10,000 prompt and 20,000 completion tokens for
   reasoning-model at $0.03/1K prompt and $0.06/1K completion. Bill shows total
   $1.50.
4. Enterprise customer using fast-model crosses a 50,000 token threshold and the
   bill splits usage across tier rates.
```

Checkpoint:

```bash
git add .
git commit -m "[000] feat: add a enhancement story"
```

### Step 14: Split The Broad Story

Use SPDD story decomposition or a direct prompt. The reference splits the
broad story into a deliverable Story 1 and a later Story 2.

Prompt:

```text
Split @./requirements/enhancement-token-usage-billing-story.md into two independent
INVEST-style stories:

1. Story 1: Multi-Plan Billing Foundation & Model-Aware Pricing.
   Include Standard plan model-aware overage and Premium plan split-rate billing.
   Defer Enterprise tiered pricing.
2. Story 2: Enterprise Plan Volume-Based Tiered Billing.
   Depend on Story 1.

Keep each story concise with Background, Business Value, Scope In, Scope Out,
and Acceptance Criteria.
Delete ./requirements/enhancement-token-usage-billing-story.md after creating the
two story files.
```

Expected files:

```text
./requirements/[User-story-1]Multi-Plan-Billing-Foundation-&-Model-Aware-Pricing.md
./requirements/[User-story-2]Enterprise-Plan-Volume-Based-Tiered-Billing.md
```

Checkpoint:

```bash
git add .
git commit -m "[001] feat: split one story into two to ensure the singularity of responsibilities and appropriate complexity"
```

### Step 15: Generate Enhancement Analysis

Prompt:

```text
$spdd-analysis @./requirements/[User-story-1]Multi-Plan-Billing-Foundation-&-Model-Aware-Pricing.md
```

Generated analysis file:
`./spdd/analysis/GGQPA-001-{timestamp}-[Analysis]-multi-plan-billing-model-aware-pricing.md`

Review for:

- Existing concepts that need extension: PricingPlan, Bill, UsageRequest, BillResponse
- New concepts: PlanType, ModelPricing, BillingStrategy, StandardBillingStrategy, PremiumBillingStrategy, BillingStrategyFactory
- Strategy/factory design
- Risk around nullable `model_id` for existing bills
- Response breakdown for prompt and completion charges

Checkpoint:

```bash
git add .
git commit -m "[001] feat: generate an analysis doc for enhanced story"
```

### Step 16: Generate Enhancement REASONS Canvas

Prompt:

```text
$spdd-reasons-canvas @./spdd/analysis/GGQPA-001-{timestamp}-[Analysis]-multi-plan-billing-model-aware-pricing.md
```

Generated structured prompt file:
`./spdd/prompt/GGQPA-001-{timestamp}-[Feat]-multi-plan-billing-model-aware-pricing.md`

At this point the reference canvas still allowed `bills.model_id` to be nullable
for backward compatibility. Do not apply the `fast-model` default yet; that is a
later prompt-first update.

Checkpoint:

```bash
git add .
git commit -m "[001] feat: generated a structured prompt based on the analysis doc"
```

### Step 17: Generate Enhancement Code

Prompt:

```text
$spdd-generate @./spdd/prompt/GGQPA-001-{timestamp}-[Feat]-multi-plan-billing-model-aware-pricing.md

Generate the code for Story 1 only.
Do not implement Enterprise tiered billing.
Keep all changes inside the scope of the structured prompt.
```

Expected changes:

- Add `modelId` to `UsageRequest`, `Bill`, `BillPO`, `BillResponse`
- Add `promptCharge` and `completionCharge`
- Add `PlanType`
- Add `ModelPricing` domain, PO, mapper, repository, and adapter
- Add `BillingContext`
- Add `BillingStrategy`, `StandardBillingStrategy`, `PremiumBillingStrategy`, and `BillingStrategyFactory`
- Add `ModelPricingNotFoundException`
- Add `./src/main/resources/db/migration/V2__Add_model_pricing.sql`
- Update existing tests enough to compile
- Add `./src/test/resources/application.yml` if needed for H2/Flyway test setup

Verify:

```bash
./gradlew test
```

Checkpoint:

```bash
git add .
git commit -m "[001] feat: generate the codes based on the structured prompt"
```

### Step 18: Generate Functional API Test Script

Generate the optional API test skill if it is not present:

```bash
openspdd --tool codex generate spdd-api-test
```

Prompt:

```text
$spdd-api-test

Generate a self-contained bash script at ./scripts/test-api.sh for the current
POST /api/usage API. Cover:
- validation errors
- Standard plan model-aware overage
- Premium plan split-rate billing
- edge cases for zero tokens, single token side, invalid JSON, and large usage
Use curl only. Do not require jq.
```

Verify manually with the app running:

```bash
docker compose up -d
./gradlew bootRun
```

In another terminal:

```bash
sh ./scripts/test-api.sh
```

Checkpoint:

```bash
git add .
git commit -m "[001] feat: generate functional testing shell script by the spdd api test command"
```

### Step 19: Refactor Remaining Magic Numbers

The article calls out magic numbers in `BillingServiceImpl.calculateRemainingQuota`.
Refactor code first, then sync the prompt.

Prompt:

```text
@./src/main/java/org/tw/token_billing/service/impl/BillingServiceImpl.java

In calculateRemainingQuota, extract magic numbers used for month-boundary
calculation into meaningful constants. Do not change observable behavior.
```

Follow-up prompt:

```text
$spdd-sync @./spdd/prompt/GGQPA-001-{timestamp}-[Feat]-multi-plan-billing-model-aware-pricing.md

Sync the magic-number refactor back into the structured prompt.
```

Checkpoint:

```bash
git add .
git commit -m "[001] feat: magic number refactoring"
```

### Step 20: Generate The Enhancement Test Prompt

Prompt:

```text
Based on the implementation details prompt
@./spdd/prompt/GGQPA-001-{timestamp}-[Feat]-multi-plan-billing-model-aware-pricing.md
combined with the template @./spdd/template/TEST-SCENARIOS-TEMPLATE.md,
generate a test prompt file:
./spdd/prompt/GGQPA-001-{timestamp}-[Test]-multi-plan-billing-model-aware-pricing.md
```

Follow-up prompt:

```text
@./spdd/prompt/GGQPA-001-{timestamp}-[Test]-multi-plan-billing-model-aware-pricing.md

There are tests that duplicate existing ones. Compare the relevant existing tests
and keep only new scenarios needed for model-aware Standard billing, Premium
split-rate billing, strategy/factory routing, model pricing persistence, and
updated DTO/mapper behavior.
```

Checkpoint:

```bash
git add .
git commit -m "[001] feat: generate a test structure prompt"
```

### Step 21: Generate Enhancement Unit Tests

Prompt:

```text
Based on the generated test prompt
@./spdd/prompt/GGQPA-001-{timestamp}-[Test]-multi-plan-billing-model-aware-pricing.md,
generate the corresponding unit and integration test code.
Only add tests for scenarios not already covered.
```

Expected new tests:

- `./src/test/java/org/tw/token_billing/service/strategy/BillingStrategyFactoryTest.java`
- `./src/test/java/org/tw/token_billing/service/strategy/StandardBillingStrategyTest.java`
- `./src/test/java/org/tw/token_billing/service/strategy/PremiumBillingStrategyTest.java`
- `./src/test/java/org/tw/token_billing/infrastructure/persistence/JpaModelPricingRepositoryAdapterTest.java`
- `./src/test/java/org/tw/token_billing/infrastructure/persistence/mapper/ModelPricingMapperTest.java`
- `./src/test/java/org/tw/token_billing/infrastructure/persistence/mapper/PricingPlanMapperTest.java`
- updates to controller, integration, and billing service tests

Verify:

```bash
./gradlew test
```

Checkpoint:

```bash
git add .
git commit -m "[001] feat: generate all the tests"
```

### Step 22: Prompt-First Update For Required `model_id`

This is a business/behavior correction. Update the prompt first.

Prompt:

```text
$spdd-prompt-update @./spdd/prompt/GGQPA-001-{timestamp}-[Feat]-multi-plan-billing-model-aware-pricing.md

model_id is a required field, and its default value is fast-model.
Based on this decision, update the corresponding parts of the structured prompt.

Required updates:
- V2 migration must add bills.model_id as VARCHAR(50) NOT NULL DEFAULT 'fast-model'.
- BillPO.modelId must be annotated nullable = false.
- Safeguards must state that bills.model_id is NOT NULL with default 'fast-model' for backward compatibility with existing bills.
- Preserve all other Story 1 behavior.
```

Checkpoint:

```bash
git add .
git commit -m "[001] feat: update the structured prompt to make model_id a required field with a default value of 'fast-model' to ensure backward compatibility with existing bills."
```

### Step 23: Generate Code From The Updated Prompt

Prompt:

```text
$spdd-generate @./spdd/prompt/GGQPA-001-{timestamp}-[Feat]-multi-plan-billing-model-aware-pricing.md

Apply only the code changes required by the updated model_id default decision.
Do not regenerate unrelated files.
```

Expected code changes:

- `./src/main/resources/db/migration/V2__Add_model_pricing.sql`
  uses `ALTER TABLE bills ADD COLUMN model_id VARCHAR(50) NOT NULL DEFAULT 'fast-model';`
- `BillPO.modelId` uses `@Column(name = "model_id", length = 50, nullable = false)`

Verify:

```bash
./gradlew test
```

Checkpoint:

```bash
git add .
git commit -m "[001] feat: update the code based on the structured prompt"
```

### Step 24: Update The API Test Command Output

The reference enhanced the generated API test command/script so the script has a
human-reviewable test case overview and a final result table. Update the project
skill bundle if it exists, then regenerate or patch the script.

Prompt:

```text
@./.agents/skills/spdd-api-test/SKILL.md
@./scripts/test-api.sh

Enhance the API test generation guidance and the current script:
- Add a TEST CASE OVERVIEW section near the top of ./scripts/test-api.sh.
- Track test IDs, descriptions, expected status, actual status, and pass/fail result.
- Print a final TEST RESULTS SUMMARY table.
- Keep the same curl-only behavior and existing scenarios.
```

Verify:

```bash
sh ./scripts/test-api.sh
```

Checkpoint:

```bash
git add .
git commit -m "[001] feat: update the api test command"
```

## Part 3: Article Story-Generation Artifacts

The last four reference commits were added after the main implementation to
support the article narrative around `/spdd-story`. Keep this order if you want
the workshop history to stay close to `../token-billing`.

### Step 25: Add The Story Command And First Initial Story

Generate the optional story skill:

```bash
openspdd --tool codex generate spdd-story
```

Prompt:

```text
$spdd-story

High-level enhancement:
We need to enhance the billing engine to support diverse subscription strategies
and variable, model-specific pricing. Add modelId to POST /api/usage, support
model-aware Standard plan overage, add Premium split-rate billing, and keep the
architecture extensible via Strategy/Factory.

Generate the Standard-plan foundation story and save it as:
./requirements/[User-story-1-initial]Multi-Plan-Billing-Foundation-&-Standard-Plan-Model-Aware-Pricing.md
```

Checkpoint:

```bash
git add .
git commit -m "[001] feat: add a command for generating stories"
```

### Step 26: Add The Initial Enhancement Idea

Create `./requirements/idea-of-the-enhancement.md`:

```text
# The enhancement content

We need to enhance the billing engine to support diverse subscription strategies
and variable, model-specific pricing. This requires several changes

- API enhancement: update the existing POST /api/usage endpoint to accept a new,
  required modelId parameter (e.g., "fast-model", "reasoning-model").
- Model-aware pricing: shift from a single global rate to dynamic pricing, where
  costs vary depending on the specific AI model invoked.
- Multi-plan billing logic: introduce distinct billing behaviors based on the
  customer's subscription tier:
  - Standard plan: retains the global monthly quota, but overage usage is now
    calculated using model-specific rates.
  - Premium plan: operates without a quota limit. Prompt tokens and completion
    tokens are charged separately at different rates depending on the model used.
- Architectural scalability: implement an extensible design pattern, such as
  Strategy or Factory, to isolate calculation formulas for different plans.
```

Checkpoint:

```bash
git add .
git commit -m "[001] feat: add an initial idea of the enhancement"
```

### Step 27: Generate The Second Initial Story

Prompt:

```text
$spdd-story @./requirements/idea-of-the-enhancement.md

The Standard plan foundation story already exists at:
@./requirements/[User-story-1-initial]Multi-Plan-Billing-Foundation-&-Standard-Plan-Model-Aware-Pricing.md

Generate the Premium Plan Split-Rate Billing story as:
./requirements/[User-story-2-initial]Premium-Plan-Split-Rate-Billing.md

Also refine the first initial story if needed so the two stories have singular
responsibilities and do not duplicate scope.
```

Checkpoint:

```bash
git add .
git commit -m "[001] feat: submit another story"
```

### Step 28: Rename Initial Story Files

Rename the two initial story files so they match the article naming:

```bash
git mv './requirements/[User-story-1-initial]Multi-Plan-Billing-Foundation-&-Standard-Plan-Model-Aware-Pricing.md' './requirements/[User-story-1-1-initial]Multi-Plan-Billing-Foundation-&-Standard-Plan-Model-Aware-Pricing.md'
git mv './requirements/[User-story-2-initial]Premium-Plan-Split-Rate-Billing.md' './requirements/[User-story-1-2-initial]Premium-Plan-Split-Rate-Billing.md'
```

Checkpoint:

```bash
git commit -m "[001] feat: update the file name"
```

## Final Verification

Run the automated tests:

```bash
./gradlew test
```

Run API regression tests:

```bash
docker compose up -d
./gradlew bootRun
```

In another terminal:

```bash
sh ./scripts/test-api.sh
```

Check the resulting history:

```bash
git log --reverse --oneline
```

Compare the shape with the reference:

```bash
git -C ../token-billing log --reverse --oneline
```

Expected final generated artifact groups:

```text
./requirements/*.md
./spdd/analysis/GGQPA-*-{timestamp}-[Analysis]-*.md
./spdd/prompt/GGQPA-*-{timestamp}-[*]-*.md
./spdd/template/TEST-SCENARIOS-TEMPLATE.md
./scripts/test-api.sh
./src/main/java/org/tw/token_billing/
./src/test/java/org/tw/token_billing/
./src/test/resources/application.yml
./.agents/skills/<skill-id>/SKILL.md
```

## Review Rules For The Workshop

- Requirements or business behavior change: update the structured prompt first,
  then run `$spdd-generate`.
- Refactoring or style-only cleanup: update code first, then run `$spdd-sync`.
- Do not manually edit generated prompt files as a shortcut during the workshop;
  use OpenSPDD commands and AI prompts so the prompt remains the governed
  artifact.
- Review every generated artifact before committing.
- Keep each commit focused on one SPDD checkpoint.
- Use `../token-billing` only for comparison after your own generated result is
  present.
