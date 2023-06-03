---
title: "Archived data about security incidents on the TTC"
author: jamie_duncan
date: 2023-06-03
tags: [media, data]
---

Violence on public transit has been attracting a lot of media attention in recent days. Toronto, where I live is no exception. After seeing several sensational headlines and a bunch of stories relying on top-line statistics from the Toronto Transit Commission (TTC), I found myself wondering what the TTC's data actually looks like. So I submitted a Freedom of Information Request for it. I'm used to waiting a long time so I was impressed when 50,000-ish rows of data came back in just under two months.

Oliver Moore and Sun Yang used this data for their story, ["Assaults are up on TTC, but rising ridership reduces risk"](https://www.theglobeandmail.com/canada/article-ttc-violence-rates/) in the *The Globe and Mail*. There is so much more that can be done with this data, so I have published it on the [Internet Archive](https://archive.org/details/ttc-security-incidents).

The text of my original request asked for: "any statistics available regarding the prevalence of security incidents and violent crime on TTC vehicles and within TTC stations extending as far back as 2000. Please include all relevant metadata including dates, type of incident, location, and any other relevant information associated with these records." I filed this request on a bit of a whim so it wasn't the most carefully crafted FOI I've ever drafted. I followed up to clarify with the coordinator that I wanted this data in tabular format.

The dataset released spans from 2002 to February of 2023 and includes details on the time and place of an incident, an 'offence type' and notes from the report. I think these notes would make for an interesting text-as-data project. Here is a glimpse at the data as it was released:

```
Rows: 50,534
Columns: 5
$ Date           <dttm> 2002-01-02 07:25:00, 2002-01-02 16:00:00, 2002-01-02 21:30:00, 2002-01-03 03:30:00, 2002-01-03 08:00:00, 2002-01-03 0…
$ Time           <dttm> 2002-01-02 07:25:00, 2002-01-02 16:00:00, 2002-01-02 21:30:00, 2002-01-03 03:30:00, 2002-01-03 08:00:00, 2002-01-03 0…
$ Location       <chr> "BLOOR YONGE", "* ONBOARD VEHICLES", "* ONBOARD VEHICLES", "* ONBOARD VEHICLES", "* ONBOARD VEHICLES", "* ONBOARD VEHI…
$ `Offence Type` <chr> "MISCHIEF", "ASSAULT", "MISCHIEF", "ASSAULT", "MISCHIEF", "ASSAULT", "TRESPASS", "MISCHIEF", "ASSAULT", "MISCHIEF", "A…
$ Notes          <chr> "RACIALLY MOTIVATED GRAFFITI IN BLACK MARKER.  INSIDE PANEL OF STALL DOOR ON FIRST TOILET STALL.  \"ALL BLACK MALES AR…
```

I hope that others can use this data to conduct further evidence-based analyses of the problems of violence and other security incidents on the TTC. With that in mind, I want to reflect a bit on the kind of insights this kind of data can inform, as well as some of the limitations inherent to government records like this.

1. This data was extracted from the TTC's databases, likely by an analyst using SQL queries. The quality of this data is linked to the quality of the query used to pull it. There are several thousand duplicated rows, which indicates *something* weird was happening when this data was pulled. 
2. This data is not a direct measure of crime. It is a measure of 'security incidents' as defined by the TTC and reported by TTC operators and TTC riders.
3. Relatedly, the meaning of these categories has shifted over time, alongside the likelihood of reporting certain types of incidents. For example there is an unexplained spike of tresspass reports in the early 2010's.
4. Crime is a complex social phenomenon. Data like this can contribute to our understandings but explaining the root causes of violence and finding appropriate solutions requires looking far beyond administrative data like this.

That's it. That's the data. Thanks to the folks at the Globe for their initial coverage of this, and thanks in advance to all who take up the call to continue analyzing it.



