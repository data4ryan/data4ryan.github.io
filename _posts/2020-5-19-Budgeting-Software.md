---
layout: post
title: Budgeting with Buckets
---

Recently my wife and I decided to aggressively revist our finances.  The decision was prompted by a free online trial of [Financial Peace University](www.financialpeace.com).  We have always wanted to take part but since it is American, the events did not take place in Canada.  We wanted to learn the principles though many of the strategies do not apply here.

One challenge we had in the past while doing this is budgeting software.  If budgeting is hard, then due to human nature, it will not get done.  Budgeting is a long-term time committment so if it requires significant work, other things will get in the way.  Over the years I have used: Quicken, Mint and GnuCash.  Each for duration of at least 18 months and in the case of Mint, several years.  I also tried EveryDollar and You Need A Budget (YNAB) for shorter periods before deciding to discontinue.  Unfortunately, all of them lacked something.  So this time, I needed to really vet software because if we are really going to commit to Dave Ramey's principles, we will (hopefully) be in it for the long haul.

I tried out several options but the real missing piece is the _savings_ feature.  If budgeting works, then you are able to actually _save_ money.  The problem is the features to calculate and redistribute funds in budgeting software are limited or non-existent.  EveryDollar does this well but it does not connect to Canadian banks so I had to find another option.  I really enjoyed budgeting with Mint, and it was super easy.  However, creating a _savings_ section required to open a bank account with one of their partners and transfer money.  This did not work for us.

So after searching around I finally settled on [Buckets](https://www.budgetwithbuckets.com/) and am enjoying it so far.  There is an infinite trial period, which I enjoy because it really takes several months to get into a rhythm with budgeting.  One challenge is that my wife does not have access to the software (unless she uses my computer which rarely happens).  One drawback to Buckets is that it does not have a print option.  This is unfortunate but it does have an SQLite back-end.  This is promising.

SQLite is a very simple relational database and I have developed simple websites with SQLite back-ends before so I am very familiar with it.  I poked around a bit to get a feel of how it works and it is pretty straightforward.  Using some SQL magic I was able to produce a single query that gives a monthly budgeting snapshot.

Here is the query:
```
select
    bkttrans.bucket_id,
    bkt.name,
    cast(bkt.deposit as float)/100 as scheduled_budget,
    cast(bkttrans.total as float)/100 as this_months_deposit,
    cast(bkttrans.total-bkt.balance as float)/100 as this_months_expenses,
    cast(bkt.balance as float)/100 as balance
from (
    select
        bucket_id,
        sum(amount) as total
    from bucket_transaction
    where amount>0
    group by bucket_id
    ) bkttrans
inner join bucket bkt on bkt.id = bkttrans.bucket_id
group by bucket_id;
```
All the values are stored as integers so casting to float is required.  This query gives the budget item, regularly scheduled amount, this month's budgeted amount, this month's expenses and the item balance.  In Buckets you can set a recurring budget amount but you can also give it extra money (or less) for each month.  The deposit amount indicates this month's actual amount.  Using this information, I can make a plot of the budget status for my wife.  I have been using Buckets for only one month so I may need to alter the query to filter transactions to the current month so we'll see if it holds up.  I plan to clean up this code and put it on Github so that others can use it.  Ultimately, I would like to create a one-page summary PDF for our monthly budget meetings.  This would be very informative.
