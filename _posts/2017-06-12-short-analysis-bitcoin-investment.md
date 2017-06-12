---
layout: post
title: "A short analysis on Bitcoin investment"
comments: true
keywords: "bitcoin, analysis, investment"
---

> I am not a regular writer. I haven't wrote a blog post in more than
> 2 years. This article is purely aimed at providing some useful insight
> in terms of BTC investment, and should not serve as a judge of my
> writing skills. :)

This is part 1 in a series of blog posts on discovering better
investment strategies in cryptocurrencies.

We will be using [Poloniex](http://poloniex.com) for our data feeds in
this series. The reason for this selection is the large volume of trades
on Poloniex, as well as the number of cryptocurrency pairs being traded
on this exchange. Having a large pool of cryptocurrency pairs will help
us refine our strategies in a much better way.

One issue with using Poloniex as our source is that we will be dealing
in `USDT` instead of Fiat currencies like `USD`, and the OHLC data for
a particular cryptocurrency pair is limited by when it started trading
on Poloniex.

> I am an experienced Ruby developer. I started fiddling with Python for
> the awesome Pandas library it provides. I am amazed. Though, I am
> still in the learning curve, and therefore, I would prefer PRs for
> code presented here, rather than being flamed for it. ;)

## Initial Setup
### Importing Bitcoin OHLC data

```python
start = datetime.now() - timedelta(days=10000)
dfBTC = polo.returnChartData(currencyPair="USDT_BTC",
                             start=int(start.timestamp()), period=86400)
dfBTC = pd.DataFrame().from_dict(dfBTC)
dfBTC['time']=pd.to_datetime(dfBTC['date'].astype(int), unit='s')
dfBTC.drop("date", axis=1, inplace=True)
dfBTC.set_index("time", inplace=True)
dfBTC[list(dfBTC)] = dfBTC[list(dfBTC)].apply(pd.to_numeric)
dfBTC = dfBTC[:-1]
dfBTC.head()
```

