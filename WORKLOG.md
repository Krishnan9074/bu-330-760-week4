# Week 4 ReAct Agent Worklog

## Goal

Run the starter ReAct agent, observe the failure caused by the missing catalog tool, implement the missing `product_lookup` tool, and verify that all 8 questions are solved correctly.

## Environment Notes

- Date: 2026-04-19
- Workspace: `/Volumes/SSK_PSSD/bu-330-760-week4`
- Model currently configured in `agent.py`: `openai:gpt-4o-mini`

## Step 1. Protect Secrets

Checked `.gitignore` to confirm that `.env` will not be committed.

### Observation

- `.gitignore` already contains `.env`

### Result

- Step 1 is satisfied without additional changes.

## Step 2. Run Agent Before Implementing Product Lookup

Command run:

```bash
uv run agent.py
```

### Observation

- Questions 1-4 completed successfully.
- The agent used `calculator_tool` correctly for direct arithmetic.
- Question 3 was solved correctly without a tool call by reasoning alone.
- Questions 5-8 did not have access to catalog prices and the agent did not recover safely.
- For Question 5, the agent attempted expressions like `3 * price_of_Alpha_Widget` and `2 * price_of_Beta_Widget`, which failed with undefined-name errors.
- For Question 6, the agent attempted `price_of_Delta_Widget - price_of_Alpha_Widget`, which failed for the same reason.
- For Question 7, the agent first failed on `price_of_Gamma_Widget`, then guessed a price of `25` and returned `8` widgets with `$0` left over. This answer is incorrect.
- For Question 8, the agent failed on both missing prices, then guessed `$10` for Gamma and `$35` for Delta and concluded Delta was the better deal. This answer is incorrect.

### Result

- Confirmed the missing `product_lookup` tool is the root cause for Questions 5-8.
- The failure is not only an inability to answer; it also allows the model to fabricate fallback prices, producing wrong results.

## Step 3. Implement `product_lookup`

Implementation summary:

- Added a `@agent.tool_plain` function named `product_lookup` in `agent.py`.
- The tool reads `products.json` with `json.load`.
- It returns the matched price as a string when the product exists.
- If the product is not found, it returns the available product names so the agent can retry with a valid catalog entry.

### Observation

- The implementation follows the same registration pattern as `calculator_tool`, which keeps tool wiring consistent and simple.

### Result

- Product lookup is now available to the ReAct agent.

## Step 4. Run Agent Again After Tool Implementation

Command run:

```bash
uv run agent.py
```

### Observation

- All 8 questions completed successfully.
- Questions 5-8 now use `product_lookup` to fetch catalog prices before doing arithmetic.
- Question 5 returned the correct total for `3 Alpha Widgets + 2 Beta Widgets`: `$180.97`.
- Question 6 returned the correct price difference between Delta and Alpha: approximately `$59.01`.
- Question 7 returned the correct budget calculation for Gamma Widgets: `15` widgets with `$8.75` left over.
- Question 8 correctly identified that `4 Gamma Widgets` cost `$51.00`, which is cheaper than `1 Delta Widget` at `$89.00`.
- The post-fix traces show a clean separation of responsibilities:
  - `product_lookup` retrieves catalog values
  - `calculator_tool` handles arithmetic when needed

### Result

- The missing tool has been implemented successfully.
- The agent now solves the full set of 8 questions correctly.
- The main behavior improvement is that the model no longer fabricates placeholder prices when product data is required.
