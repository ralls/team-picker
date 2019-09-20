<template>
    <div id="app" class="grid">
        <div class="user-input">
            <div class="user-input__content">
                <textarea v-show="!showTextRoster" id="input" v-model="input" placeholder="Add players to build your roster"></textarea>
                <textarea v-show="showTextRoster" :value="teamNames.join('\n')"></textarea>
            </div>
            <div class="user-input__actions">
                <section>
                    <button class="button button--primary" @click="makeTeams">
                        <span v-if="!teams.length">
                            Build
                            <i class="fas fa-long-arrow-right"></i>
                        </span>
                        <span v-else>
                            Shuffle
                            <i class="fas fa-sync"></i>
                        </span>
                    </button>
                </section>
            </div>
        </div>
        <div class="roster">
            <div class="roster__content">
                <h3>Roster</h3>
                <section v-if="roster.length">
                    <template v-for="(r, k) in roster">
                        <span :key="k + 'r'" class="badge" @click="toggleAce(r.name)">
                            <i :class="[ 'fad', 'fa-fire-alt', r.ace ? 'fad--ace' : null ].filter(Boolean)"></i>
                            {{ r.name }}
                        </span>
                    </template>
                </section>
            </div>
        </div>
        <div class="teams__outer">
            <h3>Teams</h3>
            <div v-if="teams.length" class="teams__header" @click="showTextRoster = !showTextRoster">
                {{ teamsCount.total }} Team{{ teamsCount.total === 1 ? '' : 's' }}
                <span v-if="sub">({{ sub.name }} sub)</span>
            </div>
            <div v-if="teams.length" class="teams">
                <template v-for="(team, key) in teams">
                    <Team :key="key" :team="team" :number="key + 1"></Team>
                </template>
            </div>
            <div v-else class="teams">
                Add players to the roster, then build
            </div>
        </div>
    </div>
</template>
<script>
const KEY = '*'
export default {
    components: {
        Team: () => import('@/components/molecules/Team')
    },
    data () {
        return {
            input: '',
            teams: [],
            sub: '',
            showTextRoster: false
        }
    },
    computed: {
        /**
         * Maps the input array into an array of objects.
         */
        roster () {
            const { shuffle, inputToArray, alphabetical } = this
            // Replaces commas with a line break
            // Then splits into an array via line breaks
            // Then trims whitespaces and returns
            // Then sorts alphabetically by name.
            return inputToArray.map(name => this.addToRoster(name)).sort(alphabetical)
        },
        /**
         * Converts the text area to an array by replacing commas with blank lines, then splitting them.
         */
        inputToArray () {
            const { input } = this
            return input.replace(',', '\n').split('\n').filter(Boolean)
        },
        /**
         * Returns ace players.
         */
        ace () {
            return this.roster.filter(r => r.ace)
        },
        /**
         * Returns non-ace players.
         */
        normal () {
            return this.roster.filter(r => !r.ace)
        },
        teamsCount () {
            const { roster } = this
            const count = roster.length
            return {
                total: Math.floor(count / 2),
                float: count % 2
            }
        },
        teamNames () {
            return this.teams.map(t => {
                return [ t[0].name, t[1].name ].join(' & ')
            })
        }
    },
    methods: {
        /**
         * Compares two object.name properties to be sorted alphabetically.
         */
        alphabetical (a, b) {
            const nameA = a.name.toUpperCase()
            const nameB = b.name.toUpperCase()

            let comparison = 0
            if (nameA > nameB) {
                comparison = 1
            } else if (nameA < nameB) {
                comparison = -1
            }
            return comparison
        },
        /**
         * Adds a name to the roster by placing it into an object
         * with its "ace" status.
         */
        addToRoster (name) {
            if (!name) {
                return false
            }
            let ace = false
            name = name.trim()
            if (name.indexOf(KEY) !== -1) {
                name = name.replace(KEY, '')
                ace = true
            }
            return {
                name,
                ace
            }
        },
        /**
         * Shuffles an array.
         * Credit: https://gist.github.com/guilhermepontes/17ae0cc71fa2b13ea8c20c94c5c35dc4
         */
        shuffle: (arr) => arr
            .map(a => [Math.random(), a])
            .sort((a, b) => a[0] - b[0])
            .map(a => a[1]),
        /**
         * Generates a random number between x and y.
         */
        randomNumber: (min, max) => Math.floor(Math.random() * (Math.floor(max) - Math.ceil(min) + 1)) + min,
        /**
         * Creates an array of random numbers that represent values in an index.
         * Each value in the array will have a difference of at least 2.
         */
        getSpliceIndexes (count, max, arr = []) {
            const { randomNumber } = this
            while (arr.length < count) {
                const num = randomNumber(0, max)
                const found = arr.sort().filter(n => {
                    const x = [ num, n ].sort()
                    return (x[1] - x[0] <= 1)
                })
                if (!found.length) {
                    arr.push(num)
                }
            }
            return arr.sort()
        },
        /**
         * Shuffles the roster, joins non-aces with aces into multiple teams.
         */
        makeTeams () {
            this.teams = []
            const { teamsCount: { total, float }, shuffle, normal, getSpliceIndexes } = this
            const ace = shuffle(this.ace)
            const roster = shuffle(normal)
            // Inserts "aces" into the roster by splicing them into random spots into the roster array
            // at least two indexes a part to avoid stacking.
            if (ace.length) {
                let spliceIndex = getSpliceIndexes(ace.length, roster.length, [])
                for (let i = 0; i < ace.length; i++) {
                    roster.splice(spliceIndex[i], 0, ace[i])
                }
            }
            // Add even/odd pairs from the roster array index into teams.
            for (let i = 0; i < total; i++) {
                let j = (i * 2)
                this.teams.push([ roster[j], roster[j + 1]])
            }
            this.sub = !!float ? roster[roster.length - 1] : ''
        },
        /**
         * Toggles a person on the roster's "ace" status.
         */
        toggleAce (name) {
            this.input = this.inputToArray.map(n => {
                if ((n === name) || (n === (name + KEY))) {
                    n = n.indexOf(KEY) === -1 ? n + KEY : n.replace(KEY, '')
                }
                return n
            }).join('\n')
        }
    },
    watch: {
        /**
         * Watches for changes in the roster.
         * If teams have been built, they will be built again automatically.
         */
        'roster': function (a, b) {
            const { makeTeams, teams } = this
            if (teams.length) {
                makeTeams()
            }
        }
    }
}
</script>

