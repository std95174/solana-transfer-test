<template>
  <v-container>
    <v-row class="text-center">
      <v-col cols="12">
        <v-img
          :src="require('../assets/logo.svg')"
          class="my-3"
          contain
          height="200"
        />
      </v-col>
      <v-row>
        <v-spacer>Your PublicKey: {{ publicKey }}</v-spacer>
      </v-row>
      <v-text-field v-model="amount"></v-text-field>
      <v-btn @click="transferSOL">Transfer SOL</v-btn>
      <v-btn @click="getProvider">Connect to phantom</v-btn>
    </v-row>
  </v-container>
</template>

<script>
import {
  Account,
  Connection,
  PublicKey,
  LAMPORTS_PER_SOL,
  SystemProgram,
  TransactionInstruction,
  Transaction,
  sendAndConfirmTransaction
} from "@solana/web3.js";
import { Token, TOKEN_PROGRAM_ID } from "@solana/spl-token";

const rpcUrl = "https://devnet.solana.com";
let connection = new Connection(rpcUrl, "confirmed");
let provider;

export default {
  name: "HelloWorld",

  data: () => ({
    publicKey: "",
    amount: 0
  }),
  methods: {
    async transferSOL() {
      const { blockhash } = await connection.getRecentBlockhash();
      let transaction = new Transaction().add(
        SystemProgram.transfer({
          fromPubkey: provider.publicKey,
          toPubkey: provider.publicKey,
          lamports: this.amount * LAMPORTS_PER_SOL
        })
      );
      transaction.feePayer = provider.publicKey;
      transaction.recentBlockhash = blockhash;
      let signed = await provider.signTransaction(transaction);
      let signature = await connection.sendRawTransaction(signed.serialize());
      await connection.confirmTransaction(signature);
      console.log("received!");
    },
    async getProvider() {
      if (window.solana) {
        console.log("solana is there");
        provider = window.solana;
        const vm = this;
        if (provider.isPhantom) {
          provider.on("connect", () => {
            console.log("Connected to wallet " + provider.publicKey.toBase58());
            vm.publicKey = provider.publicKey.toBase58();
          });
          provider.on("disconnect", () => {
            console.log("Disconnected from wallet");
          });
          // try to eagerly connect
          provider.connect(/**{ onlyIfTrusted: true }**/);
          return () => {
            provider.disconnect();
          };
        }
      }
    }
  },
  async mounted() {
    const version = await connection.getVersion();
    console.log("Connection to cluster established:", rpcUrl, version);
    // const vm = this;

    // setTimeout(async () => {
    //   window.solana.connect();
    //   window.solana.on("connect", () => {
    //     console.log("connected!");

    //     provider = window.solana;
    //     console.log(provider);
    //     vm.publicKey = provider.publicKey.toString();
    //   });
    // }, 1000);
  }
};
</script>
