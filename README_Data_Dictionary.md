# Emperor Cuisine Restaurant Group — Power BI Star-Schema Dataset

Synthetic but realistic data for the full year **2025** (Jan 1 – Dec 31), built to match the
dashboard brief: 5 Klang Valley branches, a **RM20,000,000 annual sales target**, membership /
wallet / voucher tracking, 10-platform marketing, bookings, events, reviews and AI auto-reply.

The data is generated so the dashboard tells a *real* story: group net sales land at
**≈ RM17.87M (about 89% of the RM20M target)**, KL over-performs, Cheras and Puchong lag, and
roughly a third of members are inactive — giving you genuine gaps, heatmaps and reactivation
opportunities to visualise.

---

## 1. Why this is a star schema

The model is a **star / constellation schema**: a small set of shared *dimension* tables (the
"who / what / when / where") sit in the middle, and every *fact* table (the measurable events)
points to them through integer surrogate keys.

- **11 dimensions** — `Dim_Date`, `Dim_Branch`, `Dim_Customer`, `Dim_Staff`, `Dim_MenuItem`,
  `Dim_Promotion`, `Dim_Platform`, `Dim_PaymentMethod`, `Dim_Voucher`, `Dim_Campaign`, `Dim_Table`.
- **12 fact tables** — `Fact_Sales`, `Fact_OrderItems`, `Fact_Booking`, `Fact_Wallet`,
  `Fact_Voucher`, `Fact_PlatformContent`, `Fact_PlatformEnquiry`, `Fact_Review`,
  `Fact_AIReplyLog`, `Fact_Target`, `Fact_PlatformFollowers`, `Fact_ActionPlan`.

All keys are clean integers. `0` is used as a "none / not applicable" member, platform, promotion
or voucher row so you never get blank relationships. `BillID` and `BookingID` are kept as
**degenerate dimensions** (text codes living inside the fact) so you can drill to a single receipt.

Data integrity was verified: 100% referential integrity (no orphan keys), order-item lines roll up
to each bill's `NetSales` to within rounding, and wallet balances never go negative.

---

## 2. Files in this package

| File | Grain (1 row = …) | Rows |
|---|---|---|
| Dim_Date.csv | one calendar day | 365 |
| Dim_Branch.csv | one branch | 5 |
| Dim_Customer.csv | one member (key 0 = walk-in guest) | 8,001 |
| Dim_Staff.csv | one staff member (key 0 = self-service/app) | 56 |
| Dim_MenuItem.csv | one menu dish | 45 |
| Dim_Promotion.csv | one promotion/package (key 0 = none) | 10 |
| Dim_Platform.csv | one channel (key 0 = direct) | 12 |
| Dim_PaymentMethod.csv | one payment method | 7 |
| Dim_Voucher.csv | one voucher campaign (key 0 = none) | 9 |
| Dim_Campaign.csv | one marketing campaign / UTM (key 0 = organic) | 13 |
| Dim_Table.csv | one physical table or room (key 0 = takeaway/none) | 269 |
| Fact_Sales.csv | one bill / receipt | 45,926 |
| Fact_OrderItems.csv | one dish line on a bill | 376,530 |
| Fact_Booking.csv | one booking | 12,698 |
| Fact_Wallet.csv | one wallet top-up or payment | 22,507 |
| Fact_Voucher.csv | one issued voucher | 9,585 |
| Fact_PlatformContent.csv | one social post | 1,456 |
| Fact_PlatformEnquiry.csv | one enquiry/lead | 19,000 |
| Fact_Review.csv | one customer review | 3,703 |
| Fact_AIReplyLog.csv | one AI-handled conversation | 16,500 |
| Fact_Target.csv | one branch × month target | 60 |
| Fact_PlatformFollowers.csv | one platform × month follower snapshot | 108 |
| Fact_ActionPlan.csv | one weekly action / insight item | 58 |

---

## 3. Relationships to create in Power BI

In **Model view**, create these one-to-many relationships (dimension *one* side → fact *many*
side, single-direction filter, dimension side filters the fact). The shared `Dim_Date` is the key
to all time intelligence — mark it as the date table (`Dim_Date[Date]`).

