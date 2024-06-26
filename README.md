# Worklog

## Links

[Github Repository](https://github.com/alexdcox/dashpay-extension)

[Batch One](#batch-one)  
[Batch Two](#batch-two)  
[Batch Three](#batch-three)

## Log

### Batch One

```
15.08.2023     10h
16.08.2023      6h
17.08.2023     13h
28.08.2023      6h
29.08.2023      6h
31.08.2023      4h
06.09.2023 30m
Total      30m 45h
```

#### 15.08.2023 Tuesday 10h

I'm tasked with continuing the development of the Dash Pay Extension.  

It's essentially MetaMask for dash, with the focus on easy access to dash
platform features.

There has been some previous development, need to run each project and
consolidate the best parts of each:
- Ionic chat + doc
- Dashpay web extension + doc
- Dash chat + doc
- A React/Redux Manifest V3 Chrome Extension

**Dashameter DashPay Wallet Ionic / Vue**

`dashameter/dashpay-wallet`


First up, there are a lot of scripts:

```
"serve": "vue-cli-service serve",
"local": "./scripts/runLocal.sh",
"local2": "./scripts/runLocal2.sh",
"testnet": "./scripts/runTestnet.sh",
```

Which one to pick? Do I roll a dice?  
I love it when there's not a `README` file.  

```
npm i -g @ionic/cli @vue/cli

yarn

yarn serve
yarn testnet
yarn local
yarn local2
```

> @achrinza/node-ipc@9.2.2: The engine "node" is incompatible with this module. Expected version "8 || 10 || 12 || 14 || 16 || 17". Got "20.5.0"

Rolling back:

```
brew unlink node
brew link node@16
```

> @yarnpkg/pnpify@4.0.0-rc.48: The engine "node" is incompatible with this module. Expected version ">=18.12.0". Got "16.20.1"

Hilarious.  
So it was saved in a way that no node version supports the required dependencies?  

```
node > 17    breaks `node-ipc`
node < 18.12 breaks `pnpify`
```

```
rm -rf node_modules

brew unlink node
brew link --overwrite node@18
brew link --overwrite node

npm i --force --legacy-peer-deps
```

The `secp256k1-native` package is causing `npm i` issues.  
> a function declaration without a prototype is deprecated in all versions of C

Trying to update all packages to the latest they can be...

```
MODULE_NOT_FOUND
/Users/adc/code/dashpay-wallet/scripts/registerContracts.js
```

There's a file `.evoenv` which is being referenced that I don't have.  
It was apparently build by `resetDashmate`.

```
yarn run reset:dashmate
```

`dashmate` is part of dash `platform`

Oh shit, just found this in one of the scripts:
```
docker rm -f -v $(docker ps -a -q) || true
```

That would have deleted all my containers on my devmachine, including both live
and testnet nodes for a handful of currencies, and a database full of important
notes.

So glad I didn't run it.  
I can no longer trust this codebase.  

Found a reference to a faucet that's down in `runBuildTestnet.sh`:  
`autofaucet-1.dashevo.io`

```
ionic build
npx vue-cli-service serve
```

> Invalid options in vue.config.js: child "transpileDependencies" fails because ["transpileDependencies" must be an array]

`vue-cli-service` is deprecated. That above error is bull according to the vue docs:  
https://cli.vuejs.org/config/  
`true` is totally valid.  

```
vue upgrade
```

> npm ERR! ERESOLVE unable to resolve dependency tree
> npm ERR!
> npm ERR! While resolving: dashpay-wallet@0.0.1

```
NODE_OPTIONS=--openssl-legacy-provider npx vue-cli-service serve

npm install --save @capacitor/storage axios
```

Getting closer, now I'm on:

> Module '"../../../node_modules/vue/dist/vue"' has no exported member 'onMounted'

```
curl https://registry.npmjs.org/@vue/cli-service | jq '.versions | keys'

npx @vue/cli-service@5.0.8 serve
npx @vue/cli-service@4.5.19 serve
npx @vue/cli-service@4.4.6 serve
npx @vue/cli-service@4.3.1 serve
npx @vue/cli-service@4.2.3 serve
npx @vue/cli-service@4.1.2 serve
npx @vue/cli-service@3.9.3 serve
```


**5.0.8**  
> TypeError: transpileDependencies.map is not a function

**4.5.19**  
>Cannot find module '@vue/cli-plugin-babel'

```
npm i -g @vue/cli-plugin-babel
npm i -g @babel/core @babel/preset-env
yarn add @vue/cli-plugin-babel
```

Ah we can't use node 20 with this version.  
Removed nvm, in favour of using `n` or `brew link/unlink`  
REINSTALLED NVM - ITS THE ONLY WAY TO GET NODE v17 WITHOUT MANUAL INSTALL
BREW DROPS EVERY ODD NUMBERED VERSION OF NODE FOR SOME FUN REASON

```
brew link --overwrite node@16
```

Trying to add babel again.

`@achrinza/node-ipc@9.2.2`
`yarn add @achrinza/node-ipc@9.2.7`

```
curl https://registry.npmjs.org/@achrinza/node-ipc | jq '.versions | keys'
```

10.1.9

cli-shared-utils-4.5.19
  | @achrinza/node-ipc@9.2.2


@capacitor/storage": "^1.2.5
secp256k1-native": "^1.1.3

I'm losing track here, wiping, checking out, starting again...

```
npm i --legacy-peer-deps
npm run testnet
```

Bit better, it seems to be running and gets to a
`NoAvailableAddressesForRetryError` which is thrown before the app actually
starts.

`@dashevo/dapi-client > JsonRpcTransport`

```
{
  host: "seed-1.testnet.networks.dash.org",
  httpPort: 3000,
  grpcPort: 3010,
  proRegTxHash: undefined,
  banCount: 0
},
{
  host: "seed-2.testnet.networks.dash.org",
  httpPort: 3000,
  grpcPort: 3010,
  proRegTxHash: undefined,
  banCount: 0
},
{
  host: "seed-3.testnet.networks.dash.org",
  httpPort: 3000,
  grpcPort: 3010,
  proRegTxHash: undefined,
  banCount: 0
},
{
  host: "seed-4.testnet.networks.dash.org",
  httpPort: 3000,
  grpcPort: 3010,
  proRegTxHash: undefined,
  banCount: 0
},
{
  host: "seed-5.testnet.networks.dash.org",
  httpPort: 3000,
  grpcPort: 3010,
  proRegTxHash: undefined,
  banCount: 0
}
```

```
nmap -F seed-1.testnet.networks.dash.org
nc -zv -w1 -G1 seed-1.testnet.networks.dash.org 3010
nc -zv -w1 -G1 seed-2.testnet.networks.dash.org 3010
nc -zv -w1 -G1 seed-3.testnet.networks.dash.org 3010
nc -zv -w1 -G1 seed-4.testnet.networks.dash.org 3010
nc -zv -w1 -G1 seed-5.testnet.networks.dash.org 3010
```

Yeah they're all down.

Right then, time to get a dapi server running somehow?

```
curl https://registry.npmjs.org/dash | jq '.versions | keys'
```

`pshenmic` is the DCG dev who is helping with queries about this.

```
npm i dash@3.24.21 --legacy-peer-deps
```

`*.dashevo.io` domains are owned by the original maintainer, and are all down.

Just running the scripts directly now:

```
VUE_APP_MNEMONIC="rival estate inside turn journey charge window rhythm marble audit amateur bus" \
VUE_APP_ENV_RUN="testnet" \
node ./scripts/registerContracts.js
```

```
VUE_APP_ENV_RUN="testnet" \
node ./scripts/registerContracts.js
```

Interesting. Running with the mnemonic causes it to hang. Despite `skipSynchronizationBeforeHeight: 874000`.

```
cd ~/go/src/github.com/dashpay/platform/packages/js-dash-sdk
npm link
```

For this segment of code in `registerContracts`:
```js
const i = setInterval(async () => {
  const bestBlockHash = await client.getDAPIClient().core.getBestBlockHash();
  const bestBlock = new SDK.Core.Block(
    await client.getDAPIClient().core.getBlockByHash(bestBlockHash),
  );
  const bestBlockHeight = bestBlock.transactions[0].extraPayload.height;
  console.log(`block: ${bestBlockHeight}`)
}, 5000)

console.log('BEFORE GET WALLET ACCOUNT')
client.account = await client.getWalletAccount();
console.log('AFTER GET WALLET ACCOUNT')
```

`AFTER` is never hit.

Going to have to compile/link the sources to debug this...

```
cd ./packages/js-dash-sdk/
npm link
```

Ahh the `unsafeOptions` and `skipSynchronizationBeforeHeight` was a layer too
high, on the `Client` rather than the `Wallet`. So it was syncing, even though
it was returning the latest block.

Using the latest version of the `dash` library the contract schema has changed.
I'm getting this error:

```
[StateTransitionBroadcastError: JsonSchemaError: keyword: required, instance_path: /documents/contactInfo/indices/0, schema_path:/properties/documents/additionalProperties/allOf/0/properties/indices/items/required] {
  code: 1005,
  cause: [JsonSchemaError: JsonSchemaError: keyword: required, instance_path: /documents/contactInfo/indices/0, schema_path:/properties/documents/additionalProperties/allOf/0/properties/indices/items/required] {
    __wbg_ptr: 5149328
  }
}
```

Managed to correct it using the dashpay.io validator, turns out I missed some
names on the indicies.

Anyway I don't think we even need these contracts, I'm going to keep on keeping
on...

```
nvm install 18
nvm install 17
nvm install 16

npm i --legacy-peer-deps

npm i @achrinza/node-ipc@9.2.7 --legacy-peer-deps
npm i dash@3.24.21 --legacy-peer-deps
npm i @vue/cli-plugin-babel --legacy-peer-deps

npx @vue/cli-service@5.0.8 serve
npx @vue/cli-service@4.5.19 serve

npm remove --save-dev --legacy-peer-deps @vue/cli-plugin-babel
npm remove --save-dev --legacy-peer-deps @vue/cli-plugin-e2e-cypress
npm remove --save-dev --legacy-peer-deps @vue/cli-plugin-eslint
npm remove --save-dev --legacy-peer-deps @vue/cli-plugin-router
npm remove --save-dev --legacy-peer-deps @vue/cli-plugin-typescript
npm remove --save-dev --legacy-peer-deps @vue/cli-plugin-unit-jest
npm remove --save-dev --legacy-peer-deps @vue/cli-service

npm i -g @vue/compiler-sfc
npm i --legacy-peer-deps @vue/compiler-sfc
```

```
nvm install 16
npm i --legacy-peer-deps
npm i dash@3.24.21 --legacy-peer-deps
npm i axios --legacy-peer-deps
npm -g i @vue/cli-plugin-babel
npm -g i @vue/cli-plugin-e2e-cypress
npm -g i @vue/cli-plugin-eslint
npm -g i @vue/cli-plugin-typescript
npm -g i @vue/cli-plugin-unit-jest
npx @vue/cli-service@4.5.19 serve
export NODE_PATH="/Users/adc/.nvm/versions/node/v16.20.2/lib/node_modules"
```

```
"@vue/cli-plugin-babel": "~5.0.0",
```

Okay it's working again.
```
node            v16.20.2
npm/npx           8.19.4
@vue/cli-service  4.5.15
commit            2f49d8d (this is actually the next commit on from 640c569, which I added to add logging to the register contracts script)
```

I removed LogRocket as it was sending analytics requests.  
There are a few env variables that still need to be set:

```
egrep -r -E -i -o -h "process\.env\.[A-z]+" src | sort | uniq
```

```
process.env.BASE_URL
process.env.VUE_APP_AUTOFAUCET
process.env.VUE_APP_DAPIADDRESSES
process.env.VUE_APP_DASHPAY_CONTRACT_ID
process.env.VUE_APP_DASHPAY_WALLET_CONTRACT_ID_build_testnet
process.env.VUE_APP_DASHPAY_WALLET_CONTRACT_ID_local
process.env.VUE_APP_DASHPAY_WALLET_CONTRACT_ID_testnet
process.env.VUE_APP_DPNS_CONTRACT_ID
process.env.VUE_APP_ENV_RUN
process.env.VUE_APP_USERMNEMONIC
```

I only see 3 contracts delpoyed on the testnet atm:
```
- dpns GWRSAVFMjXx8HpQFaNJMqBV7MBgMK4br5UESsB4S31Ec
- dashpay Bwr4WHCPz5rFVAD87RqTs3izo4zpzwsEdKPWUT1NS1C7
- masternodeRewardShares rUnsWrFu3PKyRMGk2mxmZVBPbQuZx2qtHeFjURoQevX
```

```
VUE_APP_MNEMONIC="rival estate inside turn journey charge window rhythm marble audit amateur bus" \
VUE_APP_USERMNEMONIC="rival estate inside turn journey charge window rhythm marble audit amateur bus" \
VUE_APP_ENV_RUN="testnet" \
VUE_APP_DAPIADDRESSES="seed-1.testnet.networks.dash.org" \
VUE_APP_DASHPAY_CONTRACT_ID="Bwr4WHCPz5rFVAD87RqTs3izo4zpzwsEdKPWUT1NS1C7" \
VUE_APP_DPNS_CONTRACT_ID="GWRSAVFMjXx8HpQFaNJMqBV7MBgMK4br5UESsB4S31Ec" \
npx @vue/cli-service@4.5.15 serve
```


#### 16.08.2023 Wednesday 6h

Right, I managed to get the dashameter project running yesterday, but didn't
actually get far in the UI.

Going to start by seeing how much functionality I can restore...

Oh right I updated the dash package to try and connect to the new nodes (no longer port 3000)

New compile error in: `DashClient.ts`:
```
Namespace '"/Users/adc/code/dashpay-wallet/node_modules/dash/build/SDK/SDK".SDK' has no exported member 'Client'.
import Dash from "dash"
let client: Dash.Client | undefined
```

```
import { Client } from "dash/src/SDK/Client";
```

```
npm link
npm link platform --legacy-peer-deps
```

```
npm audit fix --force
vue upgrade

vue serve
```

> ValidationError: Progress Plugin Invalid Options

```
npm i @vue/cli
npm remove @dashpay-wallet --force
npm remove eslint --force
npm update
```

```
npm update @typescript-eslint/eslint-plugin --legacy-peer-deps
npm update --save
```

```
vue serve
vue upgrade
```

> Module '"../node_modules/vue/dist/vue"' has no exported member 'onErrorCaptured'.

```
npm i --legacy-peer-deps
```

This isn't working. Getting weird export issues that I have no idea how to fix.
Dash.Client is not exported from 'dash', onErrorCaptured and other hooks are
not exported from 'vue'. I know it's got something to do with the way the modules
are imported but it would take a lot of learning for me to dive deeper that I
feel like I should just backtrack to a state where things were actually working.

```
git reset head^1 --hard
```

Now I'm getting `node-gyp` issues again.
`a function declaration without a prototype is deprecated in all versions of C`


```
npm i --legacy-peer-deps

npm i @achrinza/node-ipc@9.2.7 --legacy-peer-deps
npm i dash@3.24.21 --legacy-peer-deps
npm i @vue/cli-plugin-babel --legacy-peer-deps

npx @vue/cli-service@5.0.8 serve
npx @vue/cli-service@4.5.19 serve
npx @vue/cli-service@4.5.15 serve
```

• To fix node-gyp:
  Remove `secp256k1-native` from package json then run `npm i secp256k1-native --legacy-peer-deps`

• To fix `ERR_OSSL_EVP_UNSUPPORTED`:
  `export NODE_OPTIONS=--openssl-legacy-provider`


• To fix `cannot find module` with `npx` when you've installed globally
  ```
  export NODE_PATH=`cd $(dirname $(which node))/../lib/node_modules;pwd`
  npm i -g <module>
  ```

• To fix `TypeError: loaderContext.getOptions is not a function`
  UNRESOLVED
  This one is tricky because the suggestions involve either upgrading webpack or
  downgrading ts-loader, both of which are required by other packages (presumably)
  vue, so I'm not sure how best to go about that.

  ```
  npm i ts-loader@~8.2.0 --legacy-peer-deps;
  npm i -g ts-loader@~8.2.0;
  ```


• To fix `TypeError: transpileDependencies.map is not a function`
  Update `cli-plugin-babel` in package.json to `^5.0.0` this will cause knock-on issues


```
npm i ts-loader@~8.2.0 --legacy-peer-deps
npm update --legacy-peer-deps
```

• To fix `'e' is of type 'unknown'`
  This is more of a workaround, but you can set `const b: any = e` and use `b` 
  instead.

• To fix `'Dash.Client' refers to a value, but is being used as a type here. Did you mean 'typeof Dash.Client'?`

I think 2 days spent taking this approach is enough time wasted.

New idea:  
Build an entirely new blank project with the latest versions. 
Copy this project over file by file.  

```
browserify-fs
url
stream-browserify
```

```
npm i --save assert crypto-browserify path-browserify stream-browserify url
npm i --force --save browserify-fs
```

```
"vuex": "^4.0.0"
```

• `vuex` has been superseded by `pinia`  
I'll try and get vuex working first...

• `capacitor - Storage` has been superseded by `Preferences`

• `IonSlide` has been replaced with `Swiper`
https://ionicframework.com/docs/v6/vue/slides

• Need to refactor these:
```
process.env. => import.meta.env.
require( => import 
```

Actually before copying into a blank project, I tried transplanting the working
config files into the original project. It's not running smoothly yet, but I'm
making tangible progress...

It's building.

Need to setup the `vue.config.js.bak`  
`module.exports.configureWebpack.resolve.fallback` in the new vite config at
`vite.config.ts`.

He's using `require` for svgs which is no longer recommended  
https://dev.to/geowrgetudor/dealing-with-svg-icons-in-vue-vite-an9

```
const Dashcore = require("@dashevo/dashcore-lib");
VVV
import * as Dashcore from '@dashevo/dashcore-lib'
```

#### 17.08.2023 Thursday 13h (11am - 1am, 1h break)

So the old app with the new config is loading up, no errors, and yet I'm seeing
a white screen within the phone template.

The default route took you from `/` to `/device`, which was broken, so I
switched that out to `/welcome`.

Switched a lot of the background colours to white in `variables.css` so that
I can see what's actually going on.

He's using a lot of `require(svg)` calls which are not supported. Going to just
go through and fix all of those now.


This is the magic right here:
```
import { Client } from "dash";
let client: InstanceType<typeof Client>;
```

Well, been refactoring my way through. Trying to import the `Identity` type that's
returned from wallet.identities.get buut, it's coming from `@dashevo/dpp` or more
precisely, the dev package `@dashevo/wasm-dpp`, maybe I found it, maybe not.

For the choose a username form, I'd rather have 0 debounce of the static rules
and a longer debounce for the actual fetch. i.e. show the local validation
immediately, and if that passes make a request... but I'll just get this thing
running first.

The form submit event was not prevented so it was refreshing my page.

Quick vue revison / cheatsheet:
```
v-on @
v-bind `:src="url"` `:[key]="target"`
`<div .someprop="targetObj"></div>`
v-slot #
```

Can do `<form @submit.prevent></form>` which binds the `onsubmit` event with `preventDefault`
to nothing.

- [ ] Refactor all `require`
- [ ] Refactor all getClient()

Just stumbled on an issue where for some reason the password field wasn't being
bound correctly, sometimes, randomly, so the mnemonic was on occasion being
encrypted with an empty string. That's not good. Turns out, there was a
debounce on the input validation of the password field as it makes a request to
platform to verify the username is available, it only binds the changed value
once that returns, and I was submitting the form too quickly. 🤦‍♂️ That wasted
at least 10 mins goddamit.

```
store.dispatch("showToast", { text: error, color: "danger" });
```

#### 28.08.2023 Monday 6h

Mayanode setup and other work have taken priority until today.

Back to working through the dashameter wallet, learning what I can from it.

Forgot how to actually run it:

```
npm run dev
```

```
localStorage['CapacitorStorage.accounts']
console.log(JSON.stringify(JSON.parse(localStorage['CapacitorStorage.accounts']), null, 2))
```

```
[
  {
    "wishName": "something",
    "accountDPNS": null,
    "id": "i5zA4+bPWAVJFAp9PvsM2c3msPVqCZCuGHtel2ndIP8=",
    "encMnemonic": "U2FsdGVkX1/E2F3x4ZpaA/oQvnKT2aHHiBtNXYJYOIX0tpBkql5iXyNcnr1KkQnZknTCiZPU+RBtmvPRuY5tWYjcBk2wGnaIN6JpYQVk+SLY7gbVKSN1oZ96BWdn8bj/"
  }
]
```

Looks like the save + retrieval of the user account isn't working.

Because of the way the app is always calling the Dash client method it's
difficult to tell when we actually want to create a new wallet, or when we want
to wait for the user to load a wallet.

Think it'd be better to have explicit methods for:  
- create new wallet
- load wallet from mnemonic
- wait for either of the above options to be selected, then use that

It goes into this `Redeeming invite code...` screen.  
The console spits out an address that you can use to topup the balance though.

`startRefreshWalletDataLoop` not working, could be that I've broken it.

```
dash-cli getaddressbalance '{"addresses": ["ybGmBc5BPxg263Jteab83YybQzgafy6JUF"]}'
```

Where's the logic to load a 'selected' account from localStorage on refresh?

`AccountStorage` has 3 methods: `store`, `update`, and `list`.  
No `get` or `getCurrent` / `getSelected` etc.  
Does it just use the list?  

Oh yeah, and the password restore doesn't seem right.  
I'll try and recover the account...  

```
localStorage.clear()
```

Right now I'm looking at a completely different backup phrase to the one I
entered via the login/recover flow.

- Fix password / retrieving mnemonic (was set to validate 16 not 12 recovery words)
- Fix mnemonic screen being displayed when recovering a wallet
- Fix on login, post load wallet, nothing happens (My fault, needed to call recoverWallet in `ChooseAccount`)
- Sometimes you can trigger an `Undefined` toast error which doesn't disappear and cannot be dismissed
- Use the documented DPNS validation pattern: `^[a-zA-Z0-9][a-zA-Z0-9-]{0,61}[a-zA-Z0-9]$`

In the password prompt modal it's trying to display the `accountDPNS` (which I
assume they actually mean to drop the 'S/Service' like `accountDPN`, `dpnsName`
or simply `platformName`) but it can't because I'm on the finish registration
flow and it hasn't been assigned yet.

Couple of gripes as I've been using the app:
- Not having forms for submit on return key
- Going back in the flow and then forward again quite often breaks things

Right so I'm trying to follow the way the dashpay docs suggest interacting with
the DPNS service: https://github.com/dashevo/js-dpns-client

But unfortunately this library isn't compatible with my project due to no TS typings.

```
npm install -D @types/@dashevo/dapi-client
npm install -D @types/dashevo/dapi-client
npm install -D @types/dashevo__dapi-client
```

Nope.

Oh that's a red herring, you can access all this from the normal client facade.  
Brilliant.  

How do I find the dpns name for my identity?

Trying this:

```
const dpnsDoc = await client.platform?.documents?.get("dpns.domain", {
  where: [["normalizedParentDomainName", "==", "dash"]],
  orderBy: [["normalizedLabel", "asc"]],
  startAt: 1,
  limit: 25,
})
```

Getting this:

```
"AssertionError: Failure: Type not convertible to Uint8Array.
    at new goog.asserts.AssertionError (http://localhost:5173/node_modules/.vite/deps/dash.js?v=7de267c8:19652:28)
    at goog.asserts.fail (http://localhost:5173/node_modules/.vite/deps/dash.js?v=7de267c8:19674:69)
    at jspb.utils.byteSourceToUint8Array (http://localhost:5173/node_modules/.vite/deps/dash.js?v=7de267c8:21553:270)
    at jspb.BinaryWriter.writeBytes (http://localhost:5173/node_modules/.vite/deps/dash.js?v=7de267c8:22306:40)
    at C.org.dash.platform.dapi.v0.GetDocumentsRequest.serializeBinaryToWriter (http://localhost:5173/node_modules/.vite/deps/dash.js?v=7de267c8:38217:439)
    at C.org.dash.platform.dapi.v0.GetDocumentsRequest.serializeBinary (http://localhost:5173/node_modules/.vite/deps/dash.js?v=7de267c8:38214:66)
    at I2.frameRequest (http://localhost:5173/node_modules/.vite/deps/dash.js?v=7de267c8:770:25)
    at A3.send (http://localhost:5173/node_modules/.vite/deps/dash.js?v=7de267c8:435:26)
    at I2.unary (http://localhost:5173/node_modules/.vite/deps/dash.js?v=7de267c8:763:41)
    at E.getDocuments (http://localhost:5173/node_modules/.vite/deps/dash.js?v=7de267c8:38777:22)"
```

Had to do a little digging but found this: `platform/packages/js-dapi-client/docs/usage/application/getDocuments.md`
```
| parameters             | type               | required       | Description                                                                                             |
|------------------------|--------------------|----------------| ------------------------------------------------------------------------------------------------ |
| **contractId**         | String             | yes            | A valid registered contractId |
| **type**               | String             | yes            | DAP object type to fetch (e.g: 'preorder' in DPNS)    |
| **options.where**      | Object             | yes            | Mongo-like query |
| **options.orderBy**    | Object             | yes            | Mongo-like sort field |
| **options.limit**      | Number             | yes            | Limit the number of object to fetch |
| **options.startAt**    | Number             | yes            | number of objects to skip |
| **options.startAfter** | Number             | yes            | exclusive skip |
```

Here are the burning questions:

- How do I get the schema for an app
- How do I get the dpns name registered for an identity

For some reason the `app.contract` field is undefined, when called from
`client.getApps()`.

```
this.client.getDAPIClient().platform.getDataContract(contractId)
this.client.getDAPIClient().platform.getDocuments(appDefinition.contractId, fieldType, opts)
this.client.getDAPIClient().platform.getIdentity(identifier)
const blockHash = await client.getDAPIClient().core.getBestBlockHash();
const blockBinary = await client.getDAPIClient().core.getBlockByHash(blockHash);

await client.getDAPIClient().platform.getIdentitiesByPublicKeyHashes([identity.getPublicKeyById(0).hash()]);

const dapiClient = await dashClient.getDAPIClient();
const identityId = Identifier.from(dpnsOwnerId);
const identity = await dashClient.platform.identities.get(identityId);



```

This has some good hints on what those document query options look like:  
`dashpay/platform/packages/js-dash-sdk/build/SDK/Client/Platform/methods/documents/get.js`

#### 29.08.2023 Tuesday 6h

Managed to get the data contract back, but it's a protobuf schema.

```
AaVjJGlkWCDmaMZZr2au4ecsGG3ee1t+Ch1xKgnEDVch9iK/U8UxVWckc2NoZW1heDRodHRwczovL3NjaGVtYS5kYXNoLm9yZy9kcHAtMC00LTAvbWV0YS9kYXRhLWNvbnRyYWN0Z293bmVySWRYIDASwZuY7AAzrds2zWS39RBnDyo1GkMEtfaZQUQobv2sZ3ZlcnNpb24BaWRvY3VtZW50c6JmZG9tYWlupmR0eXBlZm9iamVjdGdpbmRpY2Vzg6NkbmFtZXJwYXJlbnROYW1lQW5kTGFiZWxmdW5pcXVl9Wpwcm9wZXJ0aWVzgqF4Gm5vcm1hbGl6ZWRQYXJlbnREb21haW5OYW1lY2FzY6Fvbm9ybWFsaXplZExhYmVsY2FzY6NkbmFtZW5kYXNoSWRlbnRpdHlJZGZ1bmlxdWX1anByb3BlcnRpZXOBoXgccmVjb3Jkcy5kYXNoVW5pcXVlSWRlbnRpdHlJZGNhc2OiZG5hbWVpZGFzaEFsaWFzanByb3BlcnRpZXOBoXgbcmVjb3Jkcy5kYXNoQWxpYXNJZGVudGl0eUlkY2FzY2gkY29tbWVudHkBN0luIG9yZGVyIHRvIHJlZ2lzdGVyIGEgZG9tYWluIHlvdSBuZWVkIHRvIGNyZWF0ZSBhIHByZW9yZGVyLiBUaGUgcHJlb3JkZXIgc3RlcCBpcyBuZWVkZWQgdG8gcHJldmVudCBtYW4taW4tdGhlLW1pZGRsZSBhdHRhY2tzLiBub3JtYWxpemVkTGFiZWwgKyAnLicgKyBub3JtYWxpemVkUGFyZW50RG9tYWluIG11c3Qgbm90IGJlIGxvbmdlciB0aGFuIDI1MyBjaGFycyBsZW5ndGggYXMgZGVmaW5lZCBieSBSRkMgMTAzNS4gRG9tYWluIGRvY3VtZW50cyBhcmUgaW1tdXRhYmxlOiBtb2RpZmljYXRpb24gYW5kIGRlbGV0aW9uIGFyZSByZXN0cmljdGVkaHJlcXVpcmVkhmVsYWJlbG9ub3JtYWxpemVkTGFiZWx4Gm5vcm1hbGl6ZWRQYXJlbnREb21haW5OYW1lbHByZW9yZGVyU2FsdGdyZWNvcmRzbnN1YmRvbWFpblJ1bGVzanByb3BlcnRpZXOmZWxhYmVspWR0eXBlZnN0cmluZ2dwYXR0ZXJueCpeW2EtekEtWjAtOV1bYS16QS1aMC05LV17MCw2MX1bYS16QS1aMC05XSRpbWF4TGVuZ3RoGD9pbWluTGVuZ3RoA2tkZXNjcmlwdGlvbngZRG9tYWluIGxhYmVsLiBlLmcuICdCb2InLmdyZWNvcmRzpmR0eXBlZm9iamVjdGgkY29tbWVudHiQQ29uc3RyYWludCB3aXRoIG1heCBhbmQgbWluIHByb3BlcnRpZXMgZW5zdXJlIHRoYXQgb25seSBvbmUgaWRlbnRpdHkgcmVjb3JkIGlzIHVzZWQgLSBlaXRoZXIgYSBgZGFzaFVuaXF1ZUlkZW50aXR5SWRgIG9yIGEgYGRhc2hBbGlhc0lkZW50aXR5SWRganByb3BlcnRpZXOic2Rhc2hBbGlhc0lkZW50aXR5SWSnZHR5cGVlYXJyYXloJGNvbW1lbnR4I011c3QgYmUgZXF1YWwgdG8gdGhlIGRvY3VtZW50IG93bmVyaG1heEl0ZW1zGCBobWluSXRlbXMYIGlieXRlQXJyYXn1a2Rlc2NyaXB0aW9ueD1JZGVudGl0eSBJRCB0byBiZSB1c2VkIHRvIGNyZWF0ZSBhbGlhcyBuYW1lcyBmb3IgdGhlIElkZW50aXR5cGNvbnRlbnRNZWRpYVR5cGV4IWFwcGxpY2F0aW9uL3guZGFzaC5kcHAuaWRlbnRpZmllcnRkYXNoVW5pcXVlSWRlbnRpdHlJZKdkdHlwZWVhcnJheWgkY29tbWVudHgjTXVzdCBiZSBlcXVhbCB0byB0aGUgZG9jdW1lbnQgb3duZXJobWF4SXRlbXMYIGhtaW5JdGVtcxggaWJ5dGVBcnJhefVrZGVzY3JpcHRpb254PklkZW50aXR5IElEIHRvIGJlIHVzZWQgdG8gY3JlYXRlIHRoZSBwcmltYXJ5IG5hbWUgdGhlIElkZW50aXR5cGNvbnRlbnRNZWRpYVR5cGV4IWFwcGxpY2F0aW9uL3guZGFzaC5kcHAuaWRlbnRpZmllcm1tYXhQcm9wZXJ0aWVzAW1taW5Qcm9wZXJ0aWVzAXRhZGRpdGlvbmFsUHJvcGVydGllc/RscHJlb3JkZXJTYWx0pWR0eXBlZWFycmF5aG1heEl0ZW1zGCBobWluSXRlbXMYIGlieXRlQXJyYXn1a2Rlc2NyaXB0aW9ueCJTYWx0IHVzZWQgaW4gdGhlIHByZW9yZGVyIGRvY3VtZW50bnN1YmRvbWFpblJ1bGVzpWR0eXBlZm9iamVjdGhyZXF1aXJlZIFvYWxsb3dTdWJkb21haW5zanByb3BlcnRpZXOhb2FsbG93U3ViZG9tYWluc6NkdHlwZWdib29sZWFuaCRjb21tZW50eE9Pbmx5IHRoZSBkb21haW4gb3duZXIgaXMgYWxsb3dlZCB0byBjcmVhdGUgc3ViZG9tYWlucyBmb3Igbm9uIHRvcC1sZXZlbCBkb21haW5za2Rlc2NyaXB0aW9ueFtUaGlzIG9wdGlvbiBkZWZpbmVzIHdobyBjYW4gY3JlYXRlIHN1YmRvbWFpbnM6IHRydWUgLSBhbnlvbmU7IGZhbHNlIC0gb25seSB0aGUgZG9tYWluIG93bmVya2Rlc2NyaXB0aW9ueEJTdWJkb21haW4gcnVsZXMgYWxsb3cgZG9tYWluIG93bmVycyB0byBkZWZpbmUgcnVsZXMgZm9yIHN1YmRvbWFpbnN0YWRkaXRpb25hbFByb3BlcnRpZXP0b25vcm1hbGl6ZWRMYWJlbKVkdHlwZWZzdHJpbmdncGF0dGVybnghXlthLXowLTldW2EtejAtOS1dezAsNjF9W2EtejAtOV0kaCRjb21tZW50eGlNdXN0IGJlIGVxdWFsIHRvIHRoZSBsYWJlbCBpbiBsb3dlcmNhc2UuIFRoaXMgcHJvcGVydHkgd2lsbCBiZSBkZXByZWNhdGVkIGR1ZSB0byBjYXNlIGluc2Vuc2l0aXZlIGluZGljZXNpbWF4TGVuZ3RoGD9rZGVzY3JpcHRpb254UERvbWFpbiBsYWJlbCBpbiBsb3dlcmNhc2UgZm9yIGNhc2UtaW5zZW5zaXRpdmUgdW5pcXVlbmVzcyB2YWxpZGF0aW9uLiBlLmcuICdib2IneBpub3JtYWxpemVkUGFyZW50RG9tYWluTmFtZaZkdHlwZWZzdHJpbmdncGF0dGVybngmXiR8XlthLXowLTldW2EtejAtOS1cLl17MCw2MX1bYS16MC05XSRoJGNvbW1lbnR4jE11c3QgZWl0aGVyIGJlIGVxdWFsIHRvIGFuIGV4aXN0aW5nIGRvbWFpbiBvciBlbXB0eSB0byBjcmVhdGUgYSB0b3AgbGV2ZWwgZG9tYWluLiBPbmx5IHRoZSBkYXRhIGNvbnRyYWN0IG93bmVyIGNhbiBjcmVhdGUgdG9wIGxldmVsIGRvbWFpbnMuaW1heExlbmd0aBg/aW1pbkxlbmd0aABrZGVzY3JpcHRpb254XkEgZnVsbCBwYXJlbnQgZG9tYWluIG5hbWUgaW4gbG93ZXJjYXNlIGZvciBjYXNlLWluc2Vuc2l0aXZlIHVuaXF1ZW5lc3MgdmFsaWRhdGlvbi4gZS5nLiAnZGFzaCd0YWRkaXRpb25hbFByb3BlcnRpZXP0aHByZW9yZGVypmR0eXBlZm9iamVjdGdpbmRpY2VzgaNkbmFtZWpzYWx0ZWRIYXNoZnVuaXF1ZfVqcHJvcGVydGllc4GhcHNhbHRlZERvbWFpbkhhc2hjYXNjaCRjb21tZW50eEpQcmVvcmRlciBkb2N1bWVudHMgYXJlIGltbXV0YWJsZTogbW9kaWZpY2F0aW9uIGFuZCBkZWxldGlvbiBhcmUgcmVzdHJpY3RlZGhyZXF1aXJlZIFwc2FsdGVkRG9tYWluSGFzaGpwcm9wZXJ0aWVzoXBzYWx0ZWREb21haW5IYXNopWR0eXBlZWFycmF5aG1heEl0ZW1zGCBobWluSXRlbXMYIGlieXRlQXJyYXn1a2Rlc2NyaXB0aW9ueFlEb3VibGUgc2hhLTI1NiBvZiB0aGUgY29uY2F0ZW5hdGlvbiBvZiBhIDMyIGJ5dGUgcmFuZG9tIHNhbHQgYW5kIGEgbm9ybWFsaXplZCBkb21haW4gbmFtZXRhZGRpdGlvbmFsUHJvcGVydGllc/Q=
```
```
01a5632469645820e668c659af66aee1e72c186dde7b5b7e0a1d712a09c40d5721f622bf53c531556724736368656d61783468747470733a2f2f736368656d612e646173682e6f72672f6470702d302d342d302f6d6574612f646174612d636f6e7472616374676f776e6572496458203012c19b98ec0033addb36cd64b7f510670f2a351a4304b5f6994144286efdac6776657273696f6e0169646f63756d656e7473a266646f6d61696ea66474797065666f626a65637467696e646963657383a3646e616d6572706172656e744e616d65416e644c6162656c66756e69717565f56a70726f7065727469657382a1781a6e6f726d616c697a6564506172656e74446f6d61696e4e616d6563617363a16f6e6f726d616c697a65644c6162656c63617363a3646e616d656e646173684964656e74697479496466756e69717565f56a70726f7065727469657381a1781c7265636f7264732e64617368556e697175654964656e74697479496463617363a2646e616d656964617368416c6961736a70726f7065727469657381a1781b7265636f7264732e64617368416c6961734964656e746974794964636173636824636f6d6d656e74790137496e206f7264657220746f207265676973746572206120646f6d61696e20796f75206e65656420746f206372656174652061207072656f726465722e20546865207072656f726465722073746570206973206e656564656420746f2070726576656e74206d616e2d696e2d7468652d6d6964646c652061747461636b732e206e6f726d616c697a65644c6162656c202b20272e27202b206e6f726d616c697a6564506172656e74446f6d61696e206d757374206e6f74206265206c6f6e676572207468616e20323533206368617273206c656e67746820617320646566696e65642062792052464320313033352e20446f6d61696e20646f63756d656e74732061726520696d6d757461626c653a206d6f64696669636174696f6e20616e642064656c6574696f6e20617265207265737472696374656468726571756972656486656c6162656c6f6e6f726d616c697a65644c6162656c781a6e6f726d616c697a6564506172656e74446f6d61696e4e616d656c7072656f7264657253616c74677265636f7264736e737562646f6d61696e52756c65736a70726f70657274696573a6656c6162656ca5647479706566737472696e67677061747465726e782a5e5b612d7a412d5a302d395d5b612d7a412d5a302d392d5d7b302c36317d5b612d7a412d5a302d395d24696d61784c656e677468183f696d696e4c656e677468036b6465736372697074696f6e7819446f6d61696e206c6162656c2e20652e672e2027426f62272e677265636f726473a66474797065666f626a6563746824636f6d6d656e747890436f6e73747261696e742077697468206d617820616e64206d696e2070726f7065727469657320656e737572652074686174206f6e6c79206f6e65206964656e74697479207265636f72642069732075736564202d206569746865722061206064617368556e697175654964656e74697479496460206f722061206064617368416c6961734964656e746974794964606a70726f70657274696573a27364617368416c6961734964656e746974794964a764747970656561727261796824636f6d6d656e7478234d75737420626520657175616c20746f2074686520646f63756d656e74206f776e6572686d61784974656d731820686d696e4974656d73182069627974654172726179f56b6465736372697074696f6e783d4964656e7469747920494420746f206265207573656420746f2063726561746520616c696173206e616d657320666f7220746865204964656e7469747970636f6e74656e744d656469615479706578216170706c69636174696f6e2f782e646173682e6470702e6964656e7469666965727464617368556e697175654964656e746974794964a764747970656561727261796824636f6d6d656e7478234d75737420626520657175616c20746f2074686520646f63756d656e74206f776e6572686d61784974656d731820686d696e4974656d73182069627974654172726179f56b6465736372697074696f6e783e4964656e7469747920494420746f206265207573656420746f2063726561746520746865207072696d617279206e616d6520746865204964656e7469747970636f6e74656e744d656469615479706578216170706c69636174696f6e2f782e646173682e6470702e6964656e7469666965726d6d617850726f70657274696573016d6d696e50726f7065727469657301746164646974696f6e616c50726f70657274696573f46c7072656f7264657253616c74a56474797065656172726179686d61784974656d731820686d696e4974656d73182069627974654172726179f56b6465736372697074696f6e782253616c74207573656420696e20746865207072656f7264657220646f63756d656e746e737562646f6d61696e52756c6573a56474797065666f626a656374687265717569726564816f616c6c6f77537562646f6d61696e736a70726f70657274696573a16f616c6c6f77537562646f6d61696e73a3647479706567626f6f6c65616e6824636f6d6d656e74784f4f6e6c792074686520646f6d61696e206f776e657220697320616c6c6f77656420746f2063726561746520737562646f6d61696e7320666f72206e6f6e20746f702d6c6576656c20646f6d61696e736b6465736372697074696f6e785b54686973206f7074696f6e20646566696e65732077686f2063616e2063726561746520737562646f6d61696e733a2074727565202d20616e796f6e653b2066616c7365202d206f6e6c792074686520646f6d61696e206f776e65726b6465736372697074696f6e7842537562646f6d61696e2072756c657320616c6c6f7720646f6d61696e206f776e65727320746f20646566696e652072756c657320666f7220737562646f6d61696e73746164646974696f6e616c50726f70657274696573f46f6e6f726d616c697a65644c6162656ca5647479706566737472696e67677061747465726e78215e5b612d7a302d395d5b612d7a302d392d5d7b302c36317d5b612d7a302d395d246824636f6d6d656e7478694d75737420626520657175616c20746f20746865206c6162656c20696e206c6f776572636173652e20546869732070726f70657274792077696c6c20626520646570726563617465642064756520746f206361736520696e73656e73697469766520696e6469636573696d61784c656e677468183f6b6465736372697074696f6e7850446f6d61696e206c6162656c20696e206c6f7765726361736520666f7220636173652d696e73656e73697469766520756e697175656e6573732076616c69646174696f6e2e20652e672e2027626f6227781a6e6f726d616c697a6564506172656e74446f6d61696e4e616d65a6647479706566737472696e67677061747465726e78265e247c5e5b612d7a302d395d5b612d7a302d392d5c2e5d7b302c36317d5b612d7a302d395d246824636f6d6d656e74788c4d7573742065697468657220626520657175616c20746f20616e206578697374696e6720646f6d61696e206f7220656d70747920746f20637265617465206120746f70206c6576656c20646f6d61696e2e204f6e6c7920746865206461746120636f6e7472616374206f776e65722063616e2063726561746520746f70206c6576656c20646f6d61696e732e696d61784c656e677468183f696d696e4c656e677468006b6465736372697074696f6e785e412066756c6c20706172656e7420646f6d61696e206e616d6520696e206c6f7765726361736520666f7220636173652d696e73656e73697469766520756e697175656e6573732076616c69646174696f6e2e20652e672e20276461736827746164646974696f6e616c50726f70657274696573f4687072656f72646572a66474797065666f626a65637467696e646963657381a3646e616d656a73616c7465644861736866756e69717565f56a70726f7065727469657381a17073616c746564446f6d61696e48617368636173636824636f6d6d656e74784a5072656f7264657220646f63756d656e74732061726520696d6d757461626c653a206d6f64696669636174696f6e20616e642064656c6574696f6e206172652072657374726963746564687265717569726564817073616c746564446f6d61696e486173686a70726f70657274696573a17073616c746564446f6d61696e48617368a56474797065656172726179686d61784974656d731820686d696e4974656d73182069627974654172726179f56b6465736372697074696f6e7859446f75626c65207368612d323536206f662074686520636f6e636174656e6174696f6e206f66206120333220627974652072616e646f6d2073616c7420616e642061206e6f726d616c697a656420646f6d61696e206e616d65746164646974696f6e616c50726f70657274696573f4
```

https://schema.dash.org/dpp-0-4-0/meta/data-contract  
They're using CBOR not Protobuf, wow, haven't even heard of that.

This is what a data contract looks like:
```
{
  "sefsefse": {
    "type": "object",
    "properties": {
      "awdawd": {
        "type": "integer",
        "description": "adadwa",
        "minimum": 213,
        "maximum": 12414,
        "$comment": "awdawwa"
      },
      "dgdrg": {
        "type": "array",
        "byteArray": true,
        "minItems": 0,
        "maxItems": 22,
        "contentMediaType": "application/afghu"
      }
    },
    "indices": [
      {
        "name": "awdawd",
        "properties": [
          {
            "awdawd": "asc"
          },
          {
            "dgdrg": "asc"
          }
        ]
      }
    ],
    "required": [
      "awdawd"
    ],
    "additionalProperties": false,
    "$comment": "EGawdawd"
  }
}
```

I'd like to see the same data for all the apps currently deployed to testnet.

Oh sweet jesus, a breakthrough!  
Just fully decoded the dpns data contract :D

I can confirm the hex id given below:  
`E668C659AF66AEE1E72C186DDE7B5B7E0A1D712A09C40D5721F622BF53C53155`  
Is equal to the base58 id returned by the platform api:  
`GWRSAVFMjXx8HpQFaNJMqBV7MBgMK4br5UESsB4S31Ec`

I used the following code to extract the raw data contract for dpns:  
```js
const id = Identifier.from("GWRSAVFMjXx8HpQFaNJMqBV7MBgMK4br5UESsB4S31Ec", "base58")
const response = await client.getDAPIClient().platform.getDataContract(id)
console.log(response.dataContract.toString('hex'))
```

And ran that through `https://cbor.me` with the `cborseq` option enabled.  
Formatting is nicer with cyberchef:  
`https://gchq.github.io/CyberChef/#recipe=Drop_bytes(0,2,false)From_Hex('None')CBOR_Decode()`  
After a bit of manual formatting we're left with the very useful contract data:

```
{
  "$id": h'E668C659AF66AEE1E72C186DDE7B5B7E0A1D712A09C40D5721F622BF53C53155',
  "$schema": "https://schema.dash.org/dpp-0-4-0/meta/data-contract",
  "ownerId": h'3012C19B98EC0033ADDB36CD64B7F510670F2A351A4304B5F6994144286EFDAC',
  "version": 1,
  "documents": {
    "domain": {
      "type": "object",
      "indices": [
        {
          "name": "parentNameAndLabel",
          "unique": true,
          "properties": [
            {
              "normalizedParentDomainName": "asc"
            },
            {
              "normalizedLabel": "asc"
            }
          ]
        },
        {
          "name": "dashIdentityId",
          "unique": true,
          "properties": [
            {
              "records.dashUniqueIdentityId": "asc"
            }
          ]
        },
        {
          "name": "dashAlias",
          "properties": [
            {
              "records.dashAliasIdentityId": "asc"
            }
          ]
        }
      ],
      "$comment": "In order to register a domain you need to create a preorder. The preorder step is needed to prevent man-in-the-middle attacks. normalizedLabel + '.' + normalizedParentDomain must not be longer than 253 chars length as defined by RFC 1035. Domain documents are immutable: modification and deletion are restricted",
      "required": [
        "label",
        "normalizedLabel",
        "normalizedParentDomainName",
        "preorderSalt",
        "records",
        "subdomainRules"
      ],
      "properties": {
        "label": {
          "type": "string",
          "pattern": "^[a-zA-Z0-9][a-zA-Z0-9-]{0,61}[a-zA-Z0-9]$",
          "maxLength": 63,
          "minLength": 3,
          "description": "Domain label. e.g. 'Bob'."
        },
        "records": {
          "type": "object",
          "$comment": "Constraint with max and min properties ensure that only one identity record is used - either a `dashUniqueIdentityId` or a `dashAliasIdentityId`",
          "properties": {
            "dashAliasIdentityId": {
              "type": "array",
              "$comment": "Must be equal to the document owner",
              "maxItems": 32,
              "minItems": 32,
              "byteArray": true,
              "description": "Identity ID to be used to create alias names for the Identity",
              "contentMediaType": "application/x.dash.dpp.identifier"
            },
            "dashUniqueIdentityId": {
              "type": "array",
              "$comment": "Must be equal to the document owner",
              "maxItems": 32,
              "minItems": 32,
              "byteArray": true,
              "description": "Identity ID to be used to create the primary name the Identity",
              "contentMediaType": "application/x.dash.dpp.identifier"
            }
          },
          "maxProperties": 1,
          "minProperties": 1,
          "additionalProperties": false
        },
        "preorderSalt": {
          "type": "array",
          "maxItems": 32,
          "minItems": 32,
          "byteArray": true,
          "description": "Salt used in the preorder document"
        },
        "subdomainRules": {
          "type": "object",
          "required": [
            "allowSubdomains"
          ],
          "properties": {
            "allowSubdomains": {
              "type": "boolean",
              "$comment": "Only the domain owner is allowed to create subdomains for non top-level domains",
              "description": "This option defines who can create subdomains: true - anyone; false - only the domain owner"
            }
          },
          "description": "Subdomain rules allow domain owners to define rules for subdomains",
          "additionalProperties": false
        },
        "normalizedLabel": {
          "type": "string",
          "pattern": "^[a-z0-9][a-z0-9-]{0,61}[a-z0-9]$",
          "$comment": "Must be equal to the label in lowercase. This property will be deprecated due to case insensitive indices",
          "maxLength": 63,
          "description": "Domain label in lowercase for case-insensitive uniqueness validation. e.g. 'bob'"
        },
        "normalizedParentDomainName": {
          "type": "string",
          "pattern": "^$|^[a-z0-9][a-z0-9-\\.]{0,61}[a-z0-9]$",
          "$comment": "Must either be equal to an existing domain or empty to create a top level domain. Only the data contract owner can create top level domains.",
          "maxLength": 63,
          "minLength": 0,
          "description": "A full parent domain name in lowercase for case-insensitive uniqueness validation. e.g. 'dash'"
        }
      },
      "additionalProperties": false
    },
    "preorder": {
      "type": "object",
      "indices": [
        {
          "name": "saltedHash",
          "unique": true,
          "properties": [
            {
              "saltedDomainHash": "asc"
            }
          ]
        }
      ],
      "$comment": "Preorder documents are immutable: modification and deletion are restricted",
      "required": [
        "saltedDomainHash"
      ],
      "properties": {
        "saltedDomainHash": {
          "type": "array",
          "maxItems": 32,
          "minItems": 32,
          "byteArray": true,
          "description": "Double sha-256 of the concatenation of a 32 byte random salt and a normalized domain name"
        }
      },
      "additionalProperties": false
    }
  }
}
```

This is pretty crucial, shame it took me until now to actually see this first
hand. Had to piece it together from tests and diving into the codebase - should
be so much easier!

The above shows how to find out the document names:
- `dpns.domain`
- `dpns.preorder`

To get the registered names for an identity:
```js
const identity: Identity = ...
const dpnsDocs = await this._client.platform.names.resolveByRecord('dashUniqueIdentityId', identity.getId());
dpnsDocs.forEach((doc: any) => {
  console.log(`name: ${doc.toJSON().label}`)
})
```

Dealing with the remaining `require(...svg)` issues...  
Nice that cleaned up a lot of errors.

Now that it's getting a bit further along, it's tring to find the
`dashpayWallet` app. There IS a `dashpay` app, but it's specifically looking
for `dashpayWallet.chat` documents which of course, do not exist. Ash did
mention going for a centralised chat service to start with in order to reduce
the fees to the user.

So there are some loops to watch out for in this codebase:
- `syncChatsLoop`
- `refreshRatesLoop`
- `refreshWalletDataLoop`
- `syncContactRequestsLoop`

This undefined toast is driving me nuts. Apparently the toast message is never
`undefined`. Yet there it is, taunting me.

`startSyncContactRequests` is causing an issue, going to pause that for now.

Cutting down errors like Legolas at the Black Gate.

Was working on the send dash page and now I'm thoroughly done for today.

#### 31.08.2023 Thursday 4h

Right it's kinda stable, just trying to enable as many views as possible now to
get a feel of the functionality and styling. Not sure if it's worth actually
adopting this repo. There has been a lot of work done, but there's a lot of
room for improvements.

My test dash node current balance: 1.94392366  
Going to try and send from the dashameter wallet.  
ySpE22kLLjRoFRFwd1y9oTe5UgTDaymZxo  
Nope, didn't send to the address I entered for some reason.  

Haha if you click the `copy` button next to the dash tx hash, it copies it
literally, including the `...` which is obscuring half the txid - making it
absolutely useless.  
Ah, I've hit incomplete parts of the project, that's why.

`LegacyPaymentItems` is spamming console log.

Ohh the values in `ViewRequestModal` are hard-coded. Lots of `todo`s in there.

Moved the 'redirect when not logged in' logic from the `home` component into the
router, this way you get redirected to the choose account list if you refresh, 
preventing a load of errors from cluttering up the console.


#### 06.09.2023 Wednesday 30m

Adding for admin, made these logs more readable and uploaded to github for
review.

### Batch Two

```
06.11.2023  6h
07.11.2023  3h
13.11.2023  8h
14.11.2023  6h
22.11.2023  4h
23.11.2023  6h
27.11.2023  4h
28.11.2023  3h
29.11.2023  5h
06.02.2024  6h
07.02.2024  1h 20m
08.02.2024  6h
09.02.2024  5h
10.02.2024 10h
11.02.2024  3h 10m
total      76h 30m
```

#### 06.11.2023 Monday 6h

For a quick bit of competator research, what tools do metamask use for their
chrome extension?

cryptocompare.com
etherumjs-util
etherumjs-wallet
sentry
bignumber.js
lodash :)
human-standard-token-abi

node_modules/@chainsafe
node_modules/@ensdomains/content-hash
node_modules/@keystonehq
node_modules/@metamask/address-book-controller
node_modules/@metamask/announcement-controller
node_modules/@metamask/approval-controller
node_modules/@metamask/browser-passworder
node_modules/@metamask/eth-hd-keyring
node_modules/@metamask/eth-json-rpc-infura
node_modules/@metamask/eth-json-rpc-middleware
node_modules/@metamask/eth-json-rpc-provider
node_modules/@metamask/eth-keyring-controller
node_modules/@metamask/eth-ledger-bridge-keyring
node_modules/@metamask/eth-sig-util
node_modules/@metamask/eth-simple-keyring
node_modules/@metamask/eth-snap-keyring
node_modules/@metamask/eth-trezor-keyring
node_modules/@metamask/ethjs-query
node_modules/@metamask/gas-fee-controller
node_modules/@metamask/keyring-api
node_modules/@metamask/keyring-controller
node_modules/@metamask/logging-controller
node_modules/@metamask/message-manager
node_modules/@metamask/network-controller
node_modules/@metamask/notification-controller
node_modules/@metamask/obs-store
node_modules/@metamask/phishing-controller
node_modules/@metamask/ppom-validator
node_modules/@metamask/rate-limit-controller
node_modules/@metamask/selected-network-controller
node_modules/@metamask/signature-controller
node_modules/@metamask/smart-transactions-controller
node_modules/@metamask/swappable-obj-proxy
node_modules/@multiformats/base-x
node_modules/@protobufjs
node_modules/@segment/loosely-validate-event
node_modules/@trezor
node_modules/aes-js
node_modules/await-semaphore
node_modules/base32-encode
node_modules/bip66
node_modules/bitwise
node_modules/borc
node_modules/bytebuffer
node_modules/case
node_modules/cids
node_modules/coinstring
node_modules/component-type
node_modules/debounce-stream
node_modules/debounce
node_modules/duplexer
node_modules/eth-block-tracker
node_modules/eth-eip712-util-browser
node_modules/eth-json-rpc-filters
node_modules/eth-lattice-keyring
node_modules/eth-phishing-detect/src
node_modules/eth-sig-util
node_modules/ethereumjs-wallet
node_modules/fast-json-patch
node_modules/fast-levenshtein
node_modules/gridplus-sdk
node_modules/is-retry-allowed
node_modules/iso-url
node_modules/join-component
node_modules/js-base64
node_modules/json-stable-stringify
node_modules/jsonify
node_modules/lodash
node_modules/long/src
node_modules/multibase
node_modules/multicodec
node_modules/nanoid
node_modules/node-fetch
node_modules/nonce-tracker
node_modules/protobufjs
node_modules/remove-trailing-slash
node_modules/safe-stable-stringify
node_modules/through
node_modules/to-data-view
node_modules/tweetnacl-util
node_modules/tweetnacl
node_modules/uint8arrays
node_modules/web-encoding

https://metamask.github.io/eth-ledger-bridge-keyring/

https://github.com/MetaMask/core

So they are using es modules, but no ui library??
Lodash has a template function, wonder if they're using anything like that?

axios
react
react native

All right that's enough research for my taste.

```
brew link --overwrite node@20
brew upgrade node
```

```
npx create-react-app dashpay-extension --template typescript
```

Actually scratch that I think vite is the way forward.
CRXJS Vite Plugin?
react vs preact
npm vs yarn vs pNpm
rollup vs webpack

turborepo - what is this?
monorepo - seems everyone is going this way

This looks promising:
https://github.com/JohnBra/vite-web-extension

WebExtensions API

```
pnpm create vite dashpay-extension-vite --template react-ts
pnpm run dev

pnpm install tailwindcss postcss autoprefixer
pnpm tailwindcss init -p
```

I don't think I need to use this and can just do the steps, but it's a helpful
reference:
https://vite-plugin-web-extension.aklinker1.io/

vite-plugin-web-extension
webextension-polyfill

```
pnpm install @vitejs/plugin-react-swc
```

https://github.com/Jervis2049/vite-plugin-crx-mv3

```
pnpm i -D vite-plugin-web-extension
pnpm i webextension-polyfill
```

https://github.com/aklinker1/vite-plugin-web-extension/blob/main/packages/create-vite-plugin-web-extension/package.json

Great start. I just used the actual project creator:

```
pnpm create vite-plugin-web-extension
```

After looking at the things they did, I liked what I saw.  
But. (and this isn't a nice juicy butt)  
- `vite dev` opens a blank browser window
- `localhost:whateverport` doesn't seem to run either
- you CAN `vite build`, but the hot reload and even backup file watch mode doesn't cause a rebuild

Sooo yeah atm it's not great.

http://localhost:5173

createWebExtRunner in vite-plugin-web-extension
BROKE

web-ext
pnpm run web-ext run --source-dir ./dist
pnpm -r exec web-ext run --source-dir ./dist

pnpm install web-ext
pnpm exec web-ext run --source-dir ./dist/

Wondering if I can just get it going in firefox and then build for chrome...?
Firefox gives an error: background.service_worker is currently disabled

TARGET=chrome pnpm vite build --watch
TARGET=firefox pnpm vite build

pnpm exec web-ext run -t chromium --start-url https://example.com --source-dir ./dist/

Oh so it IS working with `pnpm run dev`, it's just the extension is installed
and hidden in the extension window. You have to click through to it.

After a system restart, file watching is now working again.

So I have a fully working setup with tailwind and the speedy boi rust compiler
and can open the extension and see changes updated instantly.

Cooking with gas 🍳🍳🍳


#### 07.11.2023 Tuesday 3h

Can I have a 360x640 window automatically displayed to work on?

There is a start-url flag:
`web-ext run --start-url www.mozilla.com`

`markdown viewer` extension, for example, has it's own page at:
`chrome-extension://ckkdlimhmcjmikdlpkmbgfkaikojcbjk/options/index.html`
`chrome-extension://pehllhcpmlhimdapoaheimbljjcdmndk/test.html`

Can we not have the same kind of thing?

chrome.runtime.id                        --> "firefox@ghostery.com"
chrome.i18n.getMessage("@@extension_id") --> "e3225586-81a0-47c3-8612-d95fb0c2a609"
origin: chrome-extension://mlomiejdfkolichcflejclcbmpeaniij
https://developer.chrome.com/docs/extensions/mv3/manifest/web_accessible_resources/

I don't think we want a `web accessible resource`, unless we want sites to be
able to fingerprint the extension and detect if it is installed. Maybe we do want
this, but it doesn't fit what I'm trying to do right now.

This WILL load the test html but I'd like to show that automatically if poss.
`chrome-extension://pehllhcpmlhimdapoaheimbljjcdmndk/src/wrapper.html`

I think extensions are limited to 600px wide, so I'll have to tweak the designs
a bit.

https://vite-plugin-web-extension.aklinker1.io/


chrome-extension://pehllhcpmlhimdapoaheimbljjcdmndk/src/wrapper.html

Added to vite config, now it loads but chrome blocks it until the user hits
refresh. Weird.

It's better to redirect after installation in the `onInstalled` callback.
Right off to gym, tbc...

#### 13.11.2023 Monday 8h

360x600

Was wondering why the `Inter` font-face wasn't loading all the variable,
iterations and instead just grouping as 2, 100-500 and 600+. You have to add the
new `tech('variations')` css function after the `url` in the `@font-face` block.

That's a new one.

Really important not to forget the tailwind JIT feature:
https://v2.tailwindcss.com/docs/just-in-time-mode

Should be able to do nearly anything we need with just classes.  

Trying to lean heavily into this approach to get used to it and give it a proper
try...

Had a weird bug where exporting a named default function as a component didn't
return anything. Removing the function name `Welcome`/`WelcomePage` and just
returning an anonymous function fixed it. Weird.

Fiddled about with the was svgs are imported. Cleanest way is to use the
`vite-plugin-svgr` package which auto-generates a react component with the
ability to change the svg stroke/fill by html property.

Finished the welcome screen and almost the username select screen. Was making
the validation responses. Haven't quite hit my stride but this is the first day
properly back and reaquainting with tailwind, no doubt things will pick up
dramatically this week...

Will someone for the love of god explain the differences here:
- `flex-start`
- `content-start`
- `items-start`
- `justify-start`

Got the dialer done, refactored the text styles a bit, going to keep refactoring
down as I go to keep things as clean as possible.

WTF, my wrapper.html just completely disappeared. No git history, no local file
history, the only trace is the reference in the manifest.json

Fuck now HMR is completely broken.

pnpm exec web-ext run -t chromium --start-url chrome-extension://pehllhcpmlhimdapoaheimbljjcdmndk/index.html --source-dir ./dist/

#### 14.11.2023 Tuesday 6h

Okay so everything kind of fell apart yesterday and HMR stopped working. I
really don't want to waste too much time on this when we can just use the
standard server.

I'm on the latest release of vite, `4.5.0`
Some people suggest the vite 5 beta.

Add now this throws an error:
```
rm -rf ./node_modules/.vite
pnpm vite build --watch
```
Could not resolve entry module "index.html".

```
pnpm create vite dashpay-extension-vite --template react-ts
pnpm create vite-plugin-web-extension
```

Okay created a test project to compare the difference with mine to see what I
might have broken, nothing is obvious. I moved some of the vite plugins around, 
maybe that order fixed it, I deleted the `./node_modules/.vite` dir too, maybe
that helped - who knows.

Tailwind uses `background-color` rather than `background` so I can't use the
linear gradient variable. Instead I have to use the tailwind gradient background 
classes with potentially a custom colour. I'll come back to this...

- [ ] Gradient buttons with proper disabled style colouring

For future reference you can manually set 94% zoom which makes it almost 1:1
with my chrome window.

Fleshed out the home page
Created a `Pill` component for sent/requested/received in small/large variants.
Realised the fonts weren't being loaded properly, they are now.

#### 22.11.2023 Wednesday 4h

Accounts View
Gradient Profile Icon
Gradient Button
Standardised `Text` component to EXACTLY the same names as figma design :)
Home Overlay Buttons
Restore View
Modal Component + Unlock Account Variety

```
text-1: 16px / 1rem, 400w, 1.25rem line height
text-2: 14px / 0.875, 500w, normal line height
text-3: 14px / 0.875rem, 400w, normal line height
text-4: 12px / 0.75rem, 500w, n lh
title-1: 28px / 1.75rem, 600w, n lh
title-4: 16px / 1rem, 600w, normal line height
title-6: 0.9375rem, 400w
title-7: 14px, 600w
```

- remove `Subtitle` for `Text type=title-4`
- replace span/p with Text

Very solid day work even if it was only 4h.

Remaining views to style:
- [ ] Search Results
- [ ] Friend Profile
- [ ] Chat
- [ ] Settings
- [ ] Transactions
- [ ] Contacts

Other main components:
- [ ] Send/Request/Accept Modal
- [ ] Currency Modal
- [ ] Context Menu
- [ ] All the little chat features

#### 23.11.2023 Thursday 6h

- [x] Friend Profile
- [x] Settings
- [-] Chat
  - [x] Welcome / Empty Messages
  - [x] Dropdown / Context Menu
  - [x] Send to friends only warning
  - [x] Chat Bubbles
  - [x] Confirm Friend Buttons

The icon list / search results styling I just did for the friend profile is
an improvement on the accounts list - I managed to get the spacer to be sized
according to the content rather than with % width, so I'll refactor the accounts
list.

The grabber makes it tricky. I'll just leave it for now and try and remember to
prioritise copying the friend profile over the accounts list.

Probably should just use ion icons.

```
pnpm i react-ionicons
```

See the list here: https://ionic.io/ionicons  
Going to replace the svgs I added with those where appropriate.  

Getting this annoying error:
> React refresh runtime was loaded twice. Maybe you forgot the base path?

When using useEffect()

Just refactored the chat context menu into it's own component, then created the
transactions view outline, then the timer went. Boom.

Solid 6h

#### 27.11.2023 Monday 4h

"React refresh runtime was loaded twice" vite
vite react cannot export multiple components
vite cant have two components per file

https://github.com/vitejs/vite-plugin-react/issues/34  
https://github.com/vitejs/vite-plugin-react-swc#consistent-components-exports  
https://www.gatsbyjs.com/docs/reference/local-development/fast-refresh/#how-it-works  

So having two exports in a tsx file breaks HMR UNLESS every export is a react
component. I was using `export default function(props: any)` to declare one
component, and `export const Another = (<>...</>)` to export the other. The
latter broke HMR as it's not a react component

- [x] List Component
- [x] Transaction List Component
- [x] Transactions View

#### 28.11.2023 Tuesday 3h

- [x] Search Input Styling
- [x] Add custom colors to tailwind config
- [x] Profile Completion Banner
- [x] List component rework
- [x] Contacts View
- [x] Mock data store

#### 29.11.2023 Wednesday 5h

- [x] Password input type
- [x] Wiring up accounts view with store
- [x] Logout / Login mock
- [x] Find a way to access history with react router v6 (you cant lol, custom approach needed)
- [x] Menu component
- [x] Home screen dropdown menu

Argh I'm back to the tsx hot module issue again. I want to use a `tsx` file to
export a hook, but can't do that as it's not a react component.

I just exported it as a weird anonymous function hook.

- [x] Context menu hook

The done list isn't exactly all encompassing, I've been doing a lot of wiring up
routes correctly rather than just going to a debug menu.

- [x] Edit profile view
- [x] Label on inputs
- [x] My Address / QR View

#### 06.02.2024 Tuesday 6h

Right, aiming to have this done in the next few weeks.
Let's gooooo.

Going to get every single bit of visible UI in place that's in the design, then
switch over to the logic to wire it all together.

- [x] Refactor store methods to be async - [x] Refactor profile circle to accept profileColor and profileImage hashcode/url
- [x] Transactions filter menu style
- [x] Legacy Transactions View
- [x] Home Search Style
- [x] Refactor sent/request svg usage to react icon import
- [x] Swipe Left Style (request/send)
- [x] Swipe Reply Style
- [x] Event message type style
- [x] Request Declined Chatbox
- [x] Reply Quote Style (message controls)
- [x] Long click message (buttons + text blur styles)
- [x] Copied message popover
- [x] In-message reply quote style

Powered through the ui there.

#### 07.02.2024 Wednesday 1h 20m

- [ ] Send/Request Dash Modal

#### 08.02.2024 Thursday 6h

- [x] Send/Request Dash Modal (cont.d)
  - [x] Amount Display
  - [x] Dialer / Keypad (refactored into a component from the pin entry view)
  - [x] Enter message step
- [ ] Chat View UI
  - [x] "You sent" info message
  - [x] Amount large pill
  - [x] Requested with controls (decline/view)
  - [x] Scroll to new messages
  - [x] Accepted Request
  - [x] View Pinned requests button
  - [x] Processing transaction loading icon (loading component)

#### 09.02.2024 Friday 5h

- [ ] Chat View UI
  - [x] Spinner Indicator
  - [x] No internet connection error
  - [x] Failed to send message (message error style)
  - [x] Send again / delete controls
- [ ] Send/Receive Modal
  - [x] Error Message
  - [ ] Request variant (alternative colours)
  - [x] Currency Select Modal

Need to convert the currency svgs from android vector back to svg.

```
find . -name '*.xml' -exec bash -c "echo \$0 to \${0/xml/svg}" {} \;
# yep that works
find . -name '*.xml' -exec bash -c "npx vector-drawable-svg \$0 \${0/xml/svg}" {} \;
```

Well that's doing them all in parallel due to `bash -c` spawning a new thread.
Might not be the fastest approach but it's going.

Got a situation here where I'm setting max-width on a text element and when it
wraps, the space between the last word on the first line and the edge of the
bounding box includes all the space. This means the elem to the right is floating
way out when it should be next to the last word on the first line.

That was fiddly. No doubt there'll be more things to change later but so far so
good.


#### 10.02.2024 Saturday 10h

- [ ] Send/Receive
  - [x] Transaction Info
  - [x] You received/set (past tense)
  - [x] Accept Request style
  - [x] Your Request style
  - [x] "accept request" controls
  - [x] "your request" controls
- [x] Friend request modal

Having some issues with my ContactList component stretching beyond it's bounds.
Also the scrollbar finishes like 20px above the end??
It was due to the way I'd done the spacer with relative/absolute.

- [ ] Legacy Payments
  - [x] Internal transfer alert
  - [x] Identity topup
  - [x] Receive/send buttons
  
Noticed the shadow colour on the chat amount pills.
You can colour shadows in tailwind but not the way I've defined the vars.
Refactoring those...

Now I've broken the usage of `--var()`
DashIcon
RecoverWalelt

- [x] Refactor colours to the main 3 (blue/green/red)

Just had another revelation regarding the colors.
The bgs are just the main colours at 20% opacity, and being able to choose that
is baked into tailwind. Another refactoring opportunity to really clean things
up there. Will also make it much easier to do lighten/darken effects.

- [x] empty legacy chat
- [x] legacy send
- [x] legacy receive
- [x] `const {...} = props` can actually be destructured within the function signature!

Massive refactoring to always destructure in the function declaration because
it provides autocomplete, a healthy default which also serves as an example, and
type safety (if specified)

- [x] receive qr
- [x] receive amount
- [x] keypad logic refactor DRY 3 modes

Trying to keep the dialer logic in one place, so need to establish all the use
cases:
- pin: allow leading 0s, no decimal place (used for pin entry, obviously)
- integer: no leading 0s, no decimal place (not needed)
- decimal: no loading 0s allow decimal place (amount entry)

#### 11.02.2024 Sunday 3h 10m

- [x] linked amount display/keypad
- [x] receive modal
- [x] search chat messages
- [x] show qr modal (move to component and use in home/chat)
- [x] Address Contraction with Copy into component (My QR Code + Send/Request)

### Batch Three

```
13.02.2024   6h
14.02.2024   3h 40m
15.02.2024   2h
26.02.2024   6h
27.02.2024   6h
04.03.2024   6h
05.03.2024   3h
06.03.2024   6h
08.03.2024   6h
11.03.2024   6h
Total       50h 40m
```

#### 13.02.2024 Tuesday 6h

- [x] confirm payment with a pin modal
- [x] confirm payment with a fingerprint modal
- [x] Swipe left to Ignore/Accept

That's it! That's all the design components except the redeem modal and the scan
view, the former which is quite heavily subject to change and the latter which
requires accessing the camera so I'll just do that when I come to it.

Working on the seed phrase entry first.

```
pnpm install --save dash
```

https://www.npmjs.com/package/dash
This links to the old github repo.
The old repo then redirects to dashevo/platform.
dashevo/platform is a monorepo which contains js-dash-sdk.
Irritatingly, the package.json still links back to the old url.

I'm going to uninstall that and use the full github urls for transparency:

```
pnpm i --save ssh://github.com/dashpay/platform/tree/master/packages/js-dash-sdk
pnpm i --save ssh://github.com/dashpay/platform/tree/master/packages/wallet-lib
pnpm i --save ssh://gitahub.com/dashpay/dashcore-lib
pnpm i --save ssh://github.com/dashpay/platform/tree/master/packages/js-dapi-client
pnpm i --save ssh://github.com/dashpay/platform/tree/master/packages/wasm-dpp
```

>  ERR_PNPM_TARBALL_EXTRACT  Failed to unpack the tarball from "https://github.com/dashpay/platform/tree/master/packages/js-dapi-client": Error: Invalid checksum for TAR header at offset 0. Expected NaN, got 43081

https://github.com/dashpay/platform/tree/master/packages/wallet-lib

```
pnpm i --save @dashevo/wallet-lib
```

Getting a `process` is not defined error when including that lib.
So it's not safe for browsers.

`vite-plugin-node-polyfills`

`@esbuild-plugins/node-globals-polyfill`
`@esbuild-plugins/node-modules-polyfill`
`npmjs.com/package/rollup-plugin-node-polyfills`


mnemonic comes from `@dashevo/dashcore-lib`
`@dashevo/dashcore-lib/lib/mnemonic/words/` is the directory
These are the supported languages:
- chinese
- english
- french
- italian
- japanese
- spanish

Trying to have Webstorm load the `typings` from `dashcore-lib` properly.

Oh they are loading, it's just the `dashcore-lib` package doesn't expose the
`Mnemonic.Words` static object. Damn. Fix or override?

I'll fix it...


#### 14.02.2024 Wednesday 3h 40m

Changed my mind about fixing it.
It works, it just displays an IDE warning. I can live with that.

- allow horizontal scroll of suggestions left/right without showing scrollbars
- sort suggestions according to closest letter combinations, then alphabetically
- allow next when entered word matches first suggestion
- OR if there's only one suggestion left
- dont suggest the same word twice
- backspace to skip back to the previous input
- force uppercase first letter
- scroll horizontally without stretching

Not sure we actually need the invalid styling. We can just only enable the
button when the input is a valid word.

Need to decide how to do a global state reload.
Looking into redux...
I always find it FAR too overengineered. I just need a simple node EventEmitter
style solution which also provides a way to unregister a listener.
Implemented a 10 line `on(...)/displatch(...)` pubsub style thingy within the
`Store` that should do the trick.

Do dash usernames allow for chinese characters?
Nope, there's a regex in the docs, nice.

https://docs.dash.org/projects/platform/en/stable/docs/explanations/dpns.html
`^[a-zA-Z0-9][a-zA-Z0-9-]{0,61}[a-zA-Z0-9]$`

Copying the debounce function I created when making the dashameter project work
over into this redo.
`dashameter-dashpay-wallet`

Okay got the username input debounce working.
Debounce in react is a bit fiddly.
It will:
- wait for the username to be valid and input to pause for at least 1s
- send the dpns request only then
- if the username is changed while waiting for a response, it will go back into a 'default' validation state
- then only the latest request will actually cause ui changes (response username matches the current state)


#### 15.02.2024 Thursday 2h

Fixed an issue where the extension was not loading correctly on other machines.
Needed to remove the hardcoded extension id as well as some dependency updates
which brought the chrome manifest from v2 to v3.

Back to making the UI interactive.

- [x] disable username button until all validation passes
- [x] disabled backup button until slider checked
- [x] pin confirmation step
- [x] height of username validation is "twitching"
- [x] settings > change pin flow
- [x] settings > show qr


#### 26.02.2024 Monday 6h

- [ ] bugfix: currency modal input de-focusing on change

This is quite an annoying bug.  
Could be to do with the way I've coded the `useModal` hook.
The parent is re-rendering the entire modal on change.  
Because of setState I assume.  
So... use the child setState method?  

No I think `Modal` just needs to actually be a component rather than a hook that
returns a component, although that can't work for things like the linked keypad,
but that doesn't need to retain input focus.

> Do not call Hooks inside useEffect(...), useMemo(...), or other built-in Hooks.

FUN

ok got it. only took me an hour and a half. dog gamn.
it means I have to use the modals in a slightly different and less appealing way,
but it works.

Here's a demo of the problem / solution:

```
// useModal.tsx

export default function() {
  return ({
    Component: (props) => <Input {...props}/>,
  })
}

...


const testModal = useModal()

...

control:
<Input value={test} onChange={e => setTest(e.target.value)}></Input>

test (breaks)
<testModal.Component value={test} onChange={e => setTest(e.target.value)}/>

test (works)
{testModal.Component({value: test, onChange: e => setTest(e.target.value)})}
```

So the solution is a one-liner but which doesn't really make sense.

- [x] settings > change currency
- [x] settings > change language
- [x] hide language in settings (for now)
- [x] hide touch ID in settings (for now)
- [x] hide notification settings (for now)
- [x] change display name / status
- [x] placeholder help and feedback/contact
- [x] hide balance on home screen
- [x] hide fingerprint page (for now)
- [x] bugfix: username suggestions are multiline (`whitespace-nowrap`)
- [x] move username check to store and randomly make available with suggestions
- [x] allow click suggestion
- [x] translate language options into their own language for display
- [x] generate random seed phrase on backup view
- [x] import dash client refactoring I did before

Okay so I need to do the logic for creating accounts, and having settings stored
against those accounts.  
What's the best way to go about that?  
While keeping the local storage as resistant to unauthorized viewing?  

Also, should settings just be part of the store?  
Seeing as the store has logIn/logOut that would make sense.  
Ok I'll combine them.  

Oh. Mnemonic invalid, of course.  
It's not a valid private key because I just randomly picked words. 

- [x] bugfix: using offline wallet mnemonic from dash lib now as 
- [x] bugfix: prevent another mnemonic from being generated when clicking back
- [x] bugfix: prevent multiple dash client loads

At the home screen, we need to create a warning modal which explains the identity
creation step to reserve their username requires funds, and that it cannot be
reserved. They need to top up their account to complete.

- [x] Home screen "account registration" modal

#### 27.02.2024 Tuesday 6h

Now that I've included the dash libraries. The reload takes 4s.
That's pretty brutal. It was <200ms before.

```
dash.min.js   392ms
wallet-lib    166ms
dashcore-lib  190ms
ionicons       67ms
```

That's not super fun.  
Something like Postman probably takes that long to load, which is fine for the
first load, but that's going to really slow me down in development over hundreds
of thousands of live-reloads.

Soooooo, time to rip out the dash libraries until I've mocked EVERYTHING.

Struggling to mock this a bit:
```
const dpnsDocs = await this._client.platform.names.resolveByRecord('dashUniqueIdentityId', identityId)
```

Following it down:
platform has: `documents: Records`, `names: DomainNames`, `identities: Identities`
The `DomainNames` interface has a method called `resolveByRecord` which returns a `Function`.
That's where the intellisense trail runs cold, as that could return anything and isn't linked to the implementation.
You have to dig that out by finding the file: `Platform/methods/names/resolveByRecord.ts`.
That in turn calls `documents.get('dpns.domain', query)` where the `query` filters down to `where records.${record} == ${value}` which also returns a `Function`.
Going all the way back up the stack to the current project and the usage: `.resolveByRecord('dashUniqueIdentityId', identityId)`
means that will search `where records.dashUniqueIdentityId == ${identityId}`.
So the `DomainNames` is a wrapper around the `documents`/`Records`.

Just thinking about the simplest way to use all this.
Umm. I need a recap.
I think that's getting the identity, which is a record within the dpns.domain contract.
The record schema would be defined in the contract, along with the validation rules.

Keywords:
`Contracts`
`Records`
`Documents`
`DomainNames`
`Identities`
`DataContracts`

I think documents and records are used interchangeably, but the interface is Record so I'll use that.

`.getDPNSNameRecordsByIdentityId(id)` done?  
`.getDPNSNamesForIdentity(id)` better?  

- [x] Rip hack and slash

Okay from 4s to 200ms, that's going to speed up dev a ton.

So I'm seeing some distinct storage/data options here:
- settings: pin/language/currency etc.
- store: account -> settings, commit to local storage
- dash: platform/txs/accounts/etc

Maybe I'll keep store as the facade for everything data layer.  
All the UI calls `store` methods and subscribes to the store.  
Then there's just a clearly defined path to debug everything else.  

It might mean store gets a bit out of hand, but if we just extract when needed
into component classes I'm sure it'll work out.

- [x] Remember last page and reload to there

Got this issue where going username always takes you back to welcome, which is
jarring when coming from accounts. Looks like I'm going to have to figure out
some decent history with this router despite it loading multiple times and being
a bit crazy with redirects.

History doesn't quite cut it, because you could reload on say the 'recover' page,
then go back to the accounts page, then back would take you forward in flow -
back to the recover page. Added a state variable to back buttons which causes
the history to always "pop".

- [x] Implement reliable "back" (I think...)
- [x] Remove placeholder message from home
- [ ] Load/save accounts to/from local storage
- [ ] Bugfix: Prevent title selection on back click (needed to switch div to button)

Need to think about the localstorage format now so that reloading will seamlessly
continue from where we left off.

I've created a little dash test project to nail down some of the responses I need
to mock, and come accross an issue trying to do what I've already done in the
vite project:

`import {Client} from 'dash'`

I thought the compiler option `esModuleInterop` would fix that.

> Unknown file extension ".ts"

ts-node-esm ./src/main.ts
ts-node ./src/main.ts
pnpm exec ts-node --esm ./src/main.ts
node --loader ts-node/esm ./src/main.ts
ts-node --esm main.ts
pnpm exec ts-node-esm ./main.ts

node --loader ts-node/esm ./src/main.ts

Cray Z

#### 04.03.2024 Monday 6h

Right so switching back to `main` and now it's not building or launching chrome.  
`pnpm build` is failing.  

Had to add the typescript reference:
```
/// <reference types="vite-plugin-svgr/client" />
```

Next infuriating error:

> undefined:1
> <!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01//EN""http://www.w3.org/TR/html4/strict.dtd">

Coming from `.pnpm/vite-plugin-web-extension/dist/index.js`

Build took 30s!  
It says `built in 50ms` then hangs for 30s. wtf.  
Next time, alternates back to the strict.dtd error above.  
Next time, works fine but hangs for 30s again.  
Yeah definitely alternating between success and failure there.  

`pnpm update`

I know the platform names (dpns) are priced, but can you register a dash identity for free?

At what point do I save an account to the local cache?  
- When the username is chosen. (create path)
- When the mnemonic is provided. (restore path)

Maybe we could use a two part id for loading/restoring users.  
Search users by username first, then the md5 sum of the mnemonic.  
As they already need to know the mnemonic to derive the cached accounts from it
the faster less secure md5 algo. is fine.

- [x] Notification Logic

Notifications can be chained
Duration can be defined, 0 can hold it indefinitely and it returns a callback to close
Position can be top/bottom
Child content can be passed or simply text

There's one issue I'm having in that I'd like `const notifications = useNotifications()`
with `{notifications.Component()}` and `notifications.show({...})` but of course
the second invocation of `useNotifications` is creating a whole second state.

Switched to `useContext`/`createContext` to solve that.


Switched settings from interface to class, perhaps I can pass in the cache and
have setters which automatically write to localStorage...

tbc...

#### 05.03.2024 Tuesday 3h

Okay now I need to think about how the account balance updates happen.

Probably a good time to switch back to the test project with nothing but the
dash wallet and see...

- [x] connect to the testnet
- [x] get new address

> Error: socket hang up\n
> MaxRetriesReachedError: Max retries reached: socket hang up at connResetException (node:internal/errors:787:14)
> /code/dash-platform-testing/node_modules/.pnpm/@dashevo+dapi-client@0.25.22/node_modules/@dashevo/dapi-client/lib/transport/GrpcTransport/GrpcTransport.js:104

node --loader ts-node/esm src/main.ts
node --trace-warnings --loader ts-node/esm src/main.ts
NODE_DEBUG=cluster,net,http,fs,tls,module,timers node --stack_trace_limit=200 --no-warnings --loader ts-node/esm src/main.ts

hmm weird error:

```
node:internal/process/esm_loader:34
      internalBinding('errors').triggerUncaughtException(
                                ^
[Object: null prototype] {
  [Symbol(nodejs.util.inspect.custom)]: [Function: [nodejs.util.inspect.custom]]
}
```

latest node: 21.6.2
latest ts-node: 10.9.2

damn, up-to-date.

what now?

node --loader ts-node/esm src/main.mts
ts-node src/main.mts

> Error [ERR_REQUIRE_ESM]: Must use import to load ES Module: /Users/adc/code/dash-platform-testing/src/main.mts

What es module?

ok if I put `module: true` in the package I now get:

> TypeError: Unknown file extension ".mts" for /Users/adc/code/dash-platform-testing/src/main.mts

```
node --trace-warnings --loader ts-node/esm src/main.mts
node --watch --loader ts-node/esm src/main.mts
```

This actually seems like the most reliable way:
`npx tsc`

Oh my shit! Adding this to `ts-node` just works:

```
  "ts-node": {
    "experimentalSpecifierResolution": "node",
    "transpileOnly": true,
    "esm": true,
  }
```

`node --watch --loader ts-node/esm src/main.mts`

Keep getting socket hang up for:
54.213.204.85:1443

> // Since we don't have PoSe atm, 3rd party masternodes sometimes provide wrong data
> // that breaks test suite and application logic. Temporary solution is to hardcode
> // reliable DCG testnet masternodes to connect. Should be removed when PoSe is introduced.

Yeah that's the last address on the list with that comment.

`@dashevo/dapi-client/lib/networkConfigs.js:14`

Maybe I can just run my own testnet masternode and expose that port?

Spent most of the day at the health center so haven't run up much time on the
clock today.

nc -zv -w1 -G1 34.214.48.68 1443;
nc -zv -w1 -G1 35.166.18.166 1443;
nc -zv -w1 -G1 35.165.50.126 1443;
nc -zv -w1 -G1 52.42.202.128 1443;
nc -zv -w1 -G1 52.12.176.90 1443;
nc -zv -w1 -G1 44.233.44.95 1443;
nc -zv -w1 -G1 35.167.145.149 1443;
nc -zv -w1 -G1 52.34.144.50 1443;
nc -zv -w1 -G1 44.240.98.102 1443;
nc -zv -w1 -G1 54.201.32.131 1443;
nc -zv -w1 -G1 52.10.229.11 1443;
nc -zv -w1 -G1 52.13.132.146 1443;
nc -zv -w1 -G1 44.228.242.181 1443;
nc -zv -w1 -G1 35.82.197.197 1443;
nc -zv -w1 -G1 52.40.219.41 1443;
nc -zv -w1 -G1 44.239.39.153 1443;
nc -zv -w1 -G1 54.149.33.167 1443;
nc -zv -w1 -G1 35.164.23.245 1443;
nc -zv -w1 -G1 52.33.28.47 1443;
nc -zv -w1 -G1 52.43.86.231 1443;
nc -zv -w1 -G1 52.43.13.92 1443;
nc -zv -w1 -G1 35.163.144.230 1443;
nc -zv -w1 -G1 52.89.154.48 1443;
nc -zv -w1 -G1 52.24.124.162 1443;
nc -zv -w1 -G1 44.227.137.77 1443;
nc -zv -w1 -G1 35.85.21.179 1443;
nc -zv -w1 -G1 54.187.14.232 1443;
nc -zv -w1 -G1 54.68.235.201 1443;
nc -zv -w1 -G1 52.13.250.182 1443;
nc -zv -w1 -G1 35.82.49.196 1443;
nc -zv -w1 -G1 44.232.196.6 1443;
nc -zv -w1 -G1 54.189.164.39 1443;
nc -zv -w1 -G1 54.213.204.85 1443;

All succeeded. Hmmmmmmmm.

Well that time everything worked perfectly.

```
node --no-warnings --loader ts-node/esm src/main.mts
```

- dash@3.25.22_typanion@3.14.0/node_modules/dash/docs/examples
- dash@3.25.22_typanion@3.14.0/node_modules/@dashevo/dapi-client/docs/usage
- dash@3.25.22_typanion@3.14.0/node_modules/@dashevo/dashcore-lib/docs/examples.md
- dash@3.25.22_typanion@3.14.0/node_modules/@dashevo/wallet-lib/examples
- dash@3.25.22_typanion@3.14.0/node_modules/@dashevo/dashpay-contract/test/unit/schema.spec.js

```
account.on('FETCHED/UNCONFIRMED_TRANSACTION', (data) => {
  console.dir(data);
});
```

#### 06.03.2024 Wednesday 6h

There must be a way to update the displayname/status of a dpns name doc.

10s

How do you access a property of a doc?
Parsing it to json shows a load of fields that you cant access directly??

```
const a = {somefield: 'here', toJSON: () => ({})}
JSON.stringify(a)
```

> '{}'

That's interesting, so each doc is overriding the way it's serialised to json.

So we just call `toObject` or `toJSON` and then we can access the properties.

They behave a little differently.

`toJSON`

```
{
  '$id': '63KGufhsPK668CBFhrT6CJeTSsinqmq6a3NiJ3gWvMLn',
  '$ownerId': 'HPacb4diz5TBxbi61P9vbYurkz98sQ41wZ351YNU4yxm',
  label: 'Alex',
  normalizedLabel: 'a1ex',
  normalizedParentDomainName: 'dash',
  parentDomainName: 'dash',
  preorderSalt: 'Tvi5TaHU863G/G7K9LgqXiaUzD2vADwWchsIULCvXxk=',
  records: {
    dashUniqueIdentityId: '84PGjmiNFwShZVc6LOIDkExMn7A2wBhaWU8U//HDK4o='
  },
  subdomainRules: { allowSubdomains: false },
  '$revision': 1,
  '$createdAt': null,
  '$updatedAt': null,
  '$dataContract': {
    '$format_version': '0',
    id: 'GWRSAVFMjXx8HpQFaNJMqBV7MBgMK4br5UESsB4S31Ec',
    config: {
      '$format_version': '0',
      canBeDeleted: false,
      readonly: false,
      keepsHistory: false,
      documentsKeepHistoryContractDefault: false,
      documentsMutableContractDefault: true,
      requiresIdentityEncryptionBoundedKey: null,
      requiresIdentityDecryptionBoundedKey: null
    },
    version: 1,
    ownerId: '4EfA9Jrvv3nnCFdSf7fad59851iiTRZ6Wcu6YVJ4iSeF',
    schemaDefs: null,
    documentSchemas: { domain: [Object], preorder: [Object] }
  },
  '$type': 'domain'
}
```

`toObject`

```
{
  '$createdAt': null,
  '$dataContractId': <Buffer e6 68 c6 59 af 66 ae e1 e7 2c 18 6d de 7b 5b 7e 0a 1d 71 2a 09 c4 0d 57 21 f6 22 bf 53 c5 31 55>,
  '$id': <Buffer 4a e2 42 50 7c 6d 79 ff 84 ae 0f f3 8a fe cb d3 91 f9 b8 7c 31 91 b3 e4 64 b8 30 e2 4f 8f c5 c3>,
  '$ownerId': <Buffer f3 83 c6 8e 68 8d 17 04 a1 65 57 3a 2c e2 03 90 4c 4c 9f b0 36 c0 18 5a 59 4f 14 ff f1 c3 2b 8a>,
  '$revision': 1,
  '$type': 'domain',
  '$updatedAt': null,
  label: 'Alex',
  normalizedLabel: 'a1ex',
  normalizedParentDomainName: 'dash',
  parentDomainName: 'dash',
  preorderSalt: <Buffer 4e f8 b9 4d a1 d4 f3 ad c6 fc 6e ca f4 b8 2a 5e 26 94 cc 3d af 00 3c 16 72 1b 08 50 b0 af 5f 19>,
  records: {
    dashUniqueIdentityId: <Buffer f3 83 c6 8e 68 8d 17 04 a1 65 57 3a 2c e2 03 90 4c 4c 9f b0 36 c0 18 5a 59 4f 14 ff f1 c3 2b 8a>
  },
  subdomainRules: { allowSubdomains: false }
}
```

I wonder if it'd be better to iterate all the name documents (if that's possible)
and cache it.

- [x] search/check dpns name


tsc --project tsconfig.json
tsc --build tsconfig.json --verbose
tsc --build tsconfig.json --verbose && node build/main.mjs
tsc --build tsconfig.json --verbose && node build/main.mjs

> ERR_UNSUPPORTED_DIR_IMPORT]: Directory import
> 'node_modules/@dashevo/platform/packages/wallet-lib'
> is not supported resolving ES modules imported from
> build/main.mjs

