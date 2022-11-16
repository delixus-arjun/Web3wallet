<script>
    import {Router, Route, Link, link} from "svelte-routing";
    import Dashboard from './Dashboard.svelte';
    import SendPayment from './SendPayment.svelte';
    import RequestPayment from './RequestPayment.svelte';
    import "./css/bootstrap.css"
    import "./css/main.css"

    export let url = '';
</script>

<style>

</style>

<link href="https://fonts.googleapis.com/css2?family=PT+Sans&display=swap" rel="stylesheet">
<link rel="stylesheet" href="https://stackpath.bootstrapcdn.com/font-awesome/4.7.0/css/font-awesome.min.css"
      integrity="sha384-wvfXpqpZZVQGK6TAh5PVlGOfQNHSoD2xbE+QkPxCAFlNEevoEH3Sl0sibVcOQVnN" crossorigin="anonymous">

<Router url="{url}">
    <nav class="navbar navbar-expand-sm">
        <ul class="navbar-nav">
            <li class="nav-item pr-122">
                <div class="imgDiv" style="width: 100px;">
                    <Link to="/"><img alt="logo" src="/cryptoLogo.png" style="width: 100%;"/></Link>
                </div>
            </li>
            <li class="nav-item mt-20 m7">
                <a class="nav-link nl active" href="/" use:link>Overview</a>
            </li>
            <li class="nav-item mt-20 m7">
                <a class="nav-link nl active" href="payment/send/BTC" use:link replace>Payments</a>
            </li>
        </ul>
    </nav>

    <Route path="payment/send/:currency" component="{SendPayment}"/>
    <Route path="payment/request/:currency" component="{RequestPayment}"/>

    <div class="container-fluid containerMain mt-5">
        <Route path="/" component="{Dashboard}"/>
    </div>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/qrcodejs/1.0.0/qrcode.min.js"></script>
    <script src="https://ajax.googleapis.com/ajax/libs/jquery/3.5.1/jquery.min.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/popper.js/1.16.0/umd/popper.min.js"></script>
    <script src="https://maxcdn.bootstrapcdn.com/bootstrap/4.5.0/js/bootstrap.min.js"></script>
</Router>