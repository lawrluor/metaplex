Fair-Launch Protocol
Firstly, huge shoutout to https://twitter.com/redacted_j and https://twitter.com/baalazamon for their amazing work so far in the community. They are enabling a very fast adoption rate with Solana, and should be treated with the greatest respect.

Note: This tutorial is up to date with the latest tutorial from https://twitter.com/redacted_j 's office hours.

Also Note: Fair Launch Protocol is still in testing. Be cautious.

This is a tutorial on how to use Fair Launch Protocol integrated with Candy machine. This tutorial assumes you already have some working knowledge of how to use candy machine. If not, I find these to be the best resources to date:
https://threadreaderapp.com/thread/1433816437525659658.html
https://hackmd.io/@levicook/HJcDneEWF#Metaplex-Candy-Machine

It’s also useful to have an intuition for how Fair Launch Protocol works, best resource is this:
https://scapesnft.medium.com/fair-launch-protocol-guide-72dc9609461e

Prerequisites:
Solana Tool-Suite: https://docs.solana.com/cli/install-solana-cli-tools
Node: https://nodejs.org/
Yarn: https://yarnpkg.com/getting-started/install
Github CLI: https://cli.github.com/manual/installation
Working Knowledge of Candy Machine, Solana CLI and how Fair Launch Protocol works on a high level.

Let’s get started:
Fork the latest metaplex repo & install dependencies.

Github CLI:

$ gh repo fork https://github.com/metaplex-foundation/metaplex --clone=true

or you could fork manually on github here: https://github.com/metaplex-foundation/metaplex


then run

$ git clone https://github.com/username/metaplex

We’re going to cd into the js directory using
$ cd metaplex/js/

then run

$ yarn

This will install all dependencies.

Now we will cd into the the cli directory using
$ cd metaplex/js/packages/cli

then run

$ yarn watch

Jordan highly recommends this approach over others. Enables you to run commands against the javascript.

Create Fair Launch Protocol Flow
High level explanation of the next steps:

So if you understand how FLP works in this specific use case, we want Candy Machine only to accept as payment tokens (lottery tickets) that are given to users who won from the fair launch. But because candy machine needs to have the specific token mint address in its creation (defaults to SOL if none is specified), we will create the fair launch first, which will give us a token mint, then use the given token mint to set up candy machine, at which point they are connected. If this doesn’t make sense, it will as you run through the tutorial.

Let’s create a new fair launch by first looking at its arguments. These can be viewed by opening:fair-launch-cli.ts

This is a glossary of options and what they mean, you may skip past if you want to copy & paste the exact command.

We start every command with the prefix: node build/fair-launch-cli.js

-e, --env <string> - Solana cluster env name, default is devnet, other options include mainnet-beta, testnet, & devnet
-k, --keypair - solana wallet location, I usually just go with /.config/solana/id.json
-u, --uuid <string> - uuid of the fair launch. A 6 character alpha-numeric string that should be unique relative to your keypair. Your keypair can host multiple fair launches, therefore, each fair launch needs its own unique UUID.
-f, --fee <string> - fee that each person needs to pay inorder to interact with your fair launch. In terms of SOL (0.001, .1, etc)
-s, --price-range-start <string> - price range start, so in the fair launch, you can have a minimum bid and a maximum bid. We set the minimum here.
-pe, --price-range-end <string> - price range end, we set the maximum bid here.
-arbp, --anti-rug-reserve-bp <string> - optional anti-rug treasury reserve in units of basis points (1-10000). Essentially, if you as the product maker don’t deliver, this is exactly how much as a % collectors will get back in refund. (eg 5000 = 50%, half of the treasury is refunded equally to everyone). This is very important for community building, as take into account a scenario where the price of Solana falls as well. Collectors could take losses of more than 50% since this is based on amount of sol pledged, not cash value at time of pledging. Do not set this value to 100% if you are doing something similar to a fundraise, as you need an access to some amount of the funds in order to build your product.
-atc, --anti-rug-token-requirement <string> - optional anti-rug token requirement when reserve opens - 100 means 100 tokens remaining out of total supply. A way to program an obligation. In the case of a 10,000 NFT supply, if this option value was set to 1000, then that means in order for you to fully collect funds, there would need to be 1000 or less supply of the fair launch token mint, or said another way “You’d need to convince 9000 people to exchange their tokens into candy machine for your product in order to recieve all your funds.”

Take into account that all of these can be seen publicly, so how you decide on these can affect sentiment about your project.

