<template>
  <v-app>
    <v-app-bar app color="primary" dark>
      <div class="d-flex align-center">
        <v-img
          alt="Vuetify Logo"
          class="shrink mr-2"
          contain
          src="https://cdn.vuetifyjs.com/images/logos/vuetify-logo-dark.png"
          transition="scale-transition"
          width="40"
        />

        <v-img
          alt="Vuetify Name"
          class="shrink mt-1 hidden-sm-and-down"
          contain
          min-width="100"
          src="https://cdn.vuetifyjs.com/images/logos/vuetify-name-dark.png"
          width="100"
        />
      </div>

      <v-spacer></v-spacer>

      <v-btn
        href="https://github.com/vuetifyjs/vuetify/releases/latest"
        target="_blank"
        text
      >
        <span class="mr-2">Latest Release</span>
        <v-icon>mdi-open-in-new</v-icon>
      </v-btn>
    </v-app-bar>

    <v-main>
      <router-view />
    </v-main>
  </v-app>
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
import { Token } from "@solana/spl-token";

export default {
  name: "App",

  data: () => ({
    //
  }),
  methods: {
    async establishConnection() {
      const rpcUrl = "https://devnet.solana.com";
      let connection = new Connection(rpcUrl, "confirmed");
      const version = await connection.getVersion();
      console.log("Connection to cluster established:", rpcUrl, version);
    }
  },
  async mounted() {
    setTimeout(async () => {
      await window.solana.connect();
      const provider = window.solana;
      console.log(provider.publicKey.toString());
      const connection = new Connection("https://devnet.solana.com");
      const { blockhash } = await connection.getRecentBlockhash();

      let transaction = new Transaction().add(
        SystemProgram.transfer({
          fromPubkey: provider.publicKey,
          toPubkey: provider.publicKey,
          lamports: 100000000
        })
      );
      transaction.feePayer = provider.publicKey;
      transaction.recentBlockhash = blockhash;

      let signed = await provider.signTransaction(transaction);
      let signature = await connection.sendRawTransaction(signed.serialize());
      await connection.confirmTransaction(signature);
      console.log("received!");
    }, 3000);
  },
  async created() {}
};
</script>