**Dim_Date[DateKey]** → `Fact_Sales[DateKey]`, `Fact_OrderItems[DateKey]`,
`Fact_Wallet[DateKey]`, `Fact_Voucher[IssueDateKey]`, `Fact_PlatformContent[PostDateKey]`,
`Fact_PlatformEnquiry[DateKey]`, `Fact_Review[DateKey]`, `Fact_AIReplyLog[DateKey]`,
`Fact_Booking[VisitDateKey]`, `Fact_Target[MonthStartDateKey]`.
*(`Fact_Booking[BookingDateKey]` and `Fact_Voucher[RedeemDateKey]` can be added as inactive
relationships and activated with `USERELATIONSHIP` when you need them.)*

**Dim_Branch[BranchKey]** → the `BranchKey` column in every fact except `Fact_PlatformContent`
(use `BranchKeyMentioned` there).

**Dim_Customer[CustomerKey]** → `Fact_Sales`, `Fact_Booking`, `Fact_Wallet`, `Fact_Voucher`,
`Fact_PlatformEnquiry`.

**Dim_Staff[StaffKey]** → `Fact_Sales`, `Fact_Booking`, `Fact_PlatformEnquiry`,
`Fact_Wallet`, `Fact_Voucher`, and (acquiring staff) `Dim_Customer[AcquisitionStaffKey]`.
*Key 0 = self-service / app / no staff, so these can stay active relationships.*

**Dim_MenuItem[MenuItemKey]** → `Fact_OrderItems[MenuItemKey]`
(and inactive → `Fact_PlatformContent[DishMentionedKey]`).

**Dim_Promotion[PromotionKey]** → `Fact_Sales`, `Fact_OrderItems`, `Fact_Booking`,
`Fact_PlatformContent`, `Fact_PlatformEnquiry`.

**Dim_Platform[PlatformKey]** → `Fact_PlatformContent`, `Fact_PlatformEnquiry`, `Fact_Review`,
`Fact_AIReplyLog`, and `Fact_Booking[SourcePlatformKey]`.

**Dim_PaymentMethod[PaymentMethodKey]** → `Fact_Sales`.

**Dim_Voucher[VoucherCampaignKey]** → `Fact_Voucher`.

**Dim_Campaign[CampaignKey]** → `Fact_PlatformEnquiry[CampaignKey]`,
`Fact_PlatformContent[CampaignKey]` (use this as the **Campaign** slicer; key 0 = organic/direct).

**Dim_Table[TableKey]** → `Fact_Sales[TableKey]`, `Fact_Booking[TableKey]`
(enables true table turnover & utilisation; key 0 = takeaway / no table).

**Dim_Platform[PlatformKey]** also → `Fact_PlatformFollowers[PlatformKey]`.
**Dim_Date[DateKey]** also → `Fact_PlatformFollowers[MonthStartDateKey]`,
`Fact_ActionPlan[WeekStartDateKey]`. **Dim_Branch[BranchKey]** also → `Fact_ActionPlan[BranchKey]`
(0 = group-wide). **Dim_Staff[StaffKey]** and **Dim_MenuItem[MenuItemKey]** also →
`Fact_Review[StaffKey]` / `Fact_Review[MenuItemKey]` (0 = no staff/dish named in the review).

`Fact_OrderItems[BillID]` relates to `Fact_Sales[BillID]` (many-to-one) only if you want
item→bill drill; usually you simply slice both facts by the shared dimensions instead.

---

## 4. Data dictionary

### Dimensions

