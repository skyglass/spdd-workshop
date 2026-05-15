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

| Command | Purpose |
| --- | --- |
| `spdd-story` | Split or refine enhancement ideas into focused user stories. |
| `spdd-analysis` | Analyze a story and produce domain concepts, risks, and design direction. |
| `spdd-reasons-canvas` | Generate the REASONS Canvas used as the implementation blueprint. |
| `spdd-generate` | Generate or update code from the current structured prompt. |
| `spdd-api-test` | Generate a cURL-based functional API test script. |
| `spdd-prompt-update` | Update the structured prompt first when requirements or behavior change. |
| `spdd-sync` | Sync code-side refactors or fixes back into the structured prompt. |

OpenSPDD writes Codex skills under `.agents/skills/<skill-id>/SKILL.md`.
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
| 0 | Create the initial Spring Boot billing service |
| 1 | Normalize the database environment for plans and subscriptions |
| 2 | Initialize OpenSPDD Codex skills |
| 3 | Generate the initial analysis document |
| 4 | Generate the initial REASONS Canvas |
| 5 | Add architecture intent to the Canvas |
| 6 | Generate the initial product code |
| 7 | Correct active subscription lookup through prompt-first update |
| 8 | Refactor `calculateBill` and sync the Canvas |
| 9 | Extract magic numbers in `Bill` and sync the Canvas |
| 10 | Remove unused methods and sync the Canvas |
| 11 | Add initial tests |
| 12 | Add the broad enhancement story |
| 13 | Split the broad story into focused stories |
| 14 | Generate enhancement analysis |
| 15 | Generate enhancement REASONS Canvas |
| 16 | Generate enhancement code |
| 17 | Generate the functional API test script |
| 18 | Refactor remaining magic numbers and sync the Canvas |
| 19 | Generate the enhancement test prompt |
| 20 | Generate enhancement unit tests |
| 21 | Update the structured prompt for required `model_id` |
| 22 | Generate code from the updated prompt |
| 23 | Update the API test command output |
| 24 | Add the story command and first initial story |
| 25 | Add the initial enhancement idea |
| 26 | Generate the second initial story |
| 27 | Rename initial story files |

## Part 1: Replay The Initial Billing Service

The article focuses on the enhancement, but the reference history begins with a
minimal Spring Boot seed project. Create that seed project first, then generate
the original billing service using SPDD.

### Step 0: Create The Initial Project From Empty

Prompt:

```text
Bootstrap an empty repository for a Spring Boot token billing service.

Create only the initial project skeleton and seed requirement. Do not implement
the usage API, controller, service, repositories, domain model, or DTOs yet.

Files to create:
- .gitignore for a Java/Gradle/Spring Boot project
- settings.gradle with rootProject.name = 'token-billing'
- build.gradle for Java 21, Spring Boot 3.5.x, dependency-management, Web,
  Data JPA, Validation, Flyway, PostgreSQL runtime, H2 runtime, Lombok, and
  Spring Boot test dependencies
- compose.yaml with a postgres service:
  - POSTGRES_DB=token_billing
  - POSTGRES_USER=postgres
  - POSTGRES_PASSWORD=postgres
  - host port 54323 mapped to container port 5432
- src/main/java/org/tw/token_billing/TokenBillingApplication.java
- src/test/java/org/tw/token_billing/TokenBillingApplicationTests.java
- src/main/resources/application.yml configured for the postgres service,
  Flyway enabled, and Hibernate ddl-auto validate
- src/main/resources/db/migration/V1__Create_tables.sql with the initial
  denormalized schema:
  - customers table with id, name, monthly_quota, used_tokens_this_month,
    overage_rate_per_1k, created_at, updated_at
  - bills table with id, customer_id, prompt_tokens, completion_tokens,
    total_tokens, included_tokens_used, overage_tokens, total_charge,
    calculated_at
  - indexes on bills(customer_id) and bills(calculated_at)
- requirements/token-usage-billing-story.md with the initial billing story:
  Background, Business Value, Scope In, Scope Out, and five Given/When/Then
  acceptance criteria for customer existence, non-negative token counts,
  within-quota billing, over-quota billing, and HTTP 201 successful return
- README.md as an application README for the Token Billing Service, not this
  workshop guide

Keep the package name org.tw.token_billing.
Use ASCII text.
```