![BTC OHLC Data](https://www.dropbox.com/s/eys5hx0otencoc8/Screenshot%202017-06-12%2022.24.06.png?dl=1)

### Function to quickly visualize the data

```python
def plot_indicator(df, keys=[], names=[], line=None, sma=0.8, price=1):
    colors="gbmc"
    if not names: names = keys

    fig = plt.figure(figsize=(10,8))
    ax = fig.add_subplot(111)

    # We are mainly, interested in SMAs of interesting data.
    for key in keys:
        df['%sSMA' % key] = df[key].rolling(window=30).mean()
    df['priceSMA'] = df['close'].rolling(window=30).mean()
    
    # This helps us fade away primary data in comparison to its SMA, or
    # vice versa.
    for idx, key in enumerate(keys):
        if sma < 1: ax.plot(df[key], color=colors[idx], label=names[idx],
                            alpha=(1-sma if sma else 1))
        if sma > 0: ax.plot(df['%sSMA'%key], color=colors[idx],
                            label="SMA(%s, 30)"%names[idx])
    
    # Plot BTC price movements, if required.
    if price:
        bx = ax.twinx()
        bx.plot(df['close'], color="r", label="price", alpha=0.2)
        bx.plot(df['priceSMA'], color="r", label="SMA(price, 30)")
        bx.legend(loc=1)
    
    # Draw a line for use cases, such as base case results, etc.
    if line is not None: ax.axhline(y=line, color="k", ls="--")
    ax.legend(loc=2)

    plt.show()
    return ax
```

## Scenario 1

> What happens if we invested a one-time fixed amount of money in BTC in
> the past? How much it will grow at this moment? Was it better to
> invest in 2015 or 2016? Are the returns on my investment better now,
> or I missed the opportunity?

Well, most of us cryptocurrency enthusiasts know the answers to atleast
some of those questions, but do we know the quantitatively? This current
scenario will be focussing on answering these questions.

### Growth in investment till date

```python
df = dfBTC.copy()
df['growth'] = df.ix[-1].close/df['open']
plot_indicator(df, ["growth"], line=1, sma=0);
```

The above code does a very simple thing - it calculates the overall
growth till the current time (to be precise, yesterday). When I talk
about `growth`, I am referring to how much a unit investment of money is
worth at a given instant. So, a growth of `2.5` means that an investment
of `$100` is now `$250`, netting in a profit of `150%`, i.e. `profit
= (growth - 1)*100` for the geeks.

![Fixed Investment Growth](https://www.dropbox.com/s/xawi6g7bhgpt9oc/Screenshot%202017-06-12%2022.38.31.png?dl=1)

We can see that the graph is in a sort-of exponentially inverse
proportionate to the price of BTC at a moment. Minor changes in price in
distant-past greatly affect the growth, while major changes in near-past
have less effect on the growth. This is obvious, as the longer the money
has remained invested in the more it starts earning `interest` over it.
This `interest` is compounded with time (as we assume BTC price is
exponential as well), and hence, slight variations in the past have
dramatic effects on the growth.

Also, the growth decreases as the investment time is shortened. This is
obvious when compared with the increase in BTC price, and needs no
explaination further.

### Average ROI till date

To understand this better, and eliminate time-factor (compounding) issue
from this data, we can try to measure the average growth per day, per
week or per month. This way, we can get an unbiased view of the real
growth of invested money at a given time, regardless of how long it has
been invested for. O'course, this isn't exactly accurate, as near the
present time, money has been invested in for a very brief period and
hence, averaging the growth doesn't really work. But, well lets go ahead
and give this a try:

```python
df = dfBTC.copy()
df['growth'] = df.ix[-1].close/df['open']
df['duration'] = (df.index[-1]-df.index).days + 1
df['roid'] = ((df['growth']) ** (30/df['duration']))*100 - 100
plot_indicator(df, ["roid"], names=["30d ROI (%incr)"], line=0, sma=1);
```

We calculate the 30day ROI of our investment here. By `ROI`, I mean the
average profits reaped on an investment in a given time. Therefore, an
`roid` value of `5` here means that from the point of investment till
date, the investment made a consistent `5%` profits every 30 days,
compounded. Therefore, in about 180 days, the investment will make
a profit of `34%` nearly, which is same as a growth factor of `1.34` in
180 days.

![30day ROI](https://www.dropbox.com/s/h72mllk9vxrlpum/Screenshot%202017-06-13%2000.09.10.png?dl=1)

We can observe that as the BTC price increases (or we approach present),
the ROI value starts increasing as well, which is consistent to the rate
of increase of BTC price. Ignoring the present moment, an investment
made in July, 2015 will have netted an ROI of `10%`, whereas one made in
April, 2017 will have earned a whooping `40%` profit every 30 days.

### Growth after fixed duration

However, both these graphs don't give out the complete picture. They
still have a time value attached to them in a way, esp. for the present
values of growth. We can try to frame a better picture by visualizing
how much growth an investment made in the next `n` days, say 90 days.

```python
df=dfBTC.copy()
df['growth30']  = df.shift(1-30).close/df['open']
df['growth90']  = df.shift(1-90).close/df['open']
df['growth180'] = df.shift(1-180).close/df['open']
df['growth365'] = df.shift(1-365).close/df['open']
plot_indicator(df, ["growth30", "growth90", "growth180", "growth365"], line=1, sma=0.9, price=0);
```

![Growth Fixed Duration](https://www.dropbox.com/s/9ydriinj8mjkn8p/Screenshot%202017-06-13%2000.19.23.png?dl=1)

Now, this gives out a good picture of our investment. It says that as
the investment time increases, we see better profits (again, obvious),
but the good thing it tells us is that, our investment grows more now
that it did in the past. While the growth in 180 days has been
consistently in the range of `1x-2x` our investment in the period May,
15 to Nov, 16, the growth in the investment has exceeded `2x` after Nov,
2016.

## Epilogue

> Now, if you are really here, I would like a moment to tell you that
> there is not a great deal of statistical models, I could have built
> with BTC price data alone. But, future posts will have far advanced
> analysis on such topics. Correlations, Buy and hold analysis for
> altcoins, and so on. Please, bear with me for a while :)

I will follow up with another post (**Scenario 2**) that compares our
investment in BTC, if we were investing a fixed amount of money every
few days/weeks/months. O'course this will be lesser than a direct fixed
amount investment of comparable size, but we are mostly interested in
the numbers we could get by such an investment. Lets, say we started
investing `$1` each day since January 01, 2016 till date. Did you know,
it would be approx. worth equal to an investment of around `$7` each
day. Go figure!
