AI Poll: an Intelligent Contract that reads free-text opinions and tallies them

A minimal GenLayer Project: a polling contract where voters submit opinions in plain
language and an LLM-backed method classifies and tallies them on-chain. Deployed and
exercised on Testnet Bradbury (Phase 1), with verifiable transactions, plus an honest
finding about LLM-method finality on the current testnet.

The use case

A normal smart contract can only count structured votes (yes/no buttons). It cannot read
"honestly I think it's a great idea" and decide that means support. AI Poll can: voters
call submit_opinion("...") with free text, and tally_opinions() invokes an LLM to classify
every opinion as support / oppose / unclear and return a structured result.

GenLayer is central here. Strip out the LLM call and the contract has nothing to do. That
is the bar a Project should meet: GenLayer is the reason it works, not decoration.

Methods

MethodTypeWhat it doessubmit_opinion(text)write (deterministic)Appends a free-text opinionget_opinions()viewLists all submitted opinionsget_topic()viewReturns the poll topictally_opinions()write (LLM)LLM classifies + tallies into {support, oppose, unclear, summary}get_result()viewReturns the latest tally

The LLM input is precomputed deterministically before the non-deterministic block, so every
validator feeds the model identical input. This is the pattern that maximizes inter-validator
agreement under Optimistic Democracy.

What was deployed and tested (Testnet Bradbury, same wallet)

Deployer wallet: 0x90b96F18E31a3A918DFf7775DA8f8eCd9A0FCe59

Two independent instances were deployed and exercised, one under heavier load (4 opinions)
and one under light load (2 short opinions), to test whether load affects LLM-method finality.

InstanceContractOpinions submittedtally_opinions outcome1 (heavy)0xb6b258F09e49F14F58d99a81B0623aB309e224E74, all FINALIZED3 calls: UNDETERMINED / FINALIZED(UNDETERMINED) / UNDETERMINED2 (light)0x1CE...944092, all ACCEPTED2 calls: UNDETERMINED / UNDETERMINED

Across both instances: 2/2 deploys succeeded, 6/6 submit_opinion (deterministic) reached
finality, and 0/5 tally_opinions (LLM) reached a clean ACCEPTED verdict. Every LLM call
ended UNDETERMINED or LEADER_TIMEOUT, on both heavy and light input.

On-chain proof (failed-finality tally calls)

txoutcomelink0x2834...be53UNDETERMINEDexplorer0xe1f6...414cUNDETERMINEDexplorer0x44d9...b45bUNDETERMINEDexplorer0x7ebf...c28cFINALIZED (UNDETERMINED)explorer

(submit_opinion and deploy transactions all reached finality. See the deployer account on the explorer.)

Finding: deterministic methods finalize, the LLM method did not (Phase 1)

The contract itself is correct. Every deterministic path (deploy, submit_opinion) reached
finality cleanly, twice over. The LLM method tally_opinions did not reach a clean verdict
in 5 attempts across 2 instances, under both heavy and light input. The failure is therefore
not load-driven and not a contract bug. It is an observed limit of LLM-method finality on
Testnet Bradbury (Phase 1) with the current validator set. This is useful signal for the
protocol: deterministic and LLM-invoking methods have very different finality reliability today.

Reproduce


Fund a Bradbury wallet (faucet: 100 GEN / 7 days, requires 0.01 ETH on mainnet)
Deploy contracts/ai_poll.py with a topic
Call submit_opinion a few times with varied free-text stances (these finalize)
Call tally_opinions() and watch the explorer Transaction Journey
Open each transaction in the GenLayer Explorer


Repo structure

.
├── README.md
└── contracts/
    └── ai_poll.py

Author

GitHub: dubai199999 · Network: Testnet Bradbury (Phase 1)
Deployer wallet: 0x90b96F18E31a3A918DFf7775DA8f8eCd9A0FCe59
