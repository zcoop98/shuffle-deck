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
                                <h5 class="text-left">
                                    Pocket cards:
                                </h5>

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

                        <b-col cols="4">
                            <div v-if="riverDone">
                                <h5 class="text-left">
                                    {{ bestHandName }}
                                </h5>

                                <vue-playing-card
                                    v-for="card in bestHand"
                                    :key="card"
                                    class="px-1"
                                    :signature="card"
                                    height="140"
                                />
                            </div>
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
            bestHand: [],
            bestHandScore: 0,
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
        bestHandName() {
            switch (Math.floor(this.bestHandScore)) {
                case 0:
                    return 'High Card';
                case 1:
                    return 'Pair';
                case 2:
                    return 'Two Pair';
                case 3:
                    return 'Three of a Kind';
                case 4:
                    return 'Straight';
                case 5:
                    return 'Flush';
                case 6:
                    return 'Full House';
                case 7:
                    return 'Four of a Kind';
                case 8:
                    return 'Straight Flush';
                case 9:
                    return 'Royal Flush';
                default:
                    return 'Error';
            }
        },
    },
    watch: {
        riverDone(newVal) {
            if (newVal) this.getBestHand();
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
            for (let i = 0; i < 5; i++) {
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
        rankSet(hand) {
            return new Set([
                this.removeSuit(this.rankToNumber(hand[0])),
                this.removeSuit(this.rankToNumber(hand[1])),
                this.removeSuit(this.rankToNumber(hand[2])),
                this.removeSuit(this.rankToNumber(hand[3])),
                this.removeSuit(this.rankToNumber(hand[4]))
            ]);
        },
        flip(cardNum) {
            this.cards[cardNum].hide = !this.cards[cardNum].hide;
        },
        flop() {
            for (let i = 0; i < 3; i++) {
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
        rankToNumber(signature) {
            const suit = signature[signature.length - 1];

            switch (signature[0]) {
                case 'T':
                    return '10' + suit;
                case 'J':
                    return '11' + suit;
                case 'Q':
                    return '12' + suit;
                case 'K':
                    return '13' + suit;
                case 'A':
                    return '14' + suit;
                default:
                    return signature;
            }
        },
        removeSuit(signature) {
            return signature.slice(0, signature.length - 1);
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
                } while(!hands[handId](this.deck) && this.genShuffles < 6000000)
                var t1 = performance.now();

                this.msTimeElapsed = t1 - t0;

                if (this.genShuffles === 6000000)
                    this.genShuffles = 'TILT';

                this.isLoading = false;

                for (let j = 0; j < 5; j++) {
                    this.cards[j].face = this.deck[j];
                    this.cards[j].hide = false;
                }
            }, 100);
        },
        checkRoyalFlush(hand) {
            //suit check
            if (!this.checkFlush(hand))
                return false;

            //rank check (flush found, checking royal)
            for (let i = 0; i < 5; i++) {
                //Can use a special method since a royal's ranks are all non-numbers
                if (!isNaN(+hand[i][0])) {
                    return false;
                }
            }
            return true;
        },
        checkStraightFlush(hand) {
            return this.checkFlush(hand) && this.checkStraight(hand);
        },
        checkTwoSet(hand, isFullHouseMode) {
            var handSet = this.rankSet(hand);

            if (handSet.size === 2) {
                var matchCount = 0;

                for (let i = 1; i < 5; i++) {
                    if ((hand[i][0] === hand[0][0])) {
                        matchCount++;
                    }
                }
                return (isFullHouseMode ? (matchCount === 1 || matchCount === 2) : (matchCount === 0 || matchCount === 3));
            }
            else
                return false;
        },
        checkFourKind(hand) {
            return this.checkTwoSet(hand, false);
        },
        checkFullHouse(hand) {
            return this.checkTwoSet(hand, true);
        },
        checkFlush(hand) {
            for (let i = 1; i < 5; i++) {
                if (!(hand[i][1] === hand[0][1])) {
                    return false;
                }
            }
            return true;
        },
        checkStraight(hand) {
            var handSet = this.rankSet(hand);

            if (handSet.size === 5)
                if (handSet.has('14')) {
                    var aceLowSet = this.rankSet(hand);
                    aceLowSet.delete('14');
                    aceLowSet.add('1');

                    return (Math.max(...handSet) - Math.min(...handSet) === 4) || (Math.max(...aceLowSet) - Math.min(...aceLowSet) === 4);
                }
                else
                    return (Math.max(...handSet) - Math.min(...handSet) === 4);
            else
                return false;
        },
        checkThreeKind(hand) {
            var handSet = this.rankSet(hand);

            if (handSet.size <= 3) {
                var handArr = [hand[0][0], hand[1][0], hand[2][0], hand[3][0], hand[4][0]];
                return handArr.filter((v, i, arr) => i !== arr.indexOf(v))
                    .filter((v, i, arr) => i !== arr.indexOf(v))
                    .length > 0;
            }
            else
                return false;
        },
        checkTwoPair(hand) {
            var handSet = this.rankSet(hand);

            if (handSet.size <= 3) {
                var handArr = [hand[0][0], hand[1][0], hand[2][0], hand[3][0], hand[4][0]];
                var dupeArr = handArr.filter((v, i, arr) => i !== arr.indexOf(v));

                return new Set(dupeArr).size == 2;
            }
            else
                return false;
        },
        checkPair(hand) {
            return this.rankSet(hand).size <= 4;
        },
        getBestHand() {
            var set = [...this.cards, ...this.pockets];
            for (let i = 0; i < set.length; ++i) {
                set[i] = set[i].face;
            }
            var combos = [];
            this.getAllHandCombos(set, combos);

            var bestCombo = null;
            var bestComboScore = 0;
            var handScore = 0;
            for (let j = combos.length - 1; j >= 0; --j) { //test each combo, keep highest scoring
                handScore = this.scoreHand(combos[j]);
                if (handScore > bestComboScore) {
                    bestCombo = combos[j];
                    bestComboScore = handScore;
                }
            }
            bestCombo.sort(
                (a, b) => {
                    a = +this.removeSuit(this.rankToNumber(a));
                    b = +this.removeSuit(this.rankToNumber(b));
                    return (a > b) ? 1 : (a < b) ? -1 : 0;
                }
            );

            if (bestComboScore.toString().substring(0, 4) == '4.05' || bestComboScore.toString().substring(0, 4) == '8.05') // check for 5-high straight
                bestCombo = [bestCombo.pop(), ...bestCombo];
                

            this.bestHand = bestCombo;
            this.bestHandScore = bestComboScore;
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
                for (let i = start; i <= end && end - i + 1 >= r - index; i++) {
                    temp[index] = set[i];
                    this.handComboUtil(set, outputSet, temp, i+1, end, index+1, r);
                }
        },
        scoreHand(hand) {
            var ranksOnlyHand = hand.map(card => +this.removeSuit(this.rankToNumber(card))); // Create array of numerical signatures without suits from hand
            var maxCard = Math.max(...ranksOnlyHand);

            if (this.checkRoyalFlush(hand)) { // Royal - 9 pts; only one royal possible per draw so no tie break needed
                return 9;
            }

            else if (this.checkStraightFlush(hand)) { // Striaght Flush - 8 pts; tie break: highest card
                let score = 8;

                if (maxCard == 14 && ranksOnlyHand.some(card => card == 2)) // 2 and A in straight indicates low straight
                    maxCard = 5;

                return score + maxCard * 0.01;
            }

            else if (this.checkFourKind(hand)) { // Four of a Kind - 7 pts; tie break: highest kicker
                if (ranksOnlyHand[0] != ranksOnlyHand[1])
                    if (ranksOnlyHand[0] == ranksOnlyHand[2])
                        return 7 + ranksOnlyHand[1] * 0.01;
                    else
                        return 7 + ranksOnlyHand[0] * 0.01;
                else {
                    return 7 + ranksOnlyHand.find(card => card != ranksOnlyHand[0]) * 0.01;
                }
            }

            else if (this.checkFullHouse(hand)) { // Full House - 6 pts; tie break: highest triple, then highest double
                let score = '6.';
                let totals = {};
                let triple = 0;
                let pair = 0;

                for (let card of ranksOnlyHand) { // Count quantity of each rank
                    if (!totals[card])
                        totals[card] = 0;
                    totals[card]++;
                }

                for (let card in totals) {
                    if (totals[card] == 3)
                        triple = card;
                    else
                        pair = card;
                }

                score += triple.toString().padStart(2, '0');
                score += pair.toString().padStart(2, '0');

                return parseFloat(score);
            }

            else if (this.checkFlush(hand)) { // Flush - 5 pts; tie break: highest cards, in descending order
                ranksOnlyHand.sort((a, b) => (a < b) ? 1 : (a > b) ? -1 : 0); // Sort into descending order
                let score = '5.'; // Use string concat instead of decimal addition to avoid precision issues

                for (let i = 0; i < 5; i++)
                    score += ranksOnlyHand[i].toString().padStart(2, '0');

                return parseFloat(score); // Return as decimal
            }

            else if (this.checkStraight(hand)) { // Straight - 4 pts; tie break: highest card
                let score = 4;

                if (maxCard == 14 && ranksOnlyHand.some(card => card == 2)) // 2 and A in straight indicates low straight
                    maxCard = 5;

                return score + maxCard * 0.01;
            }

            else if (this.checkThreeKind(hand)) { // Three of a Kind - 3 pts; tie break: highest triple, then highest kickers in descending order
                let score = '3.';
                let totals = {};
                let kickers = [];

                for (let card of ranksOnlyHand) { // Count quantity of each rank
                    if (!totals[card])
                        totals[card] = 0;
                    totals[card]++;
                }

                for (let card in totals) {
                    if (totals[card] == 3)
                        score += card.toString().padStart(2, '0'); // Add triplet score
                    else
                        kickers.push(card);
                }

                score += Math.max(...kickers).toString().padStart(2, '0'); // Append kickers to score in order
                score += Math.min(...kickers).toString().padStart(2, '0');

                return parseFloat(score);
            }

            else if (this.checkTwoPair(hand)) { // Two Pair - 2 pts; tie break: highest pairs in descending order, then kicker
                let score = '2.';
                let totals = {};
                let pairs = [];
                let kicker = 0;

                for (let card of ranksOnlyHand) { // Count quantity of each rank
                    if (!totals[card])
                        totals[card] = 0;
                    totals[card]++;
                }

                for (let card in totals) {
                    if (totals[card] == 2)
                        pairs.push(card);
                    else
                        kicker = card;
                }

                score += Math.max(...pairs).toString().padStart(2, '0');
                score += Math.min(...pairs).toString().padStart(2, '0');
                score += kicker.toString().padStart(2, '0');

                return parseFloat(score);
            }

            else if (this.checkPair(hand)) { // Pair - 1 pts; tie break: highest pair, then highest kickers in descending order
                let score = '1.';
                let totals = {};
                let kickers = [];
                let pair = 0;

                for (let card of ranksOnlyHand) { // Count quantity of each rank
                    if (!totals[card])
                        totals[card] = 0;
                    totals[card]++;
                }

                for (let card in totals) {
                    if (totals[card] == 2)
                        pair = card;
                    else
                        kickers.push(card);
                }

                kickers.sort((a, b) => (a < b) ? 1 : (a > b) ? -1 : 0);

                score += pair.toString().padStart(2, '0');
                for (let card of kickers)
                    score += card.toString().padStart(2, '0');

                return parseFloat(score);
            }

            else { // High card - 0 pts; tie break: highest cards in descending order
                let score = '0.';
                ranksOnlyHand.sort((a, b) => (a < b) ? 1 : (a > b) ? -1 : 0);

                for (let card of ranksOnlyHand)
                    score += card.toString().padStart(2, '0');

                return parseFloat(score);
            }
        },
    },
}
</script>
