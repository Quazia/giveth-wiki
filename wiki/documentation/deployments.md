# Giveth Deployments
 
## Giveth Fund Forwarder
 
@quazia and @griff developed a quick fix to prevent the loss of tokens donated to Giveth.

## MyEtherWallet's Giveth Campaign

MyEtherWallet is one of Giveth's biggest supporters and has volunteered to be our first Public Alpha Tester for the Giveth Platform. We’re going to use this as an example of how to 

### Donation Flow
Donor donates -> Campaign (MYD sent to Donor) -> Vault 

The Main Donation address is the `Campaign` contract which receives Ether and generates MYD. The Giver gets the newly minted MYD at a rate of 1:1. If tokens are sent to the `Campaign` they can just be escaped so token donations are never lost!

### Contract Relationships
```
                              +------------------+
                              |                  |
                              |   MiniMeToken    |
                              |                  |
                              |  0xf7e983781...  |
                              |                  |
                              +--------+---------+
                                       |
                                       |
                                       |
                                       |
+------------------+          +--------v---------+
|                  |          |                  |
|   TokenFactory   |          |   MiniMeToken    |
|                  <----------+                  |
|  0x63a5aeb18...  |          |  0x453f473b2...  |
|                  |          |                  |
+------------------+          +-------^--+-------+
                                      |  |
                                      |  |
                                      |  |
                                      |  |
                              +-------+--v-------+          +------------------+
                              |                  |          |                  |
                              |     Campaign     |          |       Vault      |
                              |                  +---------->                  |
                              |  0xa5a8ab2c6...  |          |  0x598ab825d...  |
                              |                  |          |                  |
                              +------------------+          +------------------+
```

### MYD Token Deployment Walkthrough

#### Token Factory (Deployed, everything is static)
The Token Factory allows you to clone the token to upgrade, create a vote, offer select ICO access, basically give anyone that has this token the ability to do something special… this is permissionless too ;-) anyone can do this :-D

When setting up a campaign this is the very first step since it’s not in any way linked to any campaign. Generally speaking it’s a good idea to use the newest Token Factory available.

