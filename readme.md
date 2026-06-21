# Green Baseload Contracts: do they still work?
Analysng whether green baseload power purchase agreements still make sense today.

The regular disclaimer around my power markets terminology applies here. Please refer to my earlier work on power markets modelling in case some of the terms are confusing, although this project is a little easier to grasp.

# 1.Introduction

When I started working in the renewables space back in 2019 as a summer intern in the Green Investment Group, renewable energy assets (solar, onshore wind, offshore wind) were contracted in a few ways:
1. Government-backed contracts (FiTs, CfDs, etc.). These usually, but not always, covered the entirety of the volumes of energy produced by an asset for 10-20 years.
2. Private power purchase agreements (PPAs) covering 50-100% of energy volumes, with the remainder sold on power markets as "merchant volumes". Asset operators would contract these PPAs with a variety of counterparties, ranging from industrials (eg. Norsk Hydro) to energy brokers (Axpo), for a variety of tenors (3-30 years).
3. Merchant only daredevils. Not for the faint of heart, usually smaller projects and not very bankable with project financiers. I never worked on merchant-only renewables projects.

At the time, PPAs came in two flavours, pay-as-produced and baseload. The former is self-explanatory, the asset operator gets paid for every unit of energy they produce regardless of when, very simple for the operator but can be quite inconvenient for the recipient if they can't use the energy (eg. if wind blows at night). The latter is far more suited to industrials who need a constant and consistent flow of energy, but is harder to structure for an intermittent asset.

As you can imagine, baseload PPAs were far more popular with the largest and most bankable offtakers (eg. Microsoft, Siemens), and so renewables operators like GIG structured PPAs to meet that blue-chip demand. In doing so they were able to point to low-risk cash flows from top credit rated companies, and so raise significant amounts of bank debt to build projects.

Without going into too much detail, baseload PPAs are structured as follows:
- A pre-determined amount of capacity from a wind or solar farm is earmarked for the PPA (say 60% for 10 years). A 100MW solar farm may produce 120,000MWh a year of electricity, so the PPA would cover 72,000MWh a year. That volume, spread out over a year, is about the same as a continuous 8.2MW (72,000MWh / 8,760 hours in a year) source operating as a baseload plant.
- A power price (say €60/MWh) is agreed for the PPA volumes.
- When the solar farm produces above 8.2MW, it sells 8.2MW to the PPA offtaker, with the excess sold on the power markets. When it produces less than 8.2MW, it has to buy back the shortfall from the power markets to fulfil the PPA.
- A further layer of optimisation can occur where an asset operator engages in a "swap", exchanging volumes, say, in summer where the plant will exceed 8.2MW regularly, for volumes in winter where it will likely suffer shortfalls. This can be done by an energy trader or utility, but comes at a cost, and that cost has increased markedly in the last few years as summer prices are typically much lower than in winter (so why would you swap them?).

In the pre-renewables power market, where a single fossil or nuclear technology set a reasonably stable power price, one could make the assumption that the cost of PPA shortfalls could be met by the extra revenues from the surplus. But as solar has depressed midday prices (see my repository on cannibalisation), the value of surplus midday solar has plummeted, while shortfalls are typically at times of expensive fossil generation. **My thesis: as renewables begin to saturate the grid, baseload PPAs are actively detrimental to renewable assets.**

# 2. Project aims

Project aims:
- Using publically available generation and market price data from ENTSO-E, determine the value to a solar plant of a baseload PPA.
- I will trial a range of a baseload PPA contract %s of total asset generation (0-100%, in 20% bins). The 0% scenario is of course the "capture price" of solar, how much a solar asset makes if it sold all of its energy on the day-ahead electricity market.
- Without pre-empting the result too much, applying a form of discount rate to contracted (baseload PPA) vs uncontracted revenues, to determine which of the 0-100% scenarios have the highest net present value.

