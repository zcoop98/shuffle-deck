<template>
    <div>
        <b-card
            title="5 Card Poker"
        >
            <b-row>
                <b-btn class="mb-4" :variant="reDealHighlight" @click="reDeal">
                    Re-Deal
                </b-btn>
                <b-btn-group class="mb-4 ml-2">
                    <b-btn variant="success" :disabled="flopDone" @click="flop">
                        Flop
                    </b-btn>
                    <b-btn variant="warning" :disabled="turnDone" @click="turn">
                        Turn
                    </b-btn>
                    <b-btn variant="danger" :disabled="riverDone" @click="river">
                        River
                    </b-btn>
                </b-btn-group>
            </b-row>
            <b-row>
                <b-col v-for="(card, index) in cards" :key="card.face + index">
                    <vue-playing-card
                        :key="card.face"
                        height="200"
                        :signature="card.face"
                        :cover="card.hide"
                    />
                </b-col>
            </b-row>
            <!-- <b-row>
                <b-col v-for="(card, index) in cards" :key="card.face + index">
                    <b-btn class="mt-3" @click="flip(index)">
                        Reveal
                    </b-btn>
                </b-col>
            </b-row> -->
        </b-card>
    </div>
</template>

<script>
import { VuePlayingCard } from 'vue-playing-card';
import { knuthShuffle } from 'knuth-shuffle';

export default {
    components: {
        VuePlayingCard,
    },
    data() {
        return {
            deck: [],
            cards: [
                {
                    face: null,
                    hide: true,
                },
                {
                    face: null,
                    hide: true,
                },
                {
                    face: null,
                    hide: true,
                },
                {
                    face: null,
                    hide: true,
                },
                {
                    face: null,
                    hide: true,
                }
            ],
        };
    },
    computed: {
        reDealHighlight() {
            return this.cards[4].hide ? 'secondary' : 'info';
        },
        flopDone() {
            return !this.cards[0].hide;
        },
        turnDone() {
            return !this.cards[3].hide;
        },
        riverDone() {
            return !this.cards[4].hide;
        },
    },
    created() { //initialize deck
        const order = ['A', 2, 3, 4, 5, 6, 7, 8, 9, 'T', 'J', 'Q', 'K'];
        const suit = ['S', 'D', 'C', 'H'];

        for (let i = 0; i < 4; i++)
            for (let j = 0; j < 13; j++) {

                this.deck.push(order[j] + suit[i]);
            }

        this.reDeal();
    },
    methods: {
        reDeal() {
            this.shuffleDeck();
            for (var i = 0; i < 5; i++) {
                this.cards[i].face = this.deck[i];
                this.cards[i].hide = true;
            }
        },
        shuffleDeck() {
            knuthShuffle(this.deck);
        },
        flip(cardNum) {
            this.cards[cardNum].hide = !this.cards[cardNum].hide;
        },
        flop() {
            for (var i = 0; i < 3; i++) {
                this.cards[i].hide = false;
            }
        },
        turn() {
            this.flop();
            this.cards[3].hide = false;
        },
        river() {
            this.turn();
            this.cards[4].hide = false;
        },
        }
    },
}
</script>