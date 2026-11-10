# Codex prompt for Dana's Bakery

Build a self-contained Google Colab notebook named dana-bakery-class.ipynb. Use LangGraph and deterministic Python. Include an introduction, a %pip installation cell, the graph definition, an inline draw_mermaid_png() diagram, and executable sample orders with assertions.

## Input and output

Accept item, quantity, day, and city via graph.invoke. Normalize text with strip and casefold. Require an active product, a positive Python integer (excluding booleans), and a full English weekday name. Treat missing, null, or blank city as pickup. Reject a city supplied as a non-text value other than null.

Return status (accepted or rejected), item (the product sold), delivery (pickup or delivery), total (numeric shekels), message (customer-facing text), and steps (ordered list of trace strings). On rejection, return null item, pickup, and zero total. Keep routing data internal to the graph.

## Business rules

| Product | Price in shekels | Opening stock |
| --- | ---: | ---: |
| challah | 18 | 10 |
| babka | 45 | 0 |
| rugelach (box) | 30 | 3 |
| sourdough (loaf) | 25 | 4 |

- Use a fixed stock snapshot for each order. Reject unknown and discontinued items, including croissants.
- Follow the substitution chain babka → rugelach → sourdough → challah, starting at the requested item. Check at most two alternatives. Each check requires stock for the entire quantity. Sell the first candidate with enough stock, retaining the quantity and using the selected product's price. Cancel if there is no eligible candidate.
- Charge ₪20 for Jerusalem delivery. Pickup is free. For other cities, keep the order and explain the switch to pickup.
- Use Avi's final rule. Give 15% off baked goods above ₪150 except on Friday. Add the full delivery fee after applying any discount. Compute in integer agorot.
- Write warm reply templates. Explain substitutions, pickup changes, and rejections where applicable. Use no LLMs or credentials. Omit the Instagram request and personal anecdotes.

## Graph and trace

```text
START → validate
  invalid → reject → END
  valid   → check_stock
    enough             → fulfill → price → accept → END
    next offer exists  → substitute → check_stock
    no offer remaining → reject → END
```

Start a fresh trace for every invocation. Append the node name at every executed step. Include the checked or newly proposed item after the node name in stock-check and substitution entries. Keep the requested item and current candidate available until the reply is generated.

## Acceptance cases

| Order | Expected result |
| --- | --- |
| 2 challahs, Thursday, blank city | Accepted, challah, pickup, ₪36 |
| 2 babkas, Thursday, Jerusalem | Accepted after one offer, rugelach, delivery, ₪80 |
| 4 babkas, Thursday, Tel Aviv | Accepted after two offers, sourdough, pickup, ₪100 |
| 5 babkas, Thursday, Jerusalem | Rejected after two offers despite enough challah |
| 11 challahs, Thursday, blank city | Rejected with no alternatives |
| 10 challahs, Thursday, Jerusalem | Accepted, delivery, ₪173 |
| 10 challahs, Friday, Jerusalem | Accepted, delivery, ₪200 |
| 6 sourdough loaves, Thursday, Jerusalem, fixture with 6 in stock | Accepted, delivery, ₪170 with no threshold discount |
| Pizza, croissants, zero, negative, fractional, or boolean quantity | Rejected during validation |

Check the result keys, exact trace order, and relevant message text. Build a separate graph with the test stock snapshot for the threshold case. Run all cells from a fresh Colab runtime and check that the inline image shows the conditional branches and substitution loop.
