<script>
    import {onMount} from 'svelte';
    import {navigate} from 'svelte-routing';
    import bcoinImg from "./images/BIcon.png";
    import ethImg from "./images/bIco2.png";
    import mtcImg from "./images/polyImg.png";
    import {
        Currency,
        generateWallet,
        createAccount,
        getTransactionsByCustomer,
        getAccountById,
        generateDepositAddress,
        getDepositAddressesForAccount, getTransactionsByAccount, getExchangeRate
    } from '@tatumio/tatum';
    import "./css/bootstrap.css"
    import "./css/main.css"
    import { TatumPolygonSDK } from '@tatumio/polygon'
    import { Currency as mtcCurrency } from '@tatumio/api-client'

    export let filter = '';
    let btcAccount, ethAccount, btcAddress, ethAddress, btcExchangeRate, ethExchangeRate, mtcAccount, mtcAddress, mtcExchangeRate;
    let transactions = [];

    function redirect(currency, isSendPage) {
        navigate(isSendPage ? `payment/send/${currency}` : `payment/request/${currency}`)
    }

    async function generateAccounts() {
        const polygonSDK = TatumPolygonSDK({ apiKey: 'd00ddab8-d6b4-4e3b-a57a-794b7ca64b3d' })

        console.log("polygonSDK====>", polygonSDK)

        const { mnemonic, xpub } = await polygonSDK.wallet.generateWallet();

        console.log("mnemonic===>", mnemonic);
        console.log("xpub===>", xpub);

        const btcMnemonic = await generateWallet(Currency.BTC, true);

        console.log("Currency111====>", Currency);
        console.log("btcMnemonic====>", btcMnemonic);

        const ethMnemonic = await generateWallet(Currency.ETH, true);
        
        btcAccount = await createAccount({
            currency: Currency.BTC,
            xpub: btcMnemonic.xpub,
            accountingCurrency: 'USD',
            customer: {externalId: 'samuel.sramko@tatum.io'}
        })
        ethAccount = await createAccount({
            currency: Currency.ETH,
            xpub: ethMnemonic.xpub,
            accountingCurrency: 'USD',
            customer: {externalId: 'samuel.sramko@tatum.io'}
        })
        mtcAccount = await polygonSDK.ledger.account.create({
            currency: mtcCurrency.MATIC,
            xpub,
        })
        
        console.log("btcAccount===>", btcAccount);
        console.log("maticAccount===>", mtcAccount);

        btcAddress = await generateDepositAddress(btcAccount.id);
        ethAddress = await generateDepositAddress(ethAccount.id);
        mtcAddress = await polygonSDK.virtualAccount.depositAddress.create(mtcAccount.id);

        console.log("mtcAddress====>", mtcAddress);

        localStorage.setItem('CUSTOMER_ID', btcAccount.customerId);
        localStorage.setItem('BTC_ACCOUNT_ID', btcAccount.id);
        localStorage.setItem('ETH_ACCOUNT_ID', ethAccount.id);
        localStorage.setItem('MTC_ACCOUNT_ID', mtcAccount.id);
        localStorage.setItem('BTC_WALLET', btcMnemonic.mnemonic);
        localStorage.setItem('ETH_WALLET', ethMnemonic.mnemonic);
        localStorage.setItem('MTC_WALLET', mnemonic);
    }

    onMount(async () => {
        const rates = await Promise.all([
            getExchangeRate(Currency.ETH),
            polygonSDK.getExchangeRate(mtcCurrency.MATIC)]);

            ethExchangeRate = parseFloat(rates[0].value);
            mtcExchangeRate = parseFloat(rates[1].value);

        const customerId = localStorage.getItem('CUSTOMER_ID');
        const btcAccountId = localStorage.getItem('BTC_ACCOUNT_ID');
        const ethAccountId = localStorage.getItem('ETH_ACCOUNT_ID');
        const mtcAccountId = localStorage.getItem('MTC_ACCOUNT_ID');
        if (!customerId) {
            return;
        }
        const all = await Promise.all([
            getAccountById(btcAccountId),
            getAccountById(ethAccountId),
            getDepositAddressesForAccount(btcAccountId),
            getDepositAddressesForAccount(ethAccountId),
            getTransactionsByCustomer({id: customerId})]);
        btcAccount = all[0];
        ethAccount = all[1];
        btcAddress = all[2][0];
        ethAddress = all[3][0];
        transactions = all[4];
    })

    async function getTransactionForAccount(currency) {
        if (filter === currency) {
            transactions = await getTransactionsByCustomer({id: localStorage.getItem('CUSTOMER_ID')});
            filter = '';
            return;
        }

        filter = currency;
        const accountId = currency === Currency.ETH ? localStorage.getItem('ETH_ACCOUNT_ID') : localStorage.getItem('MTC_ACCOUNT_ID');
        transactions = await getTransactionsByAccount({id: accountId});
    }