Generate the Gradle wrapper after the AI creates `build.gradle`:

```bash
gradle wrapper --gradle-version 8.14 --distribution-type bin
```

If you do not have Gradle installed locally, create the wrapper with your normal
Spring Boot/Gradle bootstrap method before committing. The first commit should
include `gradlew`, `gradlew.bat`, `gradle/wrapper/gradle-wrapper.jar`, and
`gradle/wrapper/gradle-wrapper.properties`.

Verify the seed project:

```bash
./gradlew test
```

Checkpoint:

```bash
git status --short
git add .gitignore README.md build.gradle settings.gradle compose.yaml gradlew gradlew.bat gradle requirements src
git commit -m "[000] feat: init the repo"
```

### Step 1: Update The Environment

The initial seed schema stores quota and rate directly on `customers`. Update the
environment so the later SPDD workflow starts from a normalized customer, plan,
subscription, and bill model.

Prompt:

```text
Update the initial environment for the token billing service.

Scope:
- Keep the existing Spring Boot project structure.
- Update requirements/token-usage-billing-story.md only if it has drifted from the current billing story.
- Refactor src/main/resources/db/migration/V1__Create_tables.sql so:
  - customers contains id, name, created_at only.
  - pricing_plans contains id, name, monthly_quota, overage_rate_per_1k, created_at.
  - customer_subscriptions links customer_id to plan_id with effective_from, effective_to, created_at.
  - bills stores prompt_tokens, completion_tokens, total_tokens, included_tokens_used, overage_tokens, total_charge, calculated_at.
  - indexes exist for customer_subscriptions(customer_id), bills(customer_id), and bills(calculated_at).
  - seed plans include PLAN-FREE, PLAN-STARTER, PLAN-PRO, PLAN-ENTERPRISE.
  - seed customers include CUST-001, CUST-002, CUST-003.
  - seed subscriptions assign those customers to starter, free, and enterprise plans.
Do not implement Java code yet.
```

Checkpoint:

```bash
git add requirements/token-usage-billing-story.md src/main/resources/db/migration/V1__Create_tables.sql
git commit -m "[000] feat: update the environment"
```

### Step 2: Initialize OpenSPDD Skills

Generate the project-scoped SPDD skills:

```bash
openspdd --tool codex init
openspdd --tool codex generate --all
```

Expected OpenSPDD skill artifacts:

```text
.agents/skills/spdd-analysis/SKILL.md
.agents/skills/spdd-reasons-canvas/SKILL.md
.agents/skills/spdd-generate/SKILL.md
.agents/skills/spdd-prompt-update/SKILL.md
.agents/skills/spdd-sync/SKILL.md
```

Commit the generated `.agents/skills/` files.

Checkpoint:

```bash
git add .agents
git commit -m "[000] feat: initialize the commands for the SPDD workflow"
```

### Step 3: Generate The Initial Analysis

Prompt:

```text
$spdd-analysis @requirements/token-usage-billing-story.md

Save the generated analysis as:
spdd/analysis/GGQPA-XXX-202603131734-[Analysis]-token-usage-billing.md
```

Review for:

- Existing concepts: Customer, PricingPlan, CustomerSubscription, Bill
- New API concepts: UsageRequest, BillResponse
- Current-month usage aggregation from existing bills
- Quota-first billing rules
- Risks around current month boundaries, rounding, and active subscription lookup

Checkpoint:

