# clone-lin-compound-seize

**EXPERIMENTAL** LIN clone of Compound [`liquidateCalculateSeizeTokens`](https://github.com/compound-finance/compound-protocol/blob/a3214f67b73310d547e00fc578e8355911c9d376/contracts/Comptroller.sol) plus [`ExponentialNoError`](https://github.com/compound-finance/compound-protocol/blob/a3214f67b73310d547e00fc578e8355911c9d376/contracts/ExponentialNoError.sol) `mul_` / `div_` / `mul_ScalarTruncate` (BSD-3-Clause upstream).

This is **not** a Comptroller, not a Compound replacement, and not uint256. The kernel is uint64-scale with a 128-bit product for Exp math. Class: EXPERIMENTAL.

## Provenance

| Field | Value |
|---|---|
| Upstream | https://github.com/compound-finance/compound-protocol |
| Path | `contracts/Comptroller.sol` + `contracts/ExponentialNoError.sol` |
| Function | `liquidateCalculateSeizeTokens` |
| Commit | `a3214f67b73310d547e00fc578e8355911c9d376` |
| Comptroller sha256 | `89a4c77da4920d0d0e44d7591446fe15e6b0c8a4794646aa497f725dc72818d8` |
| Comptroller git blob | `1d94cc756d1b52b84e4cc2ed3996cd9a7074a4ab` |
| ExponentialNoError sha256 | `dd272760b2d9082db9c94f6198896402fe3681bf41fc3f9d8bba31ada290c3c5` |
| ExponentialNoError git blob | `23f833760db898ebdc6b2fef9654cb9bc1dd21c3` |
| License (upstream) | BSD-3-Clause |

Formula (Compound Exp truncation order):

```
num_m   = (incentive * priceBorrowed) / 1e18
den_m   = (priceCollateral * exchangeRate) / 1e18
ratio_m = (num_m * 1e18) / den_m
seize   = (ratio_m * actualRepayAmount) / 1e18
```

Canonical vector (recomputable): repay=1000, incentive=1.08e18, prices=1e18, exchangeRate=0.2e18 → **5400**. Zero oracle price → 0 fail-closed.

## Files

- `src/lin_compound_seize.lin` — scalar LIN kernel (128-bit schoolbook muldiv + explicit args)
- `test/compound_seize_c11.c` — independent C11 `__int128` / limb oracle
- `fixtures/ExponentialNoError.sol` — pinned upstream Exp math
- `fixtures/Comptroller_liquidateCalculateSeizeTokens.sol` — pinned function excerpt (substring of Comptroller.sol)
- `docs/PROVENANCE.rulel` — claims / non-claims
- `LICENSE.compound.txt` — upstream BSD-3-Clause notice

## Proofs live in lin-open

Results and the external harness stay in the LIN toolchain repo:

- Clone-lin: https://github.com/kbelludoo/clone-lin-compound-seize
- Results: https://github.com/kbelludoo/lin-open/tree/cursor/linguagem-lin-e-valida-o-1ef3/examples/compound_seize
- Harness: `python3 test/prove_compound_seize_external.py` (gcc == Python == `lin_c0` vm)
- Claim sheet: `docs/events/EVENT_COMPOUND_SEIZE_CLONE_LIN.rulel`

```bash
make -C transpile/c c0
./transpile/c/bin/lin_c0 vm src/lin_compound_seize.lin cls_test_suite
# value=1
./transpile/c/bin/lin_c0 vm src/lin_compound_seize.lin cls_seize 1000 1080000000000000000 1000000000000000000 1000000000000000000 200000000000000000
# value=5400
```
