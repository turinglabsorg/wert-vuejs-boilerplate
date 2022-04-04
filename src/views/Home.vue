<template>
  <div class="home">
    <h2>
      Buy an NFT with <a href="https://wert.io" target="_blank">WERT.IO</a>
    </h2>
    <div v-if="!account" style="padding: 30vh 0">
      First connect your wallet,<br />we need it to be sure you own the
      address<br />where you'll receive the NFT.<br /><br />
      <button @click="connect">CONNECT YOUR WALLET FIRST</button>
    </div>
    <div v-if="account">
      Add a fake mobile number, then use code <b>0000</b> to verify it.<br />
      Use fake credit card number: <b>4007410000000006</b> to complete
      purchase.
    </div>
    <hr v-show="account" />
    <div v-show="account" id="wert-wrapper"></div>
    <hr style="margin-top: 20px" />
    Made with love at <a href="https://yomi.digital" target="_blank">YOMI</a>
  </div>
</template>

<script>
import Web3 from "web3";
import Web3Modal from "web3modal";
import WalletConnectProvider from "@walletconnect/web3-provider";
import WertWidget from "@wert-io/widget-initializer";
import { v4 } from "uuid";
const { signSmartContractData } = require("@wert-io/widget-sc-signer");

export default {
  name: "Home",
  data() {
    return {
      account: "",
    };
  },
  mounted() {
    this.connect();
  },
  methods: {
    async connect() {
      const app = this;
      let providerOptions = {};
      if (process.env.VUE_APP_INFURA_ID !== undefined) {
        providerOptions = {
          walletconnect: {
            package: WalletConnectProvider,
            options: {
              infuraId: process.env.VUE_APP_INFURA_ID,
            },
          },
        };
      }
      // Instantiating Web3Modal
      const web3Modal = new Web3Modal({
        cacheProvider: true,
        providerOptions: providerOptions,
      });
      const provider = await web3Modal.connect();
      app.web3 = await new Web3(provider);
      // Checking if networkId matches
      const accounts = await app.web3.eth.getAccounts();
      if (accounts.length > 0) {
        app.balance = await app.web3.eth.getBalance(accounts[0]);
        app.account = accounts[0];
        app.mountWert();
      }
    },
    async mountWert() {
      const app = this;
      const identifier = v4();
      // Customize your input data matching your contract
      const data = {
        address: app.account,
        commodity: "ETH",
        commodity_amount: process.env.VUE_APP_PURCHASE_AMOUNT,
        pk_id: process.env.VUE_APP_WERT_KEY_ID,
        sc_address: process.env.VUE_APP_SMARTCONTRACT_ADDRESS,
        sc_id: Buffer(identifier).toString("hex"),
        sc_input_data: process.env.VUE_APP_SC_INPUT_DATA,
      };
      const signedData = signSmartContractData(
        data,
        process.env.VUE_APP_PRIVATE_KEY
      );
      console.log("Signed Data:", signedData);
      const options = {
        container_id: "wert-wrapper",
        partner_id: process.env.VUE_APP_WERT_PARTNER_ID,
        address: data.address,
        commodity: "ETH:rinkeby", // Specify here if rinkeby or ropsten
        height: window.innerHeight,
        origin: "https://sandbox.wert.io",
        pk_id: data.pk_id,
        commodity_amount: data.commodity_amount,
        sc_id: Buffer(identifier).toString("hex"),
        sc_address: data.sc_address,
        sc_input_data: data.sc_input_data,
        signature: signedData.signature,
      };
      console.log("Init Wert with options:", options);
      const wertWidget = new WertWidget(options);
      wertWidget.mount();
    },
  },
};
</script>
