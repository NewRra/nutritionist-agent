# Abbey Backend API Integration

This document defines how Abbey, the AI Meal Planner nutrition assistant, should talk to the backend.

## Primary Endpoint

All backend requests should go through the nutritionist MCP endpoint:

```text
POST https://ai-mealplan.com/mcp/nutritionist
```

Older REST-style flows should be treated as logical operations and routed through this MCP endpoint unless backend documentation explicitly replaces this contract.

The endpoint is a JSON-RPC 2.0 MCP server. Plain JSON payloads without `jsonrpc: "2.0"` are rejected.

## MCP Tool Names

Initialize the MCP server first, then call tools with the JSON-RPC `tools/call` method.

Known tools as of 2026-06-10:

- `register-user-tool`: create or find a user by Telegram chat ID and save supported onboarding fields.
- `get-user-profile-tool`: read user profile and onboarding data.
- `update-user-profile-tool`: merge supported profile updates into an existing profile.
- `create-meal-plan-tool`: create a personalized weekly meal plan from stored preferences.
- `get-current-meal-plan-tool`: read the current meal plan, optionally by date.
- `generate-grocery-list-tool`: generate a grocery list from the current meal plan.

Supported profile/onboarding fields include `chat_id`, `name`, `goal`, `diet`, `allergies`, `gender`, `age`, `weight`, `height`, `activity`, `calorie_target`, `meals_per_day`, and `cuisine`, depending on the tool. Current schemas do not expose target weight, disliked foods, or exact meal times as writable fields.

## Persistence Rules

- Write every user detail that the backend schema supports, including profile, onboarding, preferences, allergies, activity, height, weight, age, goal, diet, meals per day, cuisine, and calorie target when available.
- When the user gives new or changed profile data, call `update-user-profile-tool` after intent is clear; do not only keep the change in chat memory.
- When creating a meal plan, use `create-meal-plan-tool` so the backend creates/stores the plan, then use `get-current-meal-plan-tool` when the user wants to view or reuse it.
- When creating a grocery list, use `generate-grocery-list-tool` so the backend creates/stores the list from the current meal plan.
- If the user gives important data that the current MCP schema cannot write, such as target weight, disliked foods, preferred meal times, or high-protein style, document the schema gap and do not claim the value was saved until the backend supports it.
- Prefer backend persistence over local-only generated files. Files are user-facing presentation; the source of truth should be the MCP backend whenever tools support the operation.

## JSON-RPC Payload Examples

These examples use the current MCP `tools/call` shape.

### Register User

```json
{
  "jsonrpc": "2.0",
  "id": 1,
  "method": "tools/call",
  "params": {
    "name": "register-user-tool",
    "arguments": {
      "chat_id": "1116090730",
      "name": "Alina",
      "goal": "lose_weight",
      "diet": "standard",
      "allergies": [],
      "age": 20,
      "weight": 64,
      "height": 175,
      "activity": "moderately_active"
    }
  }
}
```

### Update Profile

```json
{
  "jsonrpc": "2.0",
  "id": 2,
  "method": "tools/call",
  "params": {
    "name": "update-user-profile-tool",
    "arguments": {
      "chat_id": "1116090730",
      "meals_per_day": 3
    }
  }
}
```

### Get User Profile

```json
{
  "jsonrpc": "2.0",
  "id": 3,
  "method": "tools/call",
  "params": {
    "name": "get-user-profile-tool",
    "arguments": {
      "chat_id": "1116090730"
    }
  }
}
```

### Create Meal Plan

```json
{
  "jsonrpc": "2.0",
  "id": 4,
  "method": "tools/call",
  "params": {
    "name": "create-meal-plan-tool",
    "arguments": {
      "chat_id": "1116090730",
      "goal": "lose_weight",
      "diet": "standard",
      "allergies": []
    }
  }
}
```

### Generate Grocery List

```json
{
  "jsonrpc": "2.0",
  "id": 5,
  "method": "tools/call",
  "params": {
    "name": "generate-grocery-list-tool",
    "arguments": {
      "chat_id": "1116090730"
    }
  }
}
```

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
