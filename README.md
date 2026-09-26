# Trackrecord

**A prediction agent's record that nobody, including the agent, can pad.**

Every forecasting agent advertises a hit rate, and none of them can be checked. Losses get
quietly dropped, wins get backfilled, and calls that were never made appear afterwards. The
record is written by the party it flatters.

Live: https://trackrecord-rho.vercel.app

## The mechanism

A hold invoice inverts who controls settlement. Normally the payee knows the preimage behind
their own invoice and can claim payment whenever it arrives. Here the **grader** picks the
preimage and hands the agent only its hash:

```
grader:  preimage P,  payment_hash H = SHA256(P)
agent:   issues a BOLT-11 invoice carrying H, and does not know P
```

The agent has issued an invoice it cannot settle. When the market resolves, the grader releases
`P` for calls that turned out right and does nothing at all for the rest. There is no punishment
step — a wrong call is simply an invoice with no preimage.

Three consequences, all demonstrated on the page:

- **Getting paid is the proof.** A settled invoice can only exist if the grader released the
  secret, which it only does for correct calls.
- **A win cannot be fabricated.** Claiming one means presenting a preimage that hashes to that
  invoice's payment hash. The page lets the agent try, with an invented preimage, and the audit
  flags exactly that row.
- **A loss cannot be hidden.** Its absence is as visible as a win's presence, because the record
  is the full set of invoices, not a list the agent curates.

The reader verifies the whole record themselves by hashing each preimage. They ask nobody.

## Running it

Static. No build, no server, no node, no wallet.

```
python3 -m http.server 8000
```

Then open `http://localhost:8000`. Append `?demo` to run the whole flow automatically.

## What is implemented

Hold-invoice semantics where the payment hash is supplied by a third party, BOLT-11 construction
on testnet with a real recoverable ECDSA signature, preimage-to-hash verification of every
receipt, and an audit anyone can re-run.

Not implemented: routing or settling a real Lightning payment, HTLC timeouts and refunds, and the
grader's own accountability, which is assumed here. No node is contacted and no payment is made,
so this page cannot move a satoshi on any network.

## Verification

The page's own script is loaded into Node behind a DOM shim, so the tests drive the shipped code
rather than a reimplementation. 21 assertions, including the ones that carry the claim:

```
PASS  none is settleable at this point
PASS  claimed wins all verify                            3/3/0
PASS  the fabricated win is flagged
PASS  exactly one row is unverifiable                    1
PASS  the claim count rose but the verified count did not  3->4 claimed, 3->3 verified
PASS  over six more rounds, no honest receipt ever fails   0 bad rounds
```

The fifth is the point of the product: padding the record moves the *claimed* number and leaves
the *provable* one exactly where it was.

## Dependencies

`@noble/secp256k1` v2, vendored, for the invoice signature. WebCrypto for SHA-256 and
HMAC-SHA256. bech32 and BOLT-11 written for this project. No framework, no build step.

## Licence

MIT.
