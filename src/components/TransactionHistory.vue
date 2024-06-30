<template>
    <v-container fluid id="transaction-history">
        <v-row dense>
            <v-col sm="12" md="12" lg="12">                
                <template v-if="loading">
                    <div align="center">
                        <v-progress-circular
                        :size="50"
                        color="primary"
                        indeterminate
                        ></v-progress-circular>
                    </div>
                </template>
                <template v-else-if="transactions.length > 0 && !loading">                
                    <small>TRANSACTION HISTORY</small>
                    <v-list class="no-padding">
                        <v-list-item
                            v-for="item in transactions"
                            :key="item.signature"
                            class="no-padding"
                        >
                            <template v-slot:title>
                                <span>{{ item.signature }}</span>
                            </template>
                            <template v-slot:subtitle>
                                <small>BLOCKTIME: {{ item.blockTime }}</small>
                            </template>
                            <template v-slot:prepend>
                                <v-icon size="x-small" color="red">mdi-connection</v-icon>
                            </template>
                        </v-list-item>
                    </v-list>
                </template>
                <template v-else>
                    <div align="center">
                        <small style="color:#cecece;">NO TRANSACTIONS</small>
                    </div>
                </template>
            </v-col>
        </v-row>
    </v-container>
</template>
  
<style>
#transaction-history{
    border:1px solid #eaeaea;
    border-top:5px solid #eaeaea;
    margin-top:20px;
    
}
</style>
  
<script>
  import { LAMPORTS_PER_SOL,  Transaction, SystemProgram, sendAndConfirmTransaction, PublicKey, Keypair  } from '@solana/web3.js';
  
  export default {
    mounted(){
    },
    methods:{        
        getTransactionHistory(connection,public_key){
            this.transactions = [];
            this.loading = true;
            connection.getSignaturesForAddress(public_key).then((transactions)=>{
                console.log("getTransactionHistory",transactions);
                this.transactions = transactions;
            }).catch((error)=>{
                console.log("getTransactionHistory:error",error);
            }).finally(()=>{                
                this.loading = false;
            });
        },
    },
    data(){
        return {
            transactions:[],
            loading:false,
        }
    }
  };
  </script>