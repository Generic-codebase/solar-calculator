# Solar Calculator - Testing Guide

## Overview

This document describes the regression testing setup for the Solar Calculator to ensure changes don't break existing functionality.

## Test Suite

### Running Tests

1. **Open in Browser**
   ```bash
   # Open test-calculator.html in your browser
   open test-calculator.html
   # or
   python -m http.server 8000
   # Then navigate to http://localhost:8000/test-calculator.html
   ```

2. **Click "Run All Tests"**
   - Tests will execute automatically
   - Results will display with pass/fail status
   - Summary shows total/passed/failed counts

### Test Categories

#### 1. Generation Calculations (5 tests)
Tests the core solar generation formulas:
- **3.7 kWh/kW/day formula**: Verifies 10kW system generates 37 kWh/day, 5kW generates 18.5 kWh/day
- **Panel-only sizing**: Confirms system is sized at 110% of usage
- **Battery system sizing**: Confirms system is sized at 120% of usage
- **Monthly generation formula**: Validates kW × 3.7 × 30.416 calculation

#### 2. Cost Formula Accuracy (5 tests)
Tests system pricing calculations:
- **Panel-only cost**: Validates panels × 708.75 + 3675 formula
- **Battery system cost**: Validates (panels × 703.5) + (battery kWh × 892.5) + 6562.5
- **Battery capacity**: Confirms ceil(0.55 × panel count) calculation
- Tests multiple panel counts (20, 25, 30 panels)

#### 3. Self-Consumption & Export Rates (3 tests)
Tests savings calculations:
- **Panel-only rates**: Validates 45% self-consumption, 55% export @ 17c/kWh
- **Battery rates**: Validates 75% self-consumption, 25% export @ 17c/kWh
- Tests with various power costs and generation levels

#### 4. Inflation & Degradation (5 tests)
Tests long-term projection accuracy:
- **3% annual inflation**: Tests over 5 and 10 year periods
- **0.5% panel degradation**: Tests over 10 and 25 year periods
- **25-year multiplier**: Confirms 38.55 multiplier for 3% inflation

#### 5. Financing Calculations (5 tests)
Tests loan payment formulas:
- **Standard rates**: 6.5% over 10 years
- **Zero interest**: Edge case of 0% interest rate
- **High interest**: 15% and 20% rates
- **Loan terms**: 1 year (short) and 25 years (long)
- Validates amortization formula accuracy

#### 6. Financial Thresholds (4 tests)
Tests borrowing power logic:
- **Equity requirements**: Minimum $20k equity needed for solar ready
- **LVR calculation**: (lending / property) × 100
- **Available equity**: (property × 0.8) - lending
- Tests edge cases around $20k threshold

#### 7. Monthly Serviceability (3 tests)
Tests serviceability section accuracy:
- **Actual interest rate usage**: Confirms solar loan uses user-specified rate, not 6.8%
- **Surplus updates**: Verifies monthly surplus recalculates when switching systems
- **Mortgage test rate**: Confirms existing mortgage uses 6.8% test rate

#### 8. System Consistency (2 tests)
Tests consistency across sections:
- **Payment matching**: Verifies loan payment matches between Monthly Serviceability and Loan Financing Details
- **System switching**: Validates all metrics update when switching between systems

#### 9. Edge Cases (4 tests)
Tests boundary conditions:
- **Very low usage**: 100 kWh/month
- **Very high usage**: 2000 kWh/month
- **Zero interest**: 0% rate
- **Maximum interest**: 20% rate

#### 10. Payback Period (3 tests)
Tests payback calculations:
- **Simple payback**: cost / annual savings
- **Payback with inflation**: Faster payback due to rising power costs
- **Financed vs cash**: Confirms financed payback is longer

**Total: 39 comprehensive tests across 10 categories**

## Manual Testing Checklist

While automated tests cover calculations, some features require manual verification:

### Visual/UX Tests
- [ ] System cards (panel-only and battery) are clickable
- [ ] Selected system is highlighted
- [ ] All charts render correctly
- [ ] Responsive design works on mobile/tablet
- [ ] Tooltips display properly
- [ ] Form validation shows error messages

### Functional Tests
1. **Power Bills Only** (No Financial Data)
   - [ ] Enter summer bill: $300, winter bill: $200
   - [ ] Calculator shows panel-only and battery systems
   - [ ] No financial metrics displayed
   - [ ] Webhook doesn't send financial data

2. **With Financial Data**
   - [ ] Enter all financial fields
   - [ ] Monthly Serviceability section appears
   - [ ] Borrowing Power displays (High/Medium/Low)
   - [ ] LVR calculates correctly

3. **With Financing**
   - [ ] Enter loan term and interest rate
   - [ ] Loan Financing Details section appears
   - [ ] Monthly payment displays in both sections
   - [ ] Payments match exactly
   - [ ] Monthly surplus updates correctly

4. **System Switching**
   - [ ] Click between panel-only and battery systems
   - [ ] All metrics update (cost, savings, payback)
   - [ ] Monthly loan payment updates
   - [ ] Monthly surplus updates
   - [ ] Charts update

