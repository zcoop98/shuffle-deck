<template>
    <div>
        <h4 class="card-title text-center">
            5 Card Poker
        </h4>
        <b-card no-body>
            <b-tabs
                card
                fill
                @activate-tab="reDeal"
            >
                <b-tab title="Generators" active>
                    <b-overlay :show="isLoading" :rounded="true">
                        <template #overlay>
                            <b-icon-shuffle font-scale="3" animation="throb" />
                        </template>
                        <b-row class="my-4">
                            <b-col v-for="(card, index) in cards" :key="card.face + index">
                                <vue-playing-card
                                    :key="card.face"
                                    height="200"
                                    :signature="card.face"
                                    :cover="card.hide"
                                />
                            </b-col>
                        </b-row>
                    </b-overlay>

                    <b-row>
                        <b-col>
                            <b-btn-group>
                                <b-btn variant="dark" disabled>
                                    Generators:
                                </b-btn>

                                <b-btn variant="info" @click="generateHand(0)">
                                    Royal Flush
                                </b-btn>

                                <b-btn variant="info" @click="generateHand(1)">
                                    Straight Flush
                                </b-btn>

                                <b-btn variant="info" @click="generateHand(2)">
                                    Four of a Kind
                                </b-btn>

                                <b-btn variant="info" @click="generateHand(3)">
                                    Full House
                                </b-btn>

                                <b-btn variant="info" @click="generateHand(4)">
                                    Flush
                                </b-btn>

                                <b-btn variant="info" @click="generateHand(5)">
                                    Straight
                                </b-btn>

                                <b-btn variant="info" @click="generateHand(6)">
                                    Three of a Kind
                                </b-btn>

                                <b-btn variant="info" @click="generateHand(7)">
                                    Two Pair
                                </b-btn>

                                <b-btn variant="info" @click="generateHand(8)">
                                    Pair
                                </b-btn>
                            </b-btn-group>
                        </b-col>
                    </b-row>

                    <b-row class="mt-4" align-h="center">
                        <b-col cols="4">
                            <b-overlay :show="isLoading" :rounded="true">
                                <b-card>
                                    <p>Shuffles: {{ genShuffles }}</p>

                                    <p>Time: {{ !isNaN(+msTimeElapsed) ? Math.round((msTimeElapsed + Number.EPSILON) * 10000) / 10000000 + ' seconds' : msTimeElapsed }}</p>
                                </b-card>
                            </b-overlay>
                        </b-col>
                    </b-row>
                </b-tab>
                <b-tab title="Deal">
                    <b-row>
                        <b-btn :variant="reDealHighlight" @click="reDeal">
                            Re-Deal
                        </b-btn>

                        <b-btn-group class="ml-2">
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

                    <b-overlay :show="isLoading" :rounded="true">
                        <template #overlay>
                            <b-icon-shuffle font-scale="3" animation="throb" />
                        </template>
                        <b-row class="my-4">
                            <b-col v-for="(card, index) in cards" :key="card.face + index">
                                <vue-playing-card
                                    :key="card.face"
                                    height="200"
                                    :signature="card.face"
                                    :cover="card.hide"
                                />
                            </b-col>
                        </b-row>
                    </b-overlay>

                    <b-row class="mt-4" align-h="center">
                        <b-col cols="4">
                            <b-card>
                                <vue-playing-card
                                    v-for="pocket in pockets"
                                    :key="pocket.face"
                                    class="px-2"
                                    height="200"
                                    :signature="pocket.face"
                                    :cover="pocket.hide"
                                />
                            </b-card>
                        </b-col>
                        <b-col cols="2">
                            <b-btn @click="getBestHand">
                                Check Best Hand
                            </b-btn>
                        </b-col>
                    </b-row>
                </b-tab>
            </b-tabs>
        </b-card>
    </div>
</template>

<script>
import { BIconShuffle } from 'bootstrap-vue';
import { VuePlayingCard } from 'vue-playing-card';
import { knuthShuffle } from 'knuth-shuffle';

