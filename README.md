# Foundry-zksync verify contracts

> Project created with foundry-zksync

This project contains information to debug verification errors for contracts compiled and deployed using foundry-zksync, in the ZKsync explorer.

## Commands

- `RUST_LOG=trace forge build --zksync --build-info 2>&1 | sed -r "s/\x1b\[[0-9;]*m//g" > compile.log`: Compiles contracts and saves log. Change file name at the end as desired.
- `forge create --rpc-url https://sepolia.era.zksync.dev --private-key YOUR-PRIVATE-KEY contracts/Counter.sol:Counter --zksync`: deploys contract to ZKsync sepolia testnet
- After compilation and deployment, try to verify contract by sending an API request (details below).

## solc vs zkVM solc

Foundry-zksync currently does not use [the solc fork created by MatterLabs](https://github.com/matter-labs/era-solidity), also known as **zkVM solc compiler**. This fork was created to  addresses limitations when using older versions of Solidity ([read more here](https://docs.zksync.io/build/tooling/hardhat/hardhat-zksync-solc#zksync-era-solidity-compiler)).

## Successful verification

1. Extract the compiler input params by running the `forge build --zksync` command with `RUST_LOG=trace` as mentioned above.
2. Search `TRACE zksync_compile_with:compile: foundry_compilers::compilers::zksolc: input={"language":"Solidity"` in the log file to find the compiler input.
3. Use the `verification-request-template.json` and paste the compiler input inside the `sourceCode`.
4. Double check that the `compilerSolcVersion`, `compilerZksolcVersion` and other params are correct and match the compiler settings in your `foundry.toml`.
5. Send the verfication request via POST to the explorer API:
   - ZKsync Sepolia explorer API: https://explorer.sepolia.era.zksync.dev/contract_verification
   - ZKsync Mainnet explorer API: https://zksync2-mainnet-explorer.zksync.io/contract_verification
6. Example successful verification requests:
   - [Request for contract compiled with 0.8.19](./OK-verification-request-0.8.19.json) deployed in [0x5aD9100F05E250F5C457A32503a65380E05541E8](https://sepolia.explorer.zksync.io/address/0x5aD9100F05E250F5C457A32503a65380E05541E8).
   - [Request for contract compiled with 0.8.24](./OK-verification-request-0.8.24.json) deployed in [0xD09e794E6E8DCb25Fd0cf96D0B6925202DdF0ee0](https://sepolia.explorer.zksync.io/address/0xD09e794E6E8DCb25Fd0cf96D0B6925202DdF0ee0).

## Verification errors

The ZKsync explorer API requires the a request body similar to this one: [verification-request](./verification-request-template.json)

To test the verification, I've extracted the compiler input from the log `compile-out.log` and generated the [verification request](./verification-request-body.json). This are sent as a POST request to the explorer API endpoint `https://explorer.sepolia.era.zksync.dev/contract_verification`. 

### Error 1: Passing zkVM solc instead of solc

Using solc 0.8.19 in the `foundry.toml` file. The `Counter.sol` contract was compiled and deployed to https://sepolia.explorer.zksync.io/address/0x5aD9100F05E250F5C457A32503a65380E05541E8. I sent the verification request [verification-request-0.8.19](./verification-request-0.8.19.json). The verification ID generated was 23552 which resulted in error `Deployed bytecode is not equal to generated one from given source`. Compiler parameters were extracted from the [compiler log](./compile-0.8.19.log), but **verification included zksolc compiler version instead of vanilla solc, which caused the issue**.