node --import ./ts-loader.js src/main.mts

That runs the same as my `pnpm start` script. `Cannot find module`.

Invalid "imports" target defined for '#dashevo/wallet-lib' in the package config

/Users/adc/code/dash-platform-testing/node_modules/@dashevo/platform/package.json

Right so I've learned a few things.
1. The `wallet-lib` is a `commonjs` module with typescript definitions tacked on.
2. Modifying the compiled js from:
  `import { EVENTS } from "@dashevo/platform/packages/wallet-lib";`
  to:
  `import { EVENTS } from "@dashevo/platform/packages/wallet-lib/src/index.js";`
  works as expected.

Now I'd like to import index.js without having to specify that.
It's already set in `package.json > {"main": "src/index.js"}`

I was messing with the sub-module "exports"/"imports" stuff in my package json
to see if I could alias wallet-lib.

```
pnpm add @dashevo/wallet-lib@./node_modules/@dashevo/platform/packages/wallet-lib
```

FUCK YES. That's even better than I was hoping for. So we can use the
`@dashevo/platform` monorepo and just alias/link all the sub packages using that.

Will it work with typescript though?
YESSS.

Okay now to fix the `dash` package as well. They're both commonjs modules with
typings, so this should be possible.

When it comes to importing commonjs modules into es modules:
- Default-importing from CommonJS modules always works.
- Name-importing only works sometimes.

