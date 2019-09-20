<template>
    <div class="team">
        <span class="team__abbr">{{ abbr }}</span>
        <div class="team__members">
            <span>
                Team {{ number }}
            </span>
            <span>
                {{ members }}
            </span>
        </div>
    </div>
</template>
<script>
export default {
    props: {
        number: Number,
        team: {
            type: Array,
            validator: r => r.length === 2
        }
    },
    computed: {
        members () {
            return this.team.map(m => m.name).join(' & ')
        },
        abbr () {
            const { team } = this
            return team.map(m => m.name.charAt(0)).join('')
        }
    }
}
</script>
<style lang="scss">
.team {
    display: flex;
    background-color: #fff;
    border: 1px solid #ebebeb;
    border-top: none;
    padding: 1rem;
    color: #424444;
    width: 25%;

    &__abbr {
        $size: 40px;
        display: inline-flex;
        justify-content: center;
        align-items: center;
        width: $size;
        height: $size;
        background-color: #47537a;
        color: #fff;
        border-radius: $size;
        text-transform: uppercase;
    }

    &__members {
        display: flex;
        flex-direction: column;
        margin-left: 1rem;

        span:first-of-type {
            font-weight: bold;
            text-transform: uppercase;
            letter-spacing: 1px;
        }
    }
}
</style>
