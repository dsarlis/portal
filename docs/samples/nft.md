# NFT Minting

This example demonstrates implementing an NFT canister. NFTs (non-fungible tokens) are unique tokens with arbitrary
metadata - usually an image of some kind - to form the digital equivalent of trading cards. There are a few different
NFT standards for the Internet Computer (e.g [EXT](https://github.com/Toniq-Labs/extendable-token), [IC-NFT](https://github.com/rocklabs-io/ic-nft)), but for the purposes of this tutorial we use [DIP-721](https://github.com/Psychedelic/DIP721). You can see a quick introduction on [YouTube](https://youtu.be/1po3udDADp4).

The canister is a basic implementation of the standard, with support for the minting, burning, and notification interface extensions.

The sample code is available in the [samples repository](https://github.com/dfinity/examples) in [Rust](https://github.com/dfinity/examples/tree/master/rust/dip721-nft-container) and Motoko is coming soon!
A running instance of the Rust canister for demonstration purposes is available as [t5l7c-7yaaa-aaaab-qaehq-cai](https://t5l7c-7yaaa-aaaab-qaehq-cai.ic0.app).
The interface is meant to be programmatic, but the Rust version additionally contains HTTP functionality so you can view a metadata file at `<canister URL>/<NFT ID>/<file ID>`.
It contains six NFTs, so you can look at items from `<canister URL>/0/0` to `<canister URL>/5/0`.

Command-line length limitations would prevent you from minting an NFT with a large file, like an image or video, via `dfx`. To that end,
there is a [command-line minting tool](https://github.com/dfinity/experimental-minting-tool) provided for minting simple NFTs.

## Ideas
The NFT canister is not very complicated since the [DIP-721](https://github.com/Psychedelic/DIP721) standard specifies mostly [CRUD](https://en.wikipedia.org/wiki/Create,_read,_update_and_delete) operations,
but we can still use it to explain three important concepts concerning dapp development for the Internet Computer:

### Stable Memory for Canister Upgrades
The Internet Computer employs [Orthogonal Persistence](../developer-docs/build/cdks/motoko-dfinity/motoko.md#orthogonal-persistence), so developers generally do not need to think a lot about storing their data.
When upgrading canister code, however, it is necessary to explicitly handle canister data. The NFT canister example shows how stable memory can be handled using `pre_upgrade` and `post_upgrade`.

### Certified Data
Generally, when a function only reads data (instead of modifying the state of the canister), it is
beneficial to use a [query call instead of an update call](https://smartcontracts.org/docs/current/concepts/canisters-code#query-and-update-methods).
But, since query calls do not go through consensus, [certified responses](https://smartcontracts.org/docs/current/developer-docs/build/security/general-security-best-practices#certify-query-responses-if-they-are-relevant-for-security)
should be used wherever possible. The HTTP interface of the Rust implementation shows how certified data can be handled.

### Delegating Control over Assets
For a multitude of reasons, users may want to give control over their assets to other identities, or even delete (burn) an item.
The NFT canister example contains all those cases and shows how it can be done.

## Approach
Since the basic functions required in [DIP-721](https://github.com/Psychedelic/DIP721) are very straightforward to implement, this section only discusses how the above ideas are handled/implemented.

### Stable Storage for Canister Upgrades
During canister code upgrades, memory is not persisted between different canister calls. Only memory in stable memory is carried over.
Because of that it is necessary to write all data to stable memory before the upgrade happens, which is usually done in the `pre_upgrade` function.
This function is called by the system before the upgrade happens. After the upgrade, it is normal to load data from stable memory into memory
during the `post_upgrade` function. The `post_upgrade` function is called by the system after the upgrade happened.
In case an error occurs during any part of the upgrade (including `post_upgdrade`), the entire upgrade is reverted.

The Rust CDK (Canister Development Kit) currently only supports one value in stable memory, so it is necessary to create an object that can hold everyhing you care about.
In addition, not every data type can be stored in stable memory; only ones that implement the [CandidType trait](https://docs.rs/candid/latest/candid/types/trait.CandidType.html)
(usually via the [CandidType derive macro](https://docs.rs/candid/latest/candid/derive.CandidType.html)) can be written to stable memory. 

Since the state of our canister includes an `RbTree` which does not implement the `CandidType`, it has to be converted into a data structure (in this case a `Vec`) that implements `CandidType`.
Luckily, both `RbTree` and `Vec` implement functions that allow converting to/from iterators, so the conversion can be done quite easily.
After conversion, a separate `StableState` object is used to store data during the upgrade.

### Certified Data
To serve assets via http over `<canister-id>.ic0.app` instead of `<canister-id>.raw.ic0.app`, responses have to
[contain a certificate](https://wiki.internetcomputer.org/wiki/HTTP_asset_certification) to validate their content.
Obtaining such a certificate can not happen during a query call since it has to go through consensus, so it has to be created during an update call.

A certificate is very limited in its content. At the time of writing, canisters can submit no more than 32 bytes of data to be certified.
To make the most out of that small amount of data, a `HashTree` (the `RbTree` from the previous section is also a `HashTree`) is used.
A `HashTree` is a tree-shaped data structure where the whole tree can be summarized (hashed) into one small hash of 32 bytes.
Whenever some content of the tree changes, the hash also changes. If the hash of such a tree is certified, it means that the content of the tree can be considered certified.
To see how data is certified in the NFT example canister, look at the function `add_hash` in `http.rs`.

For the response to be verified, it has to be checked that a) the served content is part of the tree, and b) the tree containing that content actually can be hashed to the certified hash.
The function `witness` is responsible for creating a tree with minimal content that still can be verified to fulfill a) and b).
Once this minimal tree is constructed, certificate and minimal hash tree are sent as part of the `IC-Certificate` header.

For a much more detailed explanation how certification works, see [this explanation video](https://internetcomputer.org/how-it-works/response-certification).

### Managing Control over Assets
[DIP-721](https://github.com/Psychedelic/DIP721) specifies multiple levels of control over the NFTs:
- Owner: This person owns an NFT. They can transfer the NFT, add/remove operators, or burn the NFT.
- Operator: Sort of a delegated owner. The operator does not own the NFT, but can do the same actions an owner can do.
- Custodian: Creator of the NFT collection/canister. They can do anything (transfer, add/remove operators, burn, and even un-burn) to NFTs, but also mint new ones or change the symbol or description of the collection.

The NFT example canister keeps access control in these three levels very simple: For every level of control, a separate list (or set) of principals is kept.
Those three levels are then manually checked every single time someone attempts to do something for which they require authorisation.
If a user is not authorised to call a certain function an error is returned.

Burning an NFT is a special case. To burn an NFT means to either delete the NFT (not intended in DIP-721) or to set ownership to `null` (or a similar value).
On the Internet Computer, this non-existing principal is called the [Management Canister](https://smartcontracts.org/docs/current/references/ic-interface-spec#the-ic-management-canister).
Quote from the link: "The IC management canister is just a facade; it does not actually exist as a canister (with isolated state, Wasm code, etc.)." and its address is `aaaaa-aa`.
Using this management canister address, we can construct its principal and set the management canister as the owner of a burned NFT.