```
export = SDK;
```

I think this is the culprit.

If they'd put:

```
export default SDK;
```

```
module.exports = SDK_1.default;
```

I think it'd work fine.

Hmmm but I could swear I had this working before.

`rm -rf build && tsc --build tsconfig.json --verbose && node build/main.mjs`

Ahh I think it's because it's working fine after going through the webpack flow,
but when building for node it doesn't go through the same compilations steps
and the exports are not as clearly defined. Oh well I'll just use the commonjs
default import style for `dash`.

wow now getting a ton of this:

> Block height changed handler is already set.

after all those warnings it blew up with:

> InvalidRequestError: invalid transaction: bad-txns-inputs-missingorspent

Wowzers.

I'm in a mind to just get all of the platform stuff done now.

There's 0 visibility into what the `dash:Client` is doing and it hangs for
several minutes. If it's catching up with block data - fine, there's just no way
of seeing that at the moment until the wallet is returned because the underlying
dapi is protected. That's annoying and everything, but there's more - it often
throws exceptions and just completely craps out. I just had 2 within a minute 
there.

> platform deserialization error: unable to deserialize ConsensusError

> MaxRetriesReachedError: Max retries reached: socket hang up at connResetException

Seriously?

I have to wrap it all within a try/catch and keep restarting it.
The frustrating thing is, I think there's actually a process.exit call in there
somewhere. Deep.

