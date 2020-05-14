<template>
    <b-card
        title="5 Card Poker"
    >
        <b-row>
            <vue-playing-card height="200" cover />
            <vue-playing-card height="200" cover />
            <vue-playing-card height="200" cover />
            <vue-playing-card height="200" cover />
            <vue-playing-card height="200" cover />
        </b-row>
        <b-row>
            <b-btn>
                Re-Deal
            </b-btn>
        </b-row>
    </b-card>
</template>

<script>
import { VuePlayingCard } from 'vue-playing-card';

export default {
    components: {
        VuePlayingCard,
    },
}
</script>