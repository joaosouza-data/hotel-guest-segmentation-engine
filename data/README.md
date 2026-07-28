# Data

## Source

[A hotel's customers dataset](https://www.kaggle.com/datasets/nantonio/a-hotels-customers-dataset), publicly available on Kaggle. Originally published as a peer-reviewed data article:

> Antonio, N., de Almeida, A., & Nunes, L. (2020). A hotel's customers personal, behavioral, demographic, and geographic dataset from Lisbon, Portugal (2015–2018). *Data in Brief*, 33, 106583. https://doi.org/10.1016/j.dib.2020.106583

83,590 customer records, 31 variables, covering three full years (2015–2018) of a real hotel's Property Management System and CRM data. The dataset was already anonymized by the original authors before publication: no name, email, or contact details are included — `NameHash` and `DocIDHash` are hashed identifiers, not personal data.

The raw data file is not included in this repository. Download it from the Kaggle link above (`HotelCustomersDataset.tsv`, tab-separated) to reproduce the notebook.

## Data dictionary

| Column | Description |
|---|---|
| `ID` | Customer identifier |
| `Nationality` | Customer's nationality |
| `Age` | Customer's age |
| `DaysSinceCreation` | Days since the customer record was created |
| `NameHash`, `DocIDHash` | Hashed identifiers (anonymized, not usable to re-identify customers) |
| `AverageLeadTime` | Average days between booking and arrival |
| `LodgingRevenue`, `OtherRevenue` | Revenue from lodging and from other services (e.g. F&B) |
| `BookingsCanceled`, `BookingsNoShowed`, `BookingsCheckedIn` | Count of bookings by outcome |
| `PersonsNights`, `RoomNights` | Total person-nights and room-nights stayed |
| `DaysSinceLastStay`, `DaysSinceFirstStay` | Recency measures (raw data uses `-1` as a sentinel for "never stayed" — handled explicitly in cleaning) |
| `DistributionChannel` | Booking channel (e.g. travel agent/operator, direct, corporate) |
| `MarketSegment` | Original hospitality-standard segment label |
| `SRHighFloor`, `SRLowFloor`, `SRAccessibleRoom`, `SRMediumFloor`, `SRBathtub`, `SRShower`, `SRCrib`, `SRKingSizeBed`, `SRTwinBed`, `SRNearElevator`, `SRAwayFromElevator`, `SRNoAlcoholInMiniBar`, `SRQuietRoom` | Special request flags (binary) |

## Engineered features

| Feature | Built from | Purpose |
|---|---|---|
| `TotalRevenue` | `LodgingRevenue + OtherRevenue` | Single measure of customer value |
| `AvgRevenuePerNight` | `TotalRevenue / RoomNights` | Value per night stayed (0 for never-stayed customers) |
| `RevenuePerBooking` | `TotalRevenue / BookingsCheckedIn` | Distinguishes high-spend-per-visit customers from frequent low-spend ones |
| `CancellationRate` | `BookingsCanceled / total bookings` | Reliability signal (excluded from final model — too sparse, see notebook 3.5) |
| `IsFromPRT` | `Nationality == Portugal` | Domestic vs. international guest, a sharp behavioral divide |
| `TotalSpecialRequests` | Sum of all `SR*` binary flags | Aggregate preference intensity |
| `HasStayed` | Derived from the `-1` sentinel in `DaysSinceLastStay`/`DaysSinceFirstStay` | Flags customers who registered but never completed a stay |

## Data quality issues found and fixed

See the notebook (`notebooks/hotel-guest-segmentation-engine.ipynb`, sections 2.4–2.10) for the full traceable issue log. In summary: 80 exact duplicate records removed, `-1` sentinel values in recency fields replaced with a flag plus an out-of-range placeholder (not treated as missing), ages above 100 or equal to 0 corrected, continuous variables capped at the 99th percentile.
