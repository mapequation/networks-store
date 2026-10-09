# networks-store

Network files that have no public source: data processed in our own papers,
benchmark inputs that cannot be regenerated, and processed versions of public
data that we cannot reproduce from the raw data. Everything with a public source
is fetched from that source instead, through the
[`mapequation-networks`](https://github.com/mapequation/networks) package. That
package is also where generators live, so a synthetic network appears here only
when its generator cannot reproduce it.

Files are stored as they were used, byte for byte. `SHA256SUMS` pins every file;
check a fetched copy with `shasum -a 256 -c SHA256SUMS`.

The repository is private: some of these files are derived from third-party data
whose redistribution terms we have not checked.

## Contents

### `first-order/`

| file | network | used by |
|---|---|---|
| `netscicoauthor2010.net` | co-authorship among network scientists, 2010; 552 nodes, 1,318 weighted undirected edges (weights like 1/3) | Infomap columnar benchmark, undirected |
| `politicalblogs.net` | **Swedish** political blogs (party tags such as SD, MP in the labels); 1,046 nodes, 13,195 weighted links. Not the Adamic–Glance `polblogs` | Infomap columnar benchmark, run `-d` |
| `science2001.net` | journal citation network, 2001; 7,170 journals with node weights, 738,282 weighted links | Infomap columnar benchmark, run `-d` and `-d --preferred-number-of-modules 25` |

Where these three came from is not yet documented here.

### `memory/`

Rosvall et al., *Memory in network flows and its effects on spreading dynamics
and community detection*, Nature Communications 5, 4630 (2014). Supplementary
Note 1 describes the processing. Trigram files have one line per `i j k weight`.
A line `i i j w` is a self-memory trigram: it marks `w` pathways starting at `i`
and is used only for the teleportation weights.

| file | network | used by |
|---|---|---|
| `air2011/air2011-trigrams.txt` | US airline itineraries from DB1B (BTS), first three quarters of 2011: 464 airports, 335,111 trigrams, total weight 44,823,256 including the self-memory trigrams | source of `air30k.net` |
| `air2011/air30k.net` | second-order state network on the 183 airports kept in the paper: 13,213 state nodes, 264,989 links, total weight 24,072,258 | Infomap columnar benchmark: plain, `-d --regularized`, and with US-state metadata |
| `enron/enron-trigrams.txt` | Enron email threads: 146 users (144 present), 2,948 trigrams | — |
| `taxi/taxi-trigrams.txt` | Uber taxi trajectories in San Francisco on a hexagonal grid: 416 cells, 8,396 trigrams | — |

`air30k.net` is exactly `air2011-trigrams.txt` with the self-memory trigrams
dropped and every trigram restricted to the 183 airports in its `*Vertices`
list. Every link and weight agrees. Airport names are the BTS names with spaces
removed. No passenger threshold we tried on the trigram file (legs in, out or
both, itinerary origins, transfers) selects exactly these 183, so the list is
defined by the file.

The trigram file can be rebuilt from the public DB1B coupons almost, but not
exactly. `networks.paths.load("db1b-coupon", year=2011, quarter=q)` for
`q = 1, 2, 3` returns passenger-weighted itineraries. Turning each one into its
triples, plus one self-memory trigram per itinerary, gives the same 464 airports
and the same totals to within 0.0005%: total weight 44,823,463 against
44,823,256, and 19,414,511 path-start passengers against 19,415,369. (Weighted
by itinerary count instead of passengers, the totals are far off.) Still, 6,601
trigrams differ, and restricted to the 183 airports that makes 5,786 of 264,989
`air30k.net` links, moving 0.11% of the weight. Open-jaw gaps are not the cause:
the 2011 Q1–Q3 coupons contain none. The likeliest causes are later BTS
revisions or an unrecorded cleaning step. So `air30k.net` is kept here as the
exact file.

### `synthetic/overlapping-memory/`

Second-order networks with planted overlapping communities, from Andrea
Lancichinetti's trigram sampler in the higher-order regularization project:
`N = 256` physical nodes, communities of `nc = 64`, every node in `om` of them,
`E` trigrams, `mu = 0.1`. These five files were made before the sampler was
seeded, so they cannot be regenerated. Every other member of the family comes
from `networks.generate.overlapping_memory_benchmark(om, E, seed=1)`, which
reproduces the seeded files byte for byte.

| file | state nodes |
|---|--:|
| `network_N256_om2_nc64_E50000_mu10_sample1.net` | 28,203 |
| `network_N256_om4_nc64_E100000_mu10_sample1.net` | 45,394 |
| `network_N256_om5_nc64_E100000_mu10_sample1.net` | 50,133 |
| `network_N256_om6_nc64_E100000_mu10_sample1.net` | 53,860 |
| `network_N256_om8_nc64_E100000_mu10_sample1.net` | 58,505 |

Each `network_<stem>.net` has a `planted_partition_<stem>.clu` with lines
`state_id module`, modules from 0. Run them directed (`infomap -d`).