```bash
git add spdd/analysis/GGQPA-XXX-202603131734-[Analysis]-token-usage-billing.md
git commit -m "[000] feat: generate an analysis document, first round"
```

### Step 4: Generate The Initial REASONS Canvas

Prompt:

```text
$spdd-reasons-canvas @spdd/analysis/GGQPA-XXX-202603131734-[Analysis]-token-usage-billing.md

Save the generated structured prompt as:
spdd/prompt/GGQPA-XXX-202603131758-[Feat]-api-token-usage-billing.md
```

Review the canvas before code generation. It should cover:

- Requirements for `POST /api/usage`
- Domain entities and DTOs
- Repository, service, controller, exception, and persistence structure
- Operations detailed enough for implementation
- Norms and safeguards for validation, rounding, and error responses

Checkpoint:

```bash
git add spdd/prompt/GGQPA-XXX-202603131758-[Feat]-api-token-usage-billing.md
git commit -m "[000] feat: generate a structured prompt based on the analysis doc, first round"
```

### Step 5: Add Architecture Intent To The Canvas

Before generating code, refine the canvas with the architecture rules that the
reference repo used.

Prompt:

```text
$spdd-prompt-update @spdd/prompt/GGQPA-XXX-202603131758-[Feat]-api-token-usage-billing.md

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
git add spdd/prompt/GGQPA-XXX-202603131758-[Feat]-api-token-usage-billing.md
git commit -m "[000] feat: communicate additional intent to the AI -- the architecture and some design practices"
```

### Step 6: Generate The Initial Product Code

Prompt:

```text
$spdd-generate @spdd/prompt/GGQPA-XXX-202603131758-[Feat]-api-token-usage-billing.md

Generate the product code task by task from the Operations section.
Do not add unit tests yet.
Keep the implementation inside the packages and boundaries defined by the canvas.
```

Expected production artifacts:

- `controller/UsageController.java`
- `domain/Customer.java`, `PricingPlan.java`, `CustomerSubscription.java`, `Bill.java`
- `dto/UsageRequest.java`, `BillResponse.java`, `ErrorResponse.java`
- `exception/CustomerNotFoundException.java`, `NoActiveSubscriptionException.java`, `GlobalExceptionHandler.java`
- repository interfaces under `repository/`
- JPA adapters, Spring Data repositories, POs, and mappers under `infrastructure/persistence/`
- `service/BillingService.java`
- `service/impl/BillingServiceImpl.java`
- optional generated smoke script `scripts/api-test.sh`

Compile before committing:

```bash
./gradlew test
```

Checkpoint:

```bash
git add scripts src/main/java
git commit -m "[000] feat: the first round of code generated based on the structured prompt"
```

### Step 7: Correct Active Subscription Lookup

Review result: the reference changed the subscription lookup from a list of active
subscriptions to one optional active subscription. This is a behavior/design
correction, so update the prompt first, then the code.

Prompt:

```text
$spdd-prompt-update @spdd/prompt/GGQPA-XXX-202603131758-[Feat]-api-token-usage-billing.md

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
$spdd-generate @spdd/prompt/GGQPA-XXX-202603131758-[Feat]-api-token-usage-billing.md

Apply only the code changes needed for the active-subscription lookup correction.
```

Checkpoint:

```bash
git add spdd/prompt/GGQPA-XXX-202603131758-[Feat]-api-token-usage-billing.md src/main/java/org/tw/token_billing
git commit -m "[000] feat: update the logic of querying customer subscription"
```

### Step 8: Refactor `calculateBill`

This is a non-behavioral refactor, so refactor code first and sync the canvas
afterwards.

Prompt:

```text
@src/main/java/org/tw/token_billing/service/impl/BillingServiceImpl.java

Refactor calculateBill into small private methods without changing observable behavior:
- validateCustomerExists(String customerId)
- resolveActivePricingPlan(String customerId)
- calculateRemainingQuota(String customerId, PricingPlan plan)
Keep calculateBill as the orchestration method.
```