Deployed TokenFactory -- [` 0x596964Ac32F690d34CaC020545a63754251aBaCd`](https://etherscan.io/address/0x596964Ac32F690d34CaC020545a63754251aBaCd#code)

#### MYD token Contract
For this example we’re going to show you the power of upgrading a MiniMeToken!
The initial MYD deployment is at `0xf7e983781609012307f2514f63d526d83d24f466`. This will be our parent token. In order to make our cloned token we need to interact with the MiniMeToken [at this address](https://etherscan.io/address/0xf7e983781609012307f2514f63d526d83d24f466#readContract). Unlike all of the other steps where you will be calling the constructor for this step we will call `createCloneToken`.

Alternatively you could easily deploy your own fresh MiniMeToken.

---

Here are the paremters we're using when calling `createCloneToke`.
1. _Cloned Token Name_ -- MyEtherWallet Donations Token
  
  Fairly self explanatory.

2. _Decimals_ -- 18 
  
  This is changed from 16. This means that MYD will be 1 to 1 with ether as Ether is an 18 decimal token.

3. _Symbol_ -- MYD 
  
  This remains the same. Not much to say here.

4. _Snapshot Block_ -- 4379762 
  
  You generally want to enter a snapshot that aligns with or is after the last transaction. If you set this before any transactions you’ll essentially “roll them back” and this will be reflected in the balances.

5. _Transfers Enabled_ -- false
  
  This was previously true. Disabling transfers helps allow for a token to be purely for governance and not for speculation. There are other reasons to disable this but preventing speculation is a pretty good one.

---

I set the GWEI costs to .1 because I’m cheap. Time to go get a glass of water while Ethereum deploys an ENTIRE CLONED VERSION of the MYD token for approximately the same amount that a credit card company charges for transaction fees on a cup of coffee (.07 cents).

Now that we’ve cloned the token we can get the address of the newly cloned token from the Event Logs of the [`createCloneToken` transaction](https://etherscan.io/tx/0x29cb7595d7c3d9cc743f44bed58ada98514db6643bcdb515dedf0e3b154dc7ac).

Deployed MiniMeToken -- [`0xe6e9282e453c7c1e2eea400a04ee93c4cb096beb`](https://etherscan.io/address/0xe6e9282e453c7c1e2eea400a04ee93c4cb096beb#code)

WHOA! A whole new token!??!

Looking at the token view through etherscan won’t display the token distributions at this point since there haven’t been any token transfers but you can see that the total supply is the same as well as the balances of all previous token holders. Pretty cool!

By default the creator of the new cloned token will start as the controller. We’re going to need to switch this to the campaign at some point but for the time being lets hold on to controller. 

#### Vault (Deployed, very changeable) 

Now that we’ve cloned our token lets get the Vault setup. There are currently two main vaults in the Giveth ecosystem, this deployment is for a classic vault. In the future LPVaults will play a major role in the giveth ecosystem so keep in mind this is a *classic* Vault and not a *Liquid Pledging* Vault. This is where funds are kept before they’re paid out for milestones. 

---

For our `Vault` we’ll be using the following parameters:
1. _Escape Caller_ -- “0xDdA882a62600C452419145781e45052fdC06382C”

  This is the individual who can escape token funds to the contract. As triggering payable with token transfers isn’t currently possible under ERC20, token funds are simply escaped to a multi-sig wallet when this trusted individual triggers an escape.

2. _Escape Hatch Destination_ -- “0x97B47fE3Ed8d68Ee4b930b27598d08097F8eA9C6”

  This is the individual who can escape token funds to the contract. As triggering payable with token transfers isn’t currently possible under ERC20, token funds are simply escaped to a multi-sig wallet when this trusted individual triggers an escape.

3. _Absolute Minimum TIme Lock_ -- 3600 (1 hour minimum delay that can be set)

  Once a payout has been authorized it has to wait at least one hour before it’s collected. You know for security and stuff.

4. _Time Lock_ -- 86400 (1 day delayed payment)

  Default Time Lock, pretty straight forward.

5. _Security Guard_ -- “0x839395e20bbb182fa440d08f850e6c7a8f6f0780”

  This person can delay payouts. That’s it, they just delay payments so presumably the owner can intervene in the case of a security issue.

6. _Max Security Guard Delay_ -- 1209600

  The amount of time the security guard can delay a payment.

---

**Bim** *Bam* ***Boom*** we’ve got ourselves a vault!

Deployed Vault -- [`0x598ab825d607ace3b00d8714c0a141c7ae2e6822`](https://etherscan.io/address/0x598ab825d607ace3b00d8714c0a141c7ae2e6822#code)

After the `milestoneTracker` is deployed, we will add it to the `allowedSpender[]` whitelist; this will need to be handled by owner

#### Campaign, Token Controller! 
The `Campaign` is generally used as the token controller for your `MiniMeToken`.

This is how tokens get minted.

---

Using our example of the My Ether Wallet campaign we’ll be using the following parameters:
1. _Escape Hatch Caller_ -- "0xDdA882a62600C452419145781e45052fdC06382C"
  
  This should look familiar from the vault, same deal.

2. _Escape Hatch Destination_ -- "0x97B47fE3Ed8d68Ee4b930b27598d08097F8eA9C6"
  
  All of the funds that are escaped end up in this address; generally you want to use a multisig but sometimes a trusted individual works.

3. _Start Funding Time_ -- 1484933219
  
  This is just when the campaign will start, we’ve picked an arbitrary time in the past. Time is in unix time.

4. _End Funding Time_ -- 1742699619 
  
  A lazy Sunday afternoon in March 2025 seems as good a time as any to end the campaign, hopefully MEW will have raised 1,000,000 ether by then ; ]

5. _Maximum Funding_ -- "1000000000000000000000000"
  
  Funding is expressed in WEI. The version we’re deploying is hard capped at 1,000,000 Ether in the code but this is trivial to change. We’re setting the maximum funding to the maximum possible in our code: 1,000,000 Ether or 1,000,000,000,000,000,000,000,000 wei. NO WEI!!
  
  Remember this is a #BigNumber so “put it in quotes” or the compiler will look at you funny.

6. _Vault Address_ -- "0x598ab825D607ACE3b00d8714c0A141c7aE2E6822"

  This is the vault we just deployed. Seems good.

7. _Token Address_ --  "0xe6e9282e453c7c1e2eea400a04ee93c4cb096beb"

  This is the address of the cloned token. Spiffy.

---

Eventually the owner of this contract needs to be changed to 
0xDdA882a62600C452419145781e45052fdC06382C but for now I’ll hold on to it to make sure everything is all set to go.

When this contract receives ether it creates MYD and sends it to the Giver. If tokens are sent to this address they’re handled through the escape hatch so no more worries about losing tokens forever!

Deployed Campaign -- [`0x98cdcf8cd44c216fe370cf5c420d9630ae916fc4`](https://etherscan.io/address/0x98cdcf8cd44c216fe370cf5c420d9630ae916fc4#readContract)


#### MilestoneTracker 
While the `MilestoneTracker` contract is complex deploying it is very simple!

---

To get the `MilestoneTracker` working we'll use the following parameters:
1. _arbitrator_ -- 0xDdA882a62600C452419145781e45052fdC06382C 
  
  This is the individual responsible for determining if the milestone has been achieved. For our example we’re using MEWs super secure ledger address.

2. _donor_ -- 0xb8Bed6a570cA87d03E4E386f27D14822179d10f9
  
  For this milestone the donor is going to be the regular secure ledger for MEW

3. _recipient_ -- 0x839395e20bbb182fa440d08f850e6c7a8f6f0780 
  
  This is wherever the milestones will pay out to.

---

Deployed MilestoneTracker -- [`0xb438202e27991b04be6cae51e9ca4c15ddcbac55`](https://etherscan.io/address/0xb438202e27991b04be6cae51e9ca4c15ddcbac55#code)

#### Finishing Steps
So now all of the contracts should be deployed, all that’s left is to transfer ownership of  [the Vault](https://etherscan.io/address/0x598ab825D607ACE3b00d8714c0A141c7aE2E6822#readContract) and the [GivethCampaign](https://etherscan.io/address/0x98cdcf8cd44c216fe370cf5c420d9630ae916fc4#readContract). We also need to  change the controller in the [MYD token](https://etherscan.io/address/0xe6e9282e453c7c1e2eea400a04ee93c4cb096beb#code) to the [GivethCampaign](https://etherscan.io/address/0x98cdcf8cd44c216fe370cf5c420d9630ae916fc4#readContract).



#### Main Addresses Used
1. Secure Ledger -- 0xb8Bed6a570cA87d03E4E386f27D14822179d10f9
  
  You will use this to approve milestones on main net.

2. Super Secure Ledger -- 0xDdA882a62600C452419145781e45052fdC06382C
  
  Can call the escapeHatch and take tokens and ether out at will.

3. Cold storage -- 0x97B47fE3Ed8d68Ee4b930b27598d08097F8eA9C6
  
  The escapeHatchDestination; Where tokens and ETH go when there is an issue.

#### All Deployed Contracts

1. Deployed TokenFactory -- [`0x596964Ac32F690d34CaC020545a63754251aBaCd`](https://etherscan.io/address/0x596964Ac32F690d34CaC020545a63754251aBaCd#code)

2. Deployed MiniMeToken -- [`0xe6e9282e453c7c1e2eea400a04ee93c4cb096beb`](https://etherscan.io/address/0xe6e9282e453c7c1e2eea400a04ee93c4cb096beb#code)

3. Deployed Vault -- [`0x598ab825d607ace3b00d8714c0a141c7ae2e6822`](https://etherscan.io/address/0x598ab825d607ace3b00d8714c0a141c7ae2e6822#code)

4. Deployed Campaign -- [`0x98cdcf8cd44c216fe370cf5c420d9630ae916fc4`](https://etherscan.io/address/0x98cdcf8cd44c216fe370cf5c420d9630ae916fc4#readContract)

5. Deployed MilestoneTracker -- [`0xb438202e27991b04be6cae51e9ca4c15ddcbac55`](https://etherscan.io/address/0xb438202e27991b04be6cae51e9ca4c15ddcbac55#code)