export default {
    components: {
        VuePlayingCard,
        BIconShuffle,
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
            pockets: [
                {
                    face: null,
                    hide: false,
                },
                {
                    face: null,
                    hide: false,
                }
            ],
            genShuffles: '-',
            msTimeElapsed: '-',
            isLoading: false,
        };
    },
    computed: {
        reDealHighlight() {
            return this.cards[4].hide ? 'outline-info' : 'info';
        },
        flopDone() {
            return !this.cards[0].hide || this.isLoading;
        },
        turnDone() {
            return !this.cards[3].hide || this.isLoading;
        },
        riverDone() {
            return !this.cards[4].hide || this.isLoading;
        },
    },
    created() { //initialize deck
        const rank = ['A', 2, 3, 4, 5, 6, 7, 8, 9, 'T', 'J', 'Q', 'K'];
        const suit = ['S', 'D', 'C', 'H'];

        for (let i = 0; i < 4; i++)
            for (let j = 0; j < 13; j++) {
                this.deck.push(rank[j] + suit[i]);
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
            this.pockets[0].face = this.deck[5];
            this.pockets[0].hide = false;

            this.pockets[1].face = this.deck[6];
            this.pockets[1].hide = false;
        },
        shuffleDeck() {
            knuthShuffle(this.deck);
        },
        rankSet() {
            return new Set([
                this.rankToNumber(this.deck[0][0]),
                this.rankToNumber(this.deck[1][0]),
                this.rankToNumber(this.deck[2][0]),
                this.rankToNumber(this.deck[3][0]),
                this.rankToNumber(this.deck[4][0])
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
        rankToNumber(rank) {
            if (isNaN(rank))
                switch (rank) {
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
                return rank;
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

            this.isLoading = true;

            this.genShuffles = '-';
            this.msTimeElapsed = '-';

            setTimeout(() => {   //Delay allows event loop to clear
                this.genShuffles = 0;
                this.msTimeElapsed = 0;

                var t0 = performance.now();
                do {
                    this.genShuffles++;
                    this.shuffleDeck();
                } while(!hands[handId]() && this.genShuffles < 6000000)
                var t1 = performance.now();

                this.msTimeElapsed = t1 - t0;

                if (this.genShuffles === 6000000)
                    this.genShuffles = 'TILT';

                this.isLoading = false;

                for (var j = 0; j < 5; j++) {
                    this.cards[j].face = this.deck[j];
                    this.cards[j].hide = false;
                }
            }, 100);
        },
        checkRoyalFlush() {
            //suit check
            if (!this.checkFlush())
                return false;

            //rank check (flush found, checking royal)
            for (var i = 0; i < 5; i++) {
                //Can use a special method since a royal's ranks are all non-numbers
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
            var hand = this.rankSet();

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
            var hand = this.rankSet();

            return hand.size === 5 && (Math.max(...hand) - Math.min(...hand) === 4);
        },
        checkThreeKind() {
            var hand = this.rankSet();

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
            var hand = this.rankSet();

            if (hand.size <= 3) {
                var handArr = [this.deck[0][0], this.deck[1][0], this.deck[2][0], this.deck[3][0], this.deck[4][0]];
                var dupeArr = handArr.filter((v, i, arr) => i !== arr.indexOf(v));

                return new Set(dupeArr).size == 2;
            }
            else
                return false;
        },
        checkPair() {
            return this.rankSet().size <= 4;
        },
        getBestHand() {
            var set = [...this.cards, ...this.pockets];
            for (var i = 0; i < set.length; ++i) {
                set[i] = set[i].face;
            }
            var combos = [];
            this.getAllHandCombos(set, combos);
        },
        getAllHandCombos(set, outputSet) {
            var temp = [];
            this.handComboUtil(set, outputSet, temp, 0, set.length-1, 0, 5);
        },
        handComboUtil(set, outputSet, temp, start, end, index, r)
        {
            // Adapted from https://www.geeksforgeeks.org/print-all-possible-combinations-of-r-elements-in-a-given-array-of-size-n/
            if (index == r)
                outputSet.push(temp.slice());
            else
                for (var i = start; i <= end && end - i + 1 >= r - index; i++) {
                    temp[index] = set[i];
                    this.handComboUtil(set, outputSet, temp, i+1, end, index+1, r);
                }
        },
    },
}
</script>