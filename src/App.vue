<template>
  <v-container fluid id="wallet">
    <v-row>
      <v-col sm="12" md="12" lg="12" align="left">        
        <v-avatar
          rounded="0"
          size="28"
        >
          <v-img :src="icon" cover></v-img>
        </v-avatar>
        &nbsp;
        <small style="color:#cecece;">SOLANA WALLET</small>
      </v-col>
      <v-col sm="12" md="12" lg="12">
        <WalletAddress ref="walletAddress" @selected="onWalletAddressSelected"/>
      </v-col>
      <template v-if="connection">
        <v-col sm="12" md="12" lg="12">
          <SendTransaction ref="sendTransaction" @succeed="onTransactionSucceed"/>
        </v-col>
        <v-col sm="12" md="12" lg="12">
          <TransactionHistory ref="transactionHistory" />
        </v-col>
      </template>
    </v-row>
  </v-container>
</template>

<style scoped>
#wallet{
  border:1px solid #cecece;
  width:500px;
  background-color: #fff;
  margin:5% auto;
  padding:50px 30px;
  border-radius: 6px;
}
</style>

<script>
import solana_icon from '@/assets/solana-icon.png';
import WalletAddress from '@/components/WalletAddress.vue';
import SendTransaction from '@/components/SendTransaction.vue';
import TransactionHistory from '@/components/TransactionHistory.vue';


import { Connection, Keypair, PublicKey, LAMPORTS_PER_SOL, Transaction, SystemProgram, sendAndConfirmTransaction } from '@solana/web3.js';
import { airdropIfRequired } from '@solana-developers/helpers';

export default{
  components:{
    WalletAddress,
    SendTransaction,
    TransactionHistory
  },
  mounted(){
  },
  methods: {
    onWalletAddressSelected(data){
      this.connection = data.connection;
      this.private_key = data.private_key;
      this.public_key = data.public_key;
      this.$nextTick(()=>{
        this.$refs.sendTransaction.setSender(data.connection,data.private_key,data.public_key,data.public_key_base58);
        this.$refs.transactionHistory.getTransactionHistory(data.connection,data.public_key);
      });
    },
    onTransactionSucceed(){
      
      this.$nextTick(()=>{
        this.$refs.walletAddress.load(this.public_key,this.private_key);
        this.$refs.transactionHistory.getTransactionHistory(this.connection,this.public_key);
      });
    }
  },
  data(){
    return {
      connection:null,
      private_key:null,
      public_key:null,
      icon: solana_icon,
    }
  }
};
</script>
