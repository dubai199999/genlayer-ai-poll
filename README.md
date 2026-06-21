# v0.2.16
# { "Depends": "py-genlayer:1jb45aa8ynh2a9c9xn3b7qqh8sm5q93hwfp7jqmwsfhh8jpz09h6" }
 
import json
 
from genlayer import *
 
 
# AI Poll: an Intelligent Contract that collects free-text opinions and uses an LLM
# to tally them into a structured result (support / oppose / unclear + a short summary).
#
# Why this needs GenLayer: a normal smart contract cannot read free-text human opinions
# and decide what they mean. Classifying "yeah I'm for it" vs "no way" vs "not sure" is a
# subjective judgement, exactly what GenLayer's LLM-backed consensus is for. Remove the
# LLM and this contract has no reason to exist.
class AiPoll(gl.Contract):
    topic: str
    opinions: DynArray[str]
    result: str
 
    def __init__(self, topic: str) -> None:
        self.topic = topic
        self.result = ""
 
    # --- deterministic methods (no LLM) ---
 
    @gl.public.write
    def submit_opinion(self, opinion: str) -> None:
        # Anyone can submit a free-text opinion on the topic. Deterministic, cheap.
        self.opinions.append(opinion)
 
    @gl.public.view
    def get_topic(self) -> str:
        return self.topic
 
    @gl.public.view
    def get_opinions(self) -> list[str]:
        return [o for o in self.opinions]
 
    @gl.public.view
    def get_result(self) -> str:
        # Returns the last tally produced by tally_opinions().
        return self.result
 
    # --- Intelligent method (calls the LLM, resolves via consensus) ---
 
    @gl.public.write
    def tally_opinions(self) -> None:
        # Precompute the LLM input deterministically BEFORE the non-deterministic block,
        # so every validator feeds the model identical input (raises inter-validator
        # agreement under Optimistic Democracy).
        opinions_json = json.dumps([o for o in self.opinions])
        topic = self.topic
 
        prompt_input = f"""
You are tallying a poll. The topic being voted on is:
"{topic}"
 
Here is the list of free-text opinions submitted, as a JSON array:
{opinions_json}
 
"""
 
        task = """Classify each opinion as one of: "support", "oppose", or "unclear",
based only on its stance toward the topic. Then produce the result.
 
Respond only using the following JSON format, nothing else.
Don't include any other words or characters, your output must be only JSON
without any formatting prefix or suffix, perfectly parsable by a JSON parser:
{{
"support": int,      // number of opinions in favor
"oppose": int,       // number of opinions against
"unclear": int,      // number that are ambiguous
"summary": str       // one short sentence summarizing the overall sentiment
}}
"""
 
        def get_tally_result() -> str:
            return (
                gl.nondet.exec_prompt(prompt_input + task)
                .replace("```json", "")
                .replace("```", "")
            )
 
        final_result = gl.eq_principle.strict_eq(get_tally_result)
 
        print("final_result: ", final_result)
        # Validate it parses as JSON before storing; store the raw JSON string.
        json.loads(final_result)
        self.result = final_result
 
