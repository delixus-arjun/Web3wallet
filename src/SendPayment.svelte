<script>
    import {link} from "svelte-routing";
    import {
        Currency, generatePrivateKeyFromMnemonic,
        getDepositAddressesForAccount,
        sendBitcoinOffchainTransaction,
        sendEthOffchainTransaction
    } from '@tatumio/tatum';

    export let currency = Currency.BTC;

    let txId = '';

    async function swapCurrency(c) {
        currency = c;
    }

    function isContinueValid() {
        return document.getElementById('address').value && document.getElementById('amount').value;
    }

    async function sendTransaction() {
        const recipient = document.getElementById('address').value;
        const amount = document.getElementById('amount').value;

        const senderAccountId = localStorage.getItem(`${currency}_ACCOUNT_ID`);
        const mnemonic = localStorage.getItem(`${currency}_WALLET`).mnemonic;

        const {address, derivationKey} = (await getDepositAddressesForAccount(senderAccountId))[0];
        const privateKey = await generatePrivateKeyFromMnemonic(currency, true, mnemonic, derivationKey);
        try {
            switch (currency) {
                case Currency.BTC:
                    txId = (await sendBitcoinOffchainTransaction(true, {
                        senderAccountId,
                        keyPair: [{address, privateKey: privateKey}],
                        address: recipient,
                        amount
                    })).txId;
                    break;
                case Currency.ETH:
                    txId = (await sendEthOffchainTransaction(true, {
                        senderAccountId,
                        privateKey,
                        address: recipient,
                        amount
                    })).txId;
                    break;
            }
        } catch (e) {
            console.error(e);
        }
    }
</script>

<style>

</style>
<nav class="navbar navbar-expand-sm navBar2 pb-0">
    <ul class="navbar-nav">
        <li class="nav-item pl-122">
            <a class="nav-link nl2 subNavActive" href="/payment/send/{currency}" use:link replace>Send</a>
        </li>
        <li class="nav-item m7 pl65">
            <a class="nav-link nl2" href="/payment/request/{currency}" use:link replace>Request</a>
        </li>
    </ul>
</nav>

<div class="container-fluid containerMain mt-5">
    <div class="row">
        <div class="col-md-8 col-sm-12 col-12 pl-0">
            <div class="card sendPaymentCard">
                <div class="card-body cardp">
                    <div class="row">
                        <div class="col-12">
                            <h5 class="cardSubHeading">
                                Choose Account
                            </h5>
                            <div class="btns btnPay mb30-mt25">
                                <button id="btnBtc"
                                        on:click={() => swapCurrency('BTC')}
                                        class="btn btn-outline ml-0 {currency === 'BTC' ? 'btnActive' : ''}">BTC
                                </button>
                                <button id="btnEth"
                                        on:click={() => swapCurrency('ETH')}
                                        class="btn btn-outline {currency === 'ETH' ? 'btnActive' : ''}">
                                    ETH
                                </button>
                            </div>
                            <h5 class="cardSubHeading">
                                Send Payment To
                            </h5>
                        </div>
                    </div>

                    <div class="row">
                        <div class="col-12 mtb10">
                            <input class="inputStyle" id="address" placeholder="Blockchain address"/>
                        </div>
                        <div class="col-12 m5020">
                            <input class="inputStyle" id="amount" placeholder="Amount in {currency}"/>
                        </div>
                    </div>
                    <div class="row">
                        <div class="col-12 col-sm-12 mt-2">
                            <button class="btn btn-sub btnActive" on:click={sendTransaction}>Continue</button>
                            <p class="reqTxt mt-4">
                                {#if txId}
                                    TransactionId:<br/>
                                    <a href="https://ropsten.etherscan.io/tx/{txId}" target="_blank">{txId}</a>
                                {/if}
                            </p>
                        </div>
                    </div>
                </div>
            </div>
        </div>
        <div class="col-md-4 col-sm-12 col-12 pl-0">

        </div>
    </div>
</div>