<style lang="scss">
@import url('https://fonts.googleapis.com/css?family=Roboto:400,700&display=swap');
$white: #feffff;
$offwhite: #F8F7FC;
$gray-1: #DADDE7;
$gray-2: #F3F5F7;
$gray-3: #424444;
$gray-4: #898989;
$primary: #55a4a2;
$secondary: #6F50FE;
$secondary: #47537a;
$accent: orangered;
$font-sans-serif: 'Roboto', sans-serif;

%scrollbars {
    &::-webkit-scrollbar {
        width: 8px;

        &-track {
            background-color: $gray-2;
        }

        &-thumb {
            background-color: darken($gray-1, 10%);
            border-radius: 4px;
        }
    }
}
*,
::before,
::after {
    box-sizing: border-box;
}
html,
body {
    margin: 0;
    padding: 0;
    font-family: $font-sans-serif;
    color: $gray-3;
}

h3 {
    margin-top: 0;
}

section {
    margin-bottom: 1rem;
}

.grid {
    display: grid;
    grid-template-rows: 1fr;
    grid-template-columns: 25% 225px 1fr;
    grid-template-areas: "input roster team";
}

.user-input {
    grid-area: input;
    display: grid;
    grid-template-columns: 1fr;
    grid-template-rows: 1fr 47px;
    grid-template-areas:
        "content"
        "actions";
    padding: 1rem;
    background-color: $secondary;
    background-color: $offwhite;

    &__content {
        grid-area: content;
        height: calc(100% - 1rem);
    }

    &__actions {
        grid-area: actions;
    }
    textarea {
        display: block;
        width: 100%;
        max-width: 100%;
        height: 100%;
        padding: 1rem;
        overflow-y: auto;
        resize: none;
        border: none;
        font-family: $font-sans-serif;
        font-size: 1.25rem;
        color: darken($gray-4, 20%);

        &:focus {
            outline: none;
        }

        @extend %scrollbars;
    }
}

.roster {
    display: grid;
    grid-area: roster;
    height: 100vh;
    max-height: 100vh;
    overflow-y: auto;
    background-color: $offwhite;
    padding: 1rem 0;

    hr {
        margin: 1rem -1rem;
    }

    &__content {
        border-right: 1px solid #ebebeb;
    }
}

.teams {

    &__outer {
        grid-area: team;
        position: relative;
        background-color: $offwhite;
        padding: 1rem;
    }
    position: relative;
    display: flex;
    flex-wrap: wrap;
    align-content: flex-start;
    max-height: 100vh;
    overflow-y: auto;

    &__header {
        position: absolute;
        bottom: 1rem;
        right: 1rem;
        background-color: $secondary;
        color: $white;
        padding: 1rem;
        font-weight: bold;
        cursor: pointer;
    }
}

.roster,
.teams {
    overflow-y: auto;
    @extend %scrollbars;
}

.badge {
    display: inline-block;
    padding: .4rem .8rem;
    background-color: $white;
    color: lighten($gray-4, 10%);
    border-radius: 1rem;
    border: 1px solid darken($gray-2, 5%);
    cursor: pointer;

    &:not(:last-child) {
        margin: 0 .5rem .5rem 0;
    }

    .fad {
        color: rgba($gray-4, 0.5);

        &--ace {
            color: lighten($accent, 20%);
            color: $accent;
        }
    }
}

.button {
    display: block;
    font-weight: 400;
    text-align: center;
    white-space: nowrap;
    width: 100%;
    user-select: none;
    border: 1px solid transparent;
    padding: 1rem .9rem;
    font-size: .85rem;
    letter-spacing: .1em;
    text-transform: uppercase;
    line-height: 1;
    border-radius: 3px;
    transition: all .25s linear;
    color: darken($gray-4, 20%);
    cursor: pointer;

    &:hover {
        border-radius: 50px;
        color: $secondary;
    }

    &:focus {
        outline: none;
    }

    &--primary {
        background-color: $primary;

        &,
        &:hover,
        &:focus {
            color: $white;
        }
    }
}
</style>
