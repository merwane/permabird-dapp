<template>
  <div class="home">
    <div>
      <div class="titleBar">PERMABIRD</div>
      <div class="subTitleBar">Store and search tweets about a given subject on Arweave's Blockchain</div>
    </div>
    <!-- main -->
    <v-layout row wrap>
      <v-flex md3 xl4 xs1></v-flex>
      <v-flex md6 xl4 xs10>
        <br>
        <br>
        <v-text-field
            label="Search a keyword (e.g: Ethereum)"
            solo
            flat
            v-model="searchInput"
            @keydown.enter="searchTweet"
          ></v-text-field>
          <div style="color: whitesmoke; text-align: center;">(The search is case sensitive)</div>
          <div style="text-align: center;">
            <v-btn @click="searchTweet" depressed outline color="white">Search</v-btn>
          </div>
          <br>
          <div
          :hidden="searchBox"
          v-for="tweet in tweets"
          v-bind:key="tweet"
          >
            <div class="tweet">
              <h3>Tweet from: <span style="color: #42A5F5;">@{{tweet.txTagsDic[0]}}</span></h3>
              <h3>Sentiment: <span style="color: #42A5F5;">{{tweet.txTagsDic[2]}}</span></h3>
              <br>
              <h4>{{tweet.txData}}</h4>
              <br>
              <div style="color: grey;"><strong>Archived by:</strong> {{tweet.txOwner}}</div>
            </div>
          </div>
      </v-flex>
      <v-flex md3 xl4 xs1></v-flex>
    </v-layout>
    <v-footer fixed color="transparent">
      <v-spacer></v-spacer>
      <div>
        <strong style="color: white;">&copy;{{ new Date().getFullYear() }} <a style="color: white;" href="https://github.com/merwane/permabird-dapp">MIT</a> – <a href="https://github.com/merwane/permabird" target="_blank">How to store tweets?</a></strong>
      </div>
      <v-spacer></v-spacer>
    </v-footer>
  </div>
</template>

<script>
import Arweave from 'arweave/web';
const arweave = Arweave.init();

export default {
  name: 'Home',
  data() {
    return {
      searchBox: false,
      searchInput: '',
      tweets: []
    }
  },
  methods: {
     async searchTweet(){
       this.tweets = [];
       let $pageData = this;
       const txids = await arweave.arql({
            op: "and",
            expr1: {
              op: 'equals',
            expr1: 'App-Name',
            expr2: 'permabird'
            },
            expr2: {
              op: 'equals',
            expr1: 'keyword',
            expr2: this.searchInput
            }
            });
            // get data
            txids.forEach(tx => {
                arweave.transactions.get(tx).then(async transaction => {
                    var txOwner = await arweave.wallets.ownerToAddress(transaction.get('owner'));
                    var txData = transaction.get('data', {decode: true, string: true});
                    var txTagsDic = []
                    // eslint-disable-next-line
                    var txTags = transaction.get('tags').forEach(tag => {
                        // eslint-disable-next-line
                        var tagName = tag.get('name', {decode: true, string: true});
                        var tagValue = txTagsDic.push(tag.get('value', {decode: true, string: true}))
                        
                        
                    });
                    // store on data
                    $pageData.tweets.push({txOwner, txTagsDic, txData});
                });
            });
     }
  },
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
a, a:visited{
  text-decoration: none;
  color: white;
}
.titleBar{
  text-align: center;
  font-size: 40px;
  color: white;
  letter-spacing: 5px;
  padding-top: 35px;
}
.subTitleBar{
  text-align: center;
  font-size: 20px;
  color: white;
  padding-top: 15px;
}
.tweet{
  background-color: white;
  border-radius: 4px;
  padding: 8px;
  margin-bottom: 10px;
}
</style>