**Dim_Date** — `DateKey` (yyyymmdd int, PK), `Date`, `Year`, `QuarterNum`, `Quarter`, `MonthNum`,
`MonthName`, `MonthShort`, `YearMonth`, `MonthYearLabel`, `Day`, `DayOfWeekNum` (1=Mon…7=Sun),
`DayOfWeek`, `DayShort`, `WeekOfYear`, `IsWeekend`, `IsPublicHoliday`, `HolidayName`,
`FestivePeriod` (CNY / Mother's Day / Mid-Autumn / Year-End windows that lift sales).

**Dim_Branch** — `BranchKey` (PK), `BranchName` (use this exact name everywhere), `ShortName`,
`Area`, `Manager`, `OpenDate`, `SeatingCapacity`, `MainHallTables`, `VIPRooms`, `SalesWeight`
(relative size used to skew the data), `Region`, `State`.

**Dim_Customer** — `CustomerKey` (PK; **0 = Walk-In Guest / non-member**), `MemberCode`,
`FullName`, `Phone`, `JoinDate`, `Tier` (Bronze/Silver/Gold/Platinum/Corporate/Non-Member),
`BirthdayMonth` (1–12), `HomeBranchKey`, `AcquisitionPlatformKey`, `AcquisitionStaffKey`
(staff who signed the member up; **0 = self sign-up via app/web**), `HasWallet`, `SpendIndex`,
`LastVisitDate`, `Status` (Active ≤90d / At Risk 91–180d / Inactive), `VisitCount2025`,
`DaysSinceLastVisit`. *(Status & last-visit are derived from the actual sales facts.)*

**Dim_Staff** — `StaffKey` (PK; **0 = Self-Service / App / No Staff**), `StaffName`, `BranchKey`,
`Role`, `HandlesSales`, `HireDate`, `Active`.

**Dim_MenuItem** — `MenuItemKey` (PK), `ItemNameCN`, `ItemNameEN`, `Category`, `UnitPrice` (RM),
`UnitCost` (RM), `FoodCostPct`, `IsSignature` (the 10 signature dishes are flagged),
`IsTikTokSuitable`, `Active`.

**Dim_Promotion** — `PromotionKey` (PK; **0 = No Promotion**), `PromoCode`, `PromoName`,
`PromoType`, `TargetMargin`, `StartDate`, `EndDate`, `IsEvent`.

**Dim_Platform** — `PlatformKey` (PK; **0 = Direct/Walk-In**), `PlatformName`, `PlatformLayer`
(Awareness / Enquiry / Owned / Search), `IsOwned`, `PaidMediaCapable`.

**Dim_PaymentMethod** — `PaymentMethodKey` (PK), `PaymentMethod`, `PaymentType`.

**Dim_Voucher** — `VoucherCampaignKey` (PK; **0 = No Voucher**), `CampaignCode`, `CampaignName`,
`VoucherType` (Cash/Percentage/FreeItem), `FaceValue`, `Channel`, `StartDate`, `EndDate`.

**Dim_Campaign** — `CampaignKey` (PK; **0 = Organic / Direct**), `CampaignName`, `UTMSource`,
`UTMMedium`, `UTMCampaign`, `PromotionKey` (the package it promotes), `StartDate`, `EndDate`,
`Objective`. *Use for Tab 12 "campaign source" and the global Campaign slicer.*

**Dim_Table** — `TableKey` (PK; **0 = Takeaway / None**), `BranchKey`, `TableCode` (e.g. `KL-T01`,
`KL-VIP1`), `TableType` (Main Hall / VIP Room / VVIP Room / Banquet Hall), `Seats`. *Enables real
table turnover (bills ÷ tables) and utilisation (tables used ÷ tables available) for Tab 3 & 7.*

### Facts

**Fact_Sales** (bill level) — `SalesKey` (PK), `BillID` (degenerate), `DateKey`, `BranchKey`,
`CustomerKey`, `StaffKey`, `PaymentMethodKey`, `PromotionKey`, `DiningPeriod` (Lunch/Dinner/Tea),
`OrderHour` (11–22, for peak-hour analysis), `OrderTime` (HH:MM),
`ServiceType` (Dine-in/Takeaway/VIP Room/Event), `Pax`, `GrossSales`, `DiscountAmount`, `NetSales`
(**← the headline "Total Sales" measure**), `ServiceCharge` (10%), `SST` (6%), `BillAmount` (total
collected), `UpsellAmount` (drink + add-on value on the bill, for staff upsell tracking),
`TableKey` (physical table/room used; 0 = takeaway), `IsMemberSale`.

**Fact_OrderItems** (dish line) — `OrderItemKey` (PK), `BillID`, `DateKey`, `BranchKey`,
`MenuItemKey`, `PromotionKey`, `Quantity`, `UnitPrice`, `GrossLineSales`, `LineDiscount`,
`NetLineSales`, `UnitCost`, `LineCost`, `GrossProfit`, `LineType` (Main / Drink / Add-On / Staple),
`IsAddOn` (Yes for desserts & soups — separates add-on sales from drink sales). Lines sum to the
bill's `NetSales`.

**Fact_Booking** — `BookingKey` (PK), `BookingID`, `BookingDateKey`, `VisitDateKey`,
`VisitHour` (11–22, for peak booking-time analysis), `VisitTime` (HH:MM), `BranchKey`,
`CustomerKey`, `StaffKey`, `SourcePlatformKey`, `PromotionKey`, `ServiceType`, `RoomType`
(Main Hall/VIP Room/VVIP Room/Banquet Hall), `Pax`, `Status` (Confirmed/Cancelled/No-Show),
`DepositAmount`, `FinalBillAmount`, `BillID` (links confirmed bookings to `Fact_Sales`),
`TableKey` (table/room assigned), `IsEvent`.

**Fact_Wallet** — `WalletTxnKey` (PK), `DateKey`, `CustomerKey`, `BranchKey`, `StaffKey`
(staff who collected the top-up/payment; **0 = app self-service**), `TxnType`
(Top-Up/Payment), `Amount` (+top-up / −payment), `BalanceAfter`, `TopUpCampaign`.

**Fact_Voucher** — `VoucherKey` (PK), `VoucherCode`, `VoucherCampaignKey`, `CustomerKey`,
`IssueDateKey`, `RedeemDateKey` (0 if never redeemed), `BranchKey` (redeeming branch),
`StaffKey` (staff who handled redemption; 0 if not redeemed),
`Status` (Issued/Redeemed/Expired), `FaceValue`, `RedeemedAmount`.

**Fact_PlatformContent** — `ContentKey` (PK), `PostID`, `PlatformKey`, `PostDateKey`,
`BranchKeyMentioned`, `DishMentionedKey`, `ContentTopic`, `PostURL`, `Views`, `Likes`, `Comments`,
`Shares`, `Saves`, `LinkClicks`, `CampaignKey` (0 = organic). *(Public/exportable metrics only, per the brief.)*

**Fact_PlatformEnquiry** — `EnquiryKey` (PK), `EnquiryID`, `PlatformKey`, `DateKey`, `BranchKey`,
`CustomerKey`, `PromotionKey`, `EnquiryTopic`, `ConvertedToBooking`, `BookingID`, `StaffKey`,
`ResponseTimeMin`, `CampaignKey` (source campaign / UTM; 0 = organic).

**Fact_Review** — `ReviewKey` (PK), `ReviewID`, `PlatformKey` (Google/Facebook), `BranchKey`,
`DateKey`, `Rating` (1–5), `Sentiment`, `ReviewTopic`, `Keyword`, `HasComplaint`, `ComplaintType`,
`StaffKey` (staff named in the review; 0 = none), `MenuItemKey` (dish named; 0 = none),
`StaffMention`, `DishMention`. *Enables per-staff review mentions/complaints (Tab 14) and per-dish
complaints (Tab 6).*

**Fact_AIReplyLog** — `AILogKey` (PK), `ConversationID`, `PlatformKey`, `DateKey`, `BranchKey`,
`Topic`, `AISuccess`, `HumanTakeover`, `ResponseTimeSec`, `AssetSent`, `BookingConversion`,
`FollowUpSent`.

**Fact_Target** — `TargetKey` (PK), `BranchKey`, `MonthStartDateKey`, `YearMonth`, `Year`,
`MonthNum`, `BaselineMonthlyTarget` (equal RM333,333 split), `BaselineDailyTarget`,
`WeightedMonthlyTarget` (branch- and season-weighted; sums to RM20M), `WeightedDailyTarget`,
`BranchYearlyTargetEqual`, `BranchYearlyTargetWeighted`.

**Fact_PlatformFollowers** — `SnapshotKey` (PK), `MonthStartDateKey`, `YearMonth`, `PlatformKey`,
`Followers`, `Members`, `Subscribers`, `NewThisMonth`, `EngagementRatePct`. *Monthly account-level
snapshot for the Tab 11 comparison table (followers/members/subscribers by platform).*

**Fact_ActionPlan** — `ActionKey` (PK), `WeekStartDateKey`, `Tab` (which dashboard tab it feeds),
`BranchKey` (0 = group-wide), `Category`, `Priority` (High/Medium/Low),
`Finding_WhatHappened`, `Rationale_WhyItMatters`, `Action_WhatToDoNext`, `ExpectedImpactRM`,
`Status` (Open/In Progress/Done). *Seeds the Boss Overview action list, the Forecast weekly plan,
and the per-tab "What happened / Why it matters / What to do next" insight boxes — edit freely.*

---

## 5. Starter DAX measures (matches the brief)

```DAX
Total Sales          = SUM ( Fact_Sales[NetSales] )
Sales Target         = SUM ( Fact_Target[WeightedMonthlyTarget] )
Target Achievement % = DIVIDE ( [Total Sales], [Sales Target] )
Sales Gap            = [Sales Target] - [Total Sales]
Bill Count           = DISTINCTCOUNT ( Fact_Sales[BillID] )
Average Bill Value   = DIVIDE ( [Total Sales], [Bill Count] )
Avg Spend / Customer = DIVIDE ( [Total Sales], DISTINCTCOUNT ( Fact_Sales[CustomerKey] ) )

Member Sales         = CALCULATE ( [Total Sales], Fact_Sales[IsMemberSale] = "Yes" )
Member Sales %       = DIVIDE ( [Member Sales], [Total Sales] )

Confirmed Bookings   = CALCULATE ( COUNTROWS ( Fact_Booking ), Fact_Booking[Status] = "Confirmed" )
No-Show Rate         = DIVIDE (
                         CALCULATE ( COUNTROWS ( Fact_Booking ), Fact_Booking[Status] = "No-Show" ),
                         COUNTROWS ( Fact_Booking ) )
Booking Conversion % = DIVIDE (
                         CALCULATE ( COUNTROWS ( Fact_PlatformEnquiry ),
                                     Fact_PlatformEnquiry[ConvertedToBooking] = "Yes" ),
                         COUNTROWS ( Fact_PlatformEnquiry ) )

Vouchers Issued      = COUNTROWS ( Fact_Voucher )
Voucher Redemption % = DIVIDE (
                         CALCULATE ( COUNTROWS ( Fact_Voucher ), Fact_Voucher[Status] = "Redeemed" ),
                         [Vouchers Issued] )

Wallet Liability     = SUMX ( VALUES ( Fact_Wallet[CustomerKey] ),
                         CALCULATE ( SUM ( Fact_Wallet[Amount] ) ) )   -- remaining balance

Active Members       = CALCULATE ( DISTINCTCOUNT ( Dim_Customer[CustomerKey] ),
                                    Dim_Customer[Status] = "Active" )
Churn Rate           = DIVIDE (
                         CALCULATE ( DISTINCTCOUNT ( Dim_Customer[CustomerKey] ),
                                     Dim_Customer[Status] = "Inactive" ),
                         CALCULATE ( DISTINCTCOUNT ( Dim_Customer[CustomerKey] ),
                                     Dim_Customer[CustomerKey] <> 0 ) )

Gross Profit         = SUM ( Fact_OrderItems[GrossProfit] )
Food Cost %          = DIVIDE ( SUM ( Fact_OrderItems[LineCost] ),
                                SUM ( Fact_OrderItems[NetLineSales] ) )
Avg Rating           = AVERAGE ( Fact_Review[Rating] )
AI Success Rate      = DIVIDE (
                         CALCULATE ( COUNTROWS ( Fact_AIReplyLog ), Fact_AIReplyLog[AISuccess] = "Yes" ),
                         COUNTROWS ( Fact_AIReplyLog ) )

-- newly enabled by the gap-fill columns --
Sales by Hour        = SUM ( Fact_Sales[NetSales] )         -- put OrderHour on the axis for peak/low hour
Drink Sales          = CALCULATE ( SUM ( Fact_OrderItems[NetLineSales] ), Fact_OrderItems[LineType] = "Drink" )
Add-On Sales         = CALCULATE ( SUM ( Fact_OrderItems[NetLineSales] ), Fact_OrderItems[IsAddOn] = "Yes" )
Upsell Amount        = SUM ( Fact_Sales[UpsellAmount] )     -- slice by StaffKey for upsell-by-staff
Member Signups       = CALCULATE ( DISTINCTCOUNT ( Dim_Customer[CustomerKey] ),
                                   Dim_Customer[CustomerKey] <> 0 )   -- slice by AcquisitionStaffKey
Wallet Collected     = CALCULATE ( SUM ( Fact_Wallet[Amount] ), Fact_Wallet[TxnType] = "Top-Up" )  -- slice by StaffKey

Table Turnover       = DIVIDE ( DISTINCTCOUNT ( Fact_Sales[BillID] ),
                                DISTINCTCOUNT ( Fact_Sales[TableKey] ) )      -- bills per table
Table Utilisation %  = DIVIDE ( DISTINCTCOUNT ( Fact_Sales[TableKey] ),
                                CALCULATE ( DISTINCTCOUNT ( Dim_Table[TableKey] ), Dim_Table[TableKey] <> 0 ) )
Campaign Conversion  = DIVIDE (
                         CALCULATE ( COUNTROWS ( Fact_PlatformEnquiry ), Fact_PlatformEnquiry[ConvertedToBooking] = "Yes" ),
                         COUNTROWS ( Fact_PlatformEnquiry ) )                 -- slice by Dim_Campaign
Total Followers      = CALCULATE ( SUM ( Fact_PlatformFollowers[Followers] )
                                 + SUM ( Fact_PlatformFollowers[Members] )
                                 + SUM ( Fact_PlatformFollowers[Subscribers] ) )
Dish Complaints      = CALCULATE ( COUNTROWS ( Fact_Review ),
                         Fact_Review[HasComplaint] = "Yes", Fact_Review[MenuItemKey] <> 0 )  -- by dish
Staff Review Mentions= CALCULATE ( COUNTROWS ( Fact_Review ), Fact_Review[StaffKey] <> 0 )    -- by staff
```

---

## 6. How to load into Power BI

1. **Get Data → Folder** and point to this folder (or import the CSVs individually). Folder import
   lets you refresh everything at once.
2. In Power Query, confirm key columns are **Whole Number** and amount columns are **Decimal**;
   set `Dim_Date[Date]` to **Date**.
3. Create the relationships in section 3 (most auto-detect correctly; check the `Dim_Date` ones).
4. **Model view → Dim_Date → Mark as date table** using `Dim_Date[Date]`.
5. Add the measures in section 5, then build the 16 tabs from the brief.
6. Apply the traffic-light rule from the brief: green ≥100% of target, yellow 90–99%, red <90%.

---

## 7. Notes & assumptions

- Synthetic data for demonstration; names, phone numbers, post URLs and reviews are fabricated.
- Public-holiday dates are representative for Klang Valley 2025, not an official gazette.
- The 10-platform tables follow the brief's rule: only *trackable* fields (views, likes, public
  comments, enquiries-if-tagged, booking source-if-tagged) — no invented private DM sales.
