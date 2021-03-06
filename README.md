NOTE: I am no longer developping sec_edgar.  For a much cleaner (more comprehensive test suite, 
better modeled) replacement, based on XBRL reports (rather than hand-parsing raw text submissions),
check out <https://github.com/jimlindstrom/FinModeling>.

## Overview

sec_edgar is a gem that can download financial statements from the SEC/Edgar 
database, parse them, perform valuations, and write the results to CSV.  It performs well
for many NASDAQ tech companies, but has not been widely tested beyond that sector.

### See Also

For an example of using this gem to perform valuations, see: <https://github.com/jimlindstrom/stock_screener>

### Example

The following example retrieves all of Google's annual reports (10k's).  It
then reformulates them to highlight the business's operating fundamentals.
Based on the reformulated statements, it performs a two-stage valuation, 
based on N years of forecast data and then a long-term growth rate.

     require 'sec_edgar'
     
     # set up the valuation with forecast data
     ticker = 'GOOG'
     beta = 1.15
     forecast_data = 
       [ { :revenue_growth => 0.25, :sales_pm => 0.20, :fi_over_nfa => 0.01, :ato => 1.50 },
         { :revenue_growth => 0.20, :sales_pm => 0.20, :fi_over_nfa => 0.01, :ato => 1.20 },
         { :revenue_growth => 0.15, :sales_pm => 0.20, :fi_over_nfa => 0.01, :ato => 1.10 },
         { :revenue_growth => 0.15, :sales_pm => 0.20, :fi_over_nfa => 0.01, :ato => 1.00 },
         { :revenue_growth => 0.15, :sales_pm => 0.17, :fi_over_nfa => 0.01, :ato => 1.00 },
         { :revenue_growth => 0.10, :sales_pm => 0.16, :fi_over_nfa => 0.01, :ato => 0.90 },
         { :revenue_growth => 0.04, :sales_pm => 0.15, :fi_over_nfa => 0.01, :ato => 0.90 } ]
     
     
     # get the financial summary (from all historical 10k's)
     summary = SecEdgar::Helpers.get_all_10ks(ticker)
     
     # calculate cost of capital
     tax_rate = 0.35
     rho_e = 1.0 + SecEdgar::Helpers.equity_cost_of_capital__capm(SecEdgar::Helpers::RISK_FREE_RATE, 
                                                                  SecEdgar::Helpers::EXPECTED_RETURN_FOR_EQUITIES, 
                                                                  beta)
     rho_d = 1.04 # decent average to work with
     rho_f = SecEdgar::Helpers.weighted_avg_cost_of_capital(ticker, summary, rho_e, rho_d, tax_rate)
     
     # perform valuation
     shares_outstanding = SecEdgar::Helpers.get_shares_outstanding(ticker) 
     v_cse_share_0 = summary.sf2_valuation(forecast_data, rho_f, shares_outstanding / 1000.0)
     
     puts "shares outstanding: #{shares_outstanding}"
     puts "             rho_f: #{rho_f}"
     puts "   value per share: #{v_cse_share_0}"
     summary.to_csv("summary.csv")