This might be a case of forming plasma in a stable orbit around a magnetic torus
in order to generate positive net energy output.

#### 08.03.2024 Friday 6h

All of the testnet seed dns servers are down:

```
dig -t A seed-1.testnet.networks.dash.org
```

Actually that dns query returns fine.  
But getting constant `ECONNRESET` from the dash library.  

```
nc -zv -w1 -G1 seed-1.testnet.networks.dash.org 1443
```

Connection succeeded??  

I'm honestly a bit lost at this point.  
It's still erroring while the actual ports are open.
Looks like I need to dive into the dashevo code and recreate things at a lower
level.

```
curl \
  --request GET \
  --header 'Content-Type: application/json' \
  --data '{"method": "getBestBlockHash"}' \
  'https://seed-3.testnet.networks.dash.org:1443/'
```

Found a status page url on the discord:
https://dash-testnet.freshstatus.io/

Dash testnet platform is DOWN.

Perhaps I create a regtest platform for now...?

- [ ] update thorchain dash go llmq project to optionally launch the platform services

```
docker pull dashpay/dapi:1.0.0-dev.7
docker pull dashpay/drive:1.0.0-dev.7
```

They also updated:
`dashpay/dashmate-helper`
and
`dashpay/platform-test-suite`
at the same time as the above 2, but I don't think they're needed atm?

