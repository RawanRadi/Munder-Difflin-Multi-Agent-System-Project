# Munder Difflin Multi-Agent System Project

## 1. System Design and Overview
This project sets up a hierarchical multi-agent workflow using the `smolagents` framework to automate custom printing orders, pricing history checks, and logistics tracking for Munder Difflin. The system runs via a main manager agent that passes tasks step-by-step to three specialized worker agents:

* **Business Orchestrator (Manager Agent):** This is the central router and coordinator. It takes the customer's raw text request, schedules the order of steps, calls the right worker agents in sequence, and packages their final text pieces into a clear confirmation message.
* **Inventory Agent (Worker):** Handles warehouse stock management. It extracts what items the user wants and talks to the `check_stock` database tool to verify exactly how many units are sitting on the shelves.
* **Quote Agent (Worker):** Handles financial calculations. It uses the `query_pricing_history` tool to match the order requirements against past transaction logs in `quotes.csv`. If it finds a mismatch or missing data, it uses baseline category fallback rules to build an estimate.
* **Ordering Agent (Worker):** Manages the final checkout log and delivery schedules. It records customer sales revenue in the system by executing `record_transaction` strictly under `transaction_type='sales'`, and runs `calculate_delivery_timeline` to check shipping dates.

---

## 2. Evaluation Results & Analysis

### Summary of System Performance
The complete multi-agent pipeline was evaluated using 20 unique customer order requests. These test cases contained loose item descriptions, varying volume requests, custom event items, and stock shortages.

### System Strengths
* **Factual Grounding:** The `InventoryAgent` works directly with real database tools. It successfully stops the LLM from making up fake stock counts when an item is unavailable.
* **Resilient Order Filtering:** The system does not crash when specialized finished goods (like balloons or streams) are missing. It notes what it cannot fulfill but continues to process the core paper items instead of breaking down.
* **Proper State Tracking:** The final run shows that the ledger works correctly. Cash balances and inventory values shift up and down dynamically across the test rows based on real execution conditions.

### Areas for Improvement (Hurdles Overcome During Development)
* **The Procurement Loop Trap:** During early tests, when an item was out of stock, the worker agent would mistakenly call `stock_orders` to buy items from external vendors. This drained company cash during a customer sale until tight boundary rules were written into the agent descriptions.
* **Loose Text Alignment:** Customers often requested general items like "A4 paper," while the database held strict terms like "A4 printer paper." The system required explicit prompting constraints to ensure shorthand terms mapped properly to database keys.
* **Parsing Blocks:** Early versions struggled when the historical quote tool returned full conversational text blocks instead of simple numbers. Adding specific fallback instructions helped the Quote Agent handle these text variations cleanly.

---

## 3. Suggestions for Further System Improvements

If this enterprise system were to be developed further, three key upgrades would make it much more powerful:

1. **Fuzzy Database Matching:** Instead of relying entirely on the LLM to fix shorthand words in prompts, the database tool (`check_stock`) should use a fuzzy matching string algorithm. This would let terms like "white copy paper" automatically map to "standard printing paper" without any prompt overhead.
2. **Automated Reorder Thresholds:** Right now, stock shortages are simply reported back to the customer. A great upgrade would be creating a separate background procurement agent that tracks warehouse drops and fires off stock replenishment orders only when inventory dips below a fixed safety minimum (e.g., under 100 units).
3. **Structured Tool Returns:** Redesigning `query_pricing_history` to pass back a clean JSON object (like `{"item": "cardstock", "unit_price": 0.15}`) rather than conversational conversation histories would make financial parsing completely bulletproof.
