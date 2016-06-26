### OP_CSV aka 'OP_HODL'

#### Overview:
"to allow execution pathways ... to be restricted based on the age of the output ..."

Complements OP_CLTV, which uses a fixed time in the future.

BIP68 introduces semantics for handling relative locktimes. 
CSV makes this treatment a consensus rule, ensuring that
spends of a CSV bearing output cannot be spent until the 
interval has passed. 

The only way to provably lock funds for a definite amount of time,
 or until they mature.

We repurpose the NOP3 opcode, renaming it OP_CHECKSEQUENCEVERIFY

https://github.com/bitcoin/bips/blob/master/bip-0112.mediawiki
https://github.com/bitcoin/bitcoin/blob/master/src/script/interpreter.cpp

#### Contract with expiration

Alice can fund a multi-signature contract with a clause s.t.s. 
        2 <Alice's pubkey> <Bob's pubkey> <Escrow's pubkey> 3 CHECKMULTISIGVERIFY
    ELSE
        "30d" CHECKSEQUENCEVERIFY DROP
        <Alice's pubkey> CHECKSIGVERIFY
    ENDIF

#### Revocable pathways in payment channels

  A payment channel is a protocol where signatures for a new version of a spend transaction are exchanged. 
  This is done multiple times until the channel is settled, instead thof frequently updating the blockchain. 
 
  The channel defines the contract under which users interact, so details will change from contract to contract. 
  
  Imagine Alice has funded a movie website with 1 BTC, and occasionally buys movies. 
  She can settle once per month, but can download as much as the channel allows, as often as she likes.  
   
  The website needs to know it will get paid, but Alice needs to be equally sure about the safety of her funds
  should the website become non-cooperative. 

#### Lightning Network

 LN makes use of the above. The protocol is defined and is enforcable for both sides of the channel. 
  
  To open a channel, we generate a script similar to the following where funds are sent. 
  
  2 [Alice] [Bob] 2 OP_CHECKMULTISIG
  
  When the balance changes for the first time, the following takes place: 
   -  Alice and Bob generate Ra, Rb (random), and Qa, Qb (public keys) independently.
   -  They hash their R value and swap Ha, Hb, Qa, Qb. 
  
  Each side creates a commitment transaction, refunding themselves using the following script.
  Each side must put their own public key behind the CSV lock, while the other can spend the funds 
  immediately upon learning the hash: 
  
   HASH160 <revokehash> EQUAL
      IF
          <Bob's pubkey>
      ELSE
          "24h" CHECKSEQUENCEVERIFY DROP
          <Alice's pubkey>
      ENDIF
      CHECKSIG
      
  The transactions remain unsigned for now.. 
      
  Any time we update the state, the R values must be revealed to the other (easily checked),
  and new commitment transactions are created against the funds locked in the contract address. 
  
  Since R values are swapped on every update, Bob possesses the revocation codes for all
  of Alices revoked payments. Say Alice decides to renege on a later commitment by broadcasting an 
  older one, and the payment enters the chain. 
  
  Bob will notice the channel closed early, but also possesses the code Alice used in that same commitment. 
  He can spend the funds immediately, whereas Alice was forced to wait for 24 hours. 
  