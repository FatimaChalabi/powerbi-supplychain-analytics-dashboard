📐 DAX Measures

All measures live in a dedicated _Measures table in the model. Grouped by category and dashboard page.

## Executive Overview (Page 1)

```dax
Total Procurement Cost = SUM(FactShipments[TotalCost])

Total Budget = SUM(FactBudget[BudgetedCost])

Cost Variance % =
DIVIDE([Total Procurement Cost] - [Total Budget], [Total Budget])

Total Cost YoY % =
VAR CurrentCost = [Total Procurement Cost]
VAR PriorCost = CALCULATE([Total Procurement Cost], SAMEPERIODLASTYEAR(DimDate[Date]))
RETURN DIVIDE(CurrentCost - PriorCost, PriorCost)

Dynamic KPI Title =
"Total cost — " &
SELECTEDVALUE(DimDate[Year], "All years") &
" (" & FORMAT([Total Cost YoY %], "+0.0%;-0.0%") & " YoY)"

Active Suppliers = DISTINCTCOUNT(FactShipments[SupplierID])

Stockout Alerts =
CALCULATE(
    COUNTROWS(FactInventorySnapshot),
    FactInventorySnapshot[StockOnHand] < FactInventorySnapshot[SafetyStock],
    FactInventorySnapshot[SnapshotDate] = MAX(FactInventorySnapshot[SnapshotDate])
)
```

