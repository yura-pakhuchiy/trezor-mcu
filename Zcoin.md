# Zcoin on Trezor1

## Installing patched trezor-python

```
$ pip3 uninstall trezor # in case unpatched version 0.9.0 is already installed
$ pip3 install https://github.com/yura-pakhuchiy/python-trezor/releases/download/0.9.0-zcoin/trezor-zcoin.tar.gz
```

## Installing patched firmware

*WARNING: Make sure you have backup of your recovery seed. Installing this firmware will wipe your device. Do NOT install this firmware if you have not written down your seed, or you will lose your money*

1. Download firmware from https://github.com/yura-pakhuchiy/trezor-mcu/releases/download/v1.7.0-zcoin/trezor-zcoin.bin. Or even better build it by yourself from sources and verify that fingerprint matches d5d1b988ce2f296277bccadc3b24ec8079c4dcf900c236c4db1a628cd85f4d4b.
2. Disconnect Trezor from USB port.
3. Press both buttons and reconnect Trezor to USB port while still holding buttons.
4. Trezor should load in bootloader mode and show serial no.
5. Make sure that you have read warning above!
6. Install firmware:
```
$ trezorctl firmware_update -f trezor-zcoin.bin
```

## Generating addresses

You can generate Zcoin addresses by specifying BIP32 path (coin type for Zcoin is 136):
```
$ trezorctl get_address -c Zcoin -n "m/44'/136'/0'/0/0" # 1st receiving address
$ trezorctl get_address -c Zcoin -n "Zcoin/0'/0/0"      # 1st receiving address
$ trezorctl get_address -c Zcoin -n "m/44'/136'/0'/0/1" # 2nd receiving address
$ trezorctl get_address -c Zcoin -n "m/44'/136'/0'/0/2" # 3rd receiving address
$ trezorctl get_address -c Zcoin -n "m/44'/136'/0'/1/0" # 1st change address
$ trezorctl get_address -c Zcoin -n "Zcoin/0'/1/1"      # 2nd change address
```

You can confirm address on Trezor by adding `-d` parameter:
```
$ trezorctl get_address -c Zcoin -n "m/44'/136'/0'/0/0" -d
```

More information about BIP32 path: https://doc.satoshilabs.com/trezor-tech/cryptography.html#hierarchical-deterministic-wallets-bip32-and-bip44

<img src="https://user-images.githubusercontent.com/4184761/36583929-39f15366-18aa-11e8-9152-bdb00a284e1b.JPG" width="300">

## Signing and verifying messages

Signing message:
```
$ trezorctl sign_message -c Zcoin -n "m/44'/136'/0'/0/0" 'Zcoin on Trezor!'
address: aJUWeqyLtwkAunuDNmdCssMxMbjmurNqeq
signature: b'H4EwBE4nWpmGorBnKBUdefH/Z24DY9VmC6jLu2XQnssXIi192qrAEp/FQYvBBufBDxA+cXjMYLILruWPn+1/F6g='
message: Zcoin on Trezor!
```
Signature is between leading `b'` and trailing `'`.

Verifying message:
```
$ trezorctl verify_message -c Zcoin aJUWeqyLtwkAunuDNmdCssMxMbjmurNqeq 'H4EwBE4nWpmGorBnKBUdefH/Z24DY9VmC6jLu2XQnssXIi192qrAEp/FQYvBBufBDxA+cXjMYLILruWPn+1/F6g=' 'Zcoin on Trezor!'
True
```

## Signing transaction

To sign transaction, you need to specify UTXO and BIP32 path for corresponding address.
I suggest to use https://chainz.cryptoid.info/xzc/ to determine output index in transaction. https://explorer.zcoin.io merges outputs to the same address thus it can not be used to determine index number.

For example, to spend output from transaction [5b219cfb27353c428c1fdd39c2db851d9f007cd872d436315cb09941fbb52e4b](https://chainz.cryptoid.info/xzc/tx.dws?498450.htm) to 2 addresses, 0.001 XZC (100000 zeetoshi) to aJUWeqyLtwkAunuDNmdCssMxMbjmurNqeq and 0.00899 XZC (899000 zeetoshi) to Zzu6qstMBVpmccb1ihyc8XL4mNWdvfJMYF, paying 1000 zeetoshi transaction fee:
```
$ trezorctl sign_tx -c Zcoin

Previous output to spend (txid:vout) []: 5b219cfb27353c428c1fdd39c2db851d9f007cd872d436315cb09941fbb52e4b:0
BIP-32 path to derive the key: m/44'/136'/0'/0/1
Input amount (satoshis) [0]: 
Sequence Number to use (RBF opt-in enabled by default) [4294967293]: 4294967294
Input type [address]: 

Previous output to spend (txid:vout) []: 

Output address (for non-change output) []: aJUWeqyLtwkAunuDNmdCssMxMbjmurNqeq
Amount to spend (satoshis): 100000
Output type [address]: 

Output address (for non-change output) []: Zzu6qstMBVpmccb1ihyc8XL4mNWdvfJMYF 
Amount to spend (satoshis): 899000
Output type [address]: 

Output address (for non-change output) []: 
BIP-32 path (for change output) []: 
Transaction version [2]: 1
Transaction locktime [0]: 

Signed Transaction:
01000000014b2eb5fb4199b05c3136d472d87c009f1d85dbc239dd1f8c423c3527fb9c215b000000006a47304402207b82148271382b35438f7e1b05025bbcb4f1ad62e07abbfe4efcc37feb91ae8002200575ca7d77e4e05ee7209459a09bc88946e3c34df264a59aba7af96f4d79ae67012103636c97d499e44d315d598fb67c9ee4cb787cf1fc1a96095f42854c9ef7e4975bfeffffff02a0860100000000001976a914c2cedf4189d4b0adac415d8e1ef0df9a53a1309e88acb8b70d00000000001976a914020321b289a80b3e128fdc2581ee3b7faf59c2a988ac00000000
```

Take output after `Signed transaction:` and paste it to https://insight.zcoin.io/tx/send

Use 4294967294 for sequence number (note last digit different from default) and 1 for transaction version (Zcoin core uses these values). Do not use other values unless you know exactly what are you doing.

Specifying change address by BIP32 path does not work, just specify address for change output as for all other outputs.

Our transaction mined: https://insight.zcoin.io/tx/7c80b73c3038cd6aec4605ed197f2b4832a5dbbd9fb2c329eea09fd9f4bc31dd

<img src="https://user-images.githubusercontent.com/4184761/36583930-3a2afe36-18aa-11e8-8c33-a3a1c04a223e.JPG" width="300">
<img src="https://user-images.githubusercontent.com/4184761/36583931-3a6486a6-18aa-11e8-8a9c-d6b0e2b3dbb5.JPG" width="300">

## Web wallet

Unfortunatelly it is not possible to use https://wallet.trezor.io for Zcoin right now. It will start working only after Trezor team merges our patch.

## Credits

Many thanks to [@sn-ntu](https://github.com/sn-ntu/) and [@Technoprenerd](https://github.com/Technoprenerd) for zcoin port of insight server. It is pre-requirement for coin integration.

## Tips

You can send me tips for this guide and firmware build with Zcoin support to [aAkamFXreUt2TmN7bB9LMcWRR2W8q7naar](https://chainz.cryptoid.info/xzc/address.dws?aAkamFXreUt2TmN7bB9LMcWRR2W8q7naar.htm)
