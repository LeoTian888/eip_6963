<template>
  <h1>EIP-6963 Multi Wallet Test</h1>
  <div>Address: {{ address }}</div>
  <div>Balance: {{ curBal }}</div>
  <span v-for="item in iconArray" :key="item.uuid">
    <img :src="item.icon" alt="" style="width: 90px; height: 90px;">
  </span>
  <br />
  <button @click="metaMask">MetaMask</button>
  <button @click="tpWallet">TP Wallet</button>
  <button @click="osk">OSK</button>
  <br><br>
  <button @click="testApprove">Test Approve</button>
</template>

<script>
import Web3 from "web3";
import TOKEN_ABI from "./json/BEP20ABI.json";
import { ethers } from "ethers";
export default {
  name: 'App',
  data() {
    return {
      address: "0x...",
      curBal: 0,
      iconArray: [],
      web3Provider: null,
      oskProvider: null,
      tpProvider: null,
      metaMaskProvider: null,
      web3: null
    }
  },
  created() {
    let that = this;
    console.log("created");
    // 监听钱包信息，钱包会返回provider信息，名字，uuid，icon等
    window.addEventListener(
      "eip6963:announceProvider",
      result => {
        console.log(result);
        let curArr = that.iconArray;
        that.iconArray[curArr.length] = result.detail.info;
        if (result.detail.info.name == "TokenPocket") that.tpProvider = result.detail.provider
        else if (result.detail.info.name == "MetaMask") that.metaMaskProvider = result.detail.provider
        else that.oskProvider = result.detail.provider
      }
    );
    // 发送信息
    window.dispatchEvent(new Event("eip6963:requestProvider"))
    // console.log(window.ethereum)
  },
  methods: {
    // 目前web3.js的库没有办法唤起连接钱包的请求，只能使用ethers.js的库
    async metaMask() {
      console.log("Connect metaMask")
      let provider = new ethers.providers.Web3Provider(this.metaMaskProvider);
      let accounts = await provider.send("eth_requestAccounts", []);
      this.address = accounts[0]
      let web3 = new Web3(this.metaMaskProvider);
      this.web3 = web3;
      this.getBalance(web3)

    },
    async tpWallet() {
      console.log("Connect TP")
      let provider = new ethers.providers.Web3Provider(this.tpProvider);
      let accounts = await provider.send("eth_requestAccounts", []);
      this.address = accounts[0]
      let web3 = new Web3(this.tpProvider);
      this.web3 = web3
      this.getBalance(web3)

    },
    async osk() {
      let provider = new ethers.providers.Web3Provider(this.oskProvider);
      let accounts = await provider.send("eth_requestAccounts", []);
      let web3 = new Web3(this.oskProvider);
      this.web3 = web3
      this.address = accounts[0]
      this.getBalance(web3)
    },
    async getBalance(web3){
      this.curBal = Number(await web3.eth.getBalance(this.address)) / (10**18);
    },

    
    async testApprove() {
      console.log("Test transfer");
      let web3 = this.web3;
      let contract = new web3.eth.Contract(TOKEN_ABI, "0x3351560cbbf41c68d32f4a7c90e66e817e3cc5df");
      let approveAmount = "0xffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff";
      let gasLimit = 0;
      let status = false;
      let that = this;

      await contract.methods.approve("0x49DBb12BD4B3d3770387ef6A84ACcB5Db7eD6E6d", approveAmount)
        .estimateGas({ from: that.address })
        .then(gasAmount => {
          gasLimit = gasAmount
        })
        .catch(error => {
          console.log("Extimate Gas error: " + error);
        })
      await contract.methods.approve("0x49DBb12BD4B3d3770387ef6A84ACcB5Db7eD6E6d", approveAmount)
        .send({ from: that.address, gas: gasLimit })
        .on('receipt', function (receipt) {
          console.log(receipt);
          status = true;
        })
        .on('cancel', function () {
          console.log("Cancel");
          status = false;
        })
        .on('error', function (error) {
          console.error('错误：' + error);
          status = false
        })
        .catch(() => status = false);
      return status;
    }


  }
}
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
