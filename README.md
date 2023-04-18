[![dependency status](https://deps.rs/repo/github/runziggurat/crunchy/status.svg)](https://deps.rs/repo/github/runziggurat/crunchy)

# crunchy
P2P network crawler data cruncher for graph metrics


# Running

`crunchy` gets three command line parameters: an input file, and an output file and ip cache file. All of them are `JSON` files.

The input file is a sample generated by our zcash crawler. Its format (a JSON-RPC response) corresponds to this:


```
{
    jsonrpc: String,
    result: {
        num_known_nodes: usize,
        num_good_nodes: usize,
        num_known_connections: usize,
        num_versions: usize,
        protocol_versions: HashMap<u32, usize>,
        user_agents: HashMap<String, usize>,
        crawler_runtime: Duration,
        node_addrs: Vec<SocketAddr>,
        nodes_indices: NodesIndices,
    }
    id: usize,
}
```

The generated output contains processed data that our renderer can directly use. It looks like this:

```
{
    elapsed: f64,
    nodes: [
        addr: SocketAddr,
        betweenness: f64,
        closeness: f64,
        connections: Vec<usize>,
        geolocation: Option<GeoInfo>
    ]
}
```
Explaination of the node fields:

- `addr`: the address as a dotted quad, with port number
- `betweenness`: the computed betweenness
- `closeness`: the computed closeness
- `connections`: an array of indices corresponding to the connected nodes.
- `geolocation`: used for latitude, longitude, city, country

### Command Line

```
Usage: ziggurat-crunchy [OPTIONS]

Options:
  -i, --input-sample <INPUT_SAMPLE>    Input file with sample data to process (overrides input from config file)
  -o, --out-state <OUT_STATE>          Output file with state of the graph (overrides output from config file)
  -g, --geocache-file <GEOCACHE_FILE>  Output file with geolocation cache (overrides cache from config file)
  -c, --config-file <CONFIG_FILE>      Configuration file path (if none defaults will be assumed)
  -f, --filter-type <FILTER_TYPE>      Optional node filtering by network type, e.g. 'Zcash'
  -h, --help                           Print help
  -V, --version                        Print version
```

The command line also prints the network parameters before and after applying the IPS algorithm output. Parameters are printed in the following format:

IPS is described in details [ips.md](doc/ips.md).

```
IPS algorithm started...
Checking for nodes connected to themselves...
172.218.177.148:16125 is connected to itself.
154.53.63.9:16125 is connected to itself.
146.59.124.83:16125 is connected to itself.
37.187.88.103:16125 is connected to itself.
54.37.141.22:16125 is connected to itself.

...
[nodes connected to themself list]
...

37.59.134.165:16125 is connected to itself.
142.167.129.14:16125 is connected to itself.
65.108.99.214:16125 is connected to itself.
Generating initial network state and its statistics...
Statistics for the initial network:
----------------------------------------
Nodes count: 6729

Degree measures:
Average: 1673.378510922871
Median: 1775
Min: 1, max: 2948, delta: 2947

Betweenness measures:
Average: 0.0006914887442478738
Median: 0.0007390585712156514
Min: 0, max: 0.002065442852915205, delta: 0.002065442852915205

Closeness measures:
Average: 2.0009819311242922
Median: 2.000098387783965
Min: 1.9989458610770605, max: 2.9992496962479973, delta: 1.0003038351709368

Eigenvector measures:
Average: 1.000000000000002
Median: 1.056610881384425
Min: 0.0005267475705698542, max: 1.7232354729355281, delta: 1.7227087253649582
----------------------------------------

Generated initial state and statistics in 383 s
IPS detected no islands
IPS detected no fragmentation possibility even when top nodes would be disconnected
The MCDA procedure is starting...
All IPS computations done in 412 s from IPS start
Statistics for the final network:
----------------------------------------
Nodes count: 6729

Degree measures:
Average: 1668.1619854361718
Median: 1776
Min: 2, max: 2910, delta: 2908

Betweenness measures:
Average: 0.0007233121796668225
Median: 0.0007800175457913357
Min: 0, max: 0.002065442852915205, delta: 0.002065442852915205

Closeness measures:
Average: 2.000953117674391
Median: 1.9996456008839623
Min: 1.9989458610770605, max: 2.9992496962479973, delta: 1.0003038351709368

Eigenvector measures:
Average: 1.0000000000000022
Median: 1.061306347054873
Min: 0.001138872735927244, max: 1.6923985711437681, delta: 1.691259698407841
----------------------------------------

Comparing if network parameters got changed on plus or minus:
Deltas for given statistics pair:
----------------------------------------
Nodes count: 0 (0.000%)

Degree measures:
Average: -5.216525486699311 (-0.312%)
Median: 1 (0.056%)
Min: 1 (100.000%), max: -38 (-1.289%), delta: -39 (-1.323%)

Betweenness measures:
Average: 0.00003182343541894867 (4.602%)
Median: 0.00004095897457568427 (5.542%)
Min: 0 (0.000%), max: 0 (0.000%), delta: 0 (0.000%)

Closeness measures:
Average: -0.00002881344990113277 (-0.001%)
Median: -0.0004527869000026108 (-0.023%)
Min: 0 (0.000%), max: 0 (0.000%), delta: 0 (0.000%)

Eigenvector measures:
Average: 0.0000000000000002220446049250313 (0.000%)
Median: 0.004695465670447874 (0.444%)
Min: 0.0006121251653573898 (116.208%), max: -0.03083690179176002 (-1.789%), delta: -0.031449026957117265 (-1.826%)
----------------------------------------

IPS has been working for 8848 seconds
```

Output can contain also information about some problems found, like nodes with assymetric connections, or nodes connected to themselves.

Example:
```
cargo run --release -- -i testdata/sample.json -o testdata/state.json -g testdata/geoip-cache.json
```
