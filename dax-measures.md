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

## Supplier Performance (Page 2)

```dax
On-Time Delivery Rate =
DIVIDE(
    CALCULATE(COUNTROWS(FactShipments), FactShipments[ActualDeliveryDate] <= FactShipments[ExpectedDate]),
    COUNTROWS(FactShipments)
)

Average Lead Time (Days) =
AVERAGEX(FactShipments, DATEDIFF(FactShipments[OrderDate], FactShipments[ActualDeliveryDate], DAY))

Supplier Defect Rate =
DIVIDE(SUM(FactShipments[DefectQty]), SUM(FactShipments[ReceivedQty]))

SLA Compliance Rate =
DIVIDE(
    CALCULATE(
        COUNTROWS(FactShipments),
        DATEDIFF(FactShipments[OrderDate], FactShipments[ActualDeliveryDate], DAY)
            <= RELATED(DimSupplier[LeadTimeSLA_Days])
    ),
    COUNTROWS(FactShipments)
)

Supplier Scorecard Index =
VAR OnTimeScore = [On-Time Delivery Rate] * 40
VAR QualityScore = (1 - [Supplier Defect Rate]) * 40
VAR RatingScore = DIVIDE(AVERAGE(DimSupplier[SupplierRating]), 5) * 20
RETURN OnTimeScore + QualityScore + RatingScore

Supplier Rank =
RANKX(ALL(DimSupplier[SupplierName]), [Supplier Scorecard Index], , DESC)
```

## Inventory & Stockout Analysis (Page 3)

```dax
Average Stock On Hand =
CALCULATE(
    AVERAGE(FactInventorySnapshot[StockOnHand]),
    FactInventorySnapshot[SnapshotDate] = MAX(FactInventorySnapshot[SnapshotDate])
)

Inventory Turnover Ratio =
DIVIDE([Total Procurement Cost], [Average Stock On Hand])

Days of Inventory On Hand =
DIVIDE([Average Stock On Hand], DIVIDE([Total Procurement Cost], 365))

Stockout Flag =
IF(
    SELECTEDVALUE(FactInventorySnapshot[StockOnHand]) < SELECTEDVALUE(FactInventorySnapshot[SafetyStock]),
    "At Risk", "OK"
)

ABC Cumulative % =
VAR ProductCost = [Total Procurement Cost]
VAR RunningTotal =
    CALCULATE(
        [Total Procurement Cost],
        FILTER(ALL(DimProduct), RANKX(ALL(DimProduct), CALCULATE([Total Procurement Cost])) <= RANKX(ALL(DimProduct[ProductID]), CALCULATE([Total Procurement Cost])))
    )
RETURN DIVIDE(RunningTotal, CALCULATE([Total Procurement Cost], ALL(DimProduct)))

ABC Class =
SWITCH(
    TRUE(),
    [ABC Cumulative %] <= 0.8, "A",
    [ABC Cumulative %] <= 0.95, "B",
    "C"
)
```





