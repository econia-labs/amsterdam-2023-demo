# Econia Workshop: Aptos Hack Holland Amsterdam Hackathon

*June 5, 2023*

- [Econia Workshop: Aptos Hack Holland Amsterdam Hackathon](#econia-workshop-aptos-hack-holland-amsterdam-hackathon)
  - [Resources](#resources)
    - [More documentation](#more-documentation)
    - [Repos](#repos)
  - [Account setup](#account-setup)
  - [Econia setup](#econia-setup)
  - [Faucet setup](#faucet-setup)
  - [Market account registration](#market-account-registration)
  - [Trading](#trading)

## Resources

- This walkthrough [is on GitHub](https://github.com/econia-labs/amsterdam-2023-demo) at https://github.com/econia-labs/amsterdam-2023-demo!
  - Pro tip:
    Search "github econia amsterdam" in your favorite search engine

### More documentation

- Econia Labs' [Teach Yourself Move on Aptos](https://github.com/econia-labs/teach-yourself-move) guide
- [Econia Documentation](https://econia.dev/)
- [Econia Move DocGen files](https://github.com/econia-labs/econia/tree/main/src/move/econia/doc)
- [Econia API docs](https://docs.econia.exchange)
- [Aptos Documentation](https://aptos.dev/)

### Repos

- [Econia](https://github.com/econia-labs/econia)
- [Econia indexer](https://github.com/econia-labs/aptos-core/tree/econia) ([`aptos-core`](https://github.com/aptos-labs/aptos-core) fork)
- Econia Labs' [`optivanty`](https://github.com/econia-labs/optivanity) tool
- [Econia reference front end](https://github.com/econia-labs/econia/tree/main/src/typescript/frontend)
- [Econia Move scraps](https://github.com/econia-labs/move-scraps)

## Account setup

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

1. Use the CLI to initialize a new devnet user named `user1`

   ```bash
   aptos init --profile user1
   ```

1. Type `devnet` when prompted for a network:

   ```bash
   devnet
   ```

1. Press `return` when prompted for a private key:

   ```
   <return>
   ```

1. Store `user1`'s address in a shell variable, and **make sure to use a leading `0x`** (you'll probably have to type `0x` manually before you copy-paste the output from the `aptos init` call):

   ```bash
   user1=0x<copy-paste-your-address-here>
   ```

1. Repeat for `user2`, `econia`, and `faucet` profiles:

   ```bash
   aptos init --profile user2
   ```

   ```bash
   aptos init --profile econia
   ```

   ```bash
   aptos init --profile faucet
   ```

1. Then verify you have stored each address as a shell variable with a leading `0x`:

   ```bash
   echo $user1
   echo $user2
   echo $econia
   echo $faucet
   ```

   ```bash
   0x445953fa27471d07027d2d8f0a87fc18e8f9d05bb9a7d673a2269ec2616267aa
   0xd572f6c2c9e7df6be78bdfff15bc72d56339b6226f3857d60eef1b9f99a2275f
   0xd4ba9f1c60a94f9a8a1349f89267c532195912392938c89f7cd91359aad28cee
   0xaaa85c7db25681d9f200e8ddd2bd10eeade20e8af69ae717063ea3405ef04fd2
   ```

1. Look up `user1` and `user2`'s account resources on the [devnet explorer](https://explorer.aptoslabs.com/?network=devnet):

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

1. Look up the `econia` account's modules section on the [devnet explorer](https://explorer.aptoslabs.com/?network=devnet):

   ```bash
   echo $econia
   ```

   ```mermaid

   flowchart TD

   econia
   ```

1. Publish Econia under the `econia` account (note this might take awhile if you haven't already downloaded the Aptos repo, which is a dependency).
   Use the `y` key to accept the transaction once the simulator has provided gas estimates:

   ```bash
   aptos move publish \
       --named-addresses econia=$econia \
       --included-artifacts none \
       --profile econia
   ```

   ```mermaid

   flowchart TD

   econia --> Econia:::new
   Econia --> assets:::new
   Econia --> avl_queue:::new
   Econia --> incentives:::new
   Econia --> market:::new
   Econia --> registry:::new
   Econia --> resource_account:::new
   Econia --> tablist:::new
   Econia --> user:::new

   classDef new fill:green
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
       --profile econia
   ```

## Faucet setup

1. Navigate to the faucet package:

   ```bash
   cd ../faucet
   ```

1. Look up the `faucet` account's modules section on the [devnet explorer](https://explorer.aptoslabs.com/?network=devnet):

   ```bash
   echo $faucet
   ```

   ```mermaid

   flowchart TD

   faucet
   ```

1. Publish the faucet under the `faucet` account:

   ```
   aptos move publish \
       --named-addresses econia=$econia,econia_faucet=$faucet \
       --profile faucet
   ```

   ```mermaid

   flowchart TD

   faucet_addr[faucet] --> EconiaFaucet:::new
   EconiaFaucet --> faucet:::new
   EconiaFaucet --> test_usdc:::new
   EconiaFaucet --> test_eth:::new

   classDef new fill:green
   ```

1. Mint test USDC to `user1`'s account, generating a new `CoinStore` with `tUSDC` inside:

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
   user1 --> usdc_coinstore[CoinStore]:::new
   usdc_coinstore --> usdc[Test USDC]:::new

   classDef new fill:green
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
   user1 --> usdc_coinstore[CoinStore]
   usdc_coinstore --> usdc[Test USDC]
   user1 --> market_account[Market account]:::new
   market_account --> usdcma[Test USDC]:::new

   classDef new fill:green
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
   user2 --> tusdc_coinstore[CoinStore]:::new
   tusdc_coinstore --> tusdc[Test USDC]:::new

   classDef new fill:green
   ```
