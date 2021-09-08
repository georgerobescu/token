# The Relevant Token Smart Contracts

This repository contains smart contracts used by [Relevant APP](https://relevant.community) to reward content curation.

## RelevantTokenV3.sol

Users of the Relevant App are able to earn rewards for curating tokens. These rewards are generated by minting new REL tokens. REL token `admin` distributes the tokens as balances in the web2 database of the Relevant App. Users can then `claim` these tokens by requesting an attestation by the `admin` account and submitting it the REL smart contract.

There is currently a limit to how many tokens can be claimed this way by any individual account. Balances over the limit can be claimed as vested `sRel` tokens. These tokens are not transferrable (for the most part) but can otherwise be used for governance and within the app.

### Key Contract Parameters

- `inflation` is the yearly APR in basis points.
- `allocatedRewards` are the tokens that still belong to the smart contract but are allocated to users as Curation rewards. These tokens can either be claimed or vested by the users.
- `admin` is a designated hot wallet address that signs user's token claims. Security note: if this account is compromised the attacker can drain the `allocatedRewards` but not more. This address should be rotate periodically by 'owner'.

### Key Methods

- `releaseTokens` can be called by anyone to either mint or allocate existing tokens. If there are enough tokens in the smart contract that are not part of the `allocatedRewards`, they will be used, otherwise new tokens will be minted.
- `claimTokens` can be used by users to claim REL from `allocatedRewards`

### Owner Methods

- `vestAllocatedTokens` sends a portion of the `allocatedTokens` to a vesting smart contract. This method should be called bafore initializing any specific vesting account.
- `updateAllocatedRewards` updates `allocatedRewards` - we may need to do this to ensure app rewards match up with allocated rewards.
- `setInflation` sets yearly inflation rate in basis points.
- `setAdmin` updates the token claim signer.
- `burn` can burn an amount of tokens that are not part of the `allocatedRewards`. We might want to do this in the future to simplify accounting. Note: yearly inflation rate should be adjusted if a significant number of tokens is burned.

# Relevant Protocol Governance Smart Contracts

## sREL token

sREL is governance wrapper for REL tokens and allows staking and vesting.

### Roles

- `owner` (this will eventually be a DAO)
  - can initialize initial vesting accounts
  - can set `vestAdmin` role
- `vestAdmin` is designed to be a hotwallet that allows automated vesting initialization via the Relevant App

### Staking

- REL tokens can be staked via the contract in exchange for sRel
- sRel cannot be transferred or exchanged back to REL unless they are 'unlocked' (unstaked) and unvested
- `lockPeriod` deterimines the time it takes to unstake the tokens

### Vesting

- Vested tokens can be added by the `owner` of the contract or via a signature from the `vestAdmin'
- There are two vesting schedules - short and long, exact params TBD, likely 4 and 16 years respectively
- The params are global - meant to distribute a set amount of tokens to users
- Vested tokens can be used to cast governance votes
- The full amount of vested tokens can be transferred to a new account

## Governor

Openzeppelin Governor contract that can self-modify the `votingPeriod`, `proposalThreshhold` and `votingDelay`.

## Timelock

Openzeppelin Timlock that will add a delay to all governance decisions.
