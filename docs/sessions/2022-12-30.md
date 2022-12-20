## 2022-12-30

Display The Graph network statistics in a CLI table

### Data

The Graph Hosted Service API URL
* https://api.thegraph.com/subgraphs/name/graphprotocol/graph-network-mainnet/

GraphQL Query

``` gql
query MyQuery {
  graphNetworks {
    activeCuratorCount
    activeDelegatorCount
    activeSubgraphCount
    activeDelegationCount
    maxAllocationEpochs
    currentEpoch
  }
}
```

### Code

Create a new cargo project and open it with VSCode (or the editor of your choice)

``` bash
cargo new network_stats_frontend
cd network_stats_frontend
code .
```

Update the boilerplate files with the following code:

**Cargo.toml**

``` toml
[package]
name = "network_stats_frontend"
version = "0.1.0"
edition = "2021"

# See more keys and their definitions at https://doc.rust-lang.org/cargo/reference/manifest.html

[dependencies]
serde = { version = "1.0.149", features = ["derive"] }
reqwest = { version = "0.11", features =  ["json"]}
tokio = { version = "1.23.0", features = ["full"] }
cli-table = "0.4"
```

**src/main.rs**

``` rust
use std::collections::HashMap;
use std::string::String;
use serde::Deserialize;
use cli_table::{format::Justify, Cell, Style, Table};

#[derive(Debug, Deserialize, PartialEq)]
struct GraphNetworksResponse {
    data: HashMap<String, Vec<NetworkStats>>
}

#[allow(non_snake_case)]
#[derive(Debug, Deserialize, PartialEq)]
struct NetworkStats {
    activeCuratorCount: i32,
    activeDelegatorCount: i32,
    activeSubgraphCount: i32,
    activeDelegationCount: i32,
    maxAllocationEpochs: i32,
    currentEpoch: i32,
}

const NETWORK_SUBGRAPH_URL: &str = "https://api.thegraph.com/subgraphs/name/graphprotocol/graph-network-mainnet";
const NETWORK_SUBGRAPH_QUERY: &str = "{graphNetworks {activeCuratorCount activeDelegatorCount activeSubgraphCount activeDelegationCount maxAllocationEpochs currentEpoch}}";

#[tokio::main]
async fn main() -> Result<(), reqwest::Error> {
    let mut map = HashMap::new();
    map.insert("query", NETWORK_SUBGRAPH_QUERY);

    let graph_network_response: GraphNetworksResponse = reqwest::Client::new()
    .post(NETWORK_SUBGRAPH_URL)
    .json(&map)
    .send()
    .await?
    .json()
    .await?;

    let table = vec![
        vec!["Active curator count".cell(), graph_network_response.data["graphNetworks"][0].activeCuratorCount.cell().justify(Justify::Right)],
        vec!["Active delegator count".cell(), graph_network_response.data["graphNetworks"][0].activeDelegatorCount.cell().justify(Justify::Right)],
        vec!["Active subgraph count".cell(), graph_network_response.data["graphNetworks"][0].activeSubgraphCount.cell().justify(Justify::Right)],
        vec!["Active delegation count".cell(), graph_network_response.data["graphNetworks"][0].activeDelegationCount.cell().justify(Justify::Right)],
        vec!["Max allocation epochs".cell(), graph_network_response.data["graphNetworks"][0].maxAllocationEpochs.cell().justify(Justify::Right)],
        vec!["Current epoch".cell(), graph_network_response.data["graphNetworks"][0].currentEpoch.cell().justify(Justify::Right)],
    ]
    .table()
    .title(vec![
        "Network stat".cell().bold(true),
        "Value".cell().bold(true),
    ])
    .bold(true);

    let table_display = table.display().unwrap();

    println!("{}", table_display);

    Ok(())
} 
```

Run the app

``` bash
cargo run
```
