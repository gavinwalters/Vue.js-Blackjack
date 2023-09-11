<script>
import { objectToString, parseStringStyle } from "@vue/shared";
import Card from "./components/cards.vue";
export default {
  components: {
    Card,
  },
  data() {
    return {
      ranks: ["A", "2", "3", "4", "5", "6", "7", "8", "9", "10", "J", "Q", "K"],
      suits: ["♥", "♦", "♣", "♠"],
      cards: [],
      player: [],
      dealer: [],
      suitColour: {
        "♥": "red",
        "♦": "red",
        "♣": "black",
        "♠": "black",
      },
      player_total: 0,
      dealer_total: 0,
      bust: false,
      isDisabled: false,
    };
  },
  created() {
    this.displayInitialDeck();
  },
  methods: {
    displayInitialDeck() {
      let id = 1;
      for (let s = 0; s < 4; s++) {
        for (let r = 0; r < 13; r++) {
          let card = {
            id: id,
            rank: this.ranks[r],
            suit: this.suits[s],
          };
          this.cards.push(card);
          id++;
        }
      }
    },
    shuffleDeck() {
      // alert("Shuffle");
      for (let i = this.cards.length - 1; i > 0; i--) {
        let randomIndex = Math.floor(Math.random() * i);
        let temp = this.cards[i];
        this.cards[i] = this.cards[randomIndex];
        this.cards[randomIndex] = temp;
      }
    },
    deal() {
      this.player.push(this.cards.shift());
      this.player.push(this.cards.shift());
      this.dealer.push(this.cards.shift());
      this.dealer.push(this.cards.shift());
      this.disableButton();
      this.calculateScore();
    },
    hit() {
      this.player.push(this.cards.shift());
      this.calculateScore();
      this.isBust(this.player_total);
      if (this.bust) {
        alert("BUST - DEALER WIN");
      }
    },
    dealerHit() {
      this.dealer.push(this.cards.shift());
      this.calculateScore();
      this.isBust(this.dealer_total);
      if (this.bust) {
        alert("BUST - PLAYER WIN");
      }
    },
    calculateScore() {
      let p_score = 0;
      let d_score = 0;
      let player_total = this.player_total;
      let dealer_total = this.dealer_total;
      this.player.forEach(function (card) {
        // console.log(card.rank);

        if (card.rank == "K" || card.rank == "Q" || card.rank == "J") {
          p_score += 10;
          // alert("Face");
        } else if (card.rank == "A" && player_total < 11) {
          p_score += 11;
          // alert("Ace High");
        } else if (card.rank == "A" && player_total > 10) {
          p_score += 1;
          // alert("Ace low");
        } else {
          p_score += parseInt(card.rank);
          // alert("number");
        }
      });
      this.dealer.forEach(function (card) {
        // console.log(card.rank);

        if (card.rank == "K" || card.rank == "Q" || card.rank == "J") {
          d_score += 10;
          // alert("Face");
        } else if (card.rank == "A" && dealer_total < 11) {
          d_score += 11;
          // alert("Ace High");
        } else if (card.rank == "A" && dealer_total > 10) {
          d_score += 1;
          // alert("Ace low");
        } else {
          d_score += parseInt(card.rank);
          // alert("number");
        }
      });
      // alert(score);
      this.player_total = p_score;
      this.dealer_total = d_score;

      if (this.player_total === 21) {
        alert("PLAYER BLACKJACK");
      }
      if (this.dealer_total === 21) {
        alert("DEALER BLACKJACK");
      }
    },
    isBust(score) {
      if (score > 21) {
        this.calculateScore();
        if (this.player_total > 21 || this.dealer_total > 21) {
          this.bust = true;
        }
      }
    },
    stand() {
      while (this.dealer_total <= 16) {
        this.dealerHit();
        this.calculateScore();
      }
      while (!this.bust) {
        if (this.dealer_total > this.player_total) {
          alert("DEALER WINS");
        } else if (this.player_total == this.dealer_total) {
          alert("DRAW");
        } else {
          alert("PLAYER WINS");
        }
        document.querySelectorAll(".button").forEach((button) => {
          button.disabled = true;
        });

        this.$refs.dealer_score.innerHTML = "Dealer - " + this.dealer_total;
        // this.$refs.first[0].style.display = "none";
        this.$refs.first[0].$el.classList.replace(
          "flipped",
          this.suitColour[this.dealer[0].suit]
        );
        break;
      }
    },
    reloadPage() {
      window.location.reload();
    },
    disableButton() {
      this.isDisabled = true;
    },
  },
};
</script>

<template>
  <button @click="shuffleDeck" class="button">Shuffle</button>
  <button @click="deal" :disabled="isDisabled">Deal</button>
  <button @click="hit" class="button">Hit</button>
  <button @click="stand" class="button">Stand</button>
  <button @click="reloadPage">Play Again</button>
  <div id="player">
    <p>Player - {{ player_total }}</p>
    <div v-for="card in player" :key="card.id">
      <Card
        :card-rank="card.rank"
        :card-suit="card.suit"
        :class="suitColour[card.suit]"
      />
    </div>
  </div>
  <div id="dealer">
    <p ref="dealer_score">Dealer</p>
    <div v-for="card in dealer" :key="card.id">
      <Card
        v-if="dealer.indexOf(card) === 0"
        ref="first"
        :card-rank="card.rank"
        :card-suit="card.suit"
        :class="(suitColour[card.suit], { flipped: !isFlipped })"
      />
      <Card
        v-else
        :card-rank="card.rank"
        :card-suit="card.suit"
        :class="suitColour[card.suit]"
      />
      <!-- <p>{{ dealer.indexOf(card) }} = {{ card.rank }}</p> -->
    </div>
  </div>
  <div id="deck">
    <div v-for="card in cards" :key="card.id">
      <Card
        :card-rank="card.rank"
        :card-suit="card.suit"
        :class="(suitColour[card.suit], { flipped: !isFlipped })"
      />
    </div>
    <div v-for="card in cards" :key="card.id">
      <Card
        :card-rank="card.rank"
        :card-suit="card.suit"
        :class="(suitColour[card.suit], { flipped: !isFlipped })"
      />
    </div>
  </div>
  <!-- <Card /> -->
  <!-- <Card /> -->
</template>

<style scoped>
.test {
  display: none;
}
button {
  max-width: 100px;
  max-height: 25px;
}
header {
  line-height: 1.5;
}

.fade-enter-active,
.fade-leave-active {
  transition: opacity 0.5s;
}

.fade-enter,
.fade-leave-to {
  opacity: 0;
}

.shuffleMedium-move {
  transition: transform 1s;
}

.logo {
  display: block;
  margin: 0 auto 2rem;
}

.deck {
  float: left;
  margin-right: 0.25em;
  margin-bottom: 0.25em;
}

p {
  color: aquamarine;
}

.red {
  color: #ff0000;
}

.blue {
  color: #0000ff;
}

.green {
  color: #00ff00;
}

.black {
  color: #000000;
}

/* .button {
  width: auto;
} */

@media (min-width: 1080px) {
  header {
    display: flex;
    place-items: center;
    padding-right: calc(var(--section-gap) / 2);
  }

  .logo {
    margin: 0 2rem 0 0;
  }

  header .wrapper {
    display: flex;
    place-items: flex-start;
    flex-wrap: wrap;
  }
}
</style>
