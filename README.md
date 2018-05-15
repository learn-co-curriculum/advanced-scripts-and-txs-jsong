# Advanced Scripting and Transactions

```python
# import everything and define a test runner function
from importlib import reload
from helper import run_test

import ecc
import helper
import script
import tx
```

### Exercise 1

#### 1.1. Make [this test](/edit/session5/tx.py) pass
```
tx.py:TxTest::test_verify_input
```


```python
# Exercise 1.1

reload(tx)
run_test(tx.TxTest('test_verify_input'))
```


```python
# Transaction Construction Example

from ecc import PrivateKey
from helper import decode_base58, p2pkh_script, SIGHASH_ALL
from script import Script
from tx import TxIn, TxOut, Tx

# Step 1
tx_ins = []
prev_tx = bytes.fromhex('0025bc3c0fa8b7eb55b9437fdbd016870d18e0df0ace7bc9864efc38414147c8')
tx_ins.append(TxIn(
            prev_tx=prev_tx,
            prev_index=0,
            script_sig=b'',
            sequence=0xffffffff,
        ))

# Step 2
tx_outs = []
h160 = decode_base58('mzx5YhAH9kNHtcN481u6WkjeHjYtVeKVh2')
tx_outs.append(TxOut(
    amount=int(0.99*100000000),
    script_pubkey=p2pkh_script(h160),
))
h160 = decode_base58('mnrVtF8DWjMu839VW3rBfgYaAfKk8983Xf')
tx_outs.append(TxOut(
    amount=int(0.1*100000000),
    script_pubkey=p2pkh_script(h160),
))
tx_obj = Tx(version=1, tx_ins=tx_ins, tx_outs=tx_outs, locktime=0, testnet=True)

# Step 3
hash_type = SIGHASH_ALL
z = tx_obj.sig_hash(0, hash_type)
pk = PrivateKey(secret=8675309)
der = pk.sign(z).der()
sig = der + bytes([hash_type])
sec = pk.point.sec()
script_sig = bytes([len(sig)]) + sig + bytes([len(sec)]) + sec
script_sig = bytes([len(script_sig)]) + script_sig
tx_obj.tx_ins[0].script_sig = Script.parse(script_sig)
print(tx_obj.serialize().hex())
```

### Exercise 2

#### 2.1. Make [this test](/edit/session5/tx.py) pass
```
tx.py:TxTest:test_sign_input
```


```python
# Exercise 2.1

reload(tx)
run_test(tx.TxTest('test_sign_input'))
```

### Exercise 3

#### 3.1. Send 0.02 TBTC to this address `mnrVtF8DWjMu839VW3rBfgYaAfKk8983Xf`

#### Go here to send your transaction: https://testnet.blockexplorer.com/tx/send


```python
# Exercise 3.1
reload(tx)
from ecc import PrivateKey
from helper import decode_base58, p2pkh_script, SIGHASH_ALL
from script import Script
from tx import TxIn, TxOut, Tx

prev_tx = bytes.fromhex('0025bc3c0fa8b7eb55b9437fdbd016870d18e0df0ace7bc9864efc38414147c8')
prev_index = 0
target_address = 'mnrVtF8DWjMu839VW3rBfgYaAfKk8983Xf'
target_amount = 0.02
change_address = 'mzx5YhAH9kNHtcN481u6WkjeHjYtVeKVh2'
change_amount = 1.07
secret = 8675309
priv = PrivateKey(secret=secret)

# initialize inputs
# create a new tx input

# initialize outputs
# decode the hash160 from the target address
# convert hash160 to p2pkh script
# convert target amount to satoshis (multiply by 100 million)
# create a new tx output for target
# decode the hash160 from the change address
# convert hash160 to p2pkh script
# convert change amount to satoshis (multiply by 100 million)
# create a new tx output for target

# create the transaction

# now sign the 0th input with the private key using SIGHASH_ALL using sign_input

# SANITY CHECK: change address corresponds to private key
if priv.point.address(testnet=True) != change_address:
    raise RuntimeError('Private Key does not correspond to Change Address, check priv_key and change_address')

# SANITY CHECK: output's script_pubkey is the same one as your address
if tx_ins[0].script_pubkey(testnet=True).elements[2] != decode_base58(change_address):
    raise RuntimeError('Output is not something you can spend with this private key. Check that the prev_tx and prev_index are correct')

# SANITY CHECK: fee is reasonable
if tx_obj.fee() > 0.05*100000000 or tx_obj.fee() <= 0:
    raise RuntimeError('Check that the change amount is reasonable. Fee is {}'.format(tx_obj.fee()))

# serialize and hex()
```