Follow-up prompt:

```text
$spdd-sync @spdd/prompt/GGQPA-XXX-202603131758-[Feat]-api-token-usage-billing.md

Synchronize the BillingServiceImpl refactor back into the Operations section.
```

Checkpoint:

```bash
git add spdd/prompt/GGQPA-XXX-202603131758-[Feat]-api-token-usage-billing.md src/main/java/org/tw/token_billing/service/impl/BillingServiceImpl.java
git commit -m "[000] feat: refactoring for calculateBill"
```

### Step 9: Extract Magic Numbers In `Bill`

Prompt:

```text
@src/main/java/org/tw/token_billing/domain/Bill.java

Extract billing calculation magic numbers into named constants:
- TOKENS_PER_PRICING_UNIT = 1000
- CALCULATION_PRECISION_SCALE = 10
- CURRENCY_SCALE = 2
Do not change billing behavior.
```

Follow-up prompt:

```text
$spdd-sync @spdd/prompt/GGQPA-XXX-202603131758-[Feat]-api-token-usage-billing.md

Sync the Bill constant refactor into the structured prompt.
```

Checkpoint:

```bash
git add spdd/prompt/GGQPA-XXX-202603131758-[Feat]-api-token-usage-billing.md src/main/java/org/tw/token_billing/domain/Bill.java
git commit -m "[000] feat: refactoring for magic numbers in the Bill"
```

### Step 10: Remove Unused Methods

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
$spdd-sync @spdd/prompt/GGQPA-XXX-202603131758-[Feat]-api-token-usage-billing.md

Sync the removed unused methods back into the structured prompt.
```

Checkpoint:

```bash
git add spdd/prompt/GGQPA-XXX-202603131758-[Feat]-api-token-usage-billing.md src/main/java/org/tw/token_billing
git commit -m "[000] feat: refactoring for all the unsed meteds"
```

Use the typo in the commit message to match the reference history.

### Step 11: Add Initial Tests

Prompt:

```text
Create spdd/template/TEST-SCENARIOS-TEMPLATE.md.
It should define sections for controller, service, repository, DAO, model class,
and integration test scenarios. It should require test names in the format:
should_return_[expected_output]_when_[action]_given_[input]
```

Follow-up prompt:

```text
Based on the implementation details prompt
@spdd/prompt/GGQPA-XXX-202603131758-[Feat]-api-token-usage-billing.md
combined with the template @spdd/template/TEST-SCENARIOS-TEMPLATE.md,
generate the test prompt file:
spdd/prompt/GGQPA-XXX-202603131758-[Test]-api-token-usage-billing.md
```

Follow-up prompt:

```text
Based on @spdd/prompt/GGQPA-XXX-202603131758-[Test]-api-token-usage-billing.md,
generate the corresponding unit and integration tests for the current billing implementation.
```

Expected tests:

- `UsageControllerIntegrationTest`
- `controller/UsageControllerTest`
- `domain/BillTest`
- `dto/BillResponseTest`
- `infrastructure/persistence/JpaBillRepositoryAdapterTest`
- `service/impl/BillingServiceImplTest`

Verify:

```bash
./gradlew test
```

Checkpoint:

```bash
git add spdd src/test
git commit -m "[000] feat: add tests for all the generated codes"
```

## Part 2: Replay The Article Enhancement

Now implement the multi-plan, model-aware billing enhancement described in the
SPDD article.

### Step 12: Add The Broad Enhancement Story

Create `requirements/enhancement-token-usage-billing-story.md`.

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
git add requirements/enhancement-token-usage-billing-story.md
git commit -m "[000] feat: add a enhancement story"
```

### Step 13: Split The Broad Story

Use SPDD story decomposition or a direct prompt. The reference splits the
broad story into a deliverable Story 1 and a later Story 2.

Prompt:

