---
title: Hello World
---

# How to Deploy Hello, World! Smart Contract in 20 Minutes

This is a fast and minimalist tutorial for deploying a "hello world"
smart contract or dapp to the IC in 20 minutes or less. All that is
necessary to run this tutorial is basic knowledge of using a terminal
(no editing of code).

## Introduction

In this tutorial, we will deploy a simple `Hello` canister smart
contract that has just one function—called `greet`. The `greet` function
accepts one text argument and returns the result with a greeting. For
example, if you call the `greet` method with the text argument `Alice`.

1.  If you call it via the command-line, dapp will return
    `Hello, Alice!` in a terminal

2.  If you access the dapp in a browser, it will alert pop-up window
    reading `Hello, Alice!`

While the code comes ready out of the box for you, this dapp consists of
back-end code written in Motoko, a programming language specifically
designed for interacting with the Internet Computer (IC), and a simple
webpack-based front-end.

### Concepts necessary for this tutorial

#### Canister smart contract/dapp

A **canister/dapp** is a type of smart contract that
bundles code and state. A canister can be deployed as a smart contract
on the Internet Computer and accessed over the Internet.

#### Cycles

On the Internet Computer, a **cycle** is the unit of measurement for
resources consumed in the form of processing, memory, storage, and
network bandwidth. Every canister has a cycles account to which
resources consumed by the canister are charged. The Internet Computer’s
utility token (ICP) can be converted to cycles and transferred to a
canister. ICP can always be converted to cycles using the current price
of ICP measured in \[SDR\] using the convention that one trillion cycles
correspond to one SDR.

In this tutorial, you will get free cycles from the cycles faucet.

## Platforms and operating systems supported

-   Linux

-   macOS (12.\* Monterey)

-   Windows

## Installing tools (5 min)

To build a dapp, users need to install the following:

### Node.js

This tutorial works best with node.js version higher than `16.*.*`.

