# Shipment Document Planner

## Purpose

Prepare a simple document checklist for international shipments and warehouse handoffs.

## When to Use

Use this skill before export, import, courier shipment, sea shipment, air shipment, bonded shipment, or warehouse delivery.

## Inputs

Useful inputs: origin, destination, product, quantity, carrier, route, invoice data, packing list data, certificates, labels, and warehouse rules.

## Workflow

1. Define route and shipment type.
2. List required documents.
3. Check product, quantity, price, carton, and label consistency.
4. Identify missing documents or mismatches.
5. Create final document checklist.

## Output

Always provide document checklist, mismatch notes, missing items, owner notes, and next actions.

## Common Mistakes

- Different quantities across documents.
- Missing carton data.
- Missing labels.
- No final check before handoff.

## Related Skills

- route-planner
- warehouse-inbound-planner
- landed-cost-planner
