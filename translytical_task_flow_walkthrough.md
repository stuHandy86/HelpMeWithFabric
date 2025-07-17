# 🧭 Walkthrough: Editable Fields with Audit Trail via Translytical Task Flows in Microsoft Fabric

## 🎯 Goal

Allow Power BI users to **edit 3 fields** in a Lakehouse Delta table:
- `Comments` (free text)
- `Status` (selected via slicer)
- `Amount` (editable number)

While **automatically capturing**:
- `RecordUpdated` timestamp (when update occurred)
- `ModifiedBy` (who made the change)

## 🧰 Prerequisites

- Fabric workspace with Lakehouse + Power BI
- Delta table (`Sales`) with editable and audit columns
- Power BI access with writeback enabled
- Optional: UDF knowledge for validation

---

## ⚙️ Step 1: Prepare Your Delta Table

Modify (or create) the Delta table to include audit fields:

```sql
CREATE TABLE Sales (
    OrderID STRING,
    Product STRING,
    Quantity INT,
    Price DOUBLE,
    OrderDate DATE,
    Comments STRING,
    Status STRING,
    Amount DOUBLE,
    RecordUpdated DATETIME,
    ModifiedBy STRING
) USING DELTA;
```

Or alter your existing table:

```sql
ALTER TABLE Sales ADD COLUMNS (
    Comments STRING,
    Status STRING,
    Amount DOUBLE,
    RecordUpdated DATETIME,
    ModifiedBy STRING
);
```

---

## 🧩 Step 2: Create a Translytical Task Flow

1. In your Fabric workspace, click **New > Translytical Task Flow**.
2. Select the Lakehouse where the `Sales` table lives.
3. On the canvas, add a new **SQL Task**, name it `UpdateSalesRecord`.

---

## 🧠 Step 3: Write SQL Logic with Audit Fields

```sql
UPDATE Sales
SET
    Comments = @Comments,
    Status = @Status,
    Amount = @Amount,
    RecordUpdated = CURRENT_TIMESTAMP(),
    ModifiedBy = @ModifiedBy
WHERE OrderID = @OrderID;
```

---

## 🛂 Step 4: Define Parameters for the Task

| Name        | Type     | Source in Power BI     |
|-------------|----------|------------------------|
| OrderID     | String   | Selected row (key)     |
| Comments    | String   | Text input             |
| Status      | String   | Slicer                 |
| Amount      | Double   | Numeric input          |
| ModifiedBy  | String   | Use `USERNAME()` DAX   |

---

## 🔍 Optional: Add UDF to Validate Amount

```sql
CREATE FUNCTION ValidateAmount(a DOUBLE)
RETURNS BOOLEAN
AS
BEGIN
    RETURN a >= 0 AND a <= 100000;
END;
```

Wrap your `UPDATE` logic:

```sql
IF ValidateAmount(@Amount)
BEGIN
    UPDATE Sales
    SET
        Comments = @Comments,
        Status = @Status,
        Amount = @Amount,
        RecordUpdated = CURRENT_TIMESTAMP(),
        ModifiedBy = @ModifiedBy
    WHERE OrderID = @OrderID;
END;
```

---

## 📊 Step 5: Set Up Power BI Report

1. Load the `Sales` table from the Lakehouse
2. Add:
   - **Table visual**: show records
   - **Text box**: for `Comments`
   - **Numeric input**: for `Amount`
   - **Slicer**: for `Status`
   - **Button**: triggers writeback
3. Bind inputs to parameters in the Translytical Task Flow

---

## 🕵️ View Change History: Building an Audit Trail

### 🧱 Create an Audit Table

```sql
CREATE TABLE SalesAudit (
    AuditID STRING,
    OrderID STRING,
    OldComments STRING,
    NewComments STRING,
    OldStatus STRING,
    NewStatus STRING,
    OldAmount DOUBLE,
    NewAmount DOUBLE,
    ModifiedBy STRING,
    RecordUpdated DATETIME
) USING DELTA;
```

---

### 🔧 Modify Task Flow to Log Changes

```sql
-- Capture old values
DECLARE @OldComments STRING;
DECLARE @OldStatus STRING;
DECLARE @OldAmount DOUBLE;

SELECT
    Comments,
    Status,
    Amount
INTO
    @OldComments,
    @OldStatus,
    @OldAmount
FROM Sales
WHERE OrderID = @OrderID;

-- Insert audit record
INSERT INTO SalesAudit
SELECT
    uuid() AS AuditID,
    @OrderID,
    @OldComments,
    @Comments,
    @OldStatus,
    @Status,
    @OldAmount,
    @Amount,
    @ModifiedBy,
    CURRENT_TIMESTAMP();

-- Perform update
UPDATE Sales
SET
    Comments = @Comments,
    Status = @Status,
    Amount = @Amount,
    RecordUpdated = CURRENT_TIMESTAMP(),
    ModifiedBy = @ModifiedBy
WHERE OrderID = @OrderID;
```

---

### 📊 Power BI Audit View

1. Load `SalesAudit` into Power BI
2. Use a matrix or table to show:
   - `OrderID`, `Field`, `Old`, `New`, `ModifiedBy`, `RecordUpdated`

---

## 📦 Optional Enhancements

### 📝 Add Change Reason

```sql
ALTER TABLE SalesAudit ADD COLUMN ChangeReason STRING;
```

Pass via Power BI input and update `INSERT` accordingly.

### 🧹 Archive Old History

```sql
DELETE FROM SalesAudit
WHERE RecordUpdated < current_date() - INTERVAL 730 DAYS;
```

---

## ✅ Summary Architecture

| Component              | Role                                  |
|------------------------|---------------------------------------|
| `Sales` Table          | Editable operational data             |
| `SalesAudit` Table     | Immutable log of changes              |
| Translytical Task Flow | Writeback + audit logic               |
| Power BI               | UI for editing and audit visibility   |

---

## 📚 References

- [Translytical Task Flow Overview](https://learn.microsoft.com/en-us/power-bi/create-reports/translytical-task-flow-overview)
- [Power BI Writeback Docs](https://learn.microsoft.com/en-us/power-bi/create-reports/translytical-task-flow-button)
- [Delta Tables in Fabric](https://learn.microsoft.com/en-us/fabric/data-engineering/delta-lakehouse)
