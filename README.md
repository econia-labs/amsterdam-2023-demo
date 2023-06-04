# Econia Workshop: Aptos Hack Holland Amsterdam Hackathon

*June 5, 2023*

- [Econia Workshop: Aptos Hack Holland Amsterdam Hackathon](#econia-workshop-aptos-hack-holland-amsterdam-hackathon)
  - [Resources](#resources)
    - [More documentation](#more-documentation)
    - [Repos](#repos)
  - [User setup](#user-setup)
  - [Econia setup](#econia-setup)
  - [Faucet setup](#faucet-setup)
  - [Market account registration](#market-account-registration)
  - [Trading](#trading)

## Resources

- This walkthrough [is on GitHub](https://github.com/econia-labs/amsterdam-2023-demo) at https://github.com/econia-labs/amsterdam-2023-demo!

### More documentation

- Econia Labs' [Teach Yourself Move on Aptos](https://github.com/econia-labs/teach-yourself-move) guide
- [Econia Documentation](https://econia.dev/)
- [Econia Move DocGen files](https://github.com/econia-labs/econia/tree/main/src/move/econia/doc)
- [Econia API docs](https://docs.econia.exchange)
- [Econia Move scraps](https://github.com/econia-labs/move-scraps)
- [Aptos Documentation](https://aptos.dev/)

### Repos

- [Econia](https://github.com/econia-labs/econia)
- [Econia indexer](https://github.com/econia-labs/aptos-core/tree/econia) ([`aptos-core`](https://github.com/aptos-labs/aptos-core) fork)
- Econia Labs' [`optivanty`](https://github.com/econia-labs/optivanity) tool
- [Econia reference front end](https://github.com/econia-labs/econia/tree/main/src/typescript/frontend)

## User setup

1. [Install the Aptos CLI](https://aptos.dev/tools/install-cli/), ideally through `brew`:

   ```bash
   brew install aptos
   ```

1. Verify you are on version `v2.0.0` or higher:

   ```bash
   aptos -h
   ```

1. Navigate to your home directory:

   ```bash
   cd ~
   ```

1. Use the CLI to initialize two devnet users:

   ```bash
   aptos init --profile user1
   ```

   ```bash
   aptos init --profile user2
   ```

1. Store their addresses in shell variables, and **make sure to use a leading `0x`** (you'll probably have to type it manually before you copy-paste the output from the `aptos init` call):

   ```bash
   # Make sure to use a leading 0x
   # Your addresses should vary
   user1=0xe5ef0125f05a2cd2bd4b68820775ac7ad6907968d6ca9b795c9cf8d67891a6d5
   user2=0xe1b1f5d8d3b3f80375b8a721bdbb59011570dd9b0816b3875d378ee864a0652a
   ```

1. Look up their account resources on the [devnet explorer](https://explorer.aptoslabs.com/?network=devnet):

   ```mermaid

   flowchart TD

   user1 --> account1[Account]
   user1 --> apt_coinstore1[CoinStore]
   apt_coinstore1 --> apt1[Aptos Coin]

   user2 --> account2[Account]
   user2 --> apt_coinstore2[CoinStore]
   apt_coinstore2 --> apt2[Aptos Coin]

   ```

## Econia setup

1. Navigate to a directory where you would like to download the Econia repo to:

   ```bash
   cd my/desired/path
   ```

1. Clone the Econia repo and navigate to the Econia Move package:

   ```bash
   mkdir econia_demo
   cd econia_demo
   git clone https://github.com/econia-labs/econia.git
   cd econia/src/move/econia
   ```

1. Update the `Move.toml` manifest to have a generic `econia` named address and an Aptos devnet dependency:

   ```toml
   [addresses]
   econia = "_"

   ...

   [dependencies.AptosFramework]
   git = "https://github.com/aptos-labs/aptos-core.git"
   rev = "devnet"
   ...
   ```

1. Publish Econia under `user1`'s account (note this might take awhile if you haven't already downloaded the Aptos repo, which is a dependency).
   Use the `y` key to accept the transaction once the simulator has provided gas estimates:

   ```bash
   aptos move publish \
       --named-addresses econia=$user1 \
       --included-artifacts none \
       --profile=user1
   ```

   ```mermaid

   flowchart TD

   user1 --> account[Account]
   user1 --> apt_coinstore[CoinStore]
   apt_coinstore --> apt[Aptos Coin]
   user1 ---> Econia
   Econia --> assets
   Econia --> avl_queue
   Econia --> ...
   Econia --> user

   ```

1. Store `user1`'s address as the `econia` address:

   ```bash
   econia=$user1
   ```

1. Lower the fee to register a market (this is used to mitigate denial-of-service attacks on mainnet):

   ```bash
   aptos move run \
       --function-id $econia::incentives::update_incentives \
       --type-args 0x1::aptos_coin::AptosCoin \
       --args \
           u64:1 \
           u64:1 \
           u64:1 \
           u64:5000 \
           u64:"[[10000,0,7],[8333,1,6],[7692,2,5],[7143,3,4],[6667,4,3],[6250,5,2],[5882,6,1]]" \
       --profile user1
   ```

## Faucet setup

1. Navigate to the faucet package:

   ```bash
   cd ../faucet
   ```

1. Publish the faucet under `user2`'s account:

   ```
   aptos move publish \
       --named-addresses econia=$user1,econia_faucet=$user2 \
       --profile=user2
   ```

   ```mermaid

   flowchart TD

   user2 --> account[Account]
   user2 --> apt_coinstore[CoinStore]
   apt_coinstore --> apt[Aptos Coin]
   user2 ---> EconiaFaucet
   EconiaFaucet --> faucet
   EconiaFaucet --> test_usdc
   EconiaFaucet --> test_eth

   ```

1. Store `user2`'s address as the `faucet` address:

   ```bash
   faucet=$user2
   ```

1. Mint test USDC to `user1`'s account:

   ```bash
   aptos move run \
       --function-id $faucet::faucet::mint \
       --args u64:1234567890 \
       --type-args $faucet::test_usdc::TestUSDC \
       --profile user1
   ```

   ```mermaid

   flowchart TD

   user1 --> account[Account]
   user1 --> apt_coinstore[CoinStore]
   apt_coinstore --> apt[Aptos Coin]
   user1 ---> Econia
   Econia --> assets
   Econia --> avl_queue
   Econia --> ...
   Econia --> user
   user1 --> usdc_coinstore[CoinStore]
   usdc_coinstore --> usdc[Test USDC]
   ```

## Market account registration

1. Have `user2` register an `APT`/`tUSDC` market (lot size 0.01 `APT`, tick size 0.001 `USDC`, minimum size 0.05 `APT`):

   ```bash
   aptos move run \
       --function-id \
           $econia::market::register_market_base_coin_from_coinstore \
       --type-args \
           0x1::aptos_coin::AptosCoin \
           $faucet::test_usdc::TestUSDC \
           0x1::aptos_coin::AptosCoin \
       --args \
           u64:1000000 \
           u64:1000 \
           u64:5 \
       --profile user2
   ```

1. Have `user1` register a market account:

   ```bash
   aptos move run \
       --function-id \
           $econia::user::register_market_account \
       --type-args \
           0x1::aptos_coin::AptosCoin \
           $faucet::test_usdc::TestUSDC \
       --args \
           u64:1 \
           u64:0 \
       --profile user1
   ```

1. Deposit `tUSDC` to `user1`'s market account:

   ```bash
   aptos move run \
       --function-id \
           $econia::user::deposit_from_coinstore \
       --type-args $faucet::test_usdc::TestUSDC \
       --args \
           u64:1 \
           u64:0 \
           u64:1234567890 \
       --profile user1
   ```

   ```mermaid

   flowchart TD

   user1 --> account[Account]
   user1 --> apt_coinstore[CoinStore]
   apt_coinstore --> apt[Aptos Coin]
   user1 ---> Econia
   Econia --> assets
   Econia --> avl_queue
   Econia --> ...
   Econia --> user
   user1 --> usdc_coinstore[CoinStore]
   usdc_coinstore --> usdc[Test USDC]
   user1 --> market_account[Market account]
   market_account --> usdcma[Test USDC]
   ```

## Trading

1. Have `user1` place a bid for 0.5 `APT` at a price of 10.50 `tUSDC` per `APT`:

   ```bash
   aptos move run \
       --function-id \
           $econia::market::place_limit_order_user_entry \
       --type-args \
           0x1::aptos_coin::AptosCoin \
           $faucet::test_usdc::TestUSDC \
       --args \
           u64:1 \
           address:"0x0" \
           bool:false \
           u64:5 \
           u64:105 \
           u8:0 \
           u8:0 \
       --profile user1
   ```

1. Have `user2` submit a swap sell of 0.5 `APT`, filling directly into a new `tUSDC` coinstore:

   ```bash
   aptos move run \
       --function-id \
           $econia::market::swap_between_coinstores_entry \
       --type-args \
           0x1::aptos_coin::AptosCoin \
           $faucet::test_usdc::TestUSDC \
       --args \
           u64:1 \
           address:"0x0" \
           bool:true \
           u64:0 \
           u64:50000000 \
           u64:0 \
           u64:124567890 \
           u64:0 \
       --profile user2
   ```

   ```mermaid

   flowchart TD

   user2 --> account[Account]
   user2 --> apt_coinstore[CoinStore]
   apt_coinstore --> apt[Aptos Coin]
   user2 ---> EconiaFaucet
   EconiaFaucet --> faucet
   EconiaFaucet --> test_usdc
   EconiaFaucet --> test_eth
   user2 --> tusdc_coinstore[CoinStore]
   tusdc_coinstore --> tusdc[Test USDC]
   ```
