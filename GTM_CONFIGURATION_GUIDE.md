# Google Tag Manager Configuration Guide
## Solar Calculator - GA4 Event Tracking

This guide explains how to configure Google Tag Manager (GTM) to capture solar power data from the calculator's `calculator_results_generated` event and send it to Google Analytics 4 (GA4).

---

## Overview

The solar calculator pushes a `calculator_results_generated` event to the dataLayer whenever a user completes a calculation. This event includes comprehensive solar power estimates and recommendations (no personal financial information).

---

## Event Data Structure

### Core Event Parameters
| Parameter | Type | Description |
|-----------|------|-------------|
| `event` | string | Always "calculator_results_generated" |
| `calculator_name` | string | Calculator identifier |
| `calculator_version` | string | Version number |
| `timestamp` | string | ISO 8601 timestamp |

### System Selection & Financing
| Parameter | Type | Description |
|-----------|------|-------------|
| `system_selected` | string | "panel-only" or "panel-battery" |
| `has_panel_only` | boolean | Whether panel-only system was calculated |
| `has_panel_battery` | boolean | Whether battery system was calculated |
| `is_financed` | boolean | Whether user selected financing |
| `loan_term` | number | Loan term in years (null if not financed) |
| `interest_rate` | number | Annual interest rate (null if not financed) |

### Power Usage Data
| Parameter | Type | Description |
|-----------|------|-------------|
| `annual_power_cost_estimate` | number | Estimated annual power cost ($) |
| `avg_monthly_kwh` | number | Average monthly kWh usage |
| `power_bill_summer` | number | Summer power bill ($) |
| `power_bill_winter` | number | Winter power bill ($) |
| `summer_kwh` | number | Summer kWh usage |
| `winter_kwh` | number | Winter kWh usage |

### Panel-Only System Metrics (if available)
| Parameter | Type | Description |
|-----------|------|-------------|
| `panel_only_system_kw` | number | System size in kilowatts |
| `panel_only_panel_count` | number | Number of solar panels |
| `panel_only_system_cost` | number | Estimated system cost ($) |
| `panel_only_monthly_generation_kwh` | number | Monthly generation (kWh) |
| `panel_only_annual_generation_kwh` | number | Annual generation (kWh) |
| `panel_only_monthly_savings` | number | Monthly savings ($) |
| `panel_only_annual_savings` | number | Annual savings ($) |
| `panel_only_payback_years` | number | Payback period (years) |
| `panel_only_25year_net_savings` | number | 25-year net savings ($) |
| `panel_only_self_consumption_percent` | number | Self-consumption rate (45%) |
| `panel_only_export_percent` | number | Export rate (55%) |

### Battery System Metrics (if available)
| Parameter | Type | Description |
|-----------|------|-------------|
| `battery_system_kw` | number | System size in kilowatts |
| `battery_panel_count` | number | Number of solar panels |
| `battery_kwh_capacity` | number | Battery storage capacity (kWh) |
| `battery_system_cost` | number | Estimated system cost ($) |
| `battery_monthly_generation_kwh` | number | Monthly generation (kWh) |
| `battery_annual_generation_kwh` | number | Annual generation (kWh) |
| `battery_monthly_savings` | number | Monthly savings ($) |
| `battery_annual_savings` | number | Annual savings ($) |
| `battery_payback_years` | number | Payback period (years) |
| `battery_25year_net_savings` | number | 25-year net savings ($) |
| `battery_self_consumption_percent` | number | Self-consumption rate (75%) |
| `battery_export_percent` | number | Export rate (25%) |

---

## GTM Configuration Steps

### Step 1: Create DataLayer Variables

In GTM, create a **DataLayer Variable** for each parameter you want to track:

1. Go to **Variables** → **New**
2. Click **Variable Configuration**
3. Select **Data Layer Variable**
4. Enter the **Data Layer Variable Name** (use exact names from the table above)
5. Name your variable (e.g., "DLV - System Selected" for `system_selected`)
6. Save

**Recommended Priority Variables to Create:**
- `system_selected`
- `has_panel_only`
- `has_panel_battery`
- `is_financed`
- `annual_power_cost_estimate`
- `panel_only_system_kw`
- `panel_only_system_cost`
- `panel_only_annual_savings`
- `panel_only_payback_years`
- `panel_only_25year_net_savings`
- `battery_system_kw`
- `battery_system_cost`
- `battery_annual_savings`
- `battery_payback_years`
- `battery_25year_net_savings`
- `avg_monthly_kwh`

### Step 2: Create Custom Event Trigger

1. Go to **Triggers** → **New**
2. Click **Trigger Configuration**
3. Select **Custom Event**
4. Event name: `calculator_results_generated`
5. Trigger fires on: **All Custom Events**
6. Name your trigger: "CE - Calculator Results Generated"
7. Save