Let's have another look at dashmate.

Do you need tenderdash for platform?
What about envoy?

```
pnpm i -g dashmate

pnpx dashmate setup local
pnpx dashmate config envs --config=local --output-file .env.local
docker compose --env-file .env.local up
```

> To allow developers to quickly test changes to DAPI and Drive, a local path
  for this repository may be specified using the `platform.sourcePath` config
  options. A Docker image will be built from the provided path and then used by
  Dashmate.

An example would be great. Not sure what that means, how to do that, what format
they want.

> required variable PLATFORM_DRIVE_ABCI_CHAIN_LOCK_LLMQ_TYPE is missing a value: err

Damn it was all going so well untill I tried docker compose up.  
Removed the required config values from the docker compose that were not set in the env file.
Not stable after 5m.

Looks like I'm going to have to become a dash platform expert now just so I can
continue development.

tenderdash keeps crashing with:
> panic: runtime error: slice bounds out of range [32:3]

https://github.com/dashpay/tenderdash/blob/v0.14-dev/docs/tools/docker-compose.md

Maybe I'll have more luck with the tenderdash docker compose.

```
docker run \
--rm \
--name tenderdash \
-it \
--entrypoint bash \
-v $(pwd)/networks/local/localnode/config-template.toml:/mnt/config-template.toml \
dashpay/tenderdash:latest

tenderdash testnet --config /mnt/config-template.toml --o .

tenderdash init seed
tenderdash testnet

 --starting-ip-address 192.167.10.2

tenderdash start
```

