---
layout: post
title: "A short analysis on Bitcoin investment - Part II"
comments: true
keywords: "bitcoin, analysis, investment"
---

As discussed in the [previous post](/2017/short-analysis-bitcoin-investment/),
I wanted to follow up with another quick writeup that analyzed
quantitavely how much a recurring investment in BTC would be worth at
the present moment. You can find supplementary code in the linked article.

> This is far from an analysis. I wanted to know if a recurring
> investment in BTC would have been a good strategy to go with. And,
> hence, I wanted to get a picture of that, quantitatively.

## Scenario 2: Recurring Investments

```python
df=dfBTC.copy()
df['invested'] = 1
df['duration'] = (df.index[-1]-df.index).days + 1
df['returns'] = df.ix[-1].close/df['open']*df['invested']
df['cumulative'] = df[['returns']][::-1].cumsum()[::-1]
df['growth'] = df['cumulative']/df['invested'][::-1].cumsum()[::-1]
df['fixed_growth'] = df.ix[-1].close/df['open']
plot_indicator(df, ["growth", "fixed_growth"], line=1, price=0, sma=0.9,
               names=["Growth per invested USDT (recurring)",
                      "Growth till date per USDT (fixed)"]);

df['roid'] = df['growth'] ** (30/df['duration'])*100 -100
df['froid'] = df['fixed_growth'] ** (30/df['duration'])*100 -100
plot_indicator(df, ["roid", "froid"], line=0, sma=1, price=0,
               names=["30d ROI (%incr, recurr)", "30d ROI (%incr, fixed)"]);
```

This gives us two graphs. I will focus on the first one, initially.

### Growth Factor
![Growth Value](https://www.dropbox.com/s/c63gnz97b9jh3f7/Screenshot%202017-06-13%2002.11.39.png?dl=1)

The blue line here represents a fixed price investment of equal size, while the green line is what we are interested in - recurring investments.

What it tells us is that a `$1` investment each day since April, 2015
(worth `$804`) will be approx. equal to a `$5600` investment at the
current moment. However, if we were to invest this `$804` at one go at
that time, it would be worth somewhere around `$8800` at the moment.

Adding in the interest paid by banks that we forego on a direct
investment, the recurring investment actually comes on par (worth
somewhere around `$6200-7000`) with the direct investment. Add in the
risks associated with a direct investment of `800x` size, I would place
a recurring investment as a more reliable investment here.

### 30day ROI
![ROI Value](https://www.dropbox.com/s/4h8uocgob1rk7qr/Screenshot%202017-06-13%2002.15.24.png?dl=1)

We can see that the 30day ROI keeps increasing with the BTC price (or time). Nothing much to talk about here.

## Epilogue

> OHLC data for Bitcoin alone does not allow me to run a host of
> strategies or analysis on them. In the next post, I will follow up
> with portfolio assessment for investing in altcoins in a hope to
> increase our BTC holding.
