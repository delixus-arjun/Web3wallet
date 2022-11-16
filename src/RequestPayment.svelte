<script>
    import {onMount} from 'svelte'
    import {link, navigate} from 'svelte-routing'
    import {Currency, getDepositAddressesForAccount} from "@tatumio/tatum";

    export let currency = Currency.BTC;
    let address = '';
    let copied = false;

    async function swapCurrency(c) {
        currency = c;
        await getAddress();
    }

    function copyToClipboard() {
        if (window.clipboardData && window.clipboardData.setData) {
            // Internet Explorer-specific code path to prevent textarea being shown while dialog is visible.
            return window.clipboardData.setData("Text", address);

        } else if (document.queryCommandSupported && document.queryCommandSupported("copy")) {
            var textarea = document.createElement("textarea");
            textarea.textContent = address;
            textarea.style.position = "fixed";  // Prevent scrolling to bottom of page in Microsoft Edge.
            document.body.appendChild(textarea);
            textarea.select();
            try {
                document.execCommand("copy");  // Security exception may be thrown by some browsers.
                copied = true;
                setTimeout(() => {
                    copied = false
                }, 1500);
            } catch (ex) {
                console.warn("Copy to clipboard failed.", ex);
                return false;
            } finally {
                document.body.removeChild(textarea);
            }
        }
    }

    async function getAddress() {
        const accountId = localStorage.getItem(currency === Currency.BTC ? 'BTC_ACCOUNT_ID' : 'ETH_ACCOUNT_ID');
        address = (await getDepositAddressesForAccount(accountId))[0].address;
        document.getElementById('qrCode').innerHTML = '';
        new window.QRCode('qrCode', {text: `${currency}:${address}`, width: 250, height: 250})
    }

    onMount(getAddress);
</script>

<style>

</style>
<nav class="navbar navbar-expand-sm navBar2 pb-0">
    <ul class="navbar-nav">
        <li class="nav-item pl-122">
            <a class="nav-link nl2" href="/payment/send/{currency}" use:link replace>Send</a>
        </li>
        <li class="nav-item m7 pl65">
            <a class="nav-link nl2 subNavActive" href="/payment/request/{currency}" use:link replace>Request</a>
        </li>
    </ul>
</nav>

<div class="container-fluid containerMain mt-5">
    <div class="row">
        <div class="col-md-8 col-sm-12 col-12 pl-0">
            <div class="card sendPaymentCard mb30">
                <div class="card-body cardp">
                    <div class="row">
                        <div class="col-12">
                            <h5 class="cardSubHeading">
                                Choose Account
                            </h5>
                            <div class="btns btnPay mb30-mt25">
                                <button
                                        on:click={() => swapCurrency('BTC')}
                                        id="btnBtc"
                                        class="btn btn-outline ml-0 {currency === 'BTC' ? 'btnActive' : ''}">BTC
                                </button>
                                <button
                                        on:click={() => swapCurrency('ETH')}
                                        id="btnEth" class="btn btn-outline {currency === 'ETH' ? 'btnActive' : ''}">
                                    ETH
                                </button>
                            </div>
                            <h5 class="cardSubHeading">
                                Your wallet address
                            </h5>
                        </div>
                    </div>

                    <div class="row">
                        <div class="col-12 mtb10">
                            <p class="reqTxt">
                                {#if copied}
                                    Address {address} copied!<br/><br/>
                                {/if}
                                Click or tap on QR to copy your wallet string to clipboard:
                            </p>
                            <p class="reqTxt txtcolr" on:click={copyToClipboard}>
                                {address}
                            </p>
                        </div>
                        <div class="row">
                            <div class="col-12 col-sm-12 mt-2">
                                <div class="qrImgDiv" on:click={copyToClipboard}>
                                    <div id="qrCode" class="qrImg"></div>
                                </div>
                            </div>
                        </div>
                    </div>
                </div>
            </div>
            <div class="col-md-4 col-sm-12 col-12 pl-0">

            </div>
        </div>
    </div>
</div>