---
description: >-
  Contractum is a functional declarative programming language designed for
  developing smart contracts which run on Bitcoin and Lightning network using
  RGB technology.
layout: landing
---

# Contractum: smart-contract language

<table data-view="cards"><thead><tr><th></th><th></th></tr></thead><tbody><tr><td><h3>Declarative</h3></td><td>Define contract state and operations in a simple declarative form, which is easy to write and understand</td></tr><tr><td><h3>Strong-typed</h3></td><td>State and contract operations take only known structured types</td></tr><tr><td><h3>Abstract</h3></td><td>Leverage interfaces, type libraries, schemata and schema hierarchy for code-reuse and reducing risks of mistakes</td></tr></tbody></table>

## RGB smart contracts

Contractum is the language to write RGB contracts. RGB is a technology which allows creation of arbitrary-complex ("Turing-complete") smart contracts that run on bitcoin and, most imporangly, Lightning network. RGB contracts are confidential, scalable (up to the speed of the Lightning transactions, with small data footprint) and robust.

<table data-view="cards"><thead><tr><th></th><th></th></tr></thead><tbody><tr><td><h3>Confidential</h3></td><td><ul><li>No blockchain footprint</li><li>No chainanalysis possible: contract breaks transaction graphs</li><li>Extensive use of zero-knowledge (bulletproofs)</li></ul></td></tr><tr><td><h3>Scalable</h3></td><td><ul><li>Do not increase transaction size and no block space usage</li><li>Sharded; data kept by contract parties and not each node</li><li>Operating with the speed of Lightning Network</li></ul></td></tr><tr><td><h3>Turing-complete</h3></td><td><ul><li>Arbitrary complex business logic</li><li>Enables #BiFi: bitcoin finance, which is much more robust and reliable than DeF</li><li>Operates without new tokens or <em>gas</em>; bitcoin is the native currency</li></ul></td></tr></tbody></table>

Contracts written with Contractum are verified with client-side-validation, which does not adds data to a bitcoin blockchain and may be thought as a sharding technology, enhanced with zero-knowledge. Client-side-validation also breaks transaction graphs, unlinking contract evolution from blockchain transactions, making chain analysis impossible.