NOTE: the data for the project that I will download and manipulate is on a fleet-level basis. This data benefits from aggregation effects, and disguises the increased volatility that comes from a single renewables asset. 

# 3. Data sources

**Data source:** This project uses data from the ENTSO-E Transparency Platform. To replicate the code you need to create a login and an individual API key.

**Python API client:** THe ENTSO-E API is difficult to use so I have used a Python client for the API, developed by EnergieID: https://github.com/EnergieID/entsoe-py. The entsoe-py package is MIT licensed.

**Geographies and technology**: I really do love solar, but here I pick on it again, and in its largest European market, Germany.

**Data pulls:**
- Actual solar generation for Germany 2016-2025, divided into 15-min bins.
- Day-ahead prices for Germany 2016-2025, divided into 15-min bins.

# 4. Methodology

The code runs through the following logic:
- For German solar, pulls the actual generation and DA prices. Note the code is quite flexible as to country and generation technology. It would be easy to replicate for any technology chosen eg. wind
- The code takes the fleet-level solar generation, and normalises against the maximum generation that year (proxying as fleet capacity in MW). This creates a generation profile (0-1)
- Using this profile, and with a fixed scenario input of asset MW, and asset MWh (the yield), this creates an asset generation profile for a dummy asset.
- Using a further assumption on PPA coverage (how much the PPA covers the total asset yield), the code calculates the baseload PPA level (in MW).
- Any production above this PPA level by the asset is then sold at the DA price, any shortfall is bought back at the DA price. The entire PPA volume is sold at an assumed price (eg. €60/MWh).
- The final revenue for the asset is computed as final revenue = PPA revenue - cost of shortfall + revenue from surplus. This is then divided by total production to find a result in €/MWh.

**My assumptions:** 100MW asset, 1200MWh/MW yield, €60/MWh baseload PPA price, (0-100% coverage, 20% buckets). The PPA price is high compared to the capture price (especially 2016-2020), but is anchored on LCOE and project-financeable requirements for asset built in the late 2010s. 

# 5. Results

Table extracted from excel, chart from Python script.

<img width="1632" height="474" alt="image" src="https://github.com/user-attachments/assets/5df2fa85-0136-4fbe-a4c4-47653bfc6eb7" />

<img width="790" height="490" alt="image" src="https://github.com/user-attachments/assets/09f590aa-5eae-4e6c-9ddb-1da2a124a414" />

# 6. Discussion

The results look like a game of two halves for the solar baseload PPA:
- **2016-2020**, the pre-crisis, low renewable saturation years:
    - The baseload PPA works well for solar, with 100% coverage by the PPA far exceeding the capture price of solar, which languishes in the €25-45/MWh range, incidentally below the solar LCOE for new-build solar in the late 2010s (>€50/MWh, IRENA).
    - The 100% coverage scenario (ie. the baseload PPA contract volume covers the whole production of the asset, though not strictly at the same time) operates at a small average discount (4.8%) to the contract price of €60/MWh. This means the surplus and shortfalls of the contract are balancing well.
    - The economic value in these years definitely lies in more contracting than less, with higher coverage levels receiving higher final prices per MWh.
- **2021-2025**, energy crisis and solar saturation:
  -  All of a sudden, the safe contracted option for a PPA at €60/MWh suddenly becomes the worst option for solar, with even the markets-only daredevils making more money than the asset holding a blue-chip baseload PPA. **The net revenue for the 100% coverage PPA trades at an average 44% discount to its contract price**
  -  What is happening? Slowly, the value of solar in the market is being cannibalised with every additional unit of solar power added. When our solar asset is in surplus vs. the PPA requirement, this is typically when the rest of the solar fleet is also producing, resulting in very low power prices due to zero marginal cost of solar. And vice-versa in shortfalls. The net effect is that the adjustment costs of the PPA are actively penalising the solar asset, **and the various coverage scenarios flip from being better than the capture price, to worse off.**
  -  The 100% baseload PPA option that did so well in 2016-2020 ends 2025 catastrophically, with net revenue of only €16.6/MWh, after having missed out on all the upside of having exposure to the high prices in 2022 and 2023!

