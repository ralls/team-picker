<template>
    <div id="app" class="grid">
        <div class="roster">
            <div class="roster__content">
                <section>
                    <textarea id="input" v-model="input" placeholder="Add players to build your roster"></textarea>
                </section>
                <section v-if="roster.length">
                    <h3>Roster</h3>
                    <template v-for="(r, k) in roster">
                        <span :key="k + 'r'" class="badge" @click="toggleAce(r.name)">
                            <i :class="[ 'fad', 'fa-fire-alt', r.ace ? 'fad--ace' : null ].filter(Boolean)"></i>
                            {{ r.name }}
                        </span>
                    </template>
                </section>
            </div>
            <div class="roster__actions">
                <section>
                    <button class="button" @click="makeTeams">
                        <span v-if="!teams.length">
                            Build
                            <i class="fas fa-plus-circle"></i>
                        </span>
                        <span v-else>
                            Shuffle
                            <i class="fas fa-sync"></i>
                        </span>
                    </button>
                </section>
            </div>
        </div>
        <div class="teams">
            <div v-if="teams.length" class="teams__header">
                Teams: {{ teamsCount.total }}
                <span v-if="sub">({{ sub.name }} sub)</span>
            </div>
            <div v-else>
                <h3>Add players to the roster, then build</h3>
            </div>
            <template v-for="(team, key) in teams">
                <Team :key="key" :team="team" :number="key + 1"></Team>
            </template>
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
            sub: ''
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
         * Shuffles the roster, joins non-aces with aces into multiple teams.
         */
        makeTeams () {
            this.teams = []
            const { teamsCount: { total, float }, shuffle, ace, normal } = this
            const roster = shuffle(normal)
            // Inserts "aces" into the roster by splicing them into the beginning of the array
            // starting at zero, and every other index after that.
            // Ex: 0, 2, 4, 6...
            if (ace.length) {
                for (let i = 0; i < ace.length; i++) {
                    let j = (i * 2)
                    roster.splice(j, 0, ace[i])
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
$gray-1: #DADDE7;
$gray-2: #F3F5F7;
$gray-3: #424444;
$gray-4: #898989;
$primary: #82C256;
$secondary: #6F50FE;
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
}

section {
    margin-bottom: 1rem;
}

.grid {
    display: grid;
    grid-template-rows: 1fr;
    grid-template-columns: 25% 25% 1fr;
    grid-template-areas: "roster team";
}

.roster {
    display: grid;
    grid-template-rows: 1fr 47px;
    grid-template-areas:
        "content",
        "actions";
    grid-area: roster;
    height: 100vh;
    max-height: 100vh;
    overflow-y: auto;
    background-color: $secondary;

    &__content {
        grid-area: "content";
    }

    &__actions {
        grid-area: "actions";
    }

    hr {
        margin: 1rem -1rem;
    }

    h3 {
        color: #fff;
    }

    textarea {
        display: block;
        width: 100%;
        max-width: 100%;
        min-height: 33vh;
        max-height: 50vh;
        overflow-y: auto;
        resize: none;
        border: none;
        font-family: $font-sans-serif;
        font-size: 1rem;
        color: darken($gray-4, 20%);

        &:focus {
            outline: none;
        }

        @extend %scrollbars;
    }
}

.teams {
    grid-area: team;
    max-height: 100vh;
    overflow-y: auto;
    background-color: #F8F7FC;

    &__header {
        background-color: $secondary;
        color: #fff;
        padding: 1rem;
        font-weight: bold;
    }
}

.roster,
.teams {
    padding: 1rem;
    overflow-y: auto;
    @extend %scrollbars;
}

.badge {
    display: inline-block;
    padding: .4rem .8rem;
    background-color: #fff;
    border-radius: 1rem;
    color: #424444;
    cursor: pointer;

    &:not(:last-child) {
        margin: 0 .5rem .5rem 0;
    }

    .fad {
        color: #666;

        &--ace {
            color: orangered;
        }
    }
}

.button {
    display: block;
    font-weight: 400;
    text-align: center;
    white-space: nowrap;
    // vertical-align: middle;
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
}
</style>