5. **GTM Tracking**
   - [ ] Open browser DevTools → Console
   - [ ] Enter data and generate results
   - [ ] Check `dataLayer` for `calculator_results_generated` event
   - [ ] Verify event includes solar data (system kW, costs, etc.)
   - [ ] Verify NO personal financial data in event

### Edge Cases
- [ ] Zero interest rate (0%)
- [ ] Very high interest rate (15%+)
- [ ] Very small power bills ($50/month)
- [ ] Very large power bills ($1000/month)
- [ ] No financial data provided
- [ ] Partial financial data (should ignore)

## Current Test Status

### ✅ Implemented
- Test framework and UI
- Test structure for all categories
- Monthly payment calculation helper
- Generation formula tests
- Financing calculation tests

### 🚧 In Progress
- Calculator iframe integration
- Input value injection
- Result extraction
- Comparison logic

### 📋 Planned
- Screenshot comparison (visual regression)
- Performance benchmarking
- Cross-browser testing
- Mobile device testing

## Test Development

### Adding New Tests

Add tests to the `TEST_SUITES` array in `test-calculator.html`:

```javascript
{
    name: 'Test Suite Name',
    tests: [
        {
            name: 'Test case description',
            inputs: {
                summerPowerBill: 300,
                winterPowerBill: 200,
                // ... other inputs
            },
            expectedValue: 123.45,
            tolerance: 0.1
        }
    ]
}
```

### Test Input Fields

Available input fields to test:
- `summerPowerBill` - Summer power bill ($)
- `winterPowerBill` - Winter power bill ($)
- `summerKwh` - Summer kWh usage
- `winterKwh` - Winter kWh usage
- `fixedDailyCharge` - Fixed daily charge ($)
- `householdIncome` - Annual household income ($)
- `propertyValue` - Property value ($)
- `currentLending` - Current lending/mortgage ($)
- `expenseAmount` - Expense amount ($)
- `expenseFrequency` - 'weekly', 'fortnightly', or 'monthly'
- `loanTerm` - Loan term (years)
- `interestRate` - Annual interest rate (%)

### Expected Output Fields

Fields you can validate:
- `panel_only_system_kw` - Panel-only system size (kW)
- `panel_only_system_cost` - Panel-only system cost ($)
- `panel_only_monthly_savings` - Panel-only monthly savings ($)
- `panel_only_payback_years` - Panel-only payback period (years)
- `battery_system_kw` - Battery system size (kW)
- `battery_system_cost` - Battery system cost ($)
- `battery_monthly_savings` - Battery monthly savings ($)
- `battery_payback_years` - Battery payback period (years)
- `monthlyLoanPayment` - Monthly loan payment ($)
- `monthlySurplus` - Monthly surplus after all expenses ($)
- `lvr` - Loan-to-value ratio (%)
- `borrowingPower` - 'high', 'medium', or 'low'

## Known Issues & Limitations

1. **Iframe Testing Complexity**
   - Current tests use mock calculations
   - Full calculator integration requires more setup
   - Consider using Puppeteer/Playwright for full automation

2. **Timing Issues**
   - Calculator uses 2-second debounce for calculations
   - Tests need to wait for results
   - Add appropriate timeouts

3. **Browser Compatibility**
   - Tests require modern browser (ES6+)
   - Chart.js requires canvas support
   - GTM requires JavaScript enabled

## Continuous Integration

For CI/CD integration, consider:

1. **Puppeteer Setup**
   ```javascript
   const puppeteer = require('puppeteer');

   async function testCalculator() {
       const browser = await puppeteer.launch();
       const page = await browser.newPage();
       await page.goto('http://localhost:8000/index.html');

       // Set input values
       await page.type('#summerPowerBill', '300');
       await page.type('#winterPowerBill', '200');

       // Wait for calculation
       await page.waitForTimeout(2500);

       // Extract results
       const result = await page.evaluate(() => {
           // Extract values from DOM
       });

       await browser.close();
   }
   ```

2. **GitHub Actions Workflow**
   ```yaml
   name: Test Calculator
   on: [push, pull_request]
   jobs:
     test:
       runs-on: ubuntu-latest
       steps:
         - uses: actions/checkout@v2
         - uses: actions/setup-node@v2
         - run: npm install puppeteer
         - run: node test-script.js
   ```

## Regression Testing Best Practices

1. **Run tests before committing**
   - Ensure all tests pass
   - Review failed tests carefully
   - Update tests if behavior intentionally changed

2. **Add tests for bug fixes**
   - When fixing a bug, add a test that would have caught it
   - Prevents regression of the same bug

3. **Keep tests maintainable**
   - Use clear, descriptive test names
   - Document expected values
   - Keep tolerance values reasonable

4. **Test edge cases**
   - Boundary values (0, very large numbers)
   - Invalid inputs
   - Missing data

## Support

For questions or issues with testing:
1. Check this documentation
2. Review test-calculator.html source code
3. Check browser console for errors
4. Verify calculator works manually first
