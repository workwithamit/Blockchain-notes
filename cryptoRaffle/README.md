# Blockchain-notes
1. create next app using cmd line
2. folder structure of next.js (pages..)

# pages are basically routes, the diffrent pages of our website.

# Everything goes through \_app.js

3. components
4. install react-moralis for simplying things (like enabling web3 wallets and all)
5. instead of using pure ethers
6. difference between dependencies(used for application code) and dev-dependecies(for smoothening the work of developer like prettier and all)
7. dependencies are bundled together for frontend

//create a component folder

1. create ManualHeader.jsx

we will be using hook ---

# useMoralis() from which we will deconstruct enableWeb3

# hooks are way to keep track of states

### In order to use moralis,

we need to wrap our whole application(i.e app.js) in moralis provider which is going to be a context provider for us.

import {MoralisProvider} from 'react-moralis'

function MyApp({component,pageprops}){
return (<MoralisProvider  initializeOnMount={false}>
<components {...pageprops} />
</MoralisProvider>
)
}

#This initalizeonmount piece here is the optionality to the hook into the server to add some more features to our website.

#we want to be open source ---> no server.

### React hooks

#Insted of using hooks is we use some variable
for example let connected = false;
but it's not going to re-render our page
so hooks are way to actually work with state especially and automatically.

### const{enableWeb3} = useMoralis()

enableWeb3 is equivalent to saying like this:

### await ethereum.request({method: "eth_requestAccounts})

### Next Section (creating button in ManualHeader.jsx)