```text
Split @requirements/enhancement-token-usage-billing-story.md into two independent
INVEST-style stories:

1. Story 1: Multi-Plan Billing Foundation & Model-Aware Pricing.
   Include Standard plan model-aware overage and Premium plan split-rate billing.
   Defer Enterprise tiered pricing.
2. Story 2: Enterprise Plan Volume-Based Tiered Billing.
   Depend on Story 1.

Keep each story concise with Background, Business Value, Scope In, Scope Out,
and Acceptance Criteria.
Delete requirements/enhancement-token-usage-billing-story.md after creating the
two story files.
```

Expected files:

```text
requirements/[User-story-1]Multi-Plan-Billing-Foundation-&-Model-Aware-Pricing.md
requirements/[User-story-2]Enterprise-Plan-Volume-Based-Tiered-Billing.md
```

Checkpoint:

```bash
git add requirements
git commit -m "[001] feat: split one story into two to ensure the singularity of responsibilities and appropriate complexity"
```

### Step 14: Generate Enhancement Analysis

Prompt:

```text
$spdd-analysis @requirements/[User-story-1]Multi-Plan-Billing-Foundation-&-Model-Aware-Pricing.md

Save the generated analysis as:
spdd/analysis/GGQPA-001-202603191100-[Analysis]-multi-plan-billing-model-aware-pricing.md
```

Review for:

- Existing concepts that need extension: PricingPlan, Bill, UsageRequest, BillResponse
- New concepts: PlanType, ModelPricing, BillingStrategy, StandardBillingStrategy, PremiumBillingStrategy, BillingStrategyFactory
- Strategy/factory design
- Risk around nullable `model_id` for existing bills
- Response breakdown for prompt and completion charges

Checkpoint:

```bash
git add spdd/analysis/GGQPA-001-202603191100-[Analysis]-multi-plan-billing-model-aware-pricing.md
git commit -m "[001] feat: generate an analysis doc for enhanced story"
```

### Step 15: Generate Enhancement REASONS Canvas

Prompt:

```text
$spdd-reasons-canvas @spdd/analysis/GGQPA-001-202603191100-[Analysis]-multi-plan-billing-model-aware-pricing.md

Save the generated structured prompt as:
spdd/prompt/GGQPA-001-202603191105-[Feat]-multi-plan-billing-model-aware-pricing.md
```

At this point the reference canvas still allowed `bills.model_id` to be nullable
for backward compatibility. Do not apply the `fast-model` default yet; that is a
later prompt-first update.

Checkpoint:

```bash
git add spdd/prompt/GGQPA-001-202603191105-[Feat]-multi-plan-billing-model-aware-pricing.md
git commit -m "[001] feat: generated a structured prompt based on the analysis doc"
```

### Step 16: Generate Enhancement Code

Prompt:

```text
$spdd-generate @spdd/prompt/GGQPA-001-202603191105-[Feat]-multi-plan-billing-model-aware-pricing.md

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
- Add `V2__Add_model_pricing.sql`
- Update existing tests enough to compile
- Add `src/test/resources/application.yml` if needed for H2/Flyway test setup

Verify:

```bash
./gradlew test
```

Checkpoint:

```bash
git add src
git commit -m "[001] feat: generate the codes based on the structured prompt"
```

### Step 17: Generate Functional API Test Script

Generate the optional API test skill if it is not present:

```bash
openspdd --tool codex generate spdd-api-test
```

Prompt:

```text
$spdd-api-test

Generate a self-contained bash script at scripts/test-api.sh for the current
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
sh scripts/test-api.sh
```

Checkpoint:

```bash
git add .agents/skills/spdd-api-test scripts/test-api.sh
git commit -m "[001] feat: generate functional testing shell script by the spdd api test command"
```

### Step 18: Refactor Remaining Magic Numbers

The article calls out magic numbers in `BillingServiceImpl.calculateRemainingQuota`.
Refactor code first, then sync the prompt.

Prompt:

```text
@src/main/java/org/tw/token_billing/service/impl/BillingServiceImpl.java

