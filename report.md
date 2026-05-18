# Multi-Agent Enterprise Workflow System Evaluation Report

## 1. System Design and Overview
This project implements a hierarchical, multi-agent workflow architecture built on the smolagents framework to automate custom order processing, pricing optimization, and logistics scheduling for Munder Difflin. The system utilizes a central manager agent that sequentially coordinates tasks across three specialized downstream worker agents:

Business Orchestrator (Manager Agent): Acts as the primary root execution entry point for incoming customer purchase requests. It reads raw user prompts, establishes the serial order of operations, delegates tasks to specific workers, and consolidates individual outputs into a clear, unified response for the client.

Inventory Agent (Worker): Responsible for real-time warehouse tracking. It receives parsed product needs from the orchestrator and interfaces with the check_stock database tool to determine exact stock volumes.

Quote Agent (Worker): Responsible for financial analysis and quote generation. It uses the query_pricing_history tool to match requested volumes against historical flat-rate agreements in quotes.csv. To ensure resilience against incomplete records, it incorporates fallback pricing rules based on standard product categories.

Ordering Agent (Worker): Responsible for financial ledger management and shipment logistics. It guarantees that incoming customer revenue is recorded via record_transaction explicitly under transaction_type='sales' and executes calculate_delivery_timeline to generate accurate shipping arrival schedules.

## 2. Evaluation Results & Analysis
### Summary of System Performance
The multi-agent system was stress-tested across 20 distinct corporate order profiles containing varying product text ambiguities, custom item demands, and out-of-stock scenarios.

### System Strengths
High Factual Grounding: The InventoryAgent checks real database thresholds, eliminating LLM hallucinations regarding fake warehouse availability.

Graceful Degradation: The system doesn't crash when encountering out-of-stock items (such as balloons, cups, or specialized tape). It filters out unavailable novelties, fulfills valid core paper requests, and clearly communicates limitations to the customer.

Strict Financial Constraints: By explicitly forcing transaction routing boundaries, the model preserves financial state consistency, preventing unauthorized spending or negative balance adjustments during customer-facing interactions.

### Areas for Improvement (Hurdles Identified During Testing)
Early Operational Losses (Procurement Trap): In early design iterations, the agents attempted to act as internal procurement managers during stockouts, triggering stock_orders to wholesale vendors instead of logging incoming customer sales. This caused cash balances to drop until strict boundary instructions were introduced.

Strict String Matching Bottlenecks: Loose customer terms like "A4 paper" failed to automatically register against strict database definitions like "A4 printer paper", resulting in initial false-positive stockout rejections.

Conversational Data Parsing: The QuoteAgent initially struggled to isolate individual numeric floats from the complex narrative logs returned by the historical search tool, occasionally causing blank pricing errors before fallback constraints were established.

## 3. Suggestions for Further System Improvements
Based on the behavioral bottlenecks discovered during development and testing, the following technical updates are recommended to enhance system performance:

- Fuzzy String & Semantic Matching for Inventory: Upgrade the check_stock database tool with a text similarity algorithm (such as Levenshtein distance) or vector embeddings. This allows user variations like "standard white paper" or "copy sheets" to map automatically to the correct database column without requiring manual LLM text cleaning.

- Asynchronous Replenishment Logic: Move supplier restocking into a separate batch routine. Instead of the fulfillment agents triggering replenishment costs during an active customer checkout, low stock levels should trigger a background ticket for a separate procurement workflow executed outside the consumer stream.

- Structured Tool Output Data: Redesign the historical search utility (query_pricing_history) to return structured JSON pairs (e.g., {"unit_cost": 0.05, "currency": "USD"}) instead of full paragraphs of old conversational emails. This entirely eliminates string parsing errors and makes financial aggregation completely reliable.