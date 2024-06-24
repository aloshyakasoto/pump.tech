# Operations

## Minting
 - The MaxSupply of all ERC404 tokens mint by `PumpTechContract` is a constant (Let's say, `10000`)
 - To mint a new ERC404 token, the `Minter` must pay ETH to `PumpTechContract`
 - `PumpTechContract` mints whole `MaxSupply` of ERC404 tokens, and transfer a certain amount of it to `Minter`
 - This amount is calculated based on the amount of ETH paid by `Minter`, after deducting a `MintFee` set by the protocol
 - If the amount of ETH paid by `Minter` is insufficient, the minting process fails
 - Bancor's Power formula is used to calculate the amount of ERC404 tokens to be transferred to `Minter`
 - The remaining ERC404 tokens are kept by `PumpTechContract` for enabling trading


```mermaid
---
title: Minting Sequence Diagram
---
sequenceDiagram
    autonumber
    actor Minter
    participant PumpTechContract
    participant ERC404Contract
    Minter ->> PumpTechContract: Mint TRUMP for 1 ETH
    break If insufficient ETH
        PumpTechContract-->>Minter: Mint Fail
    end
    PumpTechContract ->> ERC404Contract: Mint 1000 TRUMP
    ERC404Contract ->> PumpTechContract: Transfer 10000 TRUMP
    PumpTechContract ->> Minter: Transfer 1 ETH worth of TRUMP
    Note over PumpTechContract: Keep remaining TRUMP
```

## Trading
The ERC404 tokens mint by this protocol are traded on either of the following AMM:

1. BondingCurve
2. UniswapV2

The ERC404 tokens mint by `PumpTechContract` are traded on `PumpTechContract` initially, until it's Market Cap (in ETH) reaches `MinimumFundsForLP` threshold set by the protocol. After that, the remaining ERC404 tokens in `PumpTechContract` are paired with ETH collected from sales on BondingCurve to create `UniswapV2LP` tokens. These LP tokens are used for further trading on a UniswapV2 AMM. Also, these initial LP tokens are locked permanently in `PumpTechContract` to ensure base liquidity of the ERC404 token.

### BondingCurve
The buying process on BondingCurve is as follows:
 - The `Trader` pays ETH to `PumpTechContract` to buy ERC404 tokens
 - If the BondingCurve is halted, the buying process fails
 - The amount of ERC404 tokens to be transferred to `Trader` is calculated based on the amount of ETH paid by `Trader` using Bancor's Power formula
 - If the Market Cap of `PumpTechContract` reaches `MinimumFundsForLP`, the BondingCurve trading is halted and the initial LP tokens are added to UniswapV2AMM
 - Similarly, when selling ERC404 tokens on BondingCurve, the amount of ETH to be transferred to `Trader` is calculated based on the amount of ERC404 tokens paid by `Trader` using Bancor's Power formula
 - If the BondingCurve is halted, the selling process fails

```mermaid
---
title: BondingCurve Trading Sequence Diagram
---
sequenceDiagram
    autonumber
    actor Trader
    participant PumpTechContract
    participant UniswapV2AMM
    par Buy on BondingCurve
        Trader ->> PumpTechContract: Buy TRUMP for 1 ETH
        break If BondingCurve halted
            PumpTechContract-->>Trader: Buy Fail
        end
        Note over PumpTechContract: Calculate TRUMP to transfer
        PumpTechContract ->> Trader: Transfer 1 ETH worth of TRUMP
        Note over PumpTechContract: If Market Cap reached MinimumFundsForLP<br/>1. Halt BondingCurve trade<br/>2. Add initial LP to UniswapV2AMM
        activate UniswapV2AMM
        PumpTechContract ->> UniswapV2AMM: Mint initial LP<br/>with remaining TRUMP<br/>and ETH reserves of TRUMP
        UniswapV2AMM ->> PumpTechContract: Transfer LP tokens
        Note over PumpTechContract: Lock LP permanently
    end
    par Sell on BondingCurve
        Trader ->> PumpTechContract: Sell 1 TRUMP for ETH
        break If BondingCurve halted
        PumpTechContract-->>Trader: Sell Fail
        end
        Note over PumpTechContract: Calculate ETH to transfer
        PumpTechContract ->> Trader: Transfer 1 TRUMP worth of ETH
    end
```

## Revenue Sharing
The `PumpTechContract` collects fees from minting and trading operations. It also allows extracting fees accumulated from locked LP tokens. The minting and trading fees are transferred to the `ProtocolTreasury` contract during the minting and trading operations. The protocol owner can extract the LP fees from the locked LP tokens at any time, while the initial LP amount shall remain locked.

```mermaid
---
title: Revenue Sharing Sequence Diagram
---
sequenceDiagram
    autonumber
    actor Owner
    actor Minter
    actor Trader
    participant PumpTechContract
    participant ProtocolTreasury
    par Minting Fee
        Minter ->> PumpTechContract: Mint TRUMP
        Note over PumpTechContract: Collect minting fee<br/>& mint TRUMP
        PumpTechContract ->> ProtocolTreasury: Transfer minting fee
    end
    par Trading Fee
        Trader ->> PumpTechContract: Buy/ Sell TRUMP
        Note over PumpTechContract: Collect trading fee<br/>& facilitate trade
        PumpTechContract ->> ProtocolTreasury: Transfer trading fee
    end
    par LP Fee
        Owner ->> PumpTechContract: Collect LP fee revenue
        Note over PumpTechContract: Extract fee revenue from all locked LP tokens
        pumpTechContract ->> ProtocolTreasury: Transfer LP fee revenue
    end
```
