---
layout: default
title:  Listing NFTs
description: Listing NFTs
date:   2022-02-03 00:00:00 +0000
permalink: /list_nft/
category: Other Random Stuff
---
## Listing NFTs

![ethereal](/assets/media/ethereal.gif) 

Since everybody (and their dogs) are listing NFTs, I thought I should just do it too, just to understand what is involved, and hopefully learn something useful from the process.

And since I will almost certainly forget what went through my mind during this process, I decided to jot down some notes on key points that I learnt or went through my mind. 

I'm not very hopeful about selling any of my NFTs, given how crowded the marketplaces have become in the short span of a year or so, so this post would probably be my main takeaway.

Some of these points could potentially be wrong, or outdated. If so, please let me know.

I am going to assume everyone has a basic understanding of what a Non-Fungible Token (NFT) is. _TLDR: unique (i.e. non-fungible) content or data stored on the blockchain._ If not, there are plenty of references on the net on this, like [this][6].


### Content ###

The starting point is of course to create the content or items that you would like to list. For me, I had [digital illustrations][4] and [code][5] lying around, so it was really a matter of curating them into coherent _collections_ of items. 

NFTs can be minted from physical/tangible items too. If you have, say, actual collectibles lying around, you could mint a digital version of that item and send the collectible to the buyer of the NFT. The NFT is then a digital immutable record of the transaction taking place (maybe alongside some metadata on the transaction, including an image of the item).

Note that I highlighted _collections_ of items, rather than individual items. It is possible to list individual NFTs, or even individual NFTs in collections of others, so why build collections of your own? 

From my perspective, there are a few advantages. 

- First, costs. We will go into this later, but there are costs (gas fees) associated with listing NFTs, and doing it as a collection enables economies of scale. From what I understand and experienced, you will usually pay a one-time gas fee for the first item you sell in a collection, but not the subsequent ones in the collection if you are doing 'lazy-minting'. Do note that gas fees will still apply when you finally sell it though. 
- Second, a collection makes for a more coherent proposition to buyers, as it allows you to develop or create a series of related works that buyers may be interested in, similar to how people collect trading cards with a specific theme. 
- Third, everyone seems to be doing it, so I will just follow the herd.

For me, because of my dual interests in illustration and generative art, I put together two collections on [OpenSea][1], which you can find at these links - illustrations in [Whimsical Ether Reality][2], and generative art videos in [Ethereal Attractors][3].

> TLDR: Generate content or things to sell as NFTs. Curate collections of these as there are many advantages to doing so.

### Platform ###

Next, where to mint and list the NFT. There are many platforms available. 

Most operate on the Ethereuem blockchain (ETH), like [OpenSea][1] and many [others][7]. [Solanart][8] seems to be another interesting marketplace that is on the Solana blockchain. For this first attempt, I decided on OpenSea, as it seems to have the greatest reach. To learn more about OpenSea and what role OpenSea and other _centralized_ marketplaces ironically play in this supposedly brave and new _decentralized_ Web 3.0 world, read this [article][9]. 

Personally, I think deciding on a marketplace should probably be based on factors like *reach* (based on the number of people who frequent the marketplace that may be interested in your stuff), *fit* (with the nature of your content or items as some platforms focus on art, while others on collectibles) and *cost* (service charges of different platforms and gas fees for different blockchains), but I shall not delve too much into those for now, since there are resources on these considerations available via a simple Google search.

So, assuming the marketplace is OpenSea, the next steps would be get a basic understanding of gas fees, and get your wallet ready to pay such fees. 

Listing an NFT is definitely not costless (but neither was Ebay back in the Web 1.0 days).

> TLDR: Find a marketplace to sell your NFT. OpenSea is probably the largest one now.

### Gas Fees ###

From the earlier part of the article, you should have understood that gas and service fees play a key role in minting your NFT. Service fees are easy to understand, OpenSea takes 2.5% of each transaction, i.e. 2.5% of the price of the item that you finally sell.