- Group net sales ≈ RM17.87M vs RM20M target (≈89%) by design, so target-gap and forecast visuals
  have something to show. Adjust the `BASE_BILLS` knob in the generator to change this.

---

## 6. 16-tab coverage map

Every tab below is fully backed by the tables in this package. "Source" lists the main
table(s); most KPIs are a single measure or a field on a slicer/axis.

| # | Tab | Source tables |
|---|---|---|
| 1 | Boss Overview | Fact_Sales, Fact_Target, Fact_Booking, Fact_Wallet, Fact_PlatformEnquiry, Fact_ActionPlan |
| 2 | RM20 Million Target | Fact_Sales, Fact_Target, Dim_Date |
| 3 | Branch Performance | Fact_Sales, Fact_Booking, Fact_Review, Dim_Table (turnover), Dim_Branch |
| 4 | Daily Sales Monitoring | Fact_Sales (OrderHour, DiningPeriod, ServiceType), Dim_Date, Dim_PaymentMethod |
| 5 | Promotion Performance | Fact_Sales, Fact_OrderItems (LineType/IsAddOn, GrossProfit), Dim_Promotion |
| 6 | Menu & Signature Dish | Fact_OrderItems, Dim_MenuItem, Fact_Review (MenuItemKey for dish complaints) |
| 7 | Booking & Table Utilisation | Fact_Booking (VisitHour, TableKey), Dim_Table |
| 8 | VIP Room & Event Sales | Fact_Booking (RoomType, Deposit, FinalBill), Fact_Sales, Dim_Table, Dim_Promotion |
| 9 | Membership | Dim_Customer (Status, Tier, VisitCount), Fact_Sales |
| 10 | Membership Wallet / App | Fact_Wallet (StaffKey, campaign), Fact_Voucher |
| 11 | 10-Platform Overview | Fact_PlatformContent, Fact_PlatformEnquiry, Fact_PlatformFollowers, Fact_Review, Dim_Platform |
| 12 | Omnichannel Attribution | Fact_PlatformEnquiry (+CampaignKey) → Fact_Booking → Fact_Sales, Dim_Customer, Dim_Campaign |
| 13 | AI Auto-Reply Performance | Fact_AIReplyLog |
| 14 | Staff Performance | Fact_Sales (Upsell), Fact_Booking, Fact_Wallet, Fact_Voucher, Dim_Customer (AcqStaff), Fact_Review |
| 15 | Customer Review & Satisfaction | Fact_Review (StaffKey, MenuItemKey, keywords, complaints) |
| 16 | Forecast & Growth Opportunity | Fact_Sales, Fact_Target, Fact_ActionPlan, Fact_Booking, Dim_MenuItem |

**Slicers** — date (Dim_Date), branch (Dim_Branch), platform (Dim_Platform), campaign (Dim_Campaign),
promotion/package (Dim_Promotion), dining type (Fact_Sales[ServiceType]), membership tier
(Dim_Customer[Tier]), staff (Dim_Staff), table/room type (Dim_Table[TableType]).
