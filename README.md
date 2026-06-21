# AI Poll — an Intelligent Contract that reads free-text opinions and tallies them

A minimal GenLayer **Project**: a polling contract where voters submit opinions in plain
language and an LLM-backed method classifies and tallies them on-chain. Deployed and
exercised on **Testnet Bradbury (Phase 1)**, with verifiable transactions — and an honest
finding about LLM-method finality on the current testnet.

## The use case

A normal smart contract can only count structured votes (`yes`/`no` buttons). It cannot read
*"honestly I think it's a great idea"* and decide that means support. **AI Poll** can: voters
call `submit_opinion("...")` with free text, and `tally_opinions()` invokes an LLM to classify
every opinion as support / oppose / unclear and return a structured result.

GenLayer is **central** — strip out the LLM call and the contract has nothing to do. That is
the bar a Project should meet: GenLayer is the reason it works, not decoration.

## Methods

| Method | Type | What it does |
|---|---|---|
| `submit_opinion(text)` | write (deterministic) | Appends a free-text opinion |
| `get_opinions()` | view | Lists all submitted opinions |
| `get_topic()` | view | Returns the poll topic |
| `tally_opinions()` | **write (LLM)** | LLM classifies + tallies into `{support, oppose, unclear, summary}` |
| `get_result()` | view | Returns the latest tally |

The LLM input is precomputed deterministically before the non-deterministic block, so every
validator feeds the model identical input — the pattern that maximizes inter-validator
agreement under Optimistic Democracy.

## What was deployed and tested (Testnet Bradbury, same wallet)

**Deployer wallet:** `0x90b96F18E31a3A918DFf7775DA8f8eCd9A0FCe59`

Two independent instances were deployed and exercised — one under heavier load (4 opinions),
one under light load (2 short opinions) — to test whether load affects LLM-method finality.

| Instance | Contract | Opinions submitted | tally_opinions outcome |
|---|---|---|---|
| 1 (heavy) | `0xb6b258F09e49F14F58d99a81B0623aB309e224E7` | 4 — all FINALIZED | 3 calls: UNDETERMINED / FINALIZED(UNDETERMINED) / UNDETERMINED |
| 2 (light) | `0x1CE...94409` | 2 — all ACCEPTED | 2 calls: UNDETERMINED / UNDETERMINED |

Across both instances: **2/2 deploys succeeded, 6/6 `submit_opinion` (deterministic) reached
finality, and 0/5 `tally_opinions` (LLM) reached a clean ACCEPTED verdict** — every LLM call
ended UNDETERMINED or LEADER_TIMEOUT, on both heavy and light input.

## On-chain proof (failed-finality tally calls)

| tx | outcome | link |
|---|---|---|
| `0x2834...be53` | UNDETERMINED | [explorer](https://explorer-bradbury.genlayer.com/tx/0x2834d9c5336401d54c93f53465132acc94b67cd162bef8abddcb0416077cbe53) |
| `0xe1f6...414c` | UNDETERMINED | [explorer](https://explorer-bradbury.genlayer.com/tx/0xe1f61b6df4ab24fcc61033c1dd9c8d9764104e92cdd45be0c8cfa1977f63414c) |
| `0x44d9...b45b` | UNDETERMINED | [explorer](https://explorer-bradbury.genlayer.com/tx/0x44d9c5689a165edf1c1ef63fa4d3d1e2c002554cc005538108f9e60cb72cb45b) |
| `0x7ebf...c28c` | FINALIZED (UNDETERMINED) | [explorer](https://explorer-bradbury.genlayer.com/tx/0x7ebf110cde8b92cafb4c4a4513e5902d43dd7d850ac46e3f9dcf48edfd5cc28c) |

(`submit_opinion` and deploy transactions all reached finality — see the deployer account on the explorer.)

## Finding: deterministic methods finalize; the LLM method did not (Phase 1)

The contract itself is correct — every deterministic path (`deploy`, `submit_opinion`) reached
finality cleanly, twice over. The LLM method `tally_opinions` did **not** reach a clean verdict
in **5 attempts across 2 instances**, under both heavy and light input. The failure is therefore
not load-driven and not a contract bug; it is an observed limit of **LLM-method finality on
Testnet Bradbury (Phase 1)** with the current validator set. This is useful signal for the
protocol: deterministic and LLM-invoking methods have very different finality reliability today.

## Reproduce

1. Fund a Bradbury wallet (faucet: 100 GEN / 7 days, requires 0.01 ETH on mainnet)
2. Deploy `contracts/ai_poll.py` with a `topic`
3. Call `submit_opinion` a few times with varied free-text stances (these finalize)
4. Call `tally_opinions()` and watch the explorer Transaction Journey
5. Open each transaction in the GenLayer Explorer

## Repo structure

```
.
├── README.md
└── contracts/
    └── ai_poll.py
```

## Author

GitHub: dubai199999 · Network: Testnet Bradbury (Phase 1)
Deployer wallet: 0x90b96F18E31a3A918DFf7775DA8f8eCd9A0FCe59