Given the cascading descent of solar market value today and the high penalties of ther "shaping costs" (surplus/shortfalls), baseload PPAs for solar seem like a dying breed and a sub-optimal economic option!

# 7. Discount-rate adjusted results

However, let us examine these results from a project finance point of view. In finance, revenues from a contract with a blue-chip offtaker are rated more highly and less risky than selling electricity on the open markets. This is reflected, when calculating the net present value (NPV) of a set of revenues, with a lower "discount rate", the amount that the set of revenues is reduced by to reflect the risk.

This is a very high-level treatment of NPV, and it has more intricacies (and is often maligned, rightly so, as an inadequate tool for future value), but we shall stick to it.

If we assume a roughly 3% difference in the required rate of return between contracted (PPA) and uncontracted (sales to DA market, shaping costs) revenues, we can make an interesting calculation as to the NPV of our various scenarios.

In the results below I assume a project begins generating in 2016, and takes the 2025 results and extends them out another 20 years. **This is very unrealistic for several reasons (extending the PPA, assuming 2025 values going forwards, etc)**, but I will use it as an example.

<img width="1560" height="307" alt="image" src="https://github.com/user-attachments/assets/b067bce1-5017-4091-9397-e6a112ee0ea8" />

Despite the much lower total revenues earned by the baseload PPA over a lifetime (€94.1m) vs a merchant only strategy (€185m), **the fully contracted asset is worth more on an risk-adjusted NPV basis (€73.3m, vs €71.4m)!** This illustrates just how much the market values contracted cash flows vs riskier merchant cash flows. 

However, and very big however, these values are **very close**, and given the ample disclaimers above about how unrealistic these assumptions are, I think it highly probable that an uncontracted asset would yield superior NPV out to 2045: **The shaping costs for baseload PPAs will be more detrimental than the decreasing capture price.** 

# 8. Project limitations and improvements

The usual limitations apply here:
- The data used is fleet-level, and does not capture intra-asset variability.
- The PPA price used here is on the high-end of PPAs from the late 2010s (though not low since 2022), but aligns with the required LCOE for late 2010s solar. The conclusions of this analysis break down if unrealistic PPA prices are used (eg. a €150/MWh baseload PPA for solar is always the optimal scenario, and vice versa for a €10/MWh baseload PPA). The conclusions (that baseload PPAs will net severe discounts to their contract price from 2021) hold in the €20-80/MWh range, which I am confident contains most of the PPAs of this type. I append the €20/MWh and €80/MWh scenarios for reference.

<img width="790" height="490" alt="image" src="https://github.com/user-attachments/assets/1cd2ace9-0cb6-4a55-b16e-de267d62357a" />

<img width="790" height="490" alt="image" src="https://github.com/user-attachments/assets/d81f3739-65d3-4479-ba84-b7e935fa3c32" />

- Baseload PPAs, as mentioned above, typically have some form of volume swap element to reduce the magnitude of shaping costs. Whilst the PPA exhibited here has been in use, smarter operators would look to hedge out the shaping costs as much as possible. Nevertheless, these swaps are bound to become much more expensive, as the provider would become a net loser by swapping generation volumes in summer (cheap) for winter (expensive). The analysis therefore still stands directionally.
- Once again I have used maximum observed generation in the year as a proxy for fleet size. But given the normalisation, I do not think this is a problem.
- I would look to repeat this analysis for different countries and technologies. A few countries in Europe have not suffered the cratering of sunny hour power prices (yet), and may yet have durable solar baseload PPA economics. Wind power is slightly less correlated than solar, but high-wind countries are already experiencing high power price cannibalisation.
- Repeating here the disclaimers around the discount rate calculation, it serves as an illustration of the power of discounting, as opposed to a justification for the NPV of baseload PPAs.
