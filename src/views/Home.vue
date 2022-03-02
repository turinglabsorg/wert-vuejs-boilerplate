<template>
  <div class="home">
    <div id="wert-wrapper"></div>
  </div>
</template>

<script>
import WertWidget from "@wert-io/widget-initializer";
import { uuid } from "uuidv4";
const { signSmartContractData } = require('@wert-io/widget-sc-signer');

export default {
  name: "Home",
  mounted() {
    const identifier = uuid();
    // Customize your input data matching your contract
    const data = {
      address: process.env.VUE_APP_RECEIVER_ADDRESS,
      commodity: "ETH",
      commodity_amount: process.env.VUE_APP_PURCHASE_AMOUNT,
      pk_id: "key1",
      sc_address: process.env.VUE_APP_SMARTCONTRACT_ADDRESS,
      sc_id: Buffer(identifier).toString("hex"),
      sc_input_data: "0xcef220a300000000000000000000000000000000000000000000000000000000000000200000000000000000000000000000000000000000000000000000000000000000",
    };
    const signedData = signSmartContractData(data, process.env.VUE_APP_PRIVATE_KEY);
    console.log('Signed Data:', signedData)
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
};
</script>
