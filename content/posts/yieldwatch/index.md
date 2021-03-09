---
title: 'Yieldwatch IFO: What Happened?'
date: 2021-03-08T19:27:37+10:00
weight: 2
---

The Yieldwatch IFO on PancakeSwap made history with recordbreaking participation. But when the limits of IFOs are
tested in such a manner, are tokens distributed fairly?

<!--more-->


_tl:dr; [show me the graphs](#tokens-sent-to-addresses-as-a-percent-of-the-total-number-of-tokens-sent)_

_tl:dr; [show me the code](TODO: github gist)_

_tl:dr; [show me how to fix it](TODO: github gist)_


-------------

On 3/4/2021, PancakeSwap (PCS) hosted its 7th IFO, offering 8 million Yieldwatch tokens
([$WATCH](https://bscscan.com/token/0x7a9f28eb62c791422aa23ceae1da9c847cbec9b0)) for a total fundraising goal
of **$800,000**. Over 10,000 eager investors pledged a sum of money far greater: **over $560,000,000**
worth of CAKE-BNB tokens were pledged **TODO: wordeing?** in hopes of acquiring $WATCH tokens at the
IFO price of $0.10 apiece.

Once the IFO was complete and the dust had settled, the lucky $WATCH holders sold each
$WATCH token to those who did not participate for over $3 each--**yielding a 30x gain**.

Some might claim that those who speculated correctly were rewarded commensurately for their speculation. **But the data may tell a
different story.**

------------

The amount of dollars raised ($560M) clearly exceeds the sum of $WATCH tokens sold ($800K)--over
**711x the funding goal.** So when demand so grossly exceeds supply, how are the tokens actually allocated to buyers?

PancakeSwap uses an "overflow" method that assigns tokens to IFO participants
proportional to the amount that each participant pledges in comparison to the
total amount pledged. PCS explains the overflow method with each IFO offered on
the exchange: https://pancakeswap.medium.com/yieldwatch-watch-ifo-to-be-hosted-on-pancakeswap-d24301f17241

While [some]() have [claimed]() that the overflow method is "[fair]()," this post posits that the vast majority of
gains were realized by "whales," or individuals who are
able to put up an amount of funds that is **significantly disproportionate** to total amount of funds raised. All the while,
the majority of participants were left comparatively very little tokens.
In effect this means that **the rich got richer** while everyone else got very little.

To verify this hypothesis, we can use the bscscan API to gather information
about the transactions that occurred during the $WATCH IFO. The code used to gather the data
used in this post is available (on github)[TODO:link].

------------

We can retrieve data from bscscan using the HTTP APIs published by Binance. In this post, we will use the
[python requests module]() to do so.
First we need the bscscan API URL, which we can find from the [bscscan documentation](https://bscscan.com/apis):

```
code
```

In order to locate the data we are interested in, we need to determine the _contract_ address (this is
the address of the token we want to get information about), and the _wallet_ address where the IFO
tokens are distributed from. Using these two addresses, we can get a list of all transactions that
occurred involving both addresses:

```
code
```

From this point we have all the data we need, and it is all about arranging it. (See the github gist for a
complete copy of the code that generates the graphs shown in this post using live bscscan data.)

------------

We will show two data sets:

1. The amount of tokens sent to addresses **as a percent of the total number of tokens sent**
1. The amount of tokens sent to addresses **as a percent of the total number of participating adressess**

By comparing these two data sets, we will show that the total amount of $WATCH assigned 
disproportionally benefited the participants who **already have more capital to pledge** over first-time investors.

---------

#### Tokens sent to addresses as a percent of the total number of tokens sent

TODO: image of graph

We can immediately make one very telling observation about this graph:
a total of **ten addresses** (or **0.1%** of 10,000+ participants) collected **over 25%** of $WATCH tokens.

Yet, if we visualize this data as a share of participating addresses, the story becomes even more frustrating.

-------------

#### Tokens sent to addresses as a percent of the total number of participants

TODO: image of graph

This graph shows the same populations, as a share of total participants. Another piece of data becomes
immediately obvious when viewing this data: **the vast majority** of $WATCH tokens were collected by
a **tiny fraction** of total participants.

It is obvious that whales are dominating in the IFO process, and leaving little room for others to make gains.

--------------

# Is there a better way?

The first question I came up with upon reckoning with this data is: would IFOs still be successful if whales were
prevented from monopolizing all of the gains?

We can also answer this question with data. If we sort the amount of
$WATCH collected from smallest to largest, we can find the exact number of participants would have
been required to meet the IFO funding goal, along with a theoretical individual contribution cap.

```
code
```