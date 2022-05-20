### Guild Create Voting(Aragon Voting fork)

[Voting.sol](https://github.com/Versailles-heroes-com/guild-aragon-voting/blob/master/contracts/Voting.sol)
[SPECS](https://github.com/Versailles-heroes-com/guild-aragon-voting/blob/master/SPECS.md)


## Description

Guild create Voting app is a fork of Aragon Voting App with the following additions:

* Added minimum balance `minBalance` a user must have in order to successful vote

* Added minimum time between one user creating a new vote

* Removed possibility to change votes

* Vote's time must end before a vote can be enacted

* There's a switch to enable/disable creating votes controlled by roles `ENABLE_VOTE_CREATION` and `DISABLE_VOTE_CREATION`.

	Once testing votes is done, the `DISABLE_VOTE_CREATION` role should be set to `0x0000000000000000000000000000000000000000` - noone

Voting app will have `CREATE_VOTES_ROLE` permissions set to anyone(`0xffffffffffffffffffffffffffffffffffffffff`) because the balance check will be done inside the Voting app and not through a forwarder

To incentivize people to vote early and not wait and see what's the outcome of the vote and vote near the end of vote,
the voting power they had for the snapshot of for that vote is getting smaller proportional to the time left for that vote(with a cutoff period of `voteTime` / 2 between creation of vote and when decrease starts to happen)

To prevent DAO to set unrealistic limits for `minBalance` and `minTime`, `minBalance` is required to be the equivalent of at least `1000000` tokens locked for `one year` (1e18 precision) (which equals to `2500000000000000000000`)
and `minTime` is required to be between half a day and 2 weeks

Those limits are set when initializing the app 


# How to deploy and initialize the app

## Edit [.env.example](./.env.example) and modify filename to `.env`, and edit `hardhat.config.ts`, which you'll then use to install the app

## Deploying the app to aragonPM

```shell
npm install
npx hardhat compile
aragon ipfs start
npx hardhat publish major --network <network>
npx hardhat etherscan-verify --network <network> --license "MIT"
```

upload infura ipfs (download infura ipfs-upload-client)

```shell
ipfs-upload-client --id <project-id>  --secret <project-secret> dist
```

verify aragon app from aragon-cli

```shell
aragon apm versions ownership-voting.open.aragonpm.eth --env aragon:rinkeby
aragon apm info ownership-voting.open.aragonpm.eth --env aragon:rinkeby --ipfs-rpc https://ipfs.infura-ipfs.io/:5001 --ipfs-gateway https://ipfs.infura-ipfs.io/ipfs
```

## Installing the app in a DAO

`dao install $DAO_ADDRESS $APP_ADDRESS --app-init-args $VOTING_ESCROW $ACCEPTANCE_PCT $QUORUM_PCT $VOTE_TIME $MIN_BALANCE $MIN_TIME $MIN_BALANCE_LOWER_LIMIT $MIN_BALANCE_UPPER_LIMIT $MIN_TIME_LOWER_LIMIT $MIN_TIME_UPPER_LIMIT --use-frame --env aragon:mainnet`

where

`$DAO_ADDRESS` = Address of DAO to install the app to

`$APP_ADDRESS` = the `appName` you specified in `arapp.json`

`$VOTING_ESCROW` - The VotingEscrow address deployed from [Versailles DAO contracts deployment guide](https://github.com/Versailles-heroes-com/versailles-heroes-DAO/blob/master/scripts/README.md)

`$ACCEPTANCE_PCT` = Percentage of yeas in casted votes for a vote to succeed (expressed as a percentage of 10^18; eg. 10^16 = 1%, 10^18 = 100%)

i.e 510000000000000000 = 51%

`$QUORUM_PCT` = Percentage of yeas in total possible votes for a vote to succeed (expressed as a percentage of 10^18; eg. 10^16 = 1%, 10^18 = 100%)


`$VOTE_TIME` = Seconds that a vote will be open for token holders to vote (unless enough yeas or nays have been cast to make an early decision)

`MIN_TIME` = The minumum time that has to pass between a user's last creating of vote and him creating new vote

`$MIN_GUILD_BALANCE` = The minimum balance a user can have to create a new vote - `1000000` means `1000000e18`

`$MIN_GUILD_BALANCE_LOWER_LIMIT` = The lowest `MIN_GUILD_BALANCE` can be set to

`$MIN_GUILD_BALANCE_UPPER_LIMIT` = The highest `MIN_GUILD_BALANCE` can be set to

`$MIN_TIME_LOWER_LIMIT` = The lowest `MIN_TIME` can be set to

`$MIN_TIME_UPPER_LIMIT` = The highest `MIN_TIME` can be set to

`$GUILD_BALANCE` = The minimum balance a user can have to success a vote - `100` means `100e18`

## Roles added in addition to standard Voting App roles

`SET_MIN_GUILD_BALANCE_ROLE` = Role to set `MIN_GUILD_BALANCE`, should be set to and managed by Voting app itself
`SET_MIN_TIME_ROLE` = Role to set `MIN_TIME`, should be set to
`ENABLE_VOTE_CREATION` = Role to enable vote creation
`DISABLE_VOTE_CREATION` = Role to disable vote creation, should be set to and managed by `0x0000000000000000000000000000000000000000` to give up control to DAO

`SET_GUILD_BALANCE_ROLE` = Role to set `GUILD_BALANCE`, should be set to and managed by Voting app itself