Gas fees are a little more tricky. Gas fees are basically what you pay to miners to perform an operation so as to record your transaction (which can be minting, or a transfer) on the blockchain. On OpenSea, you can either use Ethereum or Polygon. If you choose to use Ethereum, you pay (a lot more) gas fees, as recording a transaction on the Ethereum blockchain has become rather slow and expensive. Gas fees vary based on when you perform the transaction, and how fast you want the transaction to happen. You would probably need to sell your NFT at a base price of around $100 or more to at least break even. 

Using Polygon means no gas fees at all. But, there is a significant caveat. 

Most buyers on OpenSea prefer to use Ethereum and also value NFTs on Ethereum more, so you have a higher chance of selling your NFT than if you were to use Polygon. But, as with all things, I think this can change rather quickly. Have a read of this [article][10] if you wish to understand a little more on the dynamics of gas fees on Ethereum versus Polygon, and this [article][11] if you wish to know a little about Polygon.

> TLDR: Ethereum expensive, higher chance of selling your NFTs; Polygon free, BUT lower chance of selling your NFTs.

There is a middle ground here, which reduces your initial costs (but not the final overall costs), by 'lazy-minting'. Under this approach, you basically pay a gas fee to validate your account (once for ETH, and another for Wrapped ETH which is compatible with other ETH-based blockchains, it seems), and then each piece you mint thereafter is free. What happens is that the gas fees apply when you actually make a sale and need to record that transaction on the blockchain. So it reduces the initial costs, but ultimately you will still need to pay the piper.

> TLDR: 'Lazy-mint' allows you to pay an initial smaller gas fee, and postpone the gas fees for each NFT til you finally sell something.

### Wallet ###

To pay the gas fees, you need to exchange your real cash for Ethereum. To hold Ethereum that you exchanged, you need a wallet. A typical approach when trading Ethereum would be to purchase Ethereum on say Coinbase, transfer it to your hot (digital, connected) wallet, say Metamask, or your cold (physical, unconnected) wallet. 

For OpenSea, what you would do is quite similar. Get your Ethereum somewhere, either via Coinbase, or via Wyre straight into Metamask, and then connect your Metamask or Coinbase wallet to OpenSea. Connecting your wallet to OpenSea is in fact how you would login to OpenSea. There are many many writeups on how to setup your Metamask or Coinbase wallets and fill it with Ethereum, so I shall not cover them here. 

Now that you have a wallet that you use to login to OpenSea and pay for gas fees, some Ethereum in the wallet, the next step would be to start listing.

> TLDR: Get a wallet. You need it to login to OpenSea, and also to deposit Ethereum within to fund your gas fees.

### Listing ###

As mentioned, listing collections of NFTs makes a lot of sense. So, we should start by going to the top-right hand corner of OpenSea, hover over your profile image, and select _My Collections_. Select _Create a collection_ on the _My Collections_ page. 

Fill in the necessary details. The interesting field here is _Creator Earnings_. This essentially allows you to earn a percentage (based on what you enter here) of each of the transactions on NFTs in this collection, i.e. a royalty.

I recommend that your profile, banner and feature image be created in the recommended dimensions shown. Too small or too large and they will look rather pixellated in the final collection page.

With the collection created, we can now add an item to the collection. On the _My Collection_ page, select the collection you just created, and then choose _Add Item_ on the top-right hand corner. Again, you just need to fill in the fields you see on the next page to add the item.

- **Image, Video, Audio, or 3D Model**: The content you are minting as an NFT
- **Name**: NFT name
- **External link**: Link to say an external landing or project page where you have a more detailed description of your NF
- **Description**: Description of the NFT
- **Collection**: Which collection you are adding the item to
- **Properties**, Level, Stats: Key characteristics of your work, say for example you have a work with different degrees of joy, you can assign different numbers as levels
- **Unlockable Content**: Bonus content that you will unlock for the winning bidder, say a link to download special items
- **Supply** - How many of the same NFT you are minting, say either a one of a kind, or 5 editions of the same NFT. This field is special. In order for it to be editable, you need to, right at the start, append this `?enable_supply=true` to your link for the add item page, i.e. it should look like this:
  > `https://opensea.io/collection/<your-collection-name>/assets/create?enable_supply=true`
