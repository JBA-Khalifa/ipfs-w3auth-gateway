# IPFS W3Auth Gateway &middot; [![GitHub license](https://img.shields.io/github/license/crustio/crust?logo=apache)](LICENSE)

IPFS W3Auth Gateway is a lightweight Web3-based authentication service basedon **IPFS gateway** and **reverse proxy**

## ❓ [About IPFS W3Auth Gateway](https://wiki.crust.network/docs/en/buildIPFSWeb3AuthGW)

## 🚀 Deployment

> Please make sure you have IPFS Gateway runnning locally, you can refer [this doc](https://docs.ipfs.io/concepts/ipfs-gateway/#overview) to config the gateway information.

### 1. Run IPFS W3Auth Gateway

- Run with docker

```shell
docker run -e PORT=5050 -e IPFS_ENDPOINT=http://localhost:5001 --network=host crustio/ipfs-w3auth
```

- Run with node native

```shell
# 1. Clone repo
git clone https://github.com/crustio/ipfs-web3-authenticator.git

# 2. Install and build
yarn && yarn build

# 3. Run
PORT=5050 IPFS_ENDPOINT=http://localhost:5001 yarn start
```

- **PORT**: W3Auth Gateway service listening port

- **IPFS_ENDPOINT**: IPFS local API endpoint

### 2. Config with reverse proxy

### 2.1 With caddy

- Auth both readable and writeable API

```txt
https://ipfs.example.com {
  reverse_proxy 127.0.0.1:5050
}
```

- Auth only writeable API

```txt
https://ipfs.example.com {
    reverse_proxy /api/* localhost:5050 {
        header_down Access-Control-Allow-Origin *
        header_down Access-Control-Allow-Methods "POST"
        header_down Access-Control-Allow-Headers *
    }

    reverse_proxy /ipfs/* localhost:8080
}
```

### 2.2 With nginx

- Auth both readable and writeable API

```conf
server {
    listen       80;
    listen  [::]:80;
    server_name  ipfs.example.com;


    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   /usr/share/nginx/html;
    }


    location / {
        proxy_http_version 1.1;
        proxy_pass   http://localhost:5050/;
        proxy_set_header Host               $http_host;
        proxy_set_header X-Real-IP          $remote_addr;
        proxy_set_header X-Forwarded-For    $proxy_add_x_forwarded_for;
        add_header Cache-Control no-cache;
    }
}
```

- Auth only writeable API

```conf
server {
    listen       80;
    listen  [::]:80;
    server_name  ipfs.example.com;


    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   /usr/share/nginx/html;
    }


    location /api {
        proxy_http_version 1.1;
        proxy_pass   http://localhost:5050/api;
        proxy_set_header Host               $http_host;
        proxy_set_header X-Real-IP          $remote_addr;
        proxy_set_header X-Forwarded-For    $proxy_add_x_forwarded_for;
        add_header Cache-Control no-cache;
    }

    location / {
        proxy_http_version 1.1;
        proxy_pass   http://localhost:8080;
        proxy_set_header Host               $http_host;
        proxy_set_header X-Real-IP          $remote_addr;
        proxy_set_header X-Forwarded-For    $proxy_add_x_forwarded_for;
        add_header Cache-Control no-cache;
    }

}
```

## 🤟🏻 Usage

The IPFS W3Auth Gateway is compatible with the official IPFS API, with the same HTTP endpoints, flags and arguments. The only additional step you must take when interacting with the IPFS W3Auth Gateway API is to configure the correct Basic Authentication header.

```curl
Authorization: Basic <base64(ChainType-PubKey:SignedMsg)>
```

Let's take `cURL` as an example 😎

```shell
curl -X POST -F file=@myfile -u "ChainType-PubKey:SignedMsg" "http://localhost:5050/api/v0/add"
```

### Get ChainType

`ChainType` now can be:

1. `sub`(or `substrate`)
2. `eth`(or `ethereum`)
3. `sol`(or `solana`)
4. `pol`(or `polygon`)
5. `nea`(or `near`)
6. `ava`(or `avalanche`)

### Get PubKey and SignedMsg

You can get `PubKey` and `SignedMsg` by using the following web3-ways:

#### 1. With Substrate

##### Get `PubKey`

`PubKey` is just the substrate address, like `5Chu5r5GA41xFgMXLQd6CDjz1ABGEGVGS276xjv93ApY6vD7`

All substrate-based chains are adapted:

- [Crust](https://apps.crust.network/?rpc=wss%3A%2F%2Frpc.crust.network#/explorer)
- [Polkadot](https://polkadot.js.org/apps/?rpc=wss%3A%2F%2Fpolkadot.elara.patract.io#/explorer)
- [Kusama](https://polkadot.js.org/apps/?rpc=wss%3A%2F%2Fkusama.api.onfinality.io%2Fpublic-ws#/explorer)
- ...

##### Get `SignedMsg`

Just sign the `PubKey` with your private key to get the `SignedMsg`

- With [Crust Apps](https://apps.crust.network/?rpc=wss%3A%2F%2Frpc.crust.network#/signing)
- With [Polkadot Apps](https://polkadot.js.org/apps/?rpc=wss%3A%2F%2Frpc.polkadot.io#/signing)
- With [Subkey](https://substrate.dev/docs/en/knowledgebase/integrate/subkey#signing-and-verifying-messages)
- With [Node SDK](https://apps.crust.network/docs/util-crypto/examples/encrypt-decrypt)

#### 2. With Ethereum

##### Get `PubKey`

`PubKey` is just the ethereum address(42-characters) start with `0x`

##### Get `SignedMsg`

Just sign the `PubKey` with your eth private key to get the `SignedMsg`

- With [MyEtherWallet](https://www.myetherwallet.com/wallet/sign)
- With [MyCrypto](https://app.mycrypto.com/sign-message)

#### 3. With Solana

##### Get `PubKey`

`PubKey` is just the solana address

##### Get `SignedMsg`

You can sign the `PubKey` with your solana private key to get the `SignedMsg`

- With [Solana Signer Sandbox](https://bafybeiexn4chci4exl54hlispdhwste6mpdcvgnu5zei53r2yl24hq2kri.ipfs.dweb.link/) (deploy with IPFS(cid: `QmYXnTQwKkup7yNLXZz2VyBvBj9eJB1knG8V8dnmjNuNnu`), source code is [here](https://github.com/zikunfan/solana-signer), you can deploy yourself)
- With [Phantom](https://docs.phantom.app/integrating/signing-a-message)

#### 4. With Polygon

##### Get `PubKey`

`PubKey` is just the polygon address(42-characters) start with `0x`. It's compatiable with the ethereum.

##### Get `SignedMsg`

Just sign the `PubKey` with your polygon private key to get the `SignedMsg`

- With [MyEtherWallet](https://www.myetherwallet.com/wallet/sign)
- With [MyCrypto](https://app.mycrypto.com/sign-message)

#### 5. With Near

You can sign the `PubKey` with one of your near private key associated with your account to get the `SignedMsg`

- With [Near Wallet Example](https://bafybeie333fxe7yw44msi4khah7pjuhn7rlvg3odquoreysj6ejbjya2oy.ipfs.dweb.link/) (deploy with IPFS(cid: `QmYq6wg5NG6mRq6e8urhLTXLsf7QAzZ8RKcgyyiX5qH45b`), source code is [here](https://github.com/MyronFanQiu/wallet-example), you can deploy yourself)
- With [Near-API-JS Example](https://github.com/near/near-api-js/blob/master/examples/cookbook/utils/verify-signature.js)

#### 6. With Avalanche

You can sign the `Address` **without** chainID prefix, such as `avax1se4e9lvhlfwhcqnzjr0vpswqcnhsy5atn5r0l3`, with your X- or P- chain private key associated with your account to get the `SignedMsg`.

- With [Avalanche Wallet](https://wallet.avax.network/wallet/advanced)

## 💻 Build

### Install

```shell
yarn
```

### Run in dev mode

```shell
yarn dev
```

### Build and run in prod mode

```shell
yarn build
yarn start
```

## 🙋🏻‍♂️ Contribute

Please feel free to send a PR

## License

[Apache 2.0](https://github.com/crustio/ipfs-web3-authenticator/blob/main/LICENSE)
