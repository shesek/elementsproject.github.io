---
layout: page
title: Confidential Transactions
permalink: features/confidential-transactions
redirect_from:
  - /elements/confidential-transactions/
  - /elements/confidential-transactions
---

# Confidential Transactions

## One of the most powerful features of Elements is Confidential Transactions

Confidential Transactions keep the amount and type of assets transferred visible only to participants in the transaction (and those they choose to [reveal the blinding key]({{ site.url }}/elements-code-tutorial/confidential-transactions#blindingkey) to), while still cryptographically guaranteeing that no more coins can be spent than are available.

This goes a step beyond the usual privacy offered by Bitcoin's blockchain, which relies purely on pseudonymous (but public) identities. This matters, because insufficient financial privacy can have serious security and privacy implications for both commercial and personal transactions. Without adequate protection, thieves can focus their efforts on high-value targets, competitors can learn business details, and negotiating positions can be undermined.

Watch <a href="https://www.youtube.com/embed/ZIugzFygviw">Gregory Maxwell explaining Confidential Transactions</a>:

<center><iframe width="560" height="315" src="https://www.youtube.com/embed/ZIugzFygviw" frameborder="0" allowfullscreen></iframe></center>

* * * 

You can also read Gregory Maxwell's [initial investigation]({{ site.url }}/features/confidential-transactions/investigation) for an in-depth look at the mathmatics behind Confidentail Transactions.

### Using Confidential Transactions in Elements

For a detailed look at using Confidential Transactions please refer to the Confidential Transactions section of the [code tutorial]({{ site.url }}/elements-code-tutorial/confidential-transactions). 

At a high-level, the transaction flow is very similar to Bitcoin's on the surface. 

### Similarities to Bitcoin's transaction flow

A new address is created:

~~~~
elements-cli getnewaddress
~~~~

By default this returns a new [Confidential Address]({{ site.url }}/features/confidential-transactions/addresses). The prefix 'CTE' is used to denote the address is confidential[1]. An example address is shown below:

<div class="console-output">CTEwQjyErENrxo8dSQ6pq5atss7Ym9S7P6GGK4PiGAgQRgoh1iPUkLQ168Kqptfnwmpxr2Bf7ipQsagi
</div>

Assets can then be sent to the address using the sendtoaddress command:

~~~~
elements-cli sendtoaddress CTEwQjyErENrxo8dSQ6pq5atss7Ym9S7P6GGK4PiGAgQRgoh1iPUkLQ168Kqptfnwmpxr2Bf7ipQsagi 0.005
~~~~

Which returns a transaction id:

<div class="console-output">82b2c5122207e5f33e7adadc6a4aab16a170e16028f0b0cf2c04f9d17d6f0321
</div>

### Differences to Bitcoin's transaction flow

The key difference from Bitcoin is the addition of cryptographic privacy. 

Confidential Transactions differ in that the the amounts and types of asset transferred are visible only to participants in the transaction, and those they choose to reveal the blinding key to.

The ``createrawtransaction`` API in Elements works similar to Bitcoin's raw
transactions with the following differences:

**1.**&nbsp;&nbsp;&nbsp;&nbsp;The intent to create a confidential output is indicated by using a confidential address for the destination.

**2.**&nbsp;&nbsp;&nbsp;&nbsp;The ``createrawtransaction`` RPC has the additional key ``nValue`` per input which must be set to the value of the input. The value can be determined with the ``listunspent`` RPC for example.

**3.**&nbsp;&nbsp;&nbsp;&nbsp;After calling ``createrawtransaction`` the user must call ``blindrawtransaction`` on the transaction if a confidential input or output address is involved.

**4.**&nbsp;&nbsp;&nbsp;&nbsp;If at least one of the inputs is confidential, at least one of the outputs must also be confidential. Note that it is perfectly fine to create a 0-valued confidential output if otherwise there would be no confidential output.

**5.**&nbsp;&nbsp;&nbsp;&nbsp;If all inputs are unconfidential, the number of confidential outputs must be ``0`` or ``>= 2``. Again, it's fine to create a 0-valued confidential output.

The following example spends a confidential and an unconfidential output and
creates a confidential and unconfidential output. Due to the size of the
transaction it is not displayed here, but will be saved in the ``TX`` variable.

~~~~
TX=elements-cli createrawtransaction '[{"txid": "421079661b117b659af3431096ce2118043396e3647e523e413cd626fa798df7", "vout": 0, "nValue": 0.001}, {"txid": "17aa26d29582a9a26f02033918aaf9823d33458239074a0d7ee638b247f1fa2c", "vout": 0, "nValue": 0.001}]' '{"XoRDiv9z12ECsYCNjtHjDvV1Nn4KSBa6xRfHT1c3nKY1CwDiz5rzSMZL7QCgvuF1T5Kq43o1fMqBxbWQ": 0.001, "QLsHc5DgnpMjPaDSuEF5gXt7ccgmLtgh2N": 0.0005}'

TX=elements-cli blindrawtransaction $TX

TX=elements-cli signrawtransaction $TX | jq -r '.hex'

elements-cli sendrawtransaction $TX
~~~~

### Limitations

The implementation of Confidential Transactions as it appears in Elements has some important limitations to be aware of.

The implementation only hides a certain number of the digits of the amount of each transaction output, dependent on the range proof's "blinding coverage" at a desired precision level.  Subsequently, there is a "minimum confidential amount" around 0.0001, and a "maximum confidential amount" that is 2<sup>32</sup> times the minimum amount.

Digits smaller than the minimum will be revealed to observers; for example, if the minimum is 0.0001, a transaction output of 123.456789 will look like "?.????89". The tiny amount of information about the value leaked in this way is unlikely to be important, but be aware that this could allow observers to "follow" coins by linking subsequent transactions with identical amounts. All values can be rounded to the minimum to avoid revealing information in this way if preferred.

A transaction output larger than the maximum will reveal the order of magnitude of the amount to observers, and will also reveal additional digits at the bottom of the amount.

For example, if the maximum is 500k, then all outputs under that amount will look the same, but an output between 500k and 5M will be visible as such to observers (but not the exact amount within that range), and will also reveal one additional digit of the low end of the amount. Revealing the range in this way could be a very significant privacy leak; splitting such extremely large transactions to keep them under the maximum confidential amount is strongly recommended.

### Find out more about Confidential Transactions

Gregory Maxwell's [original investigation]({{ site.url }}/features/confidential-transactions/investigation) into Confidential Transactions.

Try it yourself in the [Elements Code Tutorial]({{ site.url }}/elements-code-tutorial/confidential-transactions).

More details on [Confidential Addresses]({{ site.url }}/features/confidential-transactions/addresses).

[1] Elements now defaults to creating P2SH-P2WPKH addresses by default, which do not start with 'CTE'. See the note [here]({{ site.url }}/elements-code-tutorial/confidential-transactions) on how to create 'legacy' addresses starting with 'CTE'.
