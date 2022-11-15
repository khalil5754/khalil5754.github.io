## Using Tableau Visualizations to Analyze Canadian Foreign Direct Investment
For my first post let's stick to a topic I know *very* well and dive into...
#### A quick and dirty Tableau analysis of my thesis topic: Canada's dependency on Foreign Investment.

This is an opportunity for me to write about a topic tangentially related to my thesis that I couldn't explore in detail in my paper (dang word limits) but now I can!

###### (Data sources are all from the OECD)

<img width="1073" alt="Screen Shot 2022-06-08 at 5 40 36 PM" src="https://user-images.githubusercontent.com/44441178/195224920-3d9c05c0-fb0a-4394-97d9-3085041ed0af.png">

To preface this post: the first step of any data analysis pipeline is to gather data from reputable sources (in this case the OECD), then clean the data using your method of choice (in this case using Python) _before_ engaging in any analysis. This post will be less of a tutorial and more of a short essay guided by insights gathered through Tableau. This is a very different style than the rest of my works, but I hope you enjoy it nonetheless!

##### Without further ado...

The Canadian government’s use of FDI cannot be understated, with industries and cities seeing tremendous growth as a result of generous policy geared toward promoting and maintaining FDI. Some research regarding FDI “heavy-hitters” rank global economies based on total inward FDI flow, or total stock. This is an inaccurate empirical measurement and statistically a poor method of analyzing economies which differ so significantly in scale. Canada, for example, has a GDP 1/20th the size of its southern neighbour, the United States, and a population just 11.5% the size. A more appropriate method of comparison is the percentage of its total GDP that an economy takes in as inward FDI flow. Of the developed world, the only country which allocates a higher percentage of its GDP to FDI than Canada is the United Kingdom.

Obviously, from the above visualization it's clear Canada is one of the most highly-dependent countries in the G7. It’s also clear Canada has succeeded in attracting a significant amount of FDI to its economy, to the tune of approximately $0.72 for every dollar the country generates.

But _why_ do FMNE (Foreign Multinational Enterprises) consistently choose Canada to set-up base?

### Well, for starters: Tertiary Education


While many global economies share Canada's desire to accept as much FDI as possible and thereby grow their economy, the Canadian economy is in a unique position as one of the most coveted destinations for foreign enterprises.

To determine which country to select as a host country for FDI, enterprises conduct thorough research of potential destinations and consistently end up selecting Canada as a top destination for measured reasons.

Intuitively, attracting FMNEs begins with an educated populace because enterprises want to hire adaptable employees that can be trained and trusted to produce positive results. An OECD published study found that a highly educated populace is the single biggest attractor of FDI. Taking this into consideration, examining the top FDI destinations in the world’s adult education levels reveals Canada as the distinct leader in tertiary (post-secondary) education. 



<img width="678" alt="image" src="https://user-images.githubusercontent.com/44441178/195230123-e5bc7bb1-6ce4-4b99-aa4b-9ba2199f54f8.png">




With such a significant discrepancy in tertiary education between Canada and the second-place United States, a clearer picture begins to form as to why the seemingly small Canadian economy can go pound-for-pound (or dollar-for-dollar) with economic heavy hitters like the United Kingdom and the United States with regard to FDI intake. Canada places a heavy emphasis on the education of its citizens and reaps the rewards a highly educated populace offers its constituents.

### Next, Canadian Restrictiveness

Canadian Restrictiveness
Canadian foreign economic policy is flexible and largely balanced. However, it is not without its faults – while flexible and balanced, Canada is ranked as one of the most restrictive countries to foreign investing - how does that make sense?

OECD’s FDI Restrictiveness Index ranks countries by their acceptance of foreign investors, with a value of 1 indicating no foreign investors allowed to partake, and 0 indicating no investment restrictions whatsoever. Canada ranks number one among developed economies in overall restriction of FDI. This is surprisingly not in line with the premise of this paper: if the government is so restrictive, why is FDI still flowing in in such significant quantities? The answer partially lies in the details.


<img width="730" alt="image" src="https://user-images.githubusercontent.com/44441178/195230082-275cd877-da46-41ee-9e77-4f8c261ed1c2.png">

###### Highlighted Values in Red Indicate Restriction Coefficients Greater Than 0.1


Canada’s restriction coefficient is evidently high relative to its peers. The total is, however, certainly skewed up heavily due to Canada’s incredibly restrictive telecommunications and media industries. These are two industries that Canadian regulatory bodies consider “distinguished” Canadian sectors wherein it is vital that Canadian culture must be “protected”. 

Interesting. 

There are many more factors, such as taxes and foreign policy, that make Canada an attractive destination. However, this blog was never meant to be in-depth on my thesis topic, but I figured for my first real post ever (not counting the prologue) I should write about something I'm passionate about. This was a short post - but don't get used to it, I'm just learning how to do this. My next post might be less nerdy, but the post after that might be more nerdy - or maybe they'll get progressively nerdier. **It all averages out in the end.**



Signing off,

Khalil (TheStatsGuy)

