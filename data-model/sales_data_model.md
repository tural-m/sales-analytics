# Data Model Notes — Sales Analytics

## Source
Live data retrieved from **Salesforce Developer Org** via Power BI Salesforce Connector.

## Salesforce Custom Objects

### Product__c
| Field | Type | Notes |
|---|---|---|
| Product_Id__c | Text | Unique identifier (P200, P201...) |
| Product_Name__c | Text | e.g. LED Strip Light, Smart Lamp |
| Category__c | Picklist | Home, Kitchen, Accessories |
| Subcategory__c | Picklist | Lighting, Appliances, Decor |
| Unit_Price__c | Currency | Product base price |
| Active__c | Checkbox | TRUE/FALSE |

### Customer__c
| Field | Type | Notes |
|---|---|---|
| Customer_Name__c | Text | Full name |
| Segment__c | Picklist | SMB, Consumer, Enterprise |
| Province__c | Picklist | AB, ON, MB, BC |
| City__c | Text | Calgary, Edmonton, Toronto etc. |
| Channel__c | Picklist | Retail, Online |

### Sales__c
| Field | Type | Notes |
|---|---|---|
| Order_Id__c | Text | Unique order identifier |
| Order_Date__c | Date | Transaction date |
| Revenue__c | Currency | Total order revenue |
| Profit__c | Currency | Revenue minus cost |
| Customer__c | Lookup | → Customer__c |
| Product__c | Lookup | → Product__c |
| Store__c | Lookup | → Store__c |
| Promotion__c | Lookup | → Promotion__c |

### Web_Traffic__c
| Field | Type | Notes |
|---|---|---|
| Session_Date__c | Date | Date of web traffic |
| Sessions__c | Number | Total site sessions |
| Add_To_Carts__c | Number | Products added to cart |
| Checkouts__c | Number | Checkout initiated |
| Orders__c | Number | Completed purchases |
| Province__c | Picklist | Traffic source province |

---

## Power Query Transformations

### Applied Steps
- Renamed Salesforce API field names to readable labels
- Merged Sales with Customer, Product, Store, Promotion tables
- Created calculated columns:
  - `Profit Margin %` = Profit / Revenue
  - `Conversion Rate` = Orders / Sessions
  - `Cart Abandonment Rate` = 1 - (Orders / Add_To_Carts)
- Extracted Month Name and Month Number from Order_Date
- Filtered out null and inactive records

---

## Power BI Data Model (Star Schema)

```
                  ┌─────────────┐
                  │  DIM_DATE   │
                  └──────┬──────┘
                         │
┌─────────────┐   ┌──────┴──────┐   ┌───────────────┐
│DIM_CUSTOMER ├───┤  FACT_SALES ├───┤  DIM_PRODUCT  │
└─────────────┘   └──────┬──────┘   └───────────────┘
                         │
              ┌──────────┼──────────┐
              │                     │
       ┌──────┴──────┐    ┌─────────┴──────┐
       │  DIM_STORE  │    │ DIM_PROMOTION  │
       └─────────────┘    └────────────────┘

FACT_WEB_TRAFFIC (linked via Date)
```

---

## Key DAX Measures

| Measure | Logic |
|---|---|
| Total Revenue | `SUM(FACT_SALES[Revenue])` |
| Profit Margin % | `DIVIDE(SUM(Profit), SUM(Revenue))` |
| Conversion Rate | `DIVIDE(SUM(Orders), SUM(Sessions))` |
| Cart Abandonment | `1 - DIVIDE(SUM(Orders), SUM(Add_To_Carts))` |
| Revenue MoM % | `DIVIDE(Revenue - CALCULATE(Revenue, PREVIOUSMONTH), CALCULATE(Revenue, PREVIOUSMONTH))` |
