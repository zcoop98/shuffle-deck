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
        </b-card>
        <b-card>
            <b-row>
                <p>Generate: </p>
                <b-col>
                    <b-btn class="mx-1" variant="info" @click="generateHand(0)">
                        Royal Flush
                    </b-btn>
                    <b-btn class="mx-1" variant="info" @click="generateHand(1)">
                        Straight Flush
                    </b-btn>
                    <b-btn class="mx-1" variant="info" @click="generateHand(2)">
                        Four of a Kind
                    </b-btn>
                    <b-btn class="mx-1" variant="info" @click="generateHand(3)">
                        Full House
                    </b-btn>
                    <b-btn class="mx-1" variant="info" @click="generateHand(4)">
                        Flush
                    </b-btn>
                    <b-btn class="mx-1" variant="info" @click="generateHand(5)">
                        Straight
                    </b-btn>
                    <b-btn class="mx-1" variant="info" @click="generateHand(6)">
                        Three of a Kind
                    </b-btn>
                    <b-btn class="mx-1" variant="info" @click="generateHand(7)">
                        Two Pair
                    </b-btn>
                    <b-btn class="mx-1" variant="info" @click="generateHand(8)">
                        Pair
                    </b-btn>
                </b-col>
            </b-row>

            <b-row>
                <b-col>
                    <p>Shuffles: {{ genShuffles }}</p>
                    <p>Time: {{ Math.round((msTimeElapsed + Number.EPSILON) * 10000) / 10000000 }} seconds</p>
                </b-col>
            </b-row>
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
            genShuffles: 0,
            msTimeElapsed: 0,
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
        orderSet() {
            return new Set([
                this.orderToNumber(this.deck[0][0]),
                this.orderToNumber(this.deck[1][0]),
                this.orderToNumber(this.deck[2][0]),
                this.orderToNumber(this.deck[3][0]),
                this.orderToNumber(this.deck[4][0])
            ]);
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
        orderToNumber(order) {
            if (isNaN(order))
                switch (order) {
                    case 'A':
                        return 1;
                    case 'T':
                        return 10;
                    case 'J':
                        return 11;
                    case 'Q':
                        return 12;
                    case 'K':
                        return 13;
                }
            else
                return order;
        },
        generateHand(handId) {
            const hands = [
                this.checkRoyalFlush,
                this.checkStraightFlush,
                this.checkFourKind,
                this.checkFullHouse,
                this.checkFlush,
                this.checkStraight,
                this.checkThreeKind,
                this.checkTwoPair,
                this.checkPair
            ];

            if (handId >= hands.length || handId < 0) {
                this.genShuffles = 'TILT';
                return null;
            }

            for (var i = 0; i < 5; i++) {
                this.cards[i].hide = true;
            }

            this.genShuffles = 0;
            var t0 = performance.now();
            do {
                this.genShuffles++;
                this.shuffleDeck();
            } while(!hands[handId]() && this.genShuffles < 3000000)
            var t1 = performance.now();

            this.msTimeElapsed = t1 - t0;

            if (this.genShuffles === 3000000)
                this.genShuffles = 'TILT';

            for (var j = 0; j < 5; j++) {
                this.cards[j].face = this.deck[j];
                this.cards[j].hide = false;
            }
        },
        checkRoyalFlush() {
            //suit check
            if (!this.checkFlush())
                return false;

            //order check (flush found, checking royal)
            for (var i = 0; i < 5; i++) {
                //Can use a special method since a royal's orders are all non-numbers
                if (!isNaN(+this.deck[i][0])) {
                    return false;
                }
            }
            return true;
        },
        checkStraightFlush() {
            return this.checkFlush() && this.checkStraight();
        },
        checkTwoSet(isFullHouseMode) {
            var hand = this.orderSet();

            if (hand.size === 2) {
                var matchCount = 0;

                for (var i = 1; i < 5; i++) {
                    if ((this.deck[i][0] === this.deck[0][0])) {
                        matchCount++;
                    }
                }
                return (isFullHouseMode ? (matchCount === 1 || matchCount === 2) : (matchCount === 0 || matchCount === 3));
            }
            else
                return false;
        },
        checkFourKind() {
            return this.checkTwoSet(false);
        },
        checkFullHouse() {
            return this.checkTwoSet(true);
        },
        checkFlush() {
            for (var i = 1; i < 5; i++) {
                if (!(this.deck[i][1] === this.deck[0][1])) {
                    return false;
                }
            }
            return true;
        },
        checkStraight() {
            var hand = this.orderSet();

            return hand.size === 5 && (Math.max(...hand) - Math.min(...hand) === 4);
        },
        checkThreeKind() {
            var hand = this.orderSet();

            if (hand.size <= 3) {
                var handArr = [this.deck[0][0], this.deck[1][0], this.deck[2][0], this.deck[3][0], this.deck[4][0]];
                return handArr.filter((v, i, arr) => i !== arr.indexOf(v))
                    .filter((v, i, arr) => i !== arr.indexOf(v))
                    .length > 0;
            }
            else
                return false;
        },
        checkTwoPair() {
            var hand = this.orderSet();

            if (hand.size <= 3) {
                var handArr = [this.deck[0][0], this.deck[1][0], this.deck[2][0], this.deck[3][0], this.deck[4][0]];
                var dupeArr = handArr.filter((v, i, arr) => i !== arr.indexOf(v));

                return new Set(dupeArr).size == 2;
            }
            else
                return false;
        },
        checkPair() {
            return this.orderSet().size <= 4;
        },
    },
}
</script>