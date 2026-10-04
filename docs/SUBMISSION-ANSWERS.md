# Submission answers — copy-ready

Written 2026-10-05 for the entry form's long-answer questions. Each heading
shows the character count against the field's limit, so a block can be pasted
as it is. Every number is measured: the weekend figures from
`deploy/weekend-backtest.json` and `deploy/weekend-live-2026-09-18.json`, the
fee from a real devnet execution, the test count from both suites.

---

### 1. What are you building, and who is it for?  [995/1000 characters]

bozBasket is a recurring-buy robo-investor for tokenized US stocks on Solana that refuses to buy when the reference price cannot be trusted.

You define a basket once - say TSLA 50% / QQQ 30% / VOO 20%, $100 weekly - and deposit USDC into a vault only your key can withdraw from. Each period a keeper triggers the buy and the program fills every leg in one atomic transaction. Before it fills, inside that transaction, it checks how old Pyth's price is, how wide its confidence band is, how far the venue has drifted from it, and whether there is depth. If any check fails the buy is deferred: the reason is written into the plan account and emitted as an event, the transaction still succeeds, and nothing moves.

It is for people who want to own US equities in small, regular amounts without a brokerage account, and who are the ones harmed when a timer buys at 3am on a Saturday. The same guard is a public API, so any wallet or recurring-buy tool can ask for the verdict before its own swap.

### 2. Why did you decide to build this, and why build it now?  [993/1000 characters]

Tokenized stocks trade 24/7. The price you can trust them against does not: from Friday 20:00 to Sunday 20:00 ET no US equity price is published, and the last one simply ages for 48 hours while the tokens keep trading. Every recurring-buy tool on chain fires on a timer anyway.

We measured the cost instead of assuming it. Across eight weekends of real xStock pool trades, a blind weekend buy landed a median 0.45-0.53% from the next price Pyth published, three to four times the weekday gap, and anywhere from -1.96% to +1.92%. On the first weekend we measured live, 18-20 September, the check ran 496 times per stock while Pyth was silent and the guard would have deferred 495 of them.

Now, because the pieces only just arrived: xStocks trade on Solana today, Pyth publishes US equities well beyond exchange hours, and its receiver verifies signed prices on chain. That is what lets the decision live inside the program instead of in someone's server, where a user has to take it on trust.

### 3. What technologies are you using or integrating with?  [485/500 characters]

Anchor 0.30.1 (Rust) for the on-chain program. Pyth for the reference price: Hermes for signed updates, pyth-solana-receiver to post them, and the PriceUpdateV2 account the program verifies. Jupiter's quote API, read-only, for what an xStock really costs on mainnet. Token-2022 scaled UI amounts, for the issuer's share multiplier. Next.js and React for the app and the public Guard API, Postgres as a ledger cache, and a keeper that runs as a scheduled function or a worker. 97 tests.

### 4. How does your product use these chains?  [498/500 characters]

Solana only, for things that need a chain. One transaction buys every leg, all or nothing: a real three-stock fill cost 0.000055 SOL, which is what makes a $100 weekly basket worth running. The USDC sits in a vault owned by the plan's PDA, so the keeper can trigger a buy but never move the money elsewhere. The guard runs inside execute_basket, on a price account the Pyth receiver wrote. And a refusal is a cheap successful transaction, so every deferral stays auditable on chain with its reason.

### 5. Notes for judges  [479/500 characters]

Try it without a wallet: "Try with a demo wallet" makes a burner key in the browser and funds it on devnet. The demo controls push real on-chain state - divergence, depth, thresholds - so you can watch the program refuse a buy, then restore it and watch it fill.

Real: the program, vaults, schedule, Pyth updates, guard, every deferral. Not real: devnet has no xStocks or liquidity, so stock tokens and fills are synthetic and every page says so. The mainnet check is read-only.

### 6. Context about the repo  [481/500 characters]

One repository holds the whole product: the Anchor programs, the keeper, the web app and public API, the shared guard rules the program mirrors in Rust, and the docs.

It moved to this account recently, so it carries a single commit; the code is unchanged and none of it belongs to another product. mock_market exists only because devnet has no xStocks to buy; it is not part of the mainnet design. deploy/ holds the devnet program ids and the measured weekend data the site reads.

### 7. Project one-liner  [95 characters]

Recurring baskets of tokenized US stocks on Solana that only buy when the price can be trusted.

### 8. What problem are you solving, and who are you building for?  [716 characters]

Tokenized stocks trade 24/7; the price you can trust them against does not. From Friday 20:00 to Sunday 20:00 ET no US equity price is published, and the last one just ages while the tokens keep trading. Recurring-buy bots fire on a timer anyway. Measured over eight weekends of real pool trades, a blind weekend buy landed a median 0.45-0.53% from the next price Pyth published, against 0.13-0.15% on weekdays, and as far as 2% away.

We build for people who want to own US equities in small regular amounts without a brokerage account - the ones who pay that spread when a timer buys at 3am on a Saturday - and for any wallet or DCA tool that wants the same check before its own swap, through our public Guard API.

### 9. The Solana integration in the working MVP  [1048 characters]

Our own Anchor program, basket_dca, live on devnet. create_plan opens a plan and a vault owned by the plan's PDA, so only the owner's key can withdraw. execute_basket buys every leg in one atomic transaction, and before each leg it verifies the Pyth price account the Pyth Solana Receiver just wrote - its owner, its feed id, its Full verification level - then checks four numbers against limits held on chain: age (120 s), confidence (50 bps), the venue's divergence (150 bps) and depth. Any failure writes a reason code into the plan account, emits an event and returns Ok, so a refusal is a successful, auditable transaction that moves nothing.

Around it: the keeper posts signed Hermes updates with @pythnetwork/pyth-solana-receiver, Jupiter's quote API is read every five minutes on mainnet to compare a real xStock price with Pyth, xStock mints are read as Token-2022 with the issuer's scaled UI multiplier, and mock_market provides fills on devnet, which has no xStocks. The same guard rules are a public API any program or wallet can call.

### 8b. Shorter version of the problem answer  [486 characters]

Tokenized stocks trade 24/7; the price you can trust them against does not. From Friday 20:00 to Sunday 20:00 ET none is published, and recurring-buy bots fire on a timer anyway. Over eight weekends of real pool trades, a blind weekend buy landed a median 0.45-0.53% from the next Pyth price, against 0.13-0.15% on weekdays. We build it for people buying US equities in small regular amounts without a brokerage, and for any wallet or DCA tool that wants the same check before its swap.

### 9b. Shorter version of the Solana answer  [580 characters]

Our own Anchor program on devnet. One transaction buys every leg of the basket, all or nothing, from a vault owned by the plan's PDA that only its owner can withdraw from. Inside that transaction the program verifies the Pyth price account the Pyth Solana Receiver wrote - owner, feed, Full verification - and checks its age, its confidence band, the venue's divergence and depth against limits held on chain. A failure writes the reason into the plan account and returns Ok, so refusals are auditable too. The keeper posts signed Hermes updates; Jupiter's quote API is read-only.

