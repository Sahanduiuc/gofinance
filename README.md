Gofinance
=========

Financial information retrieval and munging. Planning to use Yahoo
Finance solely at first. Written in go, based in parts on [richie
rich](https://github.com/aantix/richie_rich) and

Todo ====
- Try to get data from Bloomberg as well: http://www.openbloomberg.com/
  (reduce dependency on Yahoo Finance)
- Persist data locally to avoid overloading the servers and getting
  blocked
- Calculate dividend yield based on the price you actually paid (in
  aggregate) for stock you own. Why is this handy? For example: suppose
  you paid $20 for a stock that pays out $1 in dividends each year. It's
  dividend yield would be 5% (which is very good). If that stock rises
  to $40 the next year and the dividend payouts stay the same, the
  dividend yield would be 2.5% (which is still good but only half of the
  last time). However, this is not the yield you will be getting if you
  already own the stock (and bought it at $20). The yield would still be
  5%, obviously. So, the yield as it stands is a good measure of what
  you're buying, but not of what you already bought.
- Calculate effective yields and tax drag. This is hard to do (collect
  all the info), but once done it would be wonderful to determine how
  much a fund/stock/ETF is really going to net you. An example: the US
  levies a 30% tax on dividends as far as I know. This can be reduced to
  15% if you or your broker make use of a double-tax treaty. Belgium has
  a 25% dividend tax which is (as of 2014) wholly non-negotiable, you
  always pay it, no matter where else you already paid taxes. In the
  Netherlands it is always 15%. In Ireland it appears to be 20% but I
  don't know if that counts for foreign companies who have their
  "domicile" in Ireland. It's all very confusing. At any rate, the most
  rosy tax climate I could personally get is first 15% (US) then 25%
  (BE). I'm too much of a realist to assume I will get that, but one has
  to take _a_ value. So to calculate the effective yield I would use
  this formula: `yield * tax in source country * tax in receiving
  country`. In the case of for example a company with a dividend yield
  of 2.5%, that would become an effective yield of `2.5% * 0.85 * 0.75 =
  1.6%`. Which is suddenly a whole lot less great. As such, one realizes
  it takes quite a bit more dividend yield to beat a bank account by a
  nice margin. In a slightly worse case, I would also have to pay taxes
  in Holland, as that's where I usually buy shares (on the Amsterdam
  Euronext), so that would become `2.5% * 0.85 * 0.85 * 0.75 = 1.35%`.
  And that's how you hit rock bottom, even with an initially nice
  dividend yield. It gets even worse, if you want to see how your
  purchasing power will evolve, you have to take inflation into account.
  So let's do that. The average inflation in Belgium for 2013 was 1.11%
  (1.43% for Germany). That means that €1 in 2012 is worth `€1 / 1.0111
  = 0.989` in 2013. And so the final formula becomes `2.5% * 0.85 * 0.85
  * 0.75 - 1.11% = 0.24%`. And so, there's nothing left. Note that the
  levied taxes are massively important here. In the optimal case, where
  only Belgium and the US are paid (at the reduced US rate of 15%), we
  get an effective rate of 0.48%, which is not good but already double
  the other one.
- Dividend payout history (and other indicators) analysis. Even if the
  yield is good, it's possible that the company has just had a really
  bad run, and thus its share price is really low. This _might_ mean
  that it's going to be less succesful (bad), or that the market is just
  in a slump (good). So companies that are just doing bad are to be
  avoiding, more so because they are unlikely to keep up their high
  dividend payments if they are on their way to the poor house. Some
  extra parameters gofinance could use to provide tips:
  - historical dividends (are they growing, for how long?)
  - strong growth and earnings
  - no heavy drops in share price (we can disentangle the "losing
    company" case from the "bear market" case by comparing with the
    general direction of an index, be the index as representative as
    possible).
  Put shortly, fundamentals. One needs to make sure that the dividend is
  not going to be slashed and send the yield to kingdom come.

Technical
=========

Yahoo Finance
-------------

There are broadly speaking 2 ways to easily get at the Yahoo Finance
data:

1. Query via YQL (Yahoo Query Language), a SQL lookalike. This happens
   through  a HTTP GET request. The response can be XML or JSON, depending
   on the GET parameters.
2. Request a CSV file, also with a HTTP GET request.

The first approach is a bit more high-level, and actually queries the
CSV file of method (2) under the hood. You'd think that requesting the
CSV file would be faster, since it skips a step, but this appears to be
false according to my testing. This could be for two reasons:

1. The YQL server has preferential access to Yahoo's own APIs
2. The YQL server caches the CSV file somehow (and/or is aware when it
   gets refreshed).

At any rate, **gofinance** implements both methods, and performs
requests in parallel. By default the YQL way is used, although it's easy
to switch.

There seems to be a third kind of API, possibly related with the CSV
one, possibly not, it's "described"
[here](http://www.quantshare.com/sa-426-6-ways-to-download-free-intraday-and-tick-data-for-the-us-stock-market),
the same site also lists some other interesting sources. Worth a look.
