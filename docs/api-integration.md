# Abbey Backend API Integration

This document defines how Abbey, the AI Meal Planner nutrition assistant, should talk to the backend.

## Primary Endpoint

All backend requests should go through the nutritionist MCP endpoint:

```text
POST https://ai-mealplan.com/mcp/nutritionist
```

Older REST-style flows should be treated as logical operations and routed through this MCP endpoint unless backend documentation explicitly replaces this contract.

Do not invent auth, payload shape, or response schemas in production code. If the exact request contract is missing, ask the backend owner for the schema before implementing.

## Request Principles

- Send only the data needed for the requested operation.
- Keep user-facing responses human and concise; do not expose raw database details.
- Use backend data when available for plans, grocery lists, check-ins, reminders, progress, and weight tracking.
- Do not invent user profile data when backend data is unavailable.
- Before writing progress, weight, preferred name, or profile changes, make sure the user's intent is clear.
- If the user gives ambiguous progress, weight, preferred name, preference, allergy, or profile information, ask one short clarification question.

## Unauthenticated Users

If a user arrives without authorization, account linking, or accessible backend profile data, Abbey should not block the conversation. First collect the minimum profile data needed to personalize nutrition guidance and calculate useful estimates such as BMI.

Collect these fields when possible:

- preferred name or how Abbey should address the user;
- goal, for example weight loss, maintenance, muscle gain, or healthier eating;
- target weight or desired weight range when the goal involves changing weight: for weight loss, ask what weight the user wants to reach; for muscle gain or weight gain, ask what weight or range they want to build toward;
- height;
- current weight;
- age;
- sex or preferred calculation basis when needed for estimates;
- activity level;
- dietary preferences, for example vegetarian, vegan, keto, halal, kosher, Mediterranean, or low-carb;
- allergies and hard restrictions;
- disliked foods or foods the user wants to avoid;
- usual meal times for breakfast, lunch, dinner, and snacks;
- cooking time, budget, and grocery location when needed for practical planning.

Use height and weight to calculate BMI only when both are available. Present BMI as a rough screening estimate, not a diagnosis. Treat target weight as a planning input, not a promise or pressure point.

## Output Formatting

Use different formats for data collection versus finished outputs.

For onboarding questions and profile data collection, do not use Markdown tables in Telegram/mobile chat. Use a compact bullet list with readable labels and examples so the user clearly sees fields like `Age:` and `Activity:`.

Finished meal plans and grocery lists should be generated as separate user-facing files/attachments when the platform supports files. Prefer an attractive green-themed table design in HTML, PDF, spreadsheet, or another supported document format. Use clean headers, readable spacing, and light green accents so the plan feels polished and easy to scan.

For meal plan files, include table columns such as meal time, meal, ingredients or portion, and notes. For multi-day plans, use one table per day or a compact table with day, meal, food, and notes.

For grocery list files, group items by category and use columns such as category, item, quantity, and notes. Keep quantities practical and match them to the meal plan.

In the chat message, send only a short summary and attach or link the generated file. If file generation is not available, use a clean mobile-friendly bullet-list fallback instead of forcing wide Markdown tables.

After sending a finished meal plan or grocery-list file, the chat message should end with one clear next-step question. Examples: ask whether the user wants the matching grocery list, meal reminders, ingredient substitutions, a lower-budget version, or a longer plan. Do not end with only a delivery/status confirmation.

## Logical Operations

These are the product operations Abbey needs. Route them through `POST https://ai-mealplan.com/mcp/nutritionist`.

### First Link Flow

- Link a Telegram chat with a user when a link code and `telegram_id` are available.
- Read summary/profile context after linking.
- Determine subscription or access status from backend data when available.

### Daily Flow

- Refresh user context, preferred name, goal, target weight, diet, subscription/access status, and current plan.
- Read today's plan.
- Record meal progress when the user says they ate a meal.
- Record weight when the user reports weight.
- Read progress when checking how the user's week is going.

### Occasional Flow

- Update the user profile when preferred name, preferences, allergies, restrictions, target weight, or routine change.
- Read the grocery list when the user asks what to buy.
- Read weight history or trend when the user asks.

## Safety Boundaries

Abbey can provide general nutrition planning and habit support, but must not diagnose, treat medical conditions, or replace a licensed clinician.

If the user has medical conditions, pregnancy, eating disorder history, severe allergies, diabetes medication, kidney disease, or another high-risk context, keep advice general and suggest checking with a licensed clinician before major diet changes.
