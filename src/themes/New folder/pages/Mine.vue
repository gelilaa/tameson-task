<template>
  <div id="app">
    <h5>Chains</h5>
    <label v-for="link in chain" :key="link">
      <input type="checkbox" :value="link.id" v-model="selectedChainIds" @change="onChangeChain">
      <span v-text="link.name" />
    &nbsp;
    </label>
    <h5 v-if="filteredChain.length">
      【Sub Chain】
    </h5>
    <label v-for="chain in filteredChain" :key="chain">
      <input type="checkbox" :value="chain.id" v-model="selectedSubChainIds">
      <span v-text="chain.name" />
    &nbsp;&nbsp;
    </label>
  </div>
</template>

<script>
import { SfButton } from '@storefront-ui/vue';
import Vue from 'vue/dist/vue.js'
new Vue({
  data: {
    chain: [
      { id: 1, name: 'chain 1' },
      { id: 2, name: 'chain 2' },
      { id: 3, name: 'chain 3' }
    ],
    subChain: [
      { 'id': 1, 'name': 'sub chain 1', 'chain_id': 1 },
      { 'id': 2, 'name': 'sub chain 2', 'chain_id': 1 },
      { 'id': 3, 'name': 'sub chain 3', 'chain_id': 1 },
      { 'id': 4, 'name': 'sub chain 4', 'chain_id': 1 },
      { 'id': 5, 'name': 'sub chain 5 City', 'chain_id': 2 },
      { 'id': 6, 'name': 'sub chain 6', 'chain_id': 2 },
      { 'id': 7, 'name': 'sub chain 7', 'chain_id': 3 },
      { 'id': 8, 'name': 'sub chain 8', 'chain_id': 3 }
    ],
    selectedChainIds: [],
    selectedSubChainIds: []
  },
  methods: {
    onChangeChain () {
      this.selectedSubChainIds = [];

      if (!this.selectedChainIds) {
        this.selectedChainIds = [];
      }
    }
  },
  computed: {
    filteredChain () {
      let filteredSubChain = [];

      for (let i = 0; i < this.subChain.length; i++) {
        let subChain = this.subChain[i];

        if (this.selectedChainIds.includes(subChain.chain_id)) {
          let chains = this.chain.find((chain) => {
            return (chain.id == subChain.chain_id);
          });

          let id = chains.id + '-' + subChain.id;
          let name = chains.name + '-' + subChain.name;
          filteredSubChain.push({
            id: id,
            name: name
          });
        }
      }

      return filteredSubChain;
    }
  }
}).$mount('#app');

</script>

<style>
body {
  background: #20262E;
  padding: 20px;
  font-family: Helvetica;
}

#app {
  background: #fff;
  border-radius: 4px;
  padding: 20px;
  transition: all 0.2s;
}

li {
  margin: 8px 0;
}

h2 {
  font-weight: bold;
  margin-bottom: 15px;
}

del {
  color: rgba(0, 0, 0, 0.3);
}

del {
  color: rgba(0, 0, 0, 0.3);
}

</style>