-sd, --self-destruct-date <string> - optional date when funds from anti-rug setting will be returned - eg “04 Dec 1995 00:12:00 GMT”', essentially the deadline for the you as the product maker to deliver, pass this deadline & people are given the option to refund, against your will.
-pos, --phase-one-start-date <string> - phase one (bidding) start date. entered as a timestamp.
eg “04 Dec 1995 00:12:00 GMT”,
-poe, --phase-one-end-date <string> - phase one end date, entered as a timestamp.
eg “04 Dec 1995 00:12:00 GMT”,
-pte, --phase-two-end-date <string> - phase two (accept or reject price) end date. There is no start date argument because phase two automatically starts when phase one ends. Entered as a timestamp as well. eg “04 Dec 1995 00:12:00 GMT”
-ld, --lottery-duration <string> - max duration of lottery, in units of seconds. If this passes, any person can permissionlessly call the restart_phase_2 command from the cli which will bring the entire fair launch back to beginning of phase 2 for everyone to leave if they no longer trust the project.
-ts, --tick-size <string> - tick size, so in the UI of the fair launch exists a slider, in which users can utilize to set bids. This essentially sets the increment value (eg, .1 means you can only increment in values of .1 so 1.2, 1.3,…, 2.2, 2.3,etc)
-n, --number-of-tokens <number> - number of tokens (lottery tickets) to sell, in most cases, you’d want this to match the intended supply for use in candy machine.
-mint, --treasury-mint <string> - token mint to take as payment instead of sol. If you want to enable only the users who own a custom SPL token to utilize this fair launch, you may set that here. UI currently only works with SOL.

An example of a command to create a fair launch would look like this:

$ node build/fair-launch-cli.js new_fair_launch -e devnet -k ~/.config/solana/id.json -u test01 -f 0.1 -s 0.1 -pe 2 -pos "2021 Sep 22 12:30:00 CDT" -poe "2021 Sep 22 12:35:00 CDT" -pte "2021 Sep 22 12:45:00 CDT" -ts 0.1 -n 2 -arbp 5000 -atc 1 -sd "25 Sep 2021 09:00:00 GMT"

This will give you a FairLaunchID, record it somewhere for use later.

Here, I recommend you create a notepad to store some of these variables, as you will need them for future commands and they can get lost in the shuffle.

We need to get our fair launch token mint by running:
$ node build/fair-launch-cli.js show -e devnet -k ~/.config/solana/id.json -f <fair-launch-id-here>

Note: Anyone (ANYONE) who knows the Fair Launch ID can execute this command. This was intentional, as it enables decentralization by allowing anyone to access the terms and conditions of your fair launch.

After running that command, you should get something like this in the output:
Token Mint <token mint address>

We will also need this for later, so record it.

Create and Connect Candy Machine
We are now going to use this token mint to create a candy machine that only accepts this token as payment! (Isn’t that so sick!?)
First, ensure you’ve built a candy machine:
$ node build/candy-machine-cli.js upload -e devnet -k ~/.config/solana/id.json ~/directorywithimages

Second, we need to take a couple of steps to grab the value for a parameter in our next command called spl-token-account. It’s a little tedious now, but expect it to be automated/ made easier in the future.

For now the flow goes like this:
Log in to Phantom and ensure it’s on devnet.
Click Manage Token List
Click on the plus sign to create a token account.
Grab your FLP Token Mint from earlier, and paste it inside mint address box. Enter any name & symbol, then click add.
Note that in order to have your token account show up, you must activate it by clicking the switch. However, it will be all the way at the bottom of the list so you may need to scroll.
Go to explorer.solana.com (Make sure its on devnet)
Paste FLP Token Mint address into search bar.
Scroll down, find “distribution”, click on it.
There should be an address there with rank #1 for the token, copy that account address & record it.
We’ll now create the candy machine. I assume you automatically know what most of these parameters mean, such as -e, -k, -p, and --date. So adjust as needed.

$ node build/candy-machine-cli.js create_candy_machine -e devnet -k ~/.config/solana/id.json -p 1 --spl-token <fair launch token mint here> --spl-token-account <token account from last step here> 

then

$ node build/candy-machine-cli.js update_candy_machine -e devnet -k ~/.config/solana/id.json --date "22 Sep 2021 17:30:00 CDT"

You can actually set the date for later if you are not ready to utilize candy machine and mint out your product. How this further affects Fair Launch is explained later.

After running the command, we should finally recieve our Candy Machine ID. Record this somewhere as well.

Open
metaplex/js/packages/fair-launch/.env

And enter the values into ther respective fields, ensure that the network & RPC host are what you have intended, in this case, devnet.

REACT_APP_CANDY_MACHINE_ID=

