# CXCAT-3353-Data-Trinity-Data-Visualization-Model
This model supports the Tableau Visualizations using HHH, Case, Feedback data sources.

CX Business Logic: Reliability & Support Trinity Model

Overview: This repository contains the SQL logic for the CX Data Trinity Model. This data model correlates three critical domains to understand the impact of product reliability on customer support volume and sentiment: Reliability (HHH): Device-level error logs and unhealthy events. Support (CSX): Customer support cases linked to users within a 7-day window of an error.Sentiment (PCS): Post-Contact Survey results linked to those specific support cases.

Data Lineage & ArchitectureThe pipeline consists of three layers transforming raw logs into a visualization-ready dataset.graph LR
    A[Raw: Reliability Errors] --> B[01_Base: Trinity Flags]
    C[Raw: CSX Cases] --> B
    D[Raw: PCS Surveys] --> B
    B --> E[02_Intermediate: Daily Aggregates]
    E --> F[03_Presentation: Tableau Tall View]
Layer 1: Base Tables (01_base_tables)Table: CXCAT_712_DATA_TRINITY_FLAGSGranularity: Daily per Sonos ID.Logic: Joins reliability error logs with Support Cases (using a 7-day lookahead window) and joins Surveys to those cases.Key Flags: CX_CASE_WITHIN_7_DAYS, PCS_COMPLETED, and 14 specific error type flags (e.g., HHH_SPOTIFY_PLAYREPORT).

Layer 2: Intermediate Aggregation (02_intermediate)View: CXCAT_712_DAILY_AGGREGATED_TRINITY_FLAGS_CASES_ONLYGranularity: Daily Summary.Filter: Crucial: Filters ONLY for users who created a support case (CX_CASE_WITHIN_7_DAYS = 1).Purpose: Pre-calculates sums of errors specifically for the population that contacted support, optimizing performance for downstream BI tools.

Layer 3: Presentation (03_presentation)View: CXCAT_712_DAILY_AGGREGATED_TRINITY_FLAGS_TALLStructure: Unpivoted (Tall/Long format).Purpose: Formats data specifically for Tableau. Columns ERROR_NAME and ERROR_COUNT allow for easy filtering and multi-line charting in dashboards without creating 14 separate measures.

Refresh ScheduleFrequency: DailyTime: 6:00 AM UTCDependencies: Requires HHH_DAILY_BY_ACTIVITY_NAME..., VIZ_OWNER_CSX_CONTACT_CASES_VOC, and VIZ_POST_CONTACT_SURVEY to be updated first.

MaintenanceTo update the business logic (e.g., adding a new top error), edit models/01_base_tables/trinity_flags.sql and ensure the new column is added to the unpivot list in models/03_presentation/tableau_tall_view.sql.
