# Foundry-zksync verify contracts

> Project created with foundry-zksync

This project contains information to debug verification errors for contracts compiled and deployed using foundry-zksync, in the ZKsync explorer.

## Commands

- `RUST_LOG=trace forge build --zksync --build-info 2>&1 | sed -r "s/\x1b\[[0-9;]*m//g" > compile.log`: Compiles contracts and saves log. Change file name at the end as desired.
- `forge create --rpc-url https://sepolia.era.zksync.dev --private-key YOUR-PRIVATE-KEY contracts/Counter.sol:Counter --zksync`: deploys contract to ZKsync sepolia testnet
- After compilation and deployment, try to verify contract by sending an API request (details below).

## Verification errors

The ZKsync explorer API requires the a request body similar to this one: [verification-request](./verification-request-template.json)

To test the verification, I've extracted the compiler input from the log `compile-out.log` and generated the [verification request](./verification-request-body.json). This are sent as a POST request to the explorer API endpoint `https://explorer.sepolia.era.zksync.dev/contract_verification`. 

### Error 1: Using solc 0.8.19

Using solc 0.8.19 in the `foundry.toml` file. The `Counter.sol` contract was compiled and deployed to https://sepolia.explorer.zksync.io/address/0x5aD9100F05E250F5C457A32503a65380E05541E8. I sent the verification request [verification-request-0.8.19](./verification-request-0.8.19.json). The verification ID generated was 23552 which resulted in error `Deployed bytecode is not equal to generated one from given source`. Given that all compiler parameters are extracted from the [compiler log](./compile-0.8.19.log), it's strange that verification fails.

### Error 2: Using solc 0.8.24

Using solc 0.8.24 in the `foundry.toml` file. The `Counter.sol` contract was compiled and deployed to https://sepolia.explorer.zksync.io/address/0xD09e794E6E8DCb25Fd0cf96D0B6925202DdF0ee0. I sent the verification request [verification-request-0.8.24](./verification-request-0.8.24.json). The verification ID generated was 23551 which resulted in error `{"status":"failed","error":"Compilation error","compilationErrors":["Invalid EVM version requested."]}`.