REACT_APP_SOLANA_NETWORK=devnet
REACT_APP_SOLANA_RPC_HOST=https://api.devnet.solana.com

# Phase 1
REACT_APP_FAIR_LAUNCH_ID=

save and exit file.

Then:

$ cd js/packages/fair-launch

$ yarn start

And Boom! We have successfully created a fair launch, but wait, there’s more steps.

We need to manually start the lottery process after phase two has ended. Before we run the lottery command that selects who has won a ticket, we need to run this:

$ node build/fair-launch-cli.js create_missing_sequences -e devnet -k ~/.config/solana/id.json -f <fair launch id here>

“This essentially makes sure there is a reverse look up for each ticket in existence so that the lottery can iterate from index 0 to the total number of tickets without knowing who owns each ticket.”" ~ Jordan.

Now to initiate the lottery:

$ node build/fair-launch-cli.js create_fair_launch_lottery -e devnet -k ~/.config/solana/id.json -f <fair launch id here>

If everything runs smoothly…



Note: Anyone with the fair launch ID can run this command as well to see how the lottery worked out.
$ node build/fair-launch-cli.js show_lottery -e devnet -k ~/.config/solana/id.json -f <fair-launch-id-here>

Now to start phase three and enable minting, we run:
$ node build/fair-launch-cli.js start_phase_three -e devnet -k ~/.config/solana/id.json -f <fair-launch-id-here>

You know it worked if you got the message:


If your candy machine was set to start as soon as you started phase 3, then users would be able to immediately mint. If you set it to be much later, the option that would be there would be “Punch Ticket.” Which would “mint” the ticket to them, enabling them to hold it in their wallets for later.

Administration Commands:
These are commands that the owner of the fair launch would likely find useful.

$ node build/fair-launch-cli.js punch_and_refund_all_outstanding -e devnet -k ~/.config/solana/id.json -f <fair-launch-id-here>

This automatically allows you to refund everyone who lost and give tickets to everyone who won without them ever having to interact with your UI. Clean UX if you ask me. This prevents people from forgetting to claim their refunds.

$ node build/fair-launch-cli.js withdraw_funds -e devnet -k ~/.config/solana/id.json -f <fair-launch-id-here>

This would enable you to withdraw all your funds. However, note that you cannot run this command until everyone has punched or been refunded, so it’s a good idea to run the command above this first.

That is the entire flow of setting up and utilzing a fair launch protocol. It’s not actually too much, and once you get the hang of it it becomes very easy.

End
This post will be consistently updated as more information and more changes become available. This is still in testing, so errors may occur. If that’s the case, head to the @metaplex discord for assistance.

discord.com/invite/metaplex

Thank you for reading my first blog post, feedback is appreciated!

gn.

- Arte

Twitter: https://twitter.com/artelaflame
Bulls & Bears NFT: https://twitter.com/bullsbearsNFT

Bonus: Setting the Token Metadata of the Ticket
A really cool feature that was added is the ability to add metadata to your ticket so it looks like an NFT. (Imagine a golden ticket from polar express in your wallet)

An example command looks like this:

node build/fair-launch-cli.js set_token_metadata -e mainnet-beta -k ~/.config/solana/id.json -f 7fP1DM1UK3xBgzXxQVwA3fFA4PPENaQ6YghJvFLeXFMz --name "Thanks For Testing" --symbol ALPHA1 --uri https://arweave.net/DJp9PoXrUfV60I-YjSo29zY4GD84eRi2EhMe8PMHoU8 -sfbp 500 -c 44kiGWWsSgdqPMvmqYgTS78Mx2BKCWzduATkfY4biU97,50,false,CduMjFZLBeg3A9wMP3hQCoU1RQzzCpgSvQNXfCi1GCSB,50,false

Let’s break this down real quick:
-e, --env <string> - Solana cluster network name, can be devnet, mainnet-beta, or testnet
-k, --keypair - Solana wallet location, mine is usually set to ~/.config/solana/id.json
-f, --fair-launch <string> - fair launch id
-n, --name <string> - token name
-s, --symbol <string> - token symbol
-u, --uri <string> - uri, note that this should be a uri that links to a json.
-sfbp, --seller-fee-basis-points <string> - seller fee basis points, ranges (1 - 10000)
-c, --creators <string> - comma separated creator wallets like wallet1,73,true,wallet2,27,false where it flows wallet, share, verified true/false.
-nm, --is_not_mutable - set as immutable, meaning it cannot be changed ever again. Use cautiously, think about if its really necessary. A lot of artists have had issues because they marked the NFT as immutable, unable to make changes to the metadata when there were bugs/mistakes. (Ex. Sol Bears)

And there you have it!

Select a repo