<template>
  <div class="home">
    <div id="wert-wrapper"></div>
  </div>
</template>

<script>
import WertWidget from "@wert-io/widget-initializer";
import { uuid } from "uuidv4";
import { ethers } from "ethers";
const EdDSA = require("elliptic").eddsa;

export default {
  name: "Home",
  mounted() {
    const identifier = uuid();
    const coder = new ethers.utils.AbiCoder();
    const inputData = coder.encode(["bytes32[]"], [[]]);
    const ellipticEdDSA = new EdDSA("ed25519");
    const ellipticKey = ellipticEdDSA.keyFromSecret(process.env.VUE_APP_PRIVATE_KEY);
    const data = {
      address: process.env.VUE_APP_RECEIVER_ADDRESS,
      commodity: "ETH:rinkeby",
      commodity_amount: process.env.VUE_APP_PURCHASE_AMOUNT,
      pk_id: "key1",
      sc_address: process.env.VUE_APP_SMARTCONTRACT_ADDRESS,
      sc_id: Buffer(identifier).toString("hex"),
      sc_input_data: inputData,
    };

    const dataString = Object.keys(data)
      .sort()
      .map((key) => `${key}:${data[key]}`)
      .join("\n");
    console.log("Data string:", dataString);
    const hash = Buffer.from(dataString, "utf8").toString("hex");
    const signature = ellipticKey.sign(hash).toHex();

    const options = {
      container_id: "wert-wrapper",
      partner_id: process.env.VUE_APP_WERT_PARTNER_ID,
      address: data.address,
      commodity: data.commodity,
      height: window.innerHeight,
      origin: "https://sandbox.wert.io",
      commodity_amount: data.commodity_amount,
      sc_id: Buffer(identifier).toString("hex"),
      sc_address: data.sc_address,
      sc_input_data: data.sc_input_data,
      signature: signature,
    };
    console.log("Init Wert with options:", options);
    const wertWidget = new WertWidget(options);
    wertWidget.mount();
  },
};
</script>