There are many ways of installing node.js, including from [nodejs.org
website](https://nodejs.org/en/download).

Besides installing node.js, users need to also install: \* Node Package
Manager (NPM). \* Node Version Manager (NVM), see [installing
NVM](https://github.com/nvm-sh/nvm#installing-and-updating).

### The Canister SDK (AKA "dfx")

The Canister SDK used in this article is called `dfx` and it is
maintained by the DFINITY foundation. There are many SDK’s so this is
just one.

To install, one needs to run:

    $ sh -ci "$(curl -fsSL https://sdk.dfinity.org/install.sh)"

To verify it is properly installed:

    $ dfx --version

Terminal should look like this (at least version 0.9.2):

![dfx version](_attachments/dfx-version.png)

## Create a default project (2 min)

### Create a new project

Dapps on the Internet Computer start as projects. You create projects
using the `dfx` parent command and its subcommands.

For this tutorial, we’ll start with the default sample dapp to
illustrate creating dapp using the starter files in a project. When you
create a new project, the `dfx` command-line interface adds a default
project directory structure to your workspace. We cover the template
files that make up a project directory in the Explore the default
project tutorial.

To create a new project for your first application:

2.1 Open a terminal shell on your local computer, if you don’t already
have one open.

2.2 Create a new project named hello by running the following command:

    $ dfx new hello

The `dfx new hello` command creates a new project directory named
`hello`, template files, and a new `hello` Git repository for your
project. Your terminal should look like this:

![dfx new](_attachments/dfx-new-hello-1.png)

![dfx new](_attachments/dfx-new-hello-2.png)

2.3 Move to your project directory by running the following command:

    $ cd hello

Your directory should look like this:

![cd new](_attachments/cd-hello.png)

## Deploying dapp to local machine (3 min)

### Using two terminal windows/tabs

Now that the code is there, the next step is to spin up a local version
of the IC on local machine. For this, developers should keep two
terminals open:

-   **Terminal window/tab A:** This will show the local version of the
   ICrunning

    -   This is analogous to how developers often start local servers in
        web2 projects (e.g. node.js’s Express, python’s Django, Ruby’s
        Rails, etc…)

-   **Terminal window/tab B:** This will be used to send **messages** to
    the local version of the IC

    -   This is analogous to how developers send HTTP API messages to
        servers running locally (e.g. Postman, Panic).

For ease, this tutorial will distinguish between both windows by color
scheme:

**Terminal A**

![dfx new](_attachments/dfx-new-hello-2.png)

**Terminal B**

![terminal b ls](_attachments/terminal-b-ls.png)

### Start the local version of the IC (Terminal A)

1.  Use the Terminal window or tab on your local computer.

2.  Navigate to the root directory for your project, if necessary. In
    this tutorial, you should be in the folder `hello` because that is
    the name of the project created in section 2 above.

3.  Start the local canister execution environment on your computer in
    your second terminal by running the following command:

<!-- -->

    $ dfx start

![dfx start](_attachments/terminal-a-dfx-start.png)

Note: Depending on your platform and local security settings, you might
see a warning displayed. If you are prompted to allow or deny incoming
network connections, click Allow.

**That is it, there is now a local version of the IC running on your
machine. Leave this window/tab open and running while you continue.** If
the window/tab is closed, the local version of the IC will not be
running and the rest of the tutorial will fail.

### Deploy the "hello" dapp to the local version of the IC (Terminal B)

Note: since this is a local version of the IC, this has fewer steps than
deploying to mainnet (which requires cycles).

To deploy your first dapp locally:

1.  Check that you are still in the root directory for your project, if
    needed.

Ensure that node modules are available in your project directory, if
needed, by running the following command (it does not hurt to run this
many times):

    $ npm install

![npm install](_attachments/terminal-b-npm-install.png)

1.  Register, build and deploy dapp:

<!-- -->

    $ dfx deploy

![dfx deploy](_attachments/terminal-b-dfx-deploy.png)

### Testing the dapp locally via command line (Terminal B)

Now that the canister is deployed to local replica, you can send it a
message. Since the canister has a method called `greet` (which accepts a
string as a parameter), we will send it a message.

    $ dfx canister call hello greet everyone

-   The `+dfx canister call+` command requires you to specify a canister
    name and a method—or function—to call.

-   `+hello+` specifies the name of the **canister** you want to call.

-   `+greet+` specifies the name of the **function** you want to call in
    the `+hello+` canister.

-   `+everyone+` is the text data type argument that you want to pass to
    the `+greet+` function.

### Testing the dapp locally via the browser

Now that you have verified that your dapp has been deployed and tested
its operation using the command line, let’s verify that you can access
the front-end using your web browser.

3.4.1 On terminal B, start the development server with:

    $ npm start

3.4.2 Open a browser. 3.4.2 Navigate to <http://localhost:8080/>

Navigating to this URL displays a simple HTML page with a sample asset
image file, an input field, and a button. For example:

\+ ![Sample HTML page](_attachments/front-end-prompt.png)

1.  Type a greeting, then click **Click Me** to return the greeting.

    For example:

    ![Hello](_attachments/front-end-prompt.png)
front-end-result.png)

## Stop the local canister execution environment

After testing the application in the browser, you can stop the local
canister execution environment so that it does not continue running in
the background.

To stop the local deployment:

1.  In the terminal A, press Control-C to interrupt the local network
    process.

2.  In the terminal B, press Control-C to interrupt the development
    server process.

3.  Stop the local canister execution environment running on your local
    computer by running the following command:

        dfx stop

## Deploying on-chain (10 min)

### Important note about cycles

In order to run on-chain,ICdapps require cycles to pay for compute and
storage. This means that the developer needs to acquire cycles and fill
their canister with them. Cycles can be converted from ICP token.

This flow may be surprising to people familiar with Web2 software where
they can add a credit card to a hosting provider, deploy their apps, and
get charged later. In Web3, blockchains require their smart contracts
consume **something** (whether it is Ethereum’s gas or the IC’s cycles).
The next steps will likely be familiar to those in crypto, but new
entrants may be confused as to why first step of deploying a dapp is
often "go get tokens."

### Acquiring cycles and adding them to your canister (Terminal B)

For the purposes of this tutorial, you can acquire free cycles for your
"hello world" dapp from the cycles faucet. Follow the instructions here:
[Claim your free cycles](cycles-faucet.md).

Few notes about cycles:

-   Cycles pay for computation and storage

-   Cycles faucet will grant developers 15 trillion cycles

-   It takes 4 trillion cycles to deploy a canister.

-   You can see a table of compute and storage costs here: [Computation
    and storage
    costs](../linking/computation-and-storage-costs.md).

### Check the connection to the Internet Computer blockchain mainnet (Terminal B)

As sanity check, it is good practice to check your connection to the IC
is stable:

Check the current status of the Internet Computer blockchain and your
ability to connect to it by running the following command for the
network alias ic:

    $ dfx ping ic

If successful you should see output similar to the following:

    $ {
      "ic_api_version": "0.18.0"  "impl_hash": "d639545e0f38e075ad240fd4ec45d4eeeb11e1f67a52cdd449cd664d825e7fec"  "impl_version": "8dc1a28b4fb9605558c03121811c9af9701a6142"  "replica_health_status": "healthy"  "root_key": [48, 129, 130, 48, 29, 6, 13, 43, 6, 1, 4, 1, 130, 220, 124, 5, 3, 1, 2, 1, 6, 12, 43, 6, 1, 4, 1, 130, 220, 124, 5, 3, 2, 1, 3, 97, 0, 129, 76, 14, 110, 199, 31, 171, 88, 59, 8, 189, 129, 55, 60, 37, 92, 60, 55, 27, 46, 132, 134, 60, 152, 164, 241, 224, 139, 116, 35, 93, 20, 251, 93, 156, 12, 213, 70, 217, 104, 95, 145, 58, 12, 11, 44, 197, 52, 21, 131, 191, 75, 67, 146, 228, 103, 219, 150, 214, 91, 155, 180, 203, 113, 113, 18, 248, 71, 46, 13, 90, 77, 20, 80, 95, 253, 116, 132, 176, 18, 145, 9, 28, 95, 135, 185, 136, 131, 70, 63, 152, 9, 26, 11, 170, 174]
    }

### Deploying on-chain (Terminal B)

You are now ready to deploy your dapp on-chain.

    $ npm install

    $ dfx deploy --network ic

The `--network` option specifies the network alias or URL for deploying
the dapp. This option is required to install on the Internet Computer
blockchain mainnet.

If succesful, your terminal should look like this:

    Deploying all canisters.
    Creating canisters...
    Creating canister "hello"...
    "hello" canister created on network "ic" with canister id: "5o6tz-saaaa-aaaaa-qaacq-cai"
    Creating canister "hello_assets"...
    "hello_assets" canister created on network "ic" with canister id: "5h5yf-eiaaa-aaaaa-qaada-cai"
    Building canisters...
    Building frontend...
    Installing canisters...
    Installing code for canister hello, with canister_id 5o6tz-saaaa-aaaaa-qaacq-cai
    Installing code for canister hello_assets, with canister_id 5h5yf-eiaaa-aaaaa-qaada-cai
    Authorizing our identity (default) to the asset canister...
    Uploading assets to asset canister...
      /index.html 1/1 (472 bytes)
      /index.html (gzip) 1/1 (314 bytes)
      /index.js 1/1 (260215 bytes)
      /index.js (gzip) 1/1 (87776 bytes)
      /main.css 1/1 (484 bytes)
      /main.css (gzip) 1/1 (263 bytes)
      /sample-asset.txt 1/1 (24 bytes)
      /logo.png 1/1 (25397 bytes)
      /index.js.map 1/1 (842511 bytes)
      /index.js.map (gzip) 1/1 (228404 bytes)
      /index.js.LICENSE.txt 1/1 (499 bytes)
      /index.js.LICENSE.txt (gzip) 1/1 (285 bytes)
    Deployed canisters.

Note, a common error one may get in section 4.4:
`Error: The replica returned an HTTP Error: Http Error: status 403 Forbidden`.
This error means that the canister does not have enough cycles to
deploy.

### Testing the on-chain dapp via command line (Terminal B)

Now that the canister is deployed on-chain, you can send it a message.
Since the canister has a method called `greet` (which accepts a string
as a parameter), we will send it a message.

    $ dfx canister --networkiccall hello greet '("everyone": text)'

Note the way the message is constructed: \*
`dfx canister --networkiccall` is setup for calling a canister on the
IC \* `hello greet` means we are sending a message to a canister named
`hello` and evoking its `greet` method \* `'("everyone": text)'` is the
parameter we are sending to `greet` (which accepts `Text` as its only
input).

## Troubleshooting

### Resources

-   Developers who hit any blockers are encouraged to search or post in
    [IC developer forum](https://forum.dfinity.org).