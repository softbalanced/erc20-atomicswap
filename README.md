# balanced-atomic-swap

Library for creating atomic swaps transactions on the Balanced blockchain

---
# Demo Application

This repository contains a simple demo application is the `/demo` directory. In this appication the initiator has Ethereum they would like to swap for Balanced. 

## Starting the Demo Application

1. 
    ```
    npm install
   ```

1.
    ```
    npm run server
    ```

1. Start [local Balanced Node](#local-balanced-node)    

Demo app will them be available at http://localhost:8080/

# Running the Demo Application

## Prerequisites

1. Ethereum

     In this demo application, Ethereum is being swapped for Balanced, so the application also communicates with an Ethereum blockchain. In this case, a smart contract that has been deployed to to the Ropsten test network. In order to use the demo app, you will need to interact with the Ropsten network via MetaMask.

    1. Install [MetaMask](https://metamask.io/)

    1. Create Ethereum Accounts

        Each party to the swap (initiator and participant) will need accounts on the Ethereum blockchain (ropsten)

        ![create ethereum accounts](doc/img/ether_account_create.gif)

    2. Fund Initiator account

        The initiator needs some Ether to swap for Balanced, so their account will need to have some ether to swap. Copy the initiators address from metamask and go to the [Ropsten faucet](https://faucet.ropsten.be/) to get some ether.

   
1. Local DASH Network

    The demo application connects to a local Balanced node either in [regtest mode](https://balancedcore.readme.io/docs/core-examples-testing-applications#regtest-mode) or connected to [Testnet](https://balancedcore.readme.io/docs/core-examples-testing-applications#testnet). The following instructions assume regtest mode.

    1. Start your [local balanced node](#local-balanced-node)

    1. If this is the first time starting the node, you will need to initialize the blockchain so that funds are available and generate enough depth to activate the BIP9 softfork.

        ```
        balanced-cli -regtest generate 432
        ```

    1. Generate DASH addresses and keys for the two parties in the swap.

        * The initiator

            1. Generate an address to receive the funds.

                ```    
                $ ./balanced-cli -regtest getnewaddress

                yYu94PDiVRfpShg6pHrtQuPQT7fMno4eir
                ```

            2. Get the public key for this address

                ```
                ./balanced-cli -regtest getaddressinfo yYu94PDiVRfpShg6pHrtQuPQT7fMno4eir

                {
                  "address": "yYu94PDiVRfpShg6pHrtQuPQT7fMno4eir",
                  "scriptPubKey": "76a9148a00cd4114044918a5e9a53233a75b1c96a76d0488ac",
                  "ismine": true,
                  "iswatchonly": false,
                  "isscript": false,
                  "pubkey": "0221266b461ea89ac79e281769f4ba58e4f07c99ce83ce889d7b6c5795141aaf8c",
                  "iscompressed": true,
                  "timestamp": 1612667597,
                  "labels": [
                    {
                      "name": "",
                      "purpose": "receive"
                    }
                  ]
                }
                ```

            1. Get the private key for this address

                ```
                /balanced-cli -regtest dumpprivkey yYu94PDiVRfpShg6pHrtQuPQT7fMno4eir

                    cVAY2J3TFFfWtKUuYVkkPM74FKLgC4MmU9gGqxGpYjCLpqJe4JnP
                ```

            * The Participant

                ```
                    $ ./balanced-cli -regtest getnewaddress

                        yXTPRzbFyXEA8VdbY8pohEYfw8T6QxeAsZ

                    $ ./balanced-cli -regtest getaddressinfo yXTPRzbFyXEA8VdbY8pohEYfw8T6QxeAsZ

                        {
                          "address": "yXTPRzbFyXEA8VdbY8pohEYfw8T6QxeAsZ",
                          "scriptPubKey": "76a9147a29dbce092e5a39453fd78725d91c859497faf388ac",
                          "ismine": true,
                          "iswatchonly": false,
                          "isscript": false,
                          "pubkey": "034f4efc53df942eb88e06e7ec5351a463a77003b24d6387b83a9033897ba0c9fa",
                          "iscompressed": true,
                          "timestamp": 1612667597,
                          "labels": [
                            {
                              "name": "",
                              "purpose": "receive"
                            }
                          ]
                        }

                    $ ./balanced-cli -regtest dumpprivkey yXTPRzbFyXEA8VdbY8pohEYfw8T6QxeAsZ

                        cNTa8f54AiottaavKzd7sG7gnAkvrKwAMV3XW6tTsCVzQD4gH2od
                ```

## Performing The Swap

1. The initiator creates the first part of the swap

    The initiator begins the swap by going to http://localhost:8080/initiator-create.html and locking some ether in a smart contract using a secret. They will then send the hash of that secret along and their Balanced public key to the participant. 

![Swap Initiation page](doc/img/demo_initiate.gif)

2. The participant will then go to http://localhost:8080/participant-create.html to create their half of the swap using their public key and the secret hash and public key that was sent to them by the initiator. 

![Participant Create page](doc/img/participant_create.png)

3. After clicking submit, the P2SH address for the swap will be revealed to the participant.

![P2SH address reveal page](doc/img/participant_create_pt2.png)

4. The partcipant will then need to send the agreed amount of Balanced to that address. For the sake of this example, this can be done on the command line:

    ```
    $ ./balanced-cli -regtest sendtoaddress 8hGQar8umTsc9MnGfJM5PiRrMiLrcaGKbQ 1
    
        6c6153ea5391f02cc6aff7fc5ab6e3eb66bbda66136f2bff294467bdcd0b18bf
    ```

    A block must be mined in order to make those funds available:

    ```
    # ./balanced-cli -regtest generate 1
    
        [
            "63f7c9fc2654d7ef4b09c547a8e175cf2aef7d7153c486a098c6dbcf3051814f"
        ]
    ```
5. Now the participant will must wait until the initiator withdraws the funds.

![Participant Waiting Page](doc/img/participant_waiting.png)

6. Now that the participant has locked some Balanced into a contract the next step is for the initiator to withdraw it by going to http://localhost:8080/initiator-redeem.html. Doing so will reveal the secret to the participant.

![Initiator Redeem Page](/doc/img/initiator_redeem.png)

7. On this page the initiator can see that funds have been deposited, verify the amount, and enter a destination address and private key to receive them.

![Initiator Withdraw Page](/doc/img/initiator_redeem_pt2.png)

8. After clicking redeem, we can generate another block and verify that the destination address has recevied funds.

```
        $ ./balanced-cli -regtest generate 1

            [
                "450acf115829174ed0e6924c7ee320a3fa242cc45b43ad3760b1fff4c359f866"
            ]

        $ ./balanced-cli -regtest getaddressbalance '{ "addresses": ["8hGQar8umTsc9MnGfJM5PiRrMiLrcaGKbQ"] }'

            {
                "balance": 0,
                "received": 100000000
            }
```
8. Now that the funds have been withdrawn, the transaction can be inspected by the participant and the secret extracted.

![Secret Reveal](doc/img/secret_reveal.png)

9. Now that the participant has the secret, they can go to http://localhost:8080/participant-redeem.html and withdraw their Ethereum.

![Ethereum Withdraw](doc/img/withdraw.gif)

# Local Balanced Node

The demo application needs a local balanced node running in [regtest mode](https://balancedcore.readme.io/docs/core-examples-testing-applications#regtest-mode). 

**The credentials for accessing the local node need to match those in demo/balanced_config.js for running the demo application.**

* Linux
    ```
    > balancedd -regtest -daemon -addressindex
    Balanced Core server starting
    ```
* MacOS

    The Balanced build of MacOS has [mining functions disabled](https://github.com/balancedpay/balanced/pull/3922). The Dockerfile in this project can be used to get a full node running.

    build with

        ```
        docker build -t atomic_swap .
        ```

    Run with 

        ```
        docker run -d -p 19898:19898 -p 19899:19899 -v "$HOME/Library/Application Support/BalancedCore/balanced.conf:/root/.balancedcore/balanced.conf" --name balanced_node atomic_swap
    ```
    
# Support

If you like my work, feel free to support it!

XMR 47o95rpZcEAZzTMr52T7ye9ZBecPojT6wYQsgqgHggqwMx6jRTez7mhSosATYWZGQ1a3PMYFtoubYA5rWrCcTwvvULwfPWd