In calculateRemainingQuota, extract magic numbers used for month-boundary
calculation into meaningful constants. Do not change observable behavior.
```

Follow-up prompt:

```text
$spdd-sync @spdd/prompt/GGQPA-001-202603191105-[Feat]-multi-plan-billing-model-aware-pricing.md

Sync the magic-number refactor back into the structured prompt.
```

Checkpoint:

```bash
git add spdd/prompt/GGQPA-001-202603191105-[Feat]-multi-plan-billing-model-aware-pricing.md src/main/java/org/tw/token_billing/service/impl/BillingServiceImpl.java
git commit -m "[001] feat: magic number refactoring"
```

### Step 19: Generate The Enhancement Test Prompt

Prompt:

```text
Based on the implementation details prompt
@spdd/prompt/GGQPA-001-202603191105-[Feat]-multi-plan-billing-model-aware-pricing.md
combined with the template @spdd/template/TEST-SCENARIOS-TEMPLATE.md,
generate a test prompt file:
spdd/prompt/GGQPA-001-202603191105-[Test]-multi-plan-billing-model-aware-pricing.md
```

Follow-up prompt:

```text
@spdd/prompt/GGQPA-001-202603191105-[Test]-multi-plan-billing-model-aware-pricing.md

There are tests that duplicate existing ones. Compare the relevant existing tests
and keep only new scenarios needed for model-aware Standard billing, Premium
split-rate billing, strategy/factory routing, model pricing persistence, and
updated DTO/mapper behavior.
```

Checkpoint:

```bash
git add spdd/prompt/GGQPA-001-202603191105-[Test]-multi-plan-billing-model-aware-pricing.md
git commit -m "[001] feat: generate a test structure prompt"
```

### Step 20: Generate Enhancement Unit Tests

Prompt:

```text
Based on the generated test prompt
@spdd/prompt/GGQPA-001-202603191105-[Test]-multi-plan-billing-model-aware-pricing.md,
generate the corresponding unit and integration test code.
Only add tests for scenarios not already covered.
```

Expected new tests:

- `service/strategy/BillingStrategyFactoryTest`
- `service/strategy/StandardBillingStrategyTest`
- `service/strategy/PremiumBillingStrategyTest`
- `infrastructure/persistence/JpaModelPricingRepositoryAdapterTest`
- `infrastructure/persistence/mapper/ModelPricingMapperTest`
- `infrastructure/persistence/mapper/PricingPlanMapperTest`
- updates to controller, integration, and billing service tests

Verify:

```bash
./gradlew test
```

Checkpoint:

```bash
git add src/test
git commit -m "[001] feat: generate all the tests"
```

### Step 21: Prompt-First Update For Required `model_id`

This is a business/behavior correction. Update the prompt first.

Prompt:

```text
$spdd-prompt-update @spdd/prompt/GGQPA-001-202603191105-[Feat]-multi-plan-billing-model-aware-pricing.md

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
git add spdd/prompt/GGQPA-001-202603191105-[Feat]-multi-plan-billing-model-aware-pricing.md
git commit -m "[001] feat: update the structured prompt to make model_id a required field with a default value of 'fast-model' to ensure backward compatibility with existing bills."
```

### Step 22: Generate Code From The Updated Prompt

Prompt:

```text
$spdd-generate @spdd/prompt/GGQPA-001-202603191105-[Feat]-multi-plan-billing-model-aware-pricing.md

