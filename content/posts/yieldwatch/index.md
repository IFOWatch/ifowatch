---
title: 'Yieldwatch IFO: What Happened?'
date: 2021-03-08T19:27:37+10:00
weight: 2
---

The Yieldwatch IFO on PancakeSwap made history with recordbreaking participation. With the next IFO just around
the corner, IFOWatch wonders: when the limits of the IFO process are tested in such a manner,
do IFO tokens end up distributed fairly?

<!--more-->

_tl:dr; [show me the graphs](#tokens-sent-to-addresses-as-a-percent-of-the-total-number-of-tokens-sent)_

_tl:dr; [show me the code](https://gist.github.com/IFOWatch/f000feb603461286754881bdd3c27271)_

_tl:dr; [show me how to fix it](#is-there-a-better-way)_


-------------

On 3/4/2021, PancakeSwap (PCS) hosted its 7th IFO, offering 8 million Yieldwatch tokens
([$WATCH](https://bscscan.com/token/0x7a9f28eb62c791422aa23ceae1da9c847cbec9b0)) at $0.10 apiece,
for a total fundraising goal of **$800,000**. Over 10,000 eager investors pledged a sum
of money far greater, totaling **over $560,000,000** worth of CAKE-BNB tokens.

Once the dust had settled and the IFO was complete, lucky $WATCH holders sold each
$WATCH token to those who did not (or could not) participate for over $3 each--**yielding a 30x gain**.

Some might claim that those who speculated correctly were rewarded commensurately for their speculation. **But the data may tell a different story.**

------------

The amount of dollars raised ($560M) clearly exceeds the sum of $WATCH tokens sold ($800K)--over
**711x the funding goal.** So when demand so grossly exceeds supply, how are the tokens actually allocated to buyers?

PancakeSwap uses an "overflow" method that assigns tokens to IFO participants
proportional to the amount that each participant pledges in comparison to the
total amount pledged. PCS explains the overflow method with each IFO offered on
the exchange: https://pancakeswap.medium.com/yieldwatch-watch-ifo-to-be-hosted-on-pancakeswap-d24301f17241

While [some](https://twitter.com/canpak65/status/1367444361982603269)
have [claimed](https://twitter.com/wschambach1/status/1367444965824090116) that
the overflow method is "[fair](https://twitter.com/TheBishopNehru/status/1367434422555967488)," this
post posits that the vast majority of
gains were realized by "whales," or individuals who are
able to put up an amount of funds that is **significantly disproportionate** to the number of participands. All the while,
the majority of participants were left comparatively very little tokens, even with non-negligible four-figure pledges.
In effect this means that **the rich got richer** while everyone else got very little.

To verify this hypothesis, we can use the bscscan API to gather information
about the transactions that occurred during the $WATCH IFO. The code used to gather the data
used in this post is available [on github](https://gist.github.com/IFOWatch/f000feb603461286754881bdd3c27271).

------------

We can retrieve data from bscscan using the HTTP APIs published by Binance. In this post, we will use the
[python requests module]() to do so.
First we need the bscscan API URL, which we can find from the [bscscan documentation](https://bscscan.com/apis):

```python
base_url = "https://api.bscscan.com/api?module=account&action=tokentx"
```

In order to locate the data we are interested in, we need to determine the _contract_ address (this is
the address of the token we want to get information about), and the _wallet_ address where the IFO
tokens are distributed from. Using these two addresses, we can get a list of all transactions that
occurred involving both addresses. Both addresses are fairly trivially found on bscsan (which is left
as an exercise for the reader).

Along with some python looping magic, we can iterate over all known transactions that involved both addresses.
From this point we have all the data we need, and it is all about arranging it.

```python
import requests

yieldwatch_contract = "0x7a9f28eb62c791422aa23ceae1da9c847cbec9b0"
ifo_address = "0x55344b55c71ad8834c397e6e08df5195cf84fe6d"


def get_transactions_page(contract, address, start_block=0, request_size=10000):
    your_api_key = "YourApiKeyToken"
    base_url = "https://api.bscscan.com/api?module=account&action=tokentx"
    page_url = f"&contractaddress={contract}&address={address}&page=1&offset={request_size}&sort=asc&apikey={your_api_key}&startblock={start_block}"
    result = requests.get(base_url + page_url).json()
    data = result['result']
    return data
```

_Note: A large portion of graphing code hase been omitted; see the
[github gist](https://gist.github.com/IFOWatch/f000feb603461286754881bdd3c27271) for a
complete copy of the code that generates the graphs shown in this post. You can even even run it yourself
against live bscscan data!_

------------

In this post we will show two data sets:

1. The amount of tokens sent to addresses **as a percent of the total number of tokens sent**
1. The amount of tokens sent to addresses **as a percent of the total number of participating adressess**

By comparing these two data sets, we will show that the total amount of $WATCH assigned 
disproportionally benefited the participants who **already have more capital to pledge** compared
to the majority of investors.

---------

#### Tokens sent to addresses as a percent of the total number of tokens sent

![$WATCH token distribution as a share of all tokens](https://ifowatch.github.io/ifowatch/token_v2.png)

_View interactive graphs [here](/ifowatch/yieldwatch-graphs/)_

We can immediately make one very telling observation about this graph:
a total of **ten addresses** (or **0.1%** of 10,000+ participants) collected **over 25%** of $WATCH tokens.

Yet, if we visualize this data as a share of participating addresses, the story becomes even more frustrating.

-------------

#### Tokens sent to addresses as a percent of the total number of participants

![$WATCH token distribution as a share of all transactions](https://ifowatch.github.io/ifowatch/tx_v2.png)

_View interactive graphs [here](/ifowatch/yieldwatch-graphs/)_

This graph shows the same populations, as a share of total participants. Another conclusion becomes
immediately obvious: **the majority** (over 50%) of $WATCH tokens were collected by
**a tiny fraction** of total participants.

It is clear that whales are dominating in the IFO process, and leaving little room for others to make gains.

--------------

# Is there a better way?

This is the first question this author came up with upon reckoning with this data: would IFOs still be
successful if whales were prevented from monopolizing all of the gains?

We can also answer this question with code & data. If we sort the amount of $WATCH collected from smallest to largest,
we can find the exact number of participants that would have been required to meet the IFO funding goal:

```python
def identify_funding_goal_participant_index(watch_tokens_distributed, funding_goal_dollars):
    ifo_funds_raised = 0
    contributor = len(watch_tokens_distributed)
    for idx, val in enumerate(watch_tokens_distributed):
        user_pledged = val * 0.10 * 711.0
        ifo_funds_raised += user_pledged
        if ifo_funds_raised >= funding_goal_dollars:
            contributor_s = f"Found funding limit on contributor {idx},"
            pledge_s = f" who pledged ${user_pledged:,.02f}, and collected {val} $WATCH."
            raised_s = f"We have raised {ifo_funds_raised}"
            print(contributor_s, pledge_s, raised_s)
            contributor = idx
            break
    return contributor

identify_funding_goal_participant_index(watch_token_dists, 800000)

# prints
# Found funding limit on contributor 1406, who pledged $568.80, and collected 8 $WATCH. We have raised 800301.600000015
```

When executed, this code shows that the funding limit would have been reached with only the first **1,406 participants**
when arranged from smallest to largest contribution. In fact, the funding goal would have been reached without
a single contribution over **$568.** It is safe to say the IFO would have been successful **without any whales at all.**

Certainly ~$500 may be too strict of a contribution limit. But how would the situation have fared with
more permissive limits set? We can a answer this with a simple simulation:

```python
def simulate_with_individual_contributor_limit(watch_token_dists, individual_limit=10000):
    ifo_funds_raised = 0
    users_limited = 0
    for idx, val in enumerate(watch_token_dists):
        user_pledged_dollars = val * 0.10 * 711.0
        if user_pledged_dollars > individual_limit:
            user_pledged_dollars = individual_limit
            users_limited += 1
        ifo_funds_raised += user_pledged_dollars
    funding_goal_factor = ifo_funds_raised / 800000
    limit_s = f"With an individual limit of ${individual_limit:,}, {users_limited} users were limited,"
    goal_s = f"and total funds raised: ${ifo_funds_raised:,.02f} ({funding_goal_factor:,.02f}x goal)"
    print(limit_s, goal_s)
    
simulate_with_individual_contributor_limit(watch_token_dists)
simulate_with_individual_contributor_limit(watch_token_dists, 20000)
simulate_with_individual_contributor_limit(watch_token_dists, 50000)
simulate_with_individual_contributor_limit(watch_token_dists, 100000)

# prints:
# With an individual limit of $10,000, 3234 users were limited, and total funds raised: $52,699,627.20 (65.87x goal)
# With an individual limit of $20,000, 2103 users were limited, and total funds raised: $78,496,190.40 (98.12x goal)
# With an individual limit of $50,000, 1147 users were limited, and total funds raised: $123,692,556.80 (154.62x goal)
# With an individual limit of $100,000, 666 users were limited, and total funds raised: $166,990,356.00 (208.74x goal)
```

If we consider a theoretical $10,000 limit, each user contributing up to the limit would be guaranteed nearly **10x more
watch tokens** assigned than with no limits set at all. Even with a much more permissive limit of $100,000, each user
contributing under the limit would have been assigned nearly three times as many watch tokens.

We leave the judgment of the merits of such limits as an ethical exercise for the reader, but it is obvious that
the **vast majority of the community** would benefit from some contribution limit set.

### But what about bots?

If PancakeSwap were to impose a wallet cap with no additional measures, certainly whales might look towards "botting"
(or, using code to generate many wallets in order to circumvent any such limits). PancakeSwap has
[even cited](https://twitter.com/PancakeSwap/status/1367443785051824128) such a possibility as a major
deterrant from even attempting to make the IFO process more equitable. However, anti-bot
technology has existed for nearly two decades--for example, even a simple
[captcha test](https://en.wikipedia.org/wiki/CAPTCHA) would at least reduce the total
number of times a whale could physically attempt to solve the captcha challenge within the IFO time period.
It may not be a silver bullet, but instead an incremental improvement
(and one that in this writer's opinion is sorely needed).

IFO facilitators such as PancakeSwap could also look towards more high-tech & forward-thinking on-chain solutions,
such as using ID-verifying technology like [Litentry](https://www.litentry.com/) to ensure that individuals
don't participate part their fare share _without implementing KYC_. If PancakeSwap can't solve this problem,
perhaps their community would be better served by platforms that aren't afraid to tackle complicated
challenges. (This author wonders if perhaps up-and-coming exchanges like [Goose Finance](https://goosedefi.com)
may be up to the challenge?)

### Thanks for reading

If you found this post interesting, informative, or helpful, please consider tipping some BNB or BUSD to
the author's BEP20 wallet: 0x29eBe853D435C46454bc0c40DE6EF6bAd50711E7 . Donations help rationalize
the effort and time spent researching and writing posts like these.

To all readers, good luck in the next IFO, and godspeed out in the land of Defi! May you have the good sense to buy high,
and the resolve to sell never!

-- S