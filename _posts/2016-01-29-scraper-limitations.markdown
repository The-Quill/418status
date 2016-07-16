---
layout: post
title:  "Scraper limitations"
comments: true
date:   2016-01-29 9:55:58 +1000
categories: code bug
---

> A bug to do with unexpected data length limitations.

In developing scraper-related tools, trying to use a generalised database can cause you to run into issues later on.

Say we're scraping _The Red Pages_, a local business directory. Let's also say that for each listing on this website, they have a different way of writing their location.

>  
>  **Bill's Fish and Chips**  
>  Location: _We're located at the end of Miller Street, between the **community centre** and **grocery store**._
>  
    <div class="location">
        Location:
        <span itemprop="locality">
            <i>
            We're located at the end of Miller Street, between the <b>community centre</b> and <b>grocery store</b>.
            </i>
        </span>
    </div>

Versus

>
> **Jack's falafels**  
> Location: 5 Miller Street, Wagga Wagga
>  
    <div class="location">
        Location:
        <span itemprop="locality">
            5 Miller Street
        </span>
        ,
        <span itemprop="suburb">
           Wagga Wagga
        </span>
    </div>

In the first, I get a mouthful of stuff that'd be better expressed with literals, like the second.

As well as the fact that `itemprop="suburb"` is missing from the first!

So I end up with:

>
>     Quill$ python scrape.py
>     
>     {"listingName": "Bill’s Fish and Chips", "address": "We're located at the end of Miller Street, between the community centre and grocery store.", "suburb": ""}
>     
>     {"listingName": "Jack's falafels", "address": "5 Miller Street", "suburb": "Wagga Wagga"}    

Tom, our faithful DBA says "_30 characters is enough for a street address, right? If suburb, postcode and country go in different fields, surely 30 is enough._"

But, Bill's address overflows at 90 characters and ruins our 2-week uptime.

"_Fine._" grumbles Jack as he tries to solve the problem.

    if (address.Length > 30)
    {
        address = address.Substring(0, 30);
    }

Jack fails to notice the issue with this, as he slumps over to the break room for his fourth coffee.

>
>     {"address": "We're located at the end of Mi"}

It chops the message!

So, while the DB accepts it, our data quality drops down to a stunning level of _Mediocre_, with a slight tinge of disappointment.

---

# Solutions

The way _I_ solved it was to use a "`DataExceeded`" column using `varchar(max)` and simply JSON serialise the exceeded data with their column names into the `DataExceeded` column, while changing the relevant columns text to `"DataExceeded"`.

This is not a perfect solution, but it gave more accuracy than having naive expectations about the dataset you're scraping.