- **Blockchain**, which blockchain you are minting this NFT on
- **Freeze metadata**: Select this only when you are sure your details above are correct. This freezes all the details above on the blockchain, whicn means you cannot change them after you freeze the metadata. It also costs gas fees (as you are recording a transaction on the blockchain). You can list your item first, and do this later when you are sure you are selling the item.

That's pretty much it. Once you select _Create_, the NFT should appear in your collection in a few minutes (not immediately). All the steps involved up to now do not cost money.

You can still edit everything you entered earlier, except for the **Supply**. 

But they should already be viewable, so just creating collections, and adding items to collections is a pretty nice way to get some exposure for your NFTs before you even pay a single cent on gas fees. I believe that buyers on OpenSea can already starting making offers to you at this stage (but I may be wrong).

### Sell ###

Once you are happy with what you have listed. You can sell them. This is when gas fees actually take effect. First you select the NFT you want to sell.

Select the _Sell_ button on the top-right. You can choose between a **fixed price sale**, a typical **auction** (where the seller bids up), or a **Dutch auction** (where the price declines over time til someone purchases the item). For all 3, you can select the duration of the sale, up to 6 months. The advice typically given is not to set the duration for too long, as you might change your mind, and it is better to let it expire and re-list, then to incur costs cancelling the sale. I'm not too sure about the costs involved in cancellations though. 

Also note that unless you want the buyer to buy multiple pieces at a go, you should set the quantity to 1. Setting it to, say 2, means the buyer would have to purchase both editions of the NFT at once. Not an issue if you are just listing and selling one of a kind NFTs.

When you complete the sale, you will pay a one-time gas fee at this point for initializing your account (I have this impression of being charged once for ETH and once for WETH, but I may be wrong). Prices, as mentioned, depend on the prevailing gas fee at that time, which you can check [here][12].

That's about it. 

*Repeat and rinse for more collections and more items in each collection.*

### Marketing ###

What we did above is probably the most technical (though not that technical), but also the simplest part of the entire process. 

The hard part is getting the NFTs out there, and made known to people who are interested.

I tried sharing the works on Reddit subreddits such as nft, Discord channels such as NFT Asia, and my own social media channels, but this part of the puzzle (which is probably the most important), is something I am still trying to explore. Until then, I think my OpenSea account is just another portfolio site for me.

I suppose the NFT domain is not too different from the art domain. A friend once advised me to understand the artist's marketing capabilities over his artistic or technical capabilties if the intent was to invest in his or her art (rather than just admire it). I suppose this is equally true of NFTs and NFT creators.


~~~

[1]:	https://opensea.io/
[2]:    https://opensea.io/collection/whimsicalethereality
[3]:    https://opensea.io/collection/ethereal-attractors
[4]:    https://playgrd.com/art
[5]:    https://github.com/playgrdstar
[6]:    https://ethereum.org/en/nft/
[7]:    https://www.lifestyleasia.com/ind/gear/tech/top-nft-marketplaces/
[8]:    https://solanart.io/
[9]:    https://www.theverge.com/2022/2/2/22914081/open-sea-nft-marketplace-web3-fundraising-finzer-a16z?scrolla=5eb6d68b7fedc32c19ef33b4
[10]:   https://www.publish0x.com/freelance-ghostwriting/opensea-s-gas-free-nft-marketplace-on-polygon-xelelny?a=APdRorlKeG&tid=Twitter8.15
[11]:   https://www.coindesk.com/learn/polygon-and-matic-whats-the-difference/#:~:text=Polygon%20is%20a%20secondary%20scaling,it%20becomes%20ever%20more%20popular
[12]:   https://etherscan.io/gastracker