### Step 3: Create GA4 Event Tag

1. Go to **Tags** → **New**
2. Click **Tag Configuration**
3. Select **Google Analytics: GA4 Event**
4. **Configuration Tag**: Select your GA4 Configuration tag
5. **Event Name**: `calculator_results_generated`
6. Click **Event Parameters** → **Add Row** for each parameter you want to send

**Example Event Parameters Configuration:**

| Parameter Name | Value (Variable) |
|----------------|------------------|
| `system_selected` | `{{DLV - System Selected}}` |
| `is_financed` | `{{DLV - Is Financed}}` |
| `annual_power_cost` | `{{DLV - Annual Power Cost Estimate}}` |
| `panel_only_kw` | `{{DLV - Panel Only System KW}}` |
| `panel_only_cost` | `{{DLV - Panel Only System Cost}}` |
| `panel_only_savings` | `{{DLV - Panel Only Annual Savings}}` |
| `panel_only_payback` | `{{DLV - Panel Only Payback Years}}` |
| `battery_kw` | `{{DLV - Battery System KW}}` |
| `battery_cost` | `{{DLV - Battery System Cost}}` |
| `battery_savings` | `{{DLV - Battery Annual Savings}}` |
| `battery_payback` | `{{DLV - Battery Payback Years}}` |

**Note:** GA4 has a limit of 25 event parameters per event. Select the most important parameters for your analysis.

7. **Triggering**: Select your "CE - Calculator Results Generated" trigger
8. Name your tag: "GA4 - Calculator Results Generated"
9. Save

### Step 4: Test Your Configuration

1. Click **Preview** in GTM
2. Open your calculator page
3. Complete a calculation
4. In GTM Preview mode:
   - Look for the `calculator_results_generated` event
   - Verify your tag fired
   - Check that DataLayer variables contain expected values
5. Verify in GA4:
   - Go to **Configure** → **DebugView** in GA4
   - Complete a calculation
   - Verify the event appears with correct parameters

### Step 5: Publish

1. Click **Submit** in GTM
2. Add a descriptive version name (e.g., "Added solar calculator tracking")
3. Click **Publish**

---

## Recommended GA4 Custom Dimensions

To make this data available in GA4 reports, create custom dimensions:

1. In GA4, go to **Configure** → **Custom Definitions** → **Create Custom Dimension**
2. Create dimensions for key parameters:

| Dimension Name | Event Parameter | Scope |
|----------------|----------------|-------|
| System Selected | `system_selected` | Event |
| Is Financed | `is_financed` | Event |
| Panel Only System Size | `panel_only_kw` | Event |
| Battery System Size | `battery_kw` | Event |

---

## Example Use Cases

### 1. Track Which System Type Users Prefer
**Metric:** Count of `calculator_results_generated` events
**Dimension:** `system_selected`

### 2. Analyze System Sizes by Region
**Metric:** Average `panel_only_kw` or `battery_kw`
**Dimension:** User location (from GA4)

### 3. Financing Adoption Rate
**Metric:** % of events where `is_financed` = true
**Filter:** `calculator_results_generated` events

### 4. Average Estimated Savings
**Metric:** Average `panel_only_annual_savings` or `battery_annual_savings`
**Segment:** By `system_selected`

---

## Important Notes

1. **No Personal Financial Data**: The GTM push does NOT include any personal financial information (income, property value, expenses, etc.)

2. **Conditional Parameters**: Panel-only and battery system parameters are only sent when those systems are calculated

3. **GA4 Limits**:
   - Maximum 25 event parameters per event
   - Maximum 50 custom dimensions per property
   - Choose the most valuable parameters for your analysis

4. **Data Types**: All numeric values are sent as numbers (not strings) for proper analysis in GA4

5. **Privacy**: Always ensure your GTM/GA4 configuration complies with your privacy policy and applicable regulations (GDPR, CCPA, etc.)

---

## Troubleshooting

### DataLayer Variables Show "undefined"
- Check that the variable name exactly matches the dataLayer parameter name
- Verify the event is firing (check in Preview mode)
- Ensure the parameter exists in the dataLayer for that specific calculation

### Tag Not Firing
- Verify the trigger event name matches exactly: `calculator_results_generated`
- Check that the trigger is attached to your tag
- Use Preview mode to debug

### Data Not Appearing in GA4
- Check DebugView first (enable debug mode in GTM Preview)
- Allow 24-48 hours for data to appear in standard reports
- Verify your GA4 Measurement ID is correct in your GA4 Configuration tag

---

## Version History

- **v1.0** - Initial GTM configuration guide with comprehensive solar power data tracking
