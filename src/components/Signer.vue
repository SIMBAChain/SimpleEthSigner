<template>
  <div class="hello">
    <h1>Simple Signer</h1>

    <div class="signer">
      <div class="mnemonic">
        Mnemonic:
        <input type="text" v-model="mnemonic"/>
      </div>
      <div class="transaction">
        Unsigned transaction:
        <textarea v-model="unsigned" rows="10"></textarea>
      </div>
      <div class="go">
        <button v-on:click="sign()"> Sign</button>
      </div>
      <div class="signed">
        <textarea v-model="signed" readonly rows="10"></textarea>
      </div>
      <div v-if="error" class="error">
        Error:
        <textarea v-model="error" readonly rows="10"></textarea>
      </div>
    </div>
  </div>
</template>

<script>
import {Wallet as EthersWallet} from '@ethersproject/wallet';
import {mnemonicToSeed} from 'bip39';
import {hdkey} from 'ethereumjs-wallet';

export default {
  name: 'Signer',
  data() {
    return {
      unsigned: '',
      mnemonic: localStorage.getItem('mnemonic'),
      signed: '',
      error: '',
    };
  },
  methods: {
    sign() {
      try {
        this.error = null;
        let payload = {};
        try {
          payload = JSON.parse(this.unsigned);

        } catch (e) {
          // Do nothing
          this.error = e;
          return;
        }

        const allowedKeys = ['to', 'nonce', 'gasLimit', 'gasPrice', 'data', 'value', 'chainId'];

        let cleanedPayload = {};

        Object.keys(payload).forEach(key => {
          if (allowedKeys.indexOf(key) >= 0) {
            cleanedPayload[key] = payload[key];
            if ((typeof payload[key] === 'string' || payload[key] instanceof String) &&
                payload[key].startsWith('0x') &&
                payload[key].length % 2 !== 0) {
              cleanedPayload[key] = payload[key].replace('0x', '0x0');
              console.log(`Bad Hex - txn.${key} = ${payload[key]}, reformatted to ${cleanedPayload[key]}`);
            }
          }
        });

        mnemonicToSeed(this.mnemonic).then(async seed => {
          const hdwallet = hdkey.fromMasterSeed(seed);
          const path = "m/44'/60'/0'/0/0";
          const wallet = hdwallet.derivePath(path).getWallet();
          const address = `0x${wallet.getAddress().toString('hex')}`;

          console.log(`Address: ${address}`);

          this.wallet = new EthersWallet(wallet.privKey);

          try {
            this.signed = await this.wallet.signTransaction(cleanedPayload)
          } catch (e) {
            this.error = e;
            return;
          }

          localStorage.setItem('mnemonic', this.mnemonic);
        });
      } catch (e) {
        this.error = e;
      }
    }
  }
  // watch: {
  //   unsigned(newU){
  //     try{
  //       let data = JSON.parse(newU);
  //
  //     }catch(_){
  //       // Do nothing
  //     }
  //   }
  // }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped lang="scss">
h3 {
  margin: 40px 0 0;
}

ul {
  list-style-type: none;
  padding: 0;
}

li {
  display: inline-block;
  margin: 0 10px;
}

a {
  color: #42b983;
}

.signer {
  width: 100%;
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;

  .mnemonic {
    width: 75vw;

    input {
      width: 100%;
    }
  }

  .transaction {
    width: 75vw;

    textarea {
      width: 100%;
    }
  }

  .go {
    width: 75vw;
    padding-bottom: 5px;
  }

  .signed {
    width: 75vw;

    textarea {
      width: 100%;
    }
  }

  .error {
    width: 75vw;

    textarea {
      width: 100%;
    }
  }
}
</style>
