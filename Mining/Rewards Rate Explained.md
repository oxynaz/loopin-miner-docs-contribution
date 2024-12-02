##### Rewards Rate Explained

The reward system is based on four key proofs: PoT (Proof of Time), PoL (Proof of Loyalty), PoP (Proof of Privacy), and PoW (Proof of Work). 

These factors combine to create a PoCPS ratio, which determines your daily rewards. Each proof has specific criteria that influence the reward potential, with a maximum value of 1 for optimal performance. 

**PoT**= proof of time, miner start with 10% of PoT ratio, each day the % increase until 30 days, at 30 days online + LP status your PoT ratio will be 1 = the max value (have a look here: https://loopinrewards.ddns.net/)

**PoL**= proof of loyalty 

<u>PoL calculation:</u>
PoL_ratio = (LooPIN_held_in_the_address) / (minted_LooPIN - staked_LooPIN)
Explanation: This calculates the ratio between the amount of LooPIN held in the address and the difference between minted and staked LooPIN.

PoL = exp(β * (PoL_ratio - 1))

Explanation: The formula uses an exponential function based on the PoL ratio, adjusted by a constant β, to calculate PoL.
β = 7.324
Explanation: β is a predefined constant that affects the shape of the exponential PoL curve.

**PoP** = proof of Privacy, we’ve set all machines to 1 

**PoW** (Proof of Work) is currently based on the card's performance compared to a benchmark hash rate. Best ratio value is 1. 

 

**PoCPS** = PoT x PoL x PoP x PoW 

Your estimated Daily Rewards = PoCPS ratio * card daily reward (can be found in overview page of the loopin website and here:  https://loopinrewards.ddns.net/) * number of cards into the rig