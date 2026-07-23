# 🗂️ Data Model — Star Schema

## Fact Tables

### FactShipments
| Column | Type | Description |
|---|---|---|
| ShipmentID | PK | Unique shipment identifier |
| SupplierID | FK | Links to DimSupplier |
| ProductID | FK | Links to DimProduct |
| WarehouseID | FK | Links to DimWarehouse |
| OrderDate | Date | Date order was placed |
| ExpectedDate | Date | Promised delivery date |
| ActualDeliveryDate | Date | Actual delivery date |
| OrderQty | Number | Units ordered |
| ReceivedQty | Number | Units received |
| DefectQty | Number | Units rejected/defective |
| UnitCost | Currency | Cost per unit |
| TotalCost | Currency | OrderQty × UnitCost |

### FactInventorySnapshot
| Column | Type | Description |
|---|---|---|
| SnapshotDate | Date | Daily snapshot date |
| ProductID | FK | Links to DimProduct |
| WarehouseID | FK | Links to DimWarehouse |
| StockOnHand | Number | Units in stock |
| ReorderPoint | Number | Threshold for reorder alert |

## Dimension Tables

**DimSupplier** — SupplierID, Name, Country, Category, ContractType, Rating

**DimProduct** — ProductID, Category, SubCategory, UnitOfMeasure, SafetyStock

**DimWarehouse** — WarehouseID, Location, Region, Capacity

**DimDate** — standard calendar table (Year, Quarter, Month, Week, IsWeekend)
