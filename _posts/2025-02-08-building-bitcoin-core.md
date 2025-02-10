---
layout: post
title: building bitcoin core
date: 2025-02-08 09:55 -0600
---

These are the steps that i'm using to build

1. git pull `main` from bitcoin
2. brew install cmake boost pkgconf libevent
2.   oh wait... let's see first if we need to do this. perhaps bitcoin's working already

okay, so i ended up not needing to do this... 


my major need was to connect with LND to my Bitcoind to perform RPC commands. i had previously installed a prebuilt version of bitcoin in order to start the project. so i got LND installed, and pointed it towards my bitcoind and after a couple hours of not following the directions, i finally got it to work.


here's what a LND command to build a route looks like (notice the hops are in a comma-separated string array with no spaces). this route is like: 

`(my node) -- [hop1] --> (node 3: ...f3f5) --[hop2]--> (node 2: ...017d)`


```
lncli --macaroonpath '~/Library/Application Support/Lnd/data/chain/bitcoin/signet/admin.macaroon' \
buildroute \
 --hops "02ef09cab9ba578cfa5158d6951148a276cbb005bba5d61b8f630b75769d8df3f5,03ebc7280ff486f94cee51cff45a0fc521e9fe2560c02563ab191552fa90d4017d" \
 --payment_addr 37a931abbe5d6c7def93faa72dff676252cd34303d85bafbb803b614001bfba1 \
 --amt 2342 \
 --final_cltv_delta=80
 
 ```

here's what a LND route object looks like:

```

{
    "route": {
        "total_time_lock": 6931,
        "total_fees": "1",
        "total_amt": "2343",
        "hops": [
            {
                "chan_id": "7404111301509120",
                "chan_capacity": "50000",
                "amt_to_forward": "2342",
                "fee": "1",
                "expiry": 6851,
                "amt_to_forward_msat": "2342000",
                "fee_msat": "1002",
                "pub_key": "02ef09cab9ba578cfa5158d6951148a276cbb005bba5d61b8f630b75769d8df3f5",
                "tlv_payload": true,
                "mpp_record": null,
                "amp_record": null,
                "custom_records": {},
                "metadata": "",
                "blinding_point": "",
                "encrypted_data": "",
                "total_amt_msat": "0"
            },
            {
                "chan_id": "6102289534222336",
                "chan_capacity": "100000000",
                "amt_to_forward": "2342",
                "fee": "0",
                "expiry": 6851,
                "amt_to_forward_msat": "2342000",
                "fee_msat": "0",
                "pub_key": "03ebc7280ff486f94cee51cff45a0fc521e9fe2560c02563ab191552fa90d4017d",
                "tlv_payload": true,
                "mpp_record": {
                    "payment_addr": "37a931abbe5d6c7def93faa72dff676252cd34303d85bafbb803b614001bfba1",
                    "total_amt_msat": "2342000"
                },
                "amp_record": null,
                "custom_records": {},
                "metadata": "",
                "blinding_point": "",
                "encrypted_data": "",
                "total_amt_msat": "0"
            }
        ],
        "total_fees_msat": "1002",
        "total_amt_msat": "2343002",
        "first_hop_amount_msat": "0",
        "custom_channel_data": ""
    }
}
```

lastly, here a command to send a payment across that route:

```
lncli --macaroonpath '~/Library/Application Support/Lnd/data/chain/bitcoin/signet/admin.macaroon' \
sendtoroute --payment_hash f874571681b44ee3f0e525264c9b8a573b545295b2f556a8dd63fbed44ffb9b7 - < route.json
```
