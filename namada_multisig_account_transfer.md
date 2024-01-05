# Multisig Account Tranfer via Different User

:User1
```
namadaw key gen --alias user1-key
user1_public_key=$(namadaw key list | grep -A 3 "Alias \"user1-key\"" | grep "Public key:" | awk '{print $3}')
echo $user1_public_key
tpknam1qrr4tuq3slf4wvqy8rwn6yr8yk0netjxe6khjv95dv5038nrwkzlcx9wvsr
user1_address=$(namadaw address list | grep "user1-key" | awk '{print $NF}')
echo $user1_address
tnam1qq06cumrhd6yme0tnxplswydv2ykgyhj7u2kxkay
Retrieve faucet
```
:User2
```
namadaw key gen --alias user2-key
user2_public_key=$(namadaw key list | grep -A 3 "Alias \"user2-key\"" | grep "Public key:" | awk '{print $3}')
echo $user2_public_key
tpknam1qpxhyfr6clrrfytajqdnfgr9mm5rugsr5m0keaqqrg902hc04ldnwq0kpw9
user2_address=$(namadaw address list | grep "user2-key" | awk '{print $NF}')
echo $user2_address
tnam1qp2q7xncexj2hm0eh8p56caspax57epe9q3w9lf8
Retrieve faucet
```
:User1
```
namadac init-account \
--alias multisig-account \
--public-keys user1-key,tpknam1qpxhyfr6clrrfytajqdnfgr9mm5rugsr5m0keaqqrg902hc04ldnwq0kpw9 \
--signing-keys user1-key,tpknam1qpxhyfr6clrrfytajqdnfgr9mm5rugsr5m0keaqqrg902hc04ldnwq0kpw9 \
--threshold 2
multisig_account_addr=$(namadaw address list | grep '"multisig-account"' | awk '{print $NF}')
echo $multisig_account_addr
tnam1qxmn5jzjnrhl4s4wjvuf85zd7yr6639jjg4pslpm
Retrieve faucet
namadac balance --owner multisig-account
nam: 5
```
:User2
```
namadaw address gen --alias user2-account
user2_account_addr=$(namadaw address list | grep '"user2-account"' | awk '{print $NF}')
echo $user2_account_addr
tnam1qqc4s4290lnenxwm28k40akxcjlmj80n2g6plpmz
Retrieve faucet
namadac balance --owner user2-account
nam: 1
```
:User1
```
mkdir tx_dumps
namadac transfer \
--source multisig-account \
--target tnam1qqc4s4290lnenxwm28k40akxcjlmj80n2g6plpmz \
--token NAM \
--amount 1 \
--signing-keys user1-key,tpknam1qpxhyfr6clrrfytajqdnfgr9mm5rugsr5m0keaqqrg902hc04ldnwq0kpw9 \
--dump-tx \
--output-folder-path tx_dumps
Transaction serialized to tx_dumps/660718FF878EA16F9D5B630921F54B96CAB71FEFAC8AF42372BE4C47A611E880.tx
(Send transaction file "tx_dumps/660718FF878EA16F9D5B630921F54B96CAB71FEFAC8AF42372BE4C47A611E880.tx" to User2)
namadac sign-tx \
--tx-path "tx_dumps/660718FF878EA16F9D5B630921F54B96CAB71FEFAC8AF42372BE4C47A611E880.tx" \
--signing-keys user1-key \
--owner multisig-account
Signature for tpknam1qrr4tuq3slf4wvqy8rwn6yr8yk0netjxe6khjv95dv5038nrwkzlcx9wvsr serialized at offline_signature_660718FF878EA16F9D5B630921F54B96CAB71FEFAC8AF42372BE4C47A611E880_tpknam1qrr4tuq3slf4wvqy8rwn6yr8yk0netjxe6khjv95dv5038nrwkzlcx9wvsr.tx
export SIGNATURE_ONE="offline_signature_660718FF878EA16F9D5B630921F54B96CAB71FEFAC8AF42372BE4C47A611E880_tpknam1qrr4tuq3slf4wvqy8rwn6yr8yk0netjxe6khjv95dv5038nrwkzlcx9wvsr.tx"
```
:User2
```
namadac sign-tx \
--tx-path "tx_dumps/660718FF878EA16F9D5B630921F54B96CAB71FEFAC8AF42372BE4C47A611E880.tx" \
--signing-keys user2-key \
--owner tnam1qxmn5jzjnrhl4s4wjvuf85zd7yr6639jjg4pslpm
Signature for tpknam1qpxhyfr6clrrfytajqdnfgr9mm5rugsr5m0keaqqrg902hc04ldnwq0kpw9 serialized at offline_signature_660718FF878EA16F9D5B630921F54B96CAB71FEFAC8AF42372BE4C47A611E880_tpknam1qpxhyfr6clrrfytajqdnfgr9mm5rugsr5m0keaqqrg902hc04ldnwq0kpw9.tx
Send User2 signature file of "offline_signature_660718FF878EA16F9D5B630921F54B96CAB71FEFAC8AF42372BE4C47A611E880_tpknam1qpxhyfr6clrrfytajqdnfgr9mm5rugsr5m0keaqqrg902hc04ldnwq0kpw9.tx" to User1
```
:User1
```
export SIGNATURE_TWO="offline_signature_660718FF878EA16F9D5B630921F54B96CAB71FEFAC8AF42372BE4C47A611E880_tpknam1qpxhyfr6clrrfytajqdnfgr9mm5rugsr5m0keaqqrg902hc04ldnwq0kpw9.tx"
namadac tx \
--tx-path "tx_dumps/660718FF878EA16F9D5B630921F54B96CAB71FEFAC8AF42372BE4C47A611E880.tx" \
--signatures $SIGNATURE_ONE,$SIGNATURE_TWO \
--owner multisig-account \
--gas-payer user1-key
namadac balance --owner multisig-account
nam: 4
```
:User2
```
namadac balance --owner user2-account
nam: 2
```