Learn more about RGB smart contracts on the [RGB FAQ](https://app.gitbook.com/o/-MO35HartFKtUgrkgzLy/s/-MO36nlUvK8SxfXw1MFs/) website.

## Work in progress

The contractum is a work in progress: the language design is under active development at the [LNP/BP Association](https://lnp-bp.org). Everyone is welcome to join the effort; a good starting point can be reading and writing to the [language design discussions group](https://github.com/RGB-WG/contractum-lang/discussions/categories/languague-design) in GitHub.

To understand and participate in contractum design it is important to learn more about technologies which are used by RGB smart contracts:

<table data-view="cards"><thead><tr><th></th><th></th><th></th></tr></thead><tbody><tr><td><h3>Single-use-seals</h3></td><td>Client-side-validation relies on a deterministic commitment history named single-use-seals</td><td><a href="https://app.gitbook.com/s/-MO36nlUvK8SxfXw1MFs/rgb-paradigms/single-use-seals">Learn more</a></td></tr><tr><td><h3>AluVM</h3></td><td>Contractum runs code on AluVM virtual machine: functional registry-based RISC machine</td><td><a href="https://app.gitbook.com/o/-MO35HartFKtUgrkgzLy/s/-MdUUOAyT-Nw8wDf9HPZ/">Learn more</a></td></tr><tr><td><h3>Strict encoding</h3></td><td>RGB contracts uses special deterministic portable binary data type system and encoding</td><td><a href="https://app.gitbook.com/o/-MO35HartFKtUgrkgzLy/s/-McPRmdXp1jTEY27B57G/">Learn more</a></td></tr></tbody></table>

## The feel of the language

If you'd like to get a feel of the language, here is a sample contract written in Contractum:

```haskell
types Did
   data PgpKey :: curve U8, key Bytes

schema DecentralizedIdentity
   -- This defines the atom of the contract state called `Identity` 
   -- which has data type `PgpKey`.
   -- The `owned` keyword means that there is always a party
   -- which owns the identity
   owned Identity :: Did.PgpKey

   owned IOYIssue :: Zk64
   -- `Zk64` means 64-bit unsigned integer hidden with zero-knowledge
   owned IOYTokens :: Zk64

   global IOYTicker :: String
   global IOYName :: String

   -- This says that to construct contract the user must provide
   -- information about exactly one identity and its IOY token
   genesis :: Identity, IOYTicker, IOYName

   -- Now let's define what a owner of identity can do,
   -- He can execute his rights by creating state transitions
   -- ("operation" on the state) of predefined forms, like
   op Revocation :: old Identity -> new Identity
   -- which does what it says: it revokes existing identity
   -- and creates a new one.
   
   -- This issues new IOY promises in tokenized form
   op Promise :: used IOYIssue -> given [IOYTokens]?, remaining IOYIssue?
      assert used == sum given + (remaining ?? 0)

   -- This transfers IOY tokens from one owner to another owner
   op Transfer :: spent {IOYTokens} -> received [IOYTokens]
      assert sum spent == sum received
   
interface FungibleToken:
   global Ticker -> String -- this is similar to schema definition; in fact
                           -- it is a requirement that the schema must provide
                           -- a global state of the String type and link it to
                           -- the "Ticker" name
   global Name -> String

   owned Inflation :: Zk64 -- pretty much the same applies to assigned state
   owned Asset :: Zk64

   op Issue :: Inflation -> [Asset]?, Inflation? -- and operations
   op Transfer :: {Asset} -> [Asset]

interface PgpIdentity
  owned Identity :: PgpKey
  exec Revocation :: old Identity -> new Identity

-- Specific schema state may use different naming, for instance because a schema
-- can define multiple assets with different names; in that case we will have
-- multiple interface implementations referencing different state.
implement FungibleToken for DecentralizedIdentity
   global Ticker := IOYTicker -- this creates a _binding_ of the state defined
                              -- in the schema (*IOYTicker* in this case) to the
                              -- interface 
   global Name := IOYName
   owned Inflation := IOYIssue
   owned Asset := IOYTokens
   op Issue := Promise
   op Transfer -- here we skip `:=` part since the interface operation name
               -- matches the name used in the schema. In such cases we can also
               -- skip the declaration at whole

implement PgpIdentity for DecentralizedIdentity
  -- we do not need to put anything here since schema state and operation names
  -- matches interface requirements and the compiler is able to guess the bindings
  
contract meSatoshiNakamoto implements DecentralizedIdentity
   set IOYTicker := "SATN"
   set IOYName := "Satoshi Promises"
   -- this defines a genesis state and assigns it to a single-use-seal
   assign orig Identity := (0xfac503c4641c3deda72a2d00bc9d6ff1094b15276c386efea403746a91436772, 1) 
                        -> PgpKey(0, 0x028730eeeec41802621d177507b086f390ae600ba3ca5e428b13913af4c2cd25b3)

transition iLostMyKey executes Revocation
   via meSatoshiNakamoto.orig -- specifies the single-use-seal we close to match requirements
                              -- on the valid operation execution conditions
   assign upd Identity := (~, 2) -- here we use txid of the bitcoin transaction which will be
                                 -- created to hold the commitment to this state transition, 
                                 -- called "single-use-seal witness". Since we can not know the
                                 -- txid upfront we use ~ to indicate the witness transaction id
                       -> PgpKey(0, 0x0219db0a4e0eb8cb833608c08d76b9b279ec44a851ab82cc6fd68a9b32624bfa8b)
   -- the above defines new state and assigns it to a single-use-seal
```

## About

Contractum development is managed by a non-profit [LNP/BP Standards Association](https://lnp-bp.org). The language design and compiler implementation is lead by [Dr Maxim Orlovsky](https://github.com/dr-orlovsky).
