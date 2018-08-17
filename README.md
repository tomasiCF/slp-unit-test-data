# slp-unit-test-data
### Test vectors for ensuring that validators of the Simple Ledger Protocol (SLP) follow consensus.

Consensus validity is an extremely important and sensitive topic with SLP. If one validator finds a transaction to be 'valid' and another implementation says 'invalid', the consequences are catastrophic. Since the validity of future descendant transactions will almost always be contingent on this validity judgement, this leads to a spreading infection of uncertain validity that can only grow in size, and in principle grow to affect the entire set of tokens in existence. This catastrophe is much worse than can happen with regular bitcoins, since SLP transactions get committed to the blockchain regardless.

Such a consensus break could start somewhere extremely small, such as if one validator allows a certain field to have hex value 0001 or 01, but another validator allows only 01. Such differences can be trivially exploited by malicious actors to cause the consensus catastrophe. We therefore provide a considerable set of unit tests and ask that every validator makes absolutely sure they are in agreement with these unit tests.

## Overview of tests

**Link to SLP1 specification: [https://github.com/simpleledger/protocol-spec/blob/master/Token_Type_1.md](https://github.com/simpleledger/protocol-spec/blob/master/Token_Type_1.md)**

Upon release of SLP1 (SLP protocol with `token_type` = 1), the specification will be maintained at the link above in a relatively frozen state, only updated when necessary to address an ambiguity. By design the protocol can never be upgraded, rather to make changes it is required to choose an entirely new `token_type` --- effectively creating an entirely new protocol.

These tests focus on SLP1. Part A is to test that all validators correctly accept / discard various forms of OP_RETURN messages. Part B is a test on full transactions with an emphasis on checking.

## Part A: Parsing of OP_RETURN scripts

**File: [script_tests.json](script_tests.json)**

The SLP specification demands a number of very particular formatting requirements on the OP_RETURN script. It is absolutely essential that every implementation is on the same page with parsing.

[script_tests.json](script_tests.json) includes a series of script examples and the expected pass/fail result. For every `script` entry, there is a human-readable `msg` that explains the point of the test, its construction, and also whether it must pass or must fail. The `code` attribute is `null` for all valid messages, and otherwise is an integer representing a failure mode.

Note that it is not required for consensus that implementations agree on the reason, however we strongly recommend that validator unit tests should use this code to ensure the invalidation reason matches the expected reason.

### Invalidation reason codes

Overall script error codes:
* 1 - Basic error that prevents even parsing as a bitcoin script (currently, only for truncated scripts).
* 2 - Disallowed opcodes (after OP_RETURN, only opcodes 0x01 - 0x4e may be used).
* 3 - Not an SLP message (script is not OP_RETURN, or lacks correct lokad ID). Note in some implementations, the parsers would never be given such non-SLP scripts in the first place. In such implementations, error code 3 tests may be skipped as they are not relevant.

Reasons relating to the specified limits:
* 10 - A push chunk has the wrong size.
* 11 - A push chunk has an illegal value.
* 12 - A push chunk is missing / there are too few chunks.

Specific to one particular transaction type:

* 21 - Too many optional values (currently, only if there are more than 19 token output amounts)

Other:
* 255 - The SLP token_type field is has an unsupported value. Depending on perspective this is not 'invalid' per se -- it simply cannot be parsed since the protocol for that token_type is not known. Like error code 3 these tests may not be applicable to some parsers.


## Part B: Transaction input tests

**File: [tx_input_tests.json](tx_input_tests.json)**

These tests involve providing raw transactions A,B,C,D,... that form inputs to transaction T. The SLP validity judgement (valid/invalid) of A,B,C,D,... is given as a test assumption, and the test is to determine the validity of T.

(to be expanded and created)

* Enforcement of consensus rules on transaction format: OP_RETURN message placement, amount, and forbidding of multiple OP_RETURNs.
* Tests that prove that the OP_RETURN is being understood in the correct manner.
* Tests of correct integer overflow handling in token amount summation.
* Tests of input/output summation.
* Tests involving invalid inputs.
* Tests involving SLP inputs of mismatched token_type.
* Tests involving SLP inputs of mismatched token_id.
* Tests of mint baton passing (passing baton to MINT or to SEND).
* ...