<button onClick={async()=>{await enableWeb3()}>connect</button>

## isWeb3Enabled

this is part of our hook useMoralis() which keeps track of whether or not our metamask is connected.

### account

we can do better instead of using isWeb3Enabled we can use //account

it will check if there is account

return (<div>
{account ? (<div>connected!</div>) : (<button onClick={async()=>{await enableWeb3()}>connect</button>)}

</div>)

//even better

return (<div>
{account ? (<div>connected to {account} </div>) : (<button onClick={async()=>{await enableWeb3()}>connect</button>)}

</div>)

//even more better

return (<div>
{account ? (<div>connected to {account.slice(0,6)}......{account.slice(account.length-4)} </div>) : (<button onClick={async()=>{await enableWeb3()}>connect</button>)}

# useEffect hook(this is core react hook)

second argument is optional

useEffect(<function>,<dependency>)

The useEffect Hook allows you to perform side effects in your components.

# Problem in our code:

whenever i hit refresh my app gets disconnected from my wallet
or we can say our website doesn't know we have hit enableWeb3 already

so i have to re-connect everytime i hit refresh

for removing this problem
i will use another hook

## useEffect hook

1. No dependency passed:
   useEffect(()=>{
   //Runs on every render
   });
   useEffect runs on every render. That means that when the count changes, a render happens, which then triggers another effect.

2. An empty array:
   useEffect(()=>{
   //runs only on the first render
   },[]);

## ManualHeader.js

//oh!! some value has changed, i'm going to run this function , and then i'm gonna re-render the page

useEffect(()=>{
console.log('Hi');
console.log(isWeb3Enabled)
},[])

//dependency array: run when the stuff in it changes

// no dependency array: run anytime something re-renders
// Be careful with this !! because then you can get circular render
//Blank dependency array, run once

//it may run twice in your app due to react.strict mode

# useEffect in our code

we will extract one more function from useMoralis()

# isWeb3Enabled

we will use this to check if {we are already connected to web3} then we don't need to do anything
else {we will call enableWeb3()}

```
useEffect(()=>{
    if(isWeb3Enabled)return;

    enableweb3()

},[isWeb3Enabled])
```

# problem

if connected no problem but if we disconnect the wallet then it will automically pop up the metamask again and again without clicking on the 'connect' button

# Solution to this problem is (we want our applicaton to remember that somebody hit this connect button and they went and connected to us)

# Local Storage(we will store whether our walllet is connected to site or not)

//nextjs find hard time to process window(window object) in code

so in our <button>connect</button>

```
if(typeof window!="undefined"){
window.localStorage.setItem("connected","injected")};
```

# const{enableWeb3, account, isWeb3Enabled, Moralis, deactivateWeb3} = useMoralis()

now in our useEffect hook

```
useEffect(()=>{
    if(isWeb3Enabled)return;
    if(typeof window!=="undefined"){
        if(window.localStorage.getItem("connected")){
            enableWeb3()
        }
    }
    enableweb3()

},[isWeb3Enabled])
```

one more useEffect in the town(code) Yoo!!!

//on c

```
useEffect(()=>{
    Moralis.onAccountChanged((account))=>{
    console.log('Account changed to ${account})
        if(account==null){
        window.localStorage.removeItem("connected");
        deactivateweb3();
        console.log("Null account Found");
    }
}
},[]);
```

# one more functionality

when we hit connect button, we want to disable this connect button

# isWeb3EnableLoading (used to check if metamask has popped up)

const {enableWeb3,account,isWeb3Enabled,Moralis,deactivateWeb3,isWeb3EnableLoading}

# In our connect button

```
<button onclick={jo bhi likha tha} disabled={isWeb3EnableLoading}>
```

//Above was manual way
//Now easy way

# Header.jsx

# Web3uikit

# yarn add Web3uikit

```
import {connectButton} from 'web3uikit'
export default function Header(){
    return(<div>
        Decentralized Lottery
        <ConnectButton moralisAuth={false} />
    </div>)
}
```

# moralis ConnectButton provides functionalities which we have in manualHeader.jsx

### moralisAuth = {false}

means we are super explicit about not connecting to any server, we want to be open source.

# introduction to calling Function in NextJs

let's create a new component

# lotteryEntrance.js

---> what do we want to at first
---> have a function to enter the lottery

---> In fundMe we did it using calling a function fund (using provider, signer, contract)

# but doing it like this won't rerender.

---> so here we will use react-moralis which so many hooks to do pretty much everything.

# Moralis is smart enough to know about function whether it is a view function or going to be a transaction.

# react-Moralis has a hook useWeb3Contract();

```
const {data,error,runContractFunction, isFetching, isLoading}=useWeb3Contract({
    abi:
    ...
    ...
    ....

})
```

---> This hooks will give us the data returned from a function called an error returned, a little function that we can use to call any function

---> we have some really cool things

# isFetching and isLoading

---> so if we want our UI to do something while it's fetching or while it's loading the transaction we can use these two variables.

import the above hook

---> call this function to enter the lottery

```
const {runContractFunction: enterRaffle} = useWeb3Contract({
    abi: //,
    contractAddress: //,
    functionName: //,
    params: {}, ----> this is going to be blank, as we know we don't pass any parameters to this particular enterRaffle function
    msgValue: //

})
```

---> msgValue: all we need to pass this value to enter the raffle

# Automatic Constant Value UI Updater

// If I make any changes in my smartcontract I want that change to be reflected in my front-end

// and I am able to code my frontend as such

# Pro tip: (lol)

# create an update front end deploy script

--> so after we deploy script, we run a little script that will create this constant folder(in which we will put abi,smartcontractaddress and etc.)

--> we will create a deploy script in deploy folder in backend

--> so when we make any changes in backend it will update the constant folder in frontend.

---> sometimes i don't care about front-end if we've specified a dot env variable.

---> In .env file

```
UPDATE_FRONT_END = true;
```

# 99-update-front-end.js

```
module.exports = async function(){
    if(process.env.UPDATE_FRONT_END){
        console.log("updating the front end ...)
    }
}


```

# run the scripts

```
yarn hardhat deploy
```

```
module.exports = async function(){
    if(process.env.UPDATE_FRONT_END){
        console.log("updating the front end ...)
        updateContractAddresses();
        updateAbi()
    }
}

async function updateAbi(){
    const raffle = ethers.getContract("Raffle")
    // ---> we can get abi with this one line of code
    fs.writeFileSync(FRONT_END_ABI_FILE, raffle.interface.format(ethers.utils.formatTypes.json))
}

async function updateContractAddresses(){
    const raffle = await ethers.getContract("Raffle")
    const chainId = network.confing.chainId.toString();

    const currentAddresses = JSON.parse(fs.readFileSync(FRONT_END_ADDRESS_FILE,"utf8"))

    if(chainId in currentAddresses){
        if(!currentAddresses[chainId].includes(raffle.address)){
            currentAddresses[chainId].push(raffle.address)
        }
        {
            currentAddresses[chainId] = [raffle.address]
        }
        fs.writeFileSync(FRONT_END_ADDRESSES_FILE,JSON.stringify(currentAddresses));
    }
}



```

---> we just automated the process of updating the abi and contract addresses

---> we will run hh node in frontend(folder) terminal

---> we can put both the file in index.js for accessing them in one line

# runContractFunction

---> we can call function using moralis by importing {useWeb3Contract}

```
const {runContractFunction: enterRaffle} = useWeb3Contract({
    abi:abi,
    contractAddresses:
    //specify the networkId
    contractAddresses[???][0],
    functionName: "enterRaffle",
    params:{},
    msgValue:???


})
```

# we will again import useMoralis

---> The reason moralis knows what chain we're on is because back in our header component the header actually passes up all the information about the metamask to the <MoralisProvider>

---> Then the <MoralisProvider> passes down to all the components inside this moralisProvider tag.

---> we get the hex version of our chainId.

```
const {chainId: chainIdHex} = useMoralis()
```

--> this above line means pull out the chainId object and then rename it to chainIdHex

```
const {chainId: chainIdHex} = useMoralis();
const chainId = parseInt(chainIdHex);
const raffleAddress = chainIdHex in contractAddresses ? contractAddresses[chainId][0]:null


```

```
const {runContractFunction: enterRaffle} = useWeb3Contract({
    abi:abi,
    contractAddresses:
    //specify the networkId
    contractAddresses[???][0],
    functionName: "enterRaffle",
    params:{},
    msgValue:???


})
```

---> This is one of the ways to send the transaction and we can also send functions

--> one of the ways that we're going to do it right when our lottery entrance loads, we're gonna run a function to read the entrance fee value

---> and how we're going to do that
we will be using useEffect hook

# use Effect can run when something changes

---> we are only going to want to try to get that raffle if web3 is enabled

--->pull out {isWeb3Enabled} from useMoralis()

# Revisit( couldn't grasp much) timestamp 17:40

```
let entranceFeeFromContract = "";
useEffect(()=>{
    if(isWeb3Enabled){
        //try to read the raffle entrance fee
        async function updateUI(){
            entranceFeeFromContract = await getEntranceFee();
        }
        updateUI();
    }
},[isWeb3Enabled]);
```

# problem

---> we will get the entrance fee from call it will be updated but it won't re-render.

# useState # 17:46

```
import {useState} from 'react'

const [entranceFee, setEntraceFee]=useState(0);

```

---> updated code

```

useEffect(()=>{
    if(isWeb3Enabled){
        //try to read the raffle entrance fee
        async function updateUI(){
            entranceFeeFromCall = await getEntranceFee();
            setEntranceFee(entranceFeeFromCall)
        }
        updateUI();
    }
},[isWeb3Enabled]);
```

# small question backend mai entrance fee pahle se declared h ya hum set krte h?

# one more why do we need to reset the reset account in metamask after a while?

---> the entranceFee will be wei, convert it to ether using "ethers"

```
ether.utils.formatUnits(entranceFee,"ether")
```

# Calling functions in NextJs

---> so finally we have entranceFee, and a function to enter the lottery

for using entranceFee we need keep it in raw form

do not convert it into .toString()

you can change it while manupulating html jsx whatever

```
const {runContractFunction: enterRaffle} = useWeb3Contract({
    abi:abi,
    contractAddresses:
    //specify the networkId
    contractAddresses[???][0],
    functionName: "enterRaffle",
    params:{},
    msgValue:entranceFee


})
```

---> updated the msg.value(we have make a button that's gonna do that)

# Problem: when we change the account in the metamask(other than local host hardhat) we get a kind of error

---> because we're calling

```
(await getEntranceFee()).toString()
```

on an address that doesn't exist

# It might be confusing what we're doing here

# add a button to enter the raffle

---> before we actually do that, let's make sure that we can only call the function as long as there is a raffle address

# In LotteryEntrance.js

```
return(
    <div>
        Hi from Lottery Entrance!
        {
            raffleAddress ? (
                <div>
                    <button
                        onClick = {
                            async function() {
                                await enterRaffle()
                            }
                        }
                    >
                    Enter Raffle
                    </button>

                    Entrance Fee: {ethers.utils.formatUnits(entranceFee,"ether")}ETH
                </div>
            ) :(
                <div> No Raffle Address Detected</div>
            )
        }

    </div>
)

```

# Problem: The above thing is not useful for user who are following along with this to look at this and go, Okay, did it go through? or we did it fail? like what's just happened?

# So, what we gonna do, we create notifications want a little pop up saying, hey, you sent your transaction, great job

# again we're going to use some library web3uikit, u can search notification and import it in \_app.js

---> we will wrap all the components in NotificationProvider

```
import {NotificationProvider} from "web3uikit"

<MoralisProvider>
<NotificationProvider> <componetnts>
</NotificationProvider>
</MoralisProvider>



```

# Back into our LotteryEntrance.js

---> we are gonna import a hook

```
{useNotification} from 'web3uikit'
```

---> this useNotification gives us a thing 'dispatch'(it is something that will give pop up)

```
const dispatch = useNotification()

```

# Enter Raffle Button

```
<button
        onClick = {
            async function() {
                await enterRaffle(

                    //onComplete:
                    //onError:
                    onSuccess: handleSuccess,
                    onError: (error)=>console.log(error),

                )
            }
        }
    >
    Enter Raffle
</button>

```

# Pro tip(lol)

# REMEMBER TO ADD THIS // onError:(error)=>console.log(error) to all of your runContractFunction.

# we will use isLoading and isFetching

```
<button
        onClick = {
            async function() {
                await enterRaffle()
            }
        }
        disabled={isLoading || isFetching}

        {
            isLoading || isFetching ? (
                "some spinner animation"

            ) : ("Enter Raffle")
        }
    >
    Enter Raffle
</button>


```

# create handleSuccess()

```
const handleSuccess = async function (tx){
    await tx.wait(1);
    handleNewNotification(tx);
}

const handleNewNotification = function (){
    dispatch({
        type:"info",
        message: "Transaction Complete",
        title: "Tx Notification",
        position: "topR",
        icon:"bell",
    })
}


```

# You might be confuse about this handleSuccess function()

---> This handleSuccess() function is not checking that the transaction has a block confirmation, it's just checking to see that the transaction was successfully send to metamask.

---> So, onSuccess(), checks to see a transaction is successfully sent to metamask and

# That's why up there in that other function we do tx.wait(1);

---> And that's the piece which waits for transaction to be confirmed

# Let's display how many players are playing and recent winner

const {runContractFunction: getEntranceFee}=useWeb3Contract({
abi:abi,
contractAddress: raffleAddress, //specify the networkId

    functionName: "getEntranceFee",
    params:{}

})

const {runContractFunction:getNumberOfPlayer} = useWeb3Contract({
abi:abi,
address:raffleAddress //specify chainId

    functionName: "getNumberOfPlayers"
    params:{}

})

const{runContractFunction:getRecentWinner}=useWeb3Contract({
abi:abi,
address:raffleAddress,=//specify chainId,
functionName: "getRecentWinner",
params:{}

})

# update the code of

```
useEffect(()=>{
},[])
```

```
useState()
```

```
useEffect(()=>{
    if(isWeb3Enabled){
        //try to read the raffle entrance fee
        async function updateUI(){
            entranceFeeFromCall = await getEntranceFee();
            NumPlayerFromCall = (await getNumberOfPlayers()).toString();
            setEntranceFee(entranceFeeFromCall)
            setNumberOfPlayers(NumberOfPlayers);
            setRecentWinner(RecentWinner)
        }
        updateUI();
    }
},[isWeb3Enabled]);
```

# Problem : we need to refresh for getting updated number of players(it doesn't re-render), that's kindaa annoying.

# Solution: guess who is gonna do that, the handleSuccess

---> we will make a standalone function of updateUI removing it from useEffect and put it into the handleSuccess()

```
const handleSuccess = async function (tx){
    await tx.wait(1);
    handleNewNotification(tx);
    --> updated thing
    updateUI();
}
```

# Now we're gonna test getting a recentWinner

---> create a new script in backend in scripts folder

# mockOffchain.js

---> This script is for testing for getting the recentWinner

# 18:02

---> once we call that mocking script, I have to refresh the browser to see the winner here
and number of players obviously got reset to zero.

# Problem: That's not ideal

---> we want our UI to update automatically when some events gets fired.

---> In our raffleContract, we get this event emitted.

---> Instead of In our code doing this await succes in Onclick function of button.

---> What we could do is we could set up a portion to listen for that event being emitted and update the front end accordingly.

---> with that knowledge, we can also listen for the winner event being emitted.

---> we can update our frontend instead of refreshing to get recentWinner.

# Tailwind and styling

## Tailwind with nextjs

---> Head over to the "tailwindcss.com/docs/guides/nextjs

### Tailwindcss, postcss and autoprefixer

---> once we done installing them, we gonna init tailwind and make a config file

---> then configure your template paths

---> tailwind.config.js

```
module.exports = {
    content: ........

}
```

--> anything that is content is tailwindable

### and just follow the docs and make changes to your frontend .

# Install PostCss Language Support extension

# 18:12

# Introduction to Hosting your site

---> we're gonna deploy it to some testnet that can take very long time and then we are going to deploy our website to a hosting provider

# Having a decentralized frontend, that's a little bit harder

# EtherScan is centralized, but we're using it alot.

# Now let's learn how to deploy this frontend in decentralized way

# Next.js makes a static website

# Both moralis and next.js have optionality to not to have non static code

# IPFS(decentralized database)

---> decentralized distributed data structure that is not a blockchain but it's similar to a blockchain. There's no mining though but dates and pinning.

---> You can hash the code file/folder that's actually ipfs does

---> encodes the folder or massive file data into some hash

---> we can hash our data in our IPFS node

---> It hosts our data and have these hashes.

---> Our node is connected to a network or other IPFS nodes.(massive network). It is way lighter than other blockchain.

---> That's kindaa centralized, it stores the data in node.

---> But other nodes are like I like this data and pin that data.(have a copy of the entire blockchain IPFS nodes get to optionally to choose what to have and what note to )

# Install and work with IPFS

# deploy to IPFS

---> As IPFS cannot run the code or execute the code.So our code file should be static. If it does have some server stuff, it won't work.

# Yarn build

---> optimized production build

# yarn next export

---> It will fail if we have some non static file , server stuff

---> this command will create a .out folder

---> Upload that folder to ipfs

---> Pin to our node, copy our CID and paste it in browser.

# IPFS companion(some browser have routers like brave and firefox)

# Easier Way(fleek.co)

---> open web: permissionless and etc.

---> auto deployment

watch the video from {below timestamp} for deploying it on fleek

# 18:25

# Fleek helps you create those deals and helps you pin your data with this filecoin blockchain

# You can login in Fleek using github and give authorization to particular repo and deploy it(watch the video after 18:15)

# when we make any changes in our repo it will deploy the progress automatically to same url(whatever url is previously)

# Fleek uses IPFS to host your site or App.

# And this is kinddaa just a router for ipfs. so people without ipfs connected can also connect to this still.

# 18:31

# FileCoin is actually a blockchain that helps you pin your data and uses decentralized storage to do so.

---> In Ipfs you have to have people pin your data, in order for it to stay distributed and decentralized.

---> Filecoin is data persistent, decentralized and distributed.

---> Ipfs uses content addressing while normal uses http

# FileCoin v/s IPFS

---> centralized storage (problems)

-->being an attack vector, for data mining, getting leaked through and also creating an data resilience problem.

---> Designed to be offline first for resilience.

---> Not an another peer to peer network, nice thing about IPFS protocol is the standard it uses for addressing content on the network.

---> CID(content Id) makes it unique.

---> Persistent and permanence of data on the network.

---> In Ipfs, our data need to pinned, so either it is popular or we've to pay to a pinning service.

# problems with IPFS

---> we're heading back to centralization. we're are creating new data silos with this solution and losing the trustlessness and resilience we are looking for.

# this where comes in the filecoin

---> it is designed together with crytographic proofs in order to ensure data is stored persistently, highly, reliably and verifiably.

---> user can keep it for whatever time they want.

# Filecoin uses storage deals

---> rewards for good actors and penalities for bad actors.

---> storage deals ensures that the they have generated unique copy of your data. time to time they ensure they have many copies of data.

---> proof of space time

---> proof of space time is stored on blockchain.

---> proof of stake for storage provider.(bidding happens)

---> storage deal (expires and can be renewed)

---> aim of filecoin is not data permanence rather it's data timeframe, sovereignty too.

---> IPFS:- content addressing(kinddaa http)
---> Filecoin: data persistent.(blockchain)

# Some more tools

### NFT.Storage, web3.storage, textile powergate(for ease of developer for connecting to ipfs and all)

### Estuary.tech(for storing large dataset or meaningful public data), OrbitDB(peer to peer distributed database)

### Filecoin virtual machine.
