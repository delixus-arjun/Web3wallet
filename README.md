# DEPRECATED: Tatum Wallet
This repository will no longer be maintained since the DEMO application is outdated.

*Note that you will need to have [Node.js](https://nodejs.org/en/download/current/) 14+ installed.*

*THIS IS DEMO APPLICATION USED FOR DEMONSTRATING FEATURES DURING THE ONLINE WORKSHOP. NOT FOR PRODUCTION USE!!!!!!!*

## Introduction

This repository is part of the online workshop - How to build a crypto wallet in an hour?. Video is available [here](https://www.youtube.com/watch?v=s3mLvVGM9GQ).

## Get started

Clone the repository

```bash
git clone https://github.com/tatumio/tatum-wallet.git

cd tatum-wallet
```

Install the dependencies...

```bash
npm install
```

...then start webpack:

```bash
npm run dev
```

Navigate to [localhost:8080](http://localhost:8080). You should see your app running. Edit a component file in `src`, save it, and the page should reload with your changes.

## Tatum API configuration

In order to communicate with [Tatum API](https://tatum.io), API Key must be provided in *.env* file. Create *.env* file and enter valid API Key. 
```dotenv
TATUM_API_KEY = 123456789123456789123456789
```


## Deploying to the web

### With [now](https://zeit.co/now)

Install `now` if you haven't already:

```bash
npm install -g now
```

Then, from within your project folder:

```bash
now
```

As an alternative, use the [Now desktop client](https://zeit.co/download) and simply drag the unzipped project folder to the taskbar icon.

### With [surge](https://surge.sh/)

Install `surge` if you haven't already:

```bash
npm install -g surge
```

Then, from within your project folder:

```bash
npm run build
surge public
```