### Exercise 4

#### 4.1. Find the hash160 of the RedeemScript
```
5221022626e955ea6ea6d98850c994f9107b036b1334f18ca8830bfff1295d21cfdb702103b287eaf122eea69030a0e9feed096bed8045c8b98bec453e1ffac7fbdbd4bb7152ae
```


```python
# Exercise 4.1

from helper import hash160

hex_redeem_script = '5221022626e955ea6ea6d98850c994f9107b036b1334f18ca8830bfff1295d21cfdb702103b287eaf122eea69030a0e9feed096bed8045c8b98bec453e1ffac7fbdbd4bb7152ae'

# bytes.fromhex script
# hash160 result
# hex() to display
```


```python
# P2SH address construction example

from helper import encode_base58_checksum

print(encode_base58_checksum(b'\x05'+bytes.fromhex('74d691da1574e6b3c192ecfb52cc8984ee7b6c56')))
```

### Exercise 5

#### 5.1. Make [these tests](/edit/session5/helper.py) pass
```
helper.py:HelperTest:test_p2pkh_address
helper.py:HelperTest:test_p2sh_address
```


```python
# Exercise 5.1

reload(helper)
run_test(helper.HelperTest('test_p2pkh_address'))
run_test(helper.HelperTest('test_p2sh_address'))
```


```python
# z for p2sh example

from helper import double_sha256

sha = double_sha256(bytes.fromhex('0100000001868278ed6ddfb6c1ed3ad5f8181eb0c7a385aa0836f01d5e4789e6bd304d87221a000000475221022626e955ea6ea6d98850c994f9107b036b1334f18ca8830bfff1295d21cfdb702103b287eaf122eea69030a0e9feed096bed8045c8b98bec453e1ffac7fbdbd4bb7152aeffffffff04d3b11400000000001976a914904a49878c0adfc3aa05de7afad2cc15f483a56a88ac7f400900000000001976a914418327e3f3dda4cf5b9089325a4b95abdfa0334088ac722c0c00000000001976a914ba35042cfe9fc66fd35ac2224eebdafd1028ad2788acdc4ace020000000017a91474d691da1574e6b3c192ecfb52cc8984ee7b6c56870000000001000000'))
z = int.from_bytes(sha, 'big')

print(sha.hex())
```


```python
# p2sh verification example

from ecc import S256Point, Signature
from helper import double_sha256

sha = double_sha256(bytes.fromhex('0100000001868278ed6ddfb6c1ed3ad5f8181eb0c7a385aa0836f01d5e4789e6bd304d87221a000000475221022626e955ea6ea6d98850c994f9107b036b1334f18ca8830bfff1295d21cfdb702103b287eaf122eea69030a0e9feed096bed8045c8b98bec453e1ffac7fbdbd4bb7152aeffffffff04d3b11400000000001976a914904a49878c0adfc3aa05de7afad2cc15f483a56a88ac7f400900000000001976a914418327e3f3dda4cf5b9089325a4b95abdfa0334088ac722c0c00000000001976a914ba35042cfe9fc66fd35ac2224eebdafd1028ad2788acdc4ace020000000017a91474d691da1574e6b3c192ecfb52cc8984ee7b6c56870000000001000000'))
z = int.from_bytes(sha, 'big')
point = S256Point.parse(bytes.fromhex('022626e955ea6ea6d98850c994f9107b036b1334f18ca8830bfff1295d21cfdb70'))
sig = Signature.parse(bytes.fromhex('3045022100dc92655fe37036f47756db8102e0d7d5e28b3beb83a8fef4f5dc0559bddfb94e02205a36d4e4e6c7fcd16658c50783e00c341609977aed3ad00937bf4ee942a89937'))
print(point.verify(z, sig))
```

### Exercise 6

#### 6.1. Validate the second signature of the first input

```
0100000001868278ed6ddfb6c1ed3ad5f8181eb0c7a385aa0836f01d5e4789e6bd304d87221a000000db00483045022100dc92655fe37036f47756db8102e0d7d5e28b3beb83a8fef4f5dc0559bddfb94e02205a36d4e4e6c7fcd16658c50783e00c341609977aed3ad00937bf4ee942a8993701483045022100da6bee3c93766232079a01639d07fa869598749729ae323eab8eef53577d611b02207bef15429dcadce2121ea07f233115c6f09034c0be68db99980b9a6c5e75402201475221022626e955ea6ea6d98850c994f9107b036b1334f18ca8830bfff1295d21cfdb702103b287eaf122eea69030a0e9feed096bed8045c8b98bec453e1ffac7fbdbd4bb7152aeffffffff04d3b11400000000001976a914904a49878c0adfc3aa05de7afad2cc15f483a56a88ac7f400900000000001976a914418327e3f3dda4cf5b9089325a4b95abdfa0334088ac722c0c00000000001976a914ba35042cfe9fc66fd35ac2224eebdafd1028ad2788acdc4ace020000000017a91474d691da1574e6b3c192ecfb52cc8984ee7b6c568700000000
```

The sec pubkey of the second signature is:
```
03b287eaf122eea69030a0e9feed096bed8045c8b98bec453e1ffac7fbdbd4bb71
```

The der signature of the second signature is:
```
3045022100da6bee3c93766232079a01639d07fa869598749729ae323eab8eef53577d611b02207bef15429dcadce2121ea07f233115c6f09034c0be68db99980b9a6c5e75402201475221022
```

The redeemScript is:
```
475221022626e955ea6ea6d98850c994f9107b036b1334f18ca8830bfff1295d21cfdb702103b287eaf122eea69030a0e9feed096bed8045c8b98bec453e1ffac7fbdbd4bb7152ae
```


```python
# Exercise 6.1

from io import BytesIO
from ecc import S256Point, Signature
from helper import double_sha256, int_to_little_endian
from script import Script
from tx import Tx, SIGHASH_ALL

hex_sec = '03b287eaf122eea69030a0e9feed096bed8045c8b98bec453e1ffac7fbdbd4bb71'
hex_der = '3045022100da6bee3c93766232079a01639d07fa869598749729ae323eab8eef53577d611b02207bef15429dcadce2121ea07f233115c6f09034c0be68db99980b9a6c5e754022'
hex_redeem_script = '5221022626e955ea6ea6d98850c994f9107b036b1334f18ca8830bfff1295d21cfdb702103b287eaf122eea69030a0e9feed096bed8045c8b98bec453e1ffac7fbdbd4bb7152ae'
sec = bytes.fromhex(hex_sec)
der = bytes.fromhex(hex_der)
redeem_script = bytes.fromhex(hex_redeem_script)

hex_tx = '0100000001868278ed6ddfb6c1ed3ad5f8181eb0c7a385aa0836f01d5e4789e6bd304d87221a000000db00483045022100dc92655fe37036f47756db8102e0d7d5e28b3beb83a8fef4f5dc0559bddfb94e02205a36d4e4e6c7fcd16658c50783e00c341609977aed3ad00937bf4ee942a8993701483045022100da6bee3c93766232079a01639d07fa869598749729ae323eab8eef53577d611b02207bef15429dcadce2121ea07f233115c6f09034c0be68db99980b9a6c5e75402201475221022626e955ea6ea6d98850c994f9107b036b1334f18ca8830bfff1295d21cfdb702103b287eaf122eea69030a0e9feed096bed8045c8b98bec453e1ffac7fbdbd4bb7152aeffffffff04d3b11400000000001976a914904a49878c0adfc3aa05de7afad2cc15f483a56a88ac7f400900000000001976a914418327e3f3dda4cf5b9089325a4b95abdfa0334088ac722c0c00000000001976a914ba35042cfe9fc66fd35ac2224eebdafd1028ad2788acdc4ace020000000017a91474d691da1574e6b3c192ecfb52cc8984ee7b6c568700000000'
stream = BytesIO(bytes.fromhex(hex_tx))

# parse the S256Point and Signature
# parse the Tx
# change the first input's scriptSig to redeemScript (use Script.parse on the redeemScript)
# get the serialization
# add the sighash (4 bytes, little-endian of SIGHASH_ALL)
# double_sha256 the result
# this interpreted is a big-endian number is your z
# now verify the signature using point.verify
```
