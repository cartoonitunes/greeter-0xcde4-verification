# Greeter (0xcde4de4d) - Frontier Day 1 Bytecode Verification

Byte-for-byte bytecode verification of the Greeter contract deployed at block 49,392 on August 7, 2015 - Ethereum Frontier Day 1.

## Contract Details

| Field | Value |
|---|---|
| Address | `0xcde4de4d3baa9f2cb0253de1b86271152fbf7864` |
| Block | 49,392 |
| Deployer | `0xA1E4380A3B1f749673E270229993eE55F35663b4` |
| Creation TX | `0xef1b643d0cfb01321dfda1115fe8fb3181d9b592a79dfb75d1281117180d2d65` |
| Runtime bytecode | 546 bytes (exact match) |
| Creation bytecode | 717 bytes (exact match) |

## Self-Destruct Note

This contract has self-destructed - `kill()` was called by the owner. Verification is against the creation TX input, which is immutable on-chain. The bytecode cannot change retroactively; the creation TX is the permanent record.

## Source Code

```solidity
contract mortal {
    address owner;
    function kill() { if (msg.sender == owner) suicide(owner); }
}
contract greeter is mortal {
    string greeting;
    function greeter(string _greeting) {
        greeting = _greeting;
    }
    function mortal() { owner = msg.sender; }
    function kill() { if (msg.sender == owner) suicide(owner); }
    function greet() constant returns (string) {
        return greeting;
    }
}
```

### Source Code Explanation

This is the canonical `mortal + greeter` inheritance pattern from the Ethereum Frontier tutorial, with one key difference: `mortal()` is explicitly re-declared as a callable function inside `greeter` (not just a constructor). This exposes the function selector `f1eae25c`, allowing anyone to invoke `mortal()` and reset the owner to `msg.sender` at any time.

Function order matters for bytecode matching. The dispatch table is built in declaration order: `mortal()` then `kill()` then `greet()`.

### Function Selectors

| Function | Selector |
|---|---|
| `kill()` | `41c0e1b5` |
| `greet()` | `cfae3217` |
| `mortal()` | `f1eae25c` |

## Compiler

- **Compiler:** soljson v0.1.1+commit.6ff4cd6a (JavaScript/emscripten build)
- **Optimizer:** OFF

## Reproduce

Install the compiler:

```bash
npm install soljson-v0.1.1
```

Compile and extract bytecode:

```bash
node -e "var s=require('soljson-v0.1.1');var c=s.cwrap('compileJSON','string',['string','number']);console.log(JSON.parse(c(require('fs').readFileSync('greeter.sol','utf8'),0)).contracts.greeter.bytecode)"
```

Compare the output against the creation TX input data (strip the leading `0x`).

## Historical Context

The deployer (`0xA1E4380A`) received 2000 ETH at genesis - a substantial allocation indicating a core developer or very early insider. Before settling on this contract, they deployed 3 identical pre-Greeter drafts in a 7-minute window (blocks 49,157 to 49,186), demonstrating live iteration on the Frontier network. They then deployed this final version at block 49,392 and tested it by calling `greet()` at block 49,439. Later, at block 116,219, they sent 800 ETH to the Augur crowdsale.

The 3 failed drafts before this version show a developer learning in real time on the live network - a candid snapshot of the Frontier developer experience on Day 1.

## Verification

Part of [Awesome Ethereum Proofs](https://github.com/cartoonitunes/awesome-ethereum-proofs) and [Ethereum History](https://ethereumhistory.com/contract/0xcde4de4d3baa9f2cb0253de1b86271152fbf7864).
