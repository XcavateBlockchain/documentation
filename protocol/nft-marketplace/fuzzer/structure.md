# Structure

Located in:

```
#[cfg(test)]
mod fuzz_tests;
```

Or optionally:

```
pub mod fuzzer;
```

#### Typical Components:

| Component                       | Description                                                         |
| ------------------------------- | ------------------------------------------------------------------- |
| `generate_random_nft()`         | Creates an NFT with randomized metadata, owner, and properties.     |
| `generate_random_listing()`     | Produces a listing with edge-case prices, durations, or currencies. |
| `apply_random_market_actions()` | Simulates purchases, transfers, delists, and bids randomly.         |
| `run_fuzz_market_loop(n: u32)`  | Executes a random set of market actions `n` times.                  |

