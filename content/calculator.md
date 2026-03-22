---
title: "CPI Calculator"
date: 2026-03-15
draft: false
showDate: false
showReadingTime: false
showPagination: false
---

I often catch myself using online tools to calculate what a given amount of money is worth in todays dollars. This must be my economics minor coming out. This was especially true when  watching the AMC original series [Mad Man](https://en.wikipedia.org/wiki/Mad_Men) and large sums would be negotiated and talked about during episodes. I would usually pause the episode to calculate how much a given amount was worth today. There are many websites that make this calculation for you, but I wanted to see if I could make one for myself. The following tools pulls the average Consumer Price Index by year since 1913 from the [Federal Reserve Bank of Minneapolis](https://www.minneapolisfed.org/about-us/monetary-policy/inflation-calculator/consumer-price-index-1913-) and adjust for inflation. The calculation is as follows:


{{< katex >}}
$$
\text{Price In Second Year} = \frac{\text{Second Year CPI}}{\text{First Year CPI}} * \text{Year 1 Amount (\$)}
$$

><small>Fun fact, In seasnon 7 episode 7, Joan’s 5% share of the agency worth around $1.5M at the time would be worth well over $14M today.</small>

{{< calculator >}}