Apply only the code changes required by the updated model_id default decision.
Do not regenerate unrelated files.
```

Expected code changes:

- `src/main/resources/db/migration/V2__Add_model_pricing.sql`
  uses `ALTER TABLE bills ADD COLUMN model_id VARCHAR(50) NOT NULL DEFAULT 'fast-model';`
- `BillPO.modelId` uses `@Column(name = "model_id", length = 50, nullable = false)`

Verify:

```bash
./gradlew test
```

Checkpoint:

```bash
git add src/main/resources/db/migration/V2__Add_model_pricing.sql src/main/java/org/tw/token_billing/infrastructure/persistence/entity/BillPO.java
git commit -m "[001] feat: update the code based on the structured prompt"
```

### Step 23: Update The API Test Command Output

The reference enhanced the generated API test command/script so the script has a
human-reviewable test case overview and a final result table. Update the project
skill bundle if it exists, then regenerate or patch the script.

Prompt:

```text
@.agents/skills/spdd-api-test/SKILL.md
@scripts/test-api.sh

Enhance the API test generation guidance and the current script:
- Add a TEST CASE OVERVIEW section near the top of scripts/test-api.sh.
- Track test IDs, descriptions, expected status, actual status, and pass/fail result.
- Print a final TEST RESULTS SUMMARY table.
- Keep the same curl-only behavior and existing scenarios.
```

Verify:

```bash
sh scripts/test-api.sh
```

Checkpoint:

```bash
git add .agents/skills/spdd-api-test scripts/test-api.sh
git commit -m "[001] feat: update the api test command"
```

## Part 3: Article Story-Generation Artifacts

The last four reference commits were added after the main implementation to
support the article narrative around `/spdd-story`. Keep this order if you want
the workshop history to stay close to `../token-billing`.

### Step 24: Add The Story Command And First Initial Story

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
requirements/[User-story-1-initial]Multi-Plan-Billing-Foundation-&-Standard-Plan-Model-Aware-Pricing.md
```

Checkpoint:

```bash
git add .agents/skills/spdd-story requirements/[User-story-1-initial]Multi-Plan-Billing-Foundation-\&-Standard-Plan-Model-Aware-Pricing.md
git commit -m "[001] feat: add a command for generating stories"
```

### Step 25: Add The Initial Enhancement Idea

Create `requirements/idea-of-the-enhancement.md`:

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
git add requirements/idea-of-the-enhancement.md
git commit -m "[001] feat: add an initial idea of the enhancement"
```

### Step 26: Generate The Second Initial Story

Prompt:

```text
$spdd-story @requirements/idea-of-the-enhancement.md

The Standard plan foundation story already exists at:
@requirements/[User-story-1-initial]Multi-Plan-Billing-Foundation-&-Standard-Plan-Model-Aware-Pricing.md

Generate the Premium Plan Split-Rate Billing story as:
requirements/[User-story-2-initial]Premium-Plan-Split-Rate-Billing.md

Also refine the first initial story if needed so the two stories have singular
responsibilities and do not duplicate scope.
```

Checkpoint:

```bash
git add requirements/[User-story-1-initial]Multi-Plan-Billing-Foundation-\&-Standard-Plan-Model-Aware-Pricing.md requirements/[User-story-2-initial]Premium-Plan-Split-Rate-Billing.md
git commit -m "[001] feat: submit another story"
```

### Step 27: Rename Initial Story Files

Rename the two initial story files so they match the article naming:

```bash
git mv 'requirements/[User-story-1-initial]Multi-Plan-Billing-Foundation-&-Standard-Plan-Model-Aware-Pricing.md' 'requirements/[User-story-1-1-initial]Multi-Plan-Billing-Foundation-&-Standard-Plan-Model-Aware-Pricing.md'
git mv 'requirements/[User-story-2-initial]Premium-Plan-Split-Rate-Billing.md' 'requirements/[User-story-1-2-initial]Premium-Plan-Split-Rate-Billing.md'
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
sh scripts/test-api.sh
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
requirements/
spdd/analysis/
spdd/prompt/
spdd/template/
scripts/test-api.sh
src/main/java/org/tw/token_billing/
src/test/java/org/tw/token_billing/
src/test/resources/application.yml
.agents/skills/
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
