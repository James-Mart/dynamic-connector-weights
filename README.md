
# Bonding curve background

The Bancor equations provide generics for constructing bonding curve relationships between quantities that are useful for various applications. The primary application of such equations to date are to construct automated market makers (AMMs) for the purposes of automating liquidity provision in financial markets where there are defined relationships between a quantity of Smart Tokens, an amount of reserve tokens, and the resulting "price" used to calculate the total value or market capitalization of the smart token. The Bancor whitepaper also derived the equations necessary to easily calculate the number of reserve tokens exchanged when a number of Smart Tokens are bought from and sold into the AMM, in a form that can be computed in a smart contract without risking integer overflows.

## How Bancor Works

Geometrically, the Bancor equations parameterize a bonding curve, or power function curve, and allow you to shift the supply of Smart Tokens along the curve to calculate the resulting reserve balance and corresponding price. An example curve:

<!-- 
{\color{White} y=mx^{n} }
-->
![equation](https://latex.codecogs.com/svg.image?%5Cbg%7Bblack%7D%7B%5Ccolor%7BWhite%7D%20y=mx%5E%7Bn%7D%20%7D)

![image](/images/example-curve.png)

This curve shows a token with supply 140, and resulting price 49. This curve uses `m=1/400` and `n=2`, so the equation would be:

![equation](https://latex.codecogs.com/svg.image?%5Cbg%7Bblack%7D%7B%5Ccolor%7BWhite%7D%20y=%5Cleft(%5Cfrac%7B1%7D%7B400%7D%5Cright)x%5E%7B2%7D%7D)

To calculate the balance of pool tokens, you integrate from 0 to the smart token supply (`s`) for the area under the curve:

<!-- 
{\color{White} \int_{0}^{s}\left(m\right)x^{n} = \left ( \frac{m}{n+1} \right )s^{n+1} }
-->

![equation](https://latex.codecogs.com/svg.image?%5Cbg%7Bblack%7D%7B%5Ccolor%7BWhite%7D%20%5Cint_%7B0%7D%5E%7Bs%7D%5Cleft(m%5Cright)x%5E%7Bn%7D%20=%20%5Cleft%20(%20%5Cfrac%7Bm%7D%7Bn&plus;1%7D%20%5Cright%20)s%5E%7Bn&plus;1%7D%20%7D)

Which in our case yields 2,286.7 reserve tokens. 

![image](/images/reserve-balance.png)


The current price of the Smart Token (denominated in units of the reserve token) can be calculated with one of two formulas. The obvious formula to use is simply the formula for the curve:

<!--
{\color{White} p=ms^{n} }
 -->

![equation](https://latex.codecogs.com/svg.image?%5Cbg%7Bblack%7D%7B%5Ccolor%7BWhite%7D%20p=ms%5E%7Bn%7D%20%7D)

Where `p` = price, and `s` = supply. But, this equation has the drawback of using an exponent, which can cause integer overflow problems for curves parameterized with sufficiently large values of `s` and `n`. Therefore we can use the formula that is instead parameterized on `b`, `s`, and `n`:

<!-- 
{\color{White} p=\frac{b}{s}(n+1) }
-->

![equation](https://latex.codecogs.com/svg.image?%5Cbg%7Bblack%7D%7B%5Ccolor%7BWhite%7D%20p=%5Cfrac%7Bb%7D%7Bs%7D(n&plus;1)%20%7D)

Where `b` is the reserve balance, `s` is smart token supply, and `n` is the curve exponent. Each of these equations would yield the same current price of `p=49`, meaning the price is 49 reserve tokens per smart token. 

In a bond curve AMM like this, the "price" is only used for calculating current marketcap. Unlike in traditional order-book exchanges where a all purchases come from existing tokens being offered for sale by another owner, buying and selling from a bond curve AMM results in smart token supply expansion and contraction. That is the property that allows AMMs to truly serve as "automatic" market makers and automate liquidity provision. But it also means that if one wants to purchase `k` smart tokens from the AMM, then it is *not* correct to simply multiply the aforementioned price (49) by the number of desired tokens. This is because each infinitesimally small token purchase would change the supply, and therefore change the price according to the curve formula. Therefore, two equations have been derived to allow for important calculations related to purchase/sale of smart tokens.

First, the equation that calculates the actual price `p` for purchasing `k` tokens:

<!--
{\color{White} p=b((\frac{k}{s} + 1)^\frac{1}{r} - 1) } 
-->

![equation](https://latex.codecogs.com/svg.image?%5Cbg%7Bblack%7D%7B%5Ccolor%7BWhite%7D%20p=b((%5Cfrac%7Bk%7D%7Bs%7D%20&plus;%201)%5E%5Cfrac%7B1%7D%7Br%7D%20-%201)%20%7D)

And second, the equation that calculates the number of tokens minted and recieved after sending a deposit `p` to the AMM.

<!--
{\color{White} k=s((\frac{p}{b} + 1)^\frac{1}{n+1} - 1) }
-->

![equation](https://latex.codecogs.com/svg.image?%5Cbg%7Bblack%7D%7B%5Ccolor%7BWhite%7D%20k=s((%5Cfrac%7Bp%7D%7Bb%7D%20&plus;%201)%5E%5Cfrac%7B1%7D%7Bn&plus;1%7D%20-%201)%20%7D)

For a geometric intuition, a purchase of `k` tokens would have a cost equal to the area under the curve (ie the amount of new reserve tokens) from `s` to `s+k`:

![image](/images/cost-of-purchase.png)

In the above image, the cost to purchase 10 new tokens, to bring the total supply from 140 to 150, is equal to the area under the curve colored blue. 

## Why bancor equations are insufficient

A fundamental constraint imposed on the curves used in the Bancor ecosystem is that there exists a constant ratio between the Smart Token's market cap (supply x unit price), and the balance of the reserve token.

However, for some use cases, it may be necessary inflate the Smart Token supply without injecting any new reserve currency (inflation), or to inject additional reserve currency, with no additional Smart Token supply expansion (subsidization).

## Inflation

To allow for smart token inflation, what we need is a mechanism to determine how to change the supply/price curve given some constraints. The constraints for inflation are that the Smart Token supply should increase sufficiently to achieve a predetermined unit-price decrease (0 < c < 1),

<!--
{\color{White} g(s+k)=c*f(s) }
-->

![equation](https://latex.codecogs.com/svg.image?%5Cbg%7Bblack%7D%7B%5Ccolor%7BWhite%7D%20g(s&plus;k)=c*f(s)%20%7D)


Also the reserve token balance should remain fixed:

<!--
{\color{White} \int_{0}^{s}m_0x^{n_0}=\int_{0}^{s+k}m_1x^{n_1} }
-->

![equation](https://latex.codecogs.com/svg.image?%5Cbg%7Bblack%7D%7B%5Ccolor%7BWhite%7D%20%5Cint_%7B0%7D%5E%7Bs%7Dm_0x%5E%7Bn_0%7D=%5Cint_%7B0%7D%5E%7Bs&plus;k%7Dm_1x%5E%7Bn_1%7D%20%7D)


Where `s` is initial supply, `k` is the additional supply through inflation, and `m` and `n` are the curve parameters. Solving the integrals yields:

In geometric terms, we're looking for the equation of the purple curve below:

![image](/images/second-curve.png)

Where the area under its curve (purple + blue region) is equal to the area under the original curve (purple + red region), and the price (blue horizontal line) is a known fraction of the original price (black horizontal line).

... To be continued

## Subsidization

Todo


## Notes

* I'm slightly concerned about the change in exponent (n) for the curve when inflation happens. If the exponent falls with inflation, then it has the effect of increasing what the Fractional Reserve Ratio (connector weight), which makes the token price less sensitive to buys/sells. The only force in the opposite direction is subsidization. So for people who never participate in governance (receive subsidies), their tokens would, over time, become increasingly stable.
    * Perhaps subsidization should just be a market buy, and "inflation" should just be a market sell from the supply of tokens initialized in the relay. This would greatly, greatly simplify the math.
* Rather than store the token supply and the reserve balance, as in Bancor, we could store the slope parameter (m), exponent (n), and current price. The balances could be transient.
* Two ways to form a relationship between people to incentivize the discovery of under-valued individuals: "investment" or "prediction market"

### Some equations

`p` = price
`m` = slope
`s` = smart token supply
`n` = exponent
`b` = area under the curve
`F` = Fractional reserve ratio
`k` = the amount of a change in smart token supply
`M` = market cap

* `p=ms^n` <-- Spot price
* `p=(b/s)(n+1)` <-- Spot price
* `p=(b/sF)` <-- Spot price
* `p=b((k/s)+1)^(n+1)-1)` <-- Price for a given change in supply
* `F=1/(n+1)`
* `b=m/(n+1)) * s^(n+1)`
* `M=sp`



# Sources

 - [Bancor Protocol](https://storage.googleapis.com/website-bancor/2018/04/01ba8253-bancor_protocol_whitepaper_en.pdf), by Eyal Hertzog, Guy Benartzi, Galia Benartzi
 - [How to make Bonding Curves for Continuous Token Models](https://blog.relevant.community/how-to-make-bonding-curves-for-continuous-token-models-3784653f8b17), by Slava Balasanov
 - [Bonding Curves in Depth: Intiution and Parameterization](https://blog.relevant.community/bonding-curves-in-depth-intuition-parametrization-d3905a681e0a), by Slava Balasanov
 - [Calculator for plotting curves](https://www.desmos.com/calculator/w0dusgn13n)
 - [Codecogs](https://www.codecogs.com/latex/eqneditor.php) used for generating links to LaTeX
 <!--
Included LaTeX in markdown by using the "URL Encoded" from codecogs.
White text and black background are also parameterized on codecogs.
-->