<template>
    <v-container fluid class="no-padding">
        <v-row dense>
            <v-col sm="12" md="12" lg="12">
                <template v-if="account_info.loading">
                    <v-progress-circular
                        :size="50"
                        color="primary"
                        indeterminate
                    ></v-progress-circular>
                </template>
                <template v-else>
                    <div v-if="account_info.is_exists" align="center" style="margin-bottom:20px;padding:10px;">
                        <small>CURRENT BALANCE</small>
                        <p style="font-size:32pt;margin-bottom:3px;color:#555555;">
                            {{ account_info.balance }}
                            <small style="font-size:9pt;">SOL</small>
                        </p>
                        
                    </div>
                    <div v-else align="center">
                        <v-icon style="font-size:128px;" color="#cecece" icon="mdi-emoticon-sad-outline"></v-icon>
                        <br/>
                        <small>ACCOUNT DOES NOT EXISTS.</small>
                        <br/><br/>
                    </div>
                    <v-text-field density="compact" v-if="account_info.public_key_base58" variant="outlined" id="selected_wallet_address" readonly v-model="account_info.public_key_base58"></v-text-field>
                </template>
            </v-col>
            <v-col sm="12" md="12" lg="12">
                <v-select
                    :items="addresses"
                    density="compact"
                    label="Wallet Address"         
                    variant="outlined"
                    item-title="name"
                    @update:modelValue="walletAddressSelected"
                    return-object
                ></v-select> 
            </v-col>
            <v-col sm="12" md="12" lg="12">
                <v-btn variant="outlined"  @click="showForm" block color="red">CREATE WALLET ADDRESS</v-btn>
                <v-dialog v-model="form.show" width="auto">
                    <v-card width="600">
                        <v-card-item>
                            <v-container fluid>
                                <v-row dense>
                                    <v-col md="12" sm="12" lg="12">
                                        <small>GENERATE WALLET ADDRESS</small>
                                    </v-col>
                                    <v-col md="12" sm="12" lg="12">
                                        <v-text-field v-model="form.name" density="compact" variant="outlined" placeholder="Enter Address Name"></v-text-field>                            
                                    </v-col>
                                    <v-col md="12" sm="12" lg="12" align="right">
                                        <v-btn variant="outlined" @click="generate" color="primary">GENERATE</v-btn>
                                        &nbsp;
                                        <v-btn variant="tonal" color="grey" @click="hideForm" >CANCEL</v-btn>
                                    </v-col>
                                </v-row>
                            </v-container>
                        </v-card-item>
                    </v-card>
                </v-dialog>
            </v-col>
        </v-row>
    </v-container> 
</template>
  
<style>
#selected_wallet_address{
    font-size:9pt;
    background-color: #f4f4f4;
}
</style>
  
<script>
import { RPC_ADDRESS } from '@/config.js';
import { Connection, PublicKey, Keypair, LAMPORTS_PER_SOL } from '@solana/web3.js';

export default {
    mounted(){
        this.loadKeyPairs();
    },
    methods:{
        connect(){
            return new Promise((resolve,reject)=>{

                try {
                    let connection = new Connection(RPC_ADDRESS,{
                        commitment: "confirmed"
                    });
                    resolve(connection);
                } catch (error) {
                    reject(error);
                }
            });
        },
        getAccountInfo(public_key){
            if(this.connection != null){
                this.account_info.loading = true;
                this.connection.getAccountInfo(public_key).then((account)=>{
                this.account_info.public_key_base58 = public_key.toBase58();
                this.account_info.public_key = public_key
                if(account == null){
                    this.account_info.is_exists = false;            
                }
                else{
                    this.account_info.balance = account.lamports / LAMPORTS_PER_SOL;
                    this.account_info.is_exists = true;
                }
                }).catch((error)=>{
                    console.log("Account Info Error: ",error);
                }).finally(()=>{
                    this.account_info.loading = false;
                })
            }
            else {
                alert("Not connected.");
            }
        },
        walletAddressSelected(e){
            let public_key = new PublicKey(e.public_key)
            let private_key = new Uint8Array(e.private_key.split(","));
            this.load(public_key,private_key);
        },
        load(public_key,private_key){

            this.connect().then((connection)=>{
                this.connection = connection;
                console.log("Connected");
                this.$emit("selected",{
                    "connection":connection,
                    "public_key": public_key,
                    "private_key": private_key,
                    "public_key_base58": public_key.toBase58()
                });
                this.getAccountInfo(public_key);
                
            }).catch((error)=>{
                console.error(error);
            })
        },
        generate(){
            const form = this.form;
            const keypair = Keypair.generate();
            const public_key = keypair.publicKey.toBase58();
            const private_key = keypair.secretKey.toString();
            this.storeKeyPair(form.name,public_key,private_key);
            this.hideForm();
        },
        loadKeyPairs(){
            let keypairs = localStorage.getItem("wallet_keypairs");
            if(keypairs != null){
                keypairs = JSON.parse(keypairs);
            }
            else{
                keypairs = [];
            }
            this.addresses = keypairs;
            return keypairs;
        },
        storeKeyPair(name, public_key, private_key){
            let keypairs = this.loadKeyPairs();
            keypairs.push({
                "name": name,
                "public_key": public_key,
                "private_key": private_key,
            })
            localStorage.setItem("wallet_keypairs",JSON.stringify(keypairs));
            return keypairs;
        },
        showForm(){
            this.form.show = {
                show:true,
                name:"",
            };
        },
        hideForm(){
            this.form.show = false;
        }
    },
    data(){
        return {
            addresses:[],
            loading:false,
            account_info:{
                public_key:null,
                public_key_base58:null,
                private_key: null,
                balance:0,
                is_exists: false,
                loading:false,
            },
            connection:null,
            form:{
                show:false,
                name:"",
            }
        }
    }
};
</script>