</script>

<style>

</style>

<div class="row">
    <div class="col-md-6 col-sm-12 col-12 pl-0">
        <h4 class="title">Your Wallets</h4>

        {#if mtcAccount}
            <div class="card cardWalet">
                <div class="card-body cardBodyP20">
                    <div class="row">
                        <div class="col-12">
                            <h5 class="cardSubHeading">
                                MTC balance
                            </h5>
                        </div>
                    </div>
                    <div class="icoRight">
                        <span><img alt="mtc" src="{mtcImg}"></span>
                    </div>
                    <div class="row">
                        <div class="col-12 pl-0">
                            <h4 class="totalBalance mb-0">{mtcAccount.balance.accountBalance} MATIC</h4>
                            <p class="m-0 amountDol">
                                ${parseFloat(mtcAccount.balance.accountBalance) * mtcExchangeRate}</p>
                            <p class="amountDol code pt-3">
                                {mtcAddress ? mtcAddress.address : ''}
                            </p>
                        </div>
                    </div>
                    <div class="row">
                        <div class="col-md-6 col-12 col-sm-12">
                            <button class="btn btn-outlineSR w-sm-100" on:click={() => redirect('MATIC', true)}>Send MATIC
                            </button>
                        </div>
                        <div class="col-md-6 col-12 col-sm-12">
                            <button class="btn btn-outlineSR w-sm-100" on:click={() => redirect('MATIC', false)}>Request
                                MATIC
                            </button>
                        </div>
                    </div>
                </div>
            </div>
        {/if}
        {#if ethAccount}
            <div class="card cardWalet mt-4">
                <div class="card-body cardBodyP20">
                    <div class="row">
                        <div class="col-12">
                            <h5 class="cardSubHeading">
                                ETH balance
                            </h5>
                        </div>
                    </div>
                    <div class="icoRight">
                        <span><img alt="eth" src="{ethImg}"></span>
                    </div>
                    <div class="row">
                        <div class="col-12 pl-0">
                            <h4 class="totalBalance mb-0">{ethAccount.balance.accountBalance} ETH</h4>
                            <p class="m-0 amountDol">
                                ${parseFloat(ethAccount.balance.accountBalance) * ethExchangeRate}</p>
                            <p class="amountDol code pt-3">
                                {ethAddress ? ethAddress.address : ''}
                            </p>
                        </div>
                    </div>
                    <div class="row">
                        <div class="col-md-6 col-12 col-sm-12">
                            <button class="btn btn-outlineSR w-sm-100" on:click={() => redirect('ETH', true)}>Send ETH
                            </button>
                        </div>
                        <div class="col-md-6 col-12 col-sm-12">
                            <button class="btn btn-outlineSR w-sm-100" on:click={() => redirect('ETH', false)}>Request
                                ETH
                            </button>
                        </div>
                    </div>
                </div>
            </div>
        {/if}
        <p class="mt20">
            <a href="/#add" class="atag" on:click={generateAccounts}><i class="fa fa-plus-circle"></i> Add a wallet</a>
        </p>
    </div>
    {#if transactions.length}
        <div class="col-md-6 col-sm-12 col-12 pl-0">
            <h4 class="title">Latest Transactions</h4>
            <div class="btns mb20">
                <button class="btn btn-outline {filter === 'MATIC' ? 'btn-outline-active' : ''}"
                        on:click={() => getTransactionForAccount('MATIC')}>MATIC
                </button>
                <button class="btn btn-outline {filter === 'ETH' ? 'btn-outline-active' : ''}"
                        on:click={() => getTransactionForAccount('ETH')}>ETH
                </button>
            </div>
            <div class="card cardTrans mb30">
                <div class="card-body">
                    {#each transactions as transaction}
                        <div class="row">
                            <div class="col-12">
                                <div class="row">
                                    <div class="col-9 pr5">
                                        <p class="date m-0">
                                            {new Date(transaction.created).toLocaleString()}
                                        </p>
                                        <p class="id m5-6">
                                            {transaction.address || transaction.counterAccountId}
                                        </p>
                                        <p class="m-0 id">
                                            <b>Transaction ID:</b> <span><a
                                                href="https://sepolia.etherscan.io/tx/{transaction.txId}"
                                                target="_blank">{transaction.txId}</a></span>
                                        </p>
                                    </div>
                                    <div class="col-3 pl-0">
                                        <p class="amt mb10">
                                            {transaction.amount} {transaction.currency}
                                        </p>
                                        <p class="amtDol">
                                            ${parseFloat(transaction.marketValue.amount).toFixed(2)}
                                        </p>
                                    </div>
                                </div>
                            </div>
                        </div>
                        <hr/>
                    {/each}
                </div>
            </div>
        </div>
    {/if}
</div>
