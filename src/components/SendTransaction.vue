<template>
  <div style="width:100%">
    <v-btn block variant="outlined" @click="showForm"  color="primary">SEND</v-btn>
    <v-dialog v-model="show_form" width="auto">
        <v-card width="600">
            <v-card-item>
                <v-container fluid>
                    <v-row dense>
                        <v-col md="12" sm="12" lg="12">
                            <small>SEND TRANSACTION</small>
                        </v-col>
                        <v-col md="12" sm="12" lg="12">
                            <v-text-field label="Sender Wallet Address" readonly v-model="public_key_base58" density="compact" variant="outlined" placeholder="Enter Receive Address"></v-text-field>                            
                        </v-col>
                        <v-col md="12" sm="12" lg="12">
                            <v-text-field label="Receiver Wallet Address" v-model="receiver_public_key" density="compact" variant="outlined" placeholder="Enter Receive Address"></v-text-field>                            
                        </v-col>
                        <v-col md="12" sm="12" lg="12" align="right">
                            <v-text-field width="150" type="number" min="0" suffix="SOL" style="text-align:right !important;" v-model="amount" density="compact" variant="outlined" placeholder="0"></v-text-field>
                        </v-col>
                        <v-col md="12" sm="12" lg="12" align="right">
                            <v-divider></v-divider>
                            <br/>
                            <v-btn :loading="loading" :disabled="amount==0 && receiver_public_key == null " block variant="outlined"  color="primary" @click="sendAmount">SEND {{ amount }} SOL</v-btn>
                            <div style="height:10px;"></div>
                            <v-btn :disabled="loading" variant="tonal" block color="grey" @click="hideForm" >CANCEL</v-btn>
                        </v-col>
                    </v-row>
                </v-container>
            </v-card-item>
        </v-card>
    </v-dialog>
  </div>
</template>

<style>
#selected_wallet_address{
  font-size:9pt;
}
</style>

<script>
import { LAMPORTS_PER_SOL,  Transaction, SystemProgram, sendAndConfirmTransaction, PublicKey, Keypair  } from '@solana/web3.js';

export default {
  mounted(){
  },
  methods:{
      sendAmount(){
        let transaction = new Transaction();
        let connection = this.connection;
        let amount = this.amount;
        let sender_public_key = this.public_key;
        let sender_private_key = this.private_key;
        let receiver_public_key = new PublicKey(this.receiver_public_key);
        let signer = Keypair.fromSecretKey(sender_private_key);
        transaction.add(
          SystemProgram.transfer({
            fromPubkey: sender_public_key,
            toPubkey: receiver_public_key,
            lamports: amount * LAMPORTS_PER_SOL
          })
        )
        this.loading = true;
        sendAndConfirmTransaction(connection,transaction,[signer]).then((signature)=>{
          this.$emit("succeed");
          this.hideForm();
          alert("Transaction succeed.");
        }).catch((error)=>{
          console.error("sendAndConfirmTransaction",error);
          alert("Transaction failed.")
        }).finally(()=>{        
          this.loading = false;
        })

      },
      setSender(connection, private_key, public_key, public_key_base58){
        this.private_key = private_key;
        this.public_key = public_key;
        this.public_key_base58 = public_key_base58;
        this.connection = connection;
      },
      showForm(){
          this.show_form = true;
      },
      hideForm(){
          this.show_form = false;
          this.receiver_public_key = null;
      }
  },
  data(){
      return {
        receiver_public_key:null,
        amount:0,
        connection:null,
        private_key:null,
        public_key:null,
        public_key_base58:null,
        show_form:false,
        loading:false,
      }
  }
};
</script>