> ✖ externalIp option is not set in local config

```
dashmate setup local
dashmate setup local --verbose

dashmate start --config local
dashmate start --config local_1

dashmate status --config local

dashmate group reset --group local --hard
rm -rf ~/.dashmate
```

Where are the configs created?
`~/.dashmate`


> Wallet file verification failed. Failed to create database path '/home/dash/.dashcore/regtest/wallets/main'. Database already exists.

The group reset command above doesn't seem to be clearing out the docker volumes??

Okay the externalIp field really wasn't set in `~/.dashmate/config.json > configs.local.externalIp`

I set it to `192.168.65.254` because that was in `local_1`

> core.masternode.operator.privateKey option is not set in local config

tenderdash crashed with:

> 2024-03-09T03:37:49.9599195Z INFO Stopping abci.socketClient reason="core rpc
  error: JSON-RPC error: RPC error response: RpcError { code: -32603,
  message: \"Unable to find any ChainLock\", data: None }"

It's like someone asked me to dig a hole, but gave me a shovel that
spontaneously disassembles itself. It's infuriating and mentally draining.
It makes me think I'm better off learning how to make shovels than digging
holes, but that's not why I'm here.


#### 11.03.2024 Monday 6h

```
dashmate status --config local
dashmate setup local
dashmate config --config local
dashmate start --config local
dashmate update --config local
```

Right that's it, stuck on this:

> core.masternode.operator.privateKey option is not set in local config

```
dashmate stop --force --config local
dashmate group:stop --force
dashmate group reset
dashmate group start
```

Well `group start` seems to be a lot more reasonable.

So the `dashapi` is NOT exposed on any of the host ports.  
`envoy` is available. Is that what I want?  
Yes. It runs on port 2443, all the public nodes I tested before were running
on 1443.  

So configured my little test project to connect to that.

Getting: DEPTH_ZERO_SELF_SIGNED_CERT

```
pnpm add @dashevo/wallet-lib@./node_modules/@dashevo/platform/packages/wallet-lib
```





