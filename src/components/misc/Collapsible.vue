<template>
    <div :id="href" class="d-flex flex-column" :class="{'gap-3': !collapsed}">
        <button @click="collapsed = !collapsed" class="d-flex align-items-center justify-content-between fw-bold gap-2 collapse-button" :class="{collapsed}" data-toggle="collapse" :data-target="'#' + href + '-body'" aria-expanded="false" :aria-controls="href + '-body'">
            <span class="d-flex"><component v-if="arrow" :is="collapsed ? MenuRight : MenuDown" />{{ clickableText }}<slot name="additionalButtonText" /></span>
            <span class="d-flex flex-grow-1">
                <slot name="buttonRight" :collapsed="collapsed" />
            </span>
        </button>
        <div :id="href + '-body'" class="collapsible-body" :class="{'expanded': !collapsed}">
            <div>
                <slot name="content" v-if="!collapsed" />
            </div>
        </div>
    </div>
</template>

<script setup lang="ts">
    import {ref} from "vue";
    import MenuRight from "vue-material-design-icons/MenuRight.vue";
    import MenuDown from "vue-material-design-icons/MenuDown.vue";

    withDefaults(defineProps<{ href?: string, clickableText: string, arrow: boolean }>(), {
        href: Math.random().toString(36).substring(2, 5),
        arrow: true
    });

    const collapsed = ref(true);
</script>

<style scoped lang="scss">
    .collapse-button {
        padding: 0;
        border: none;
        background: none;

        &:focus {
            outline:none;
            box-shadow: none;
        }
    }

    .collapsible-body {
        overflow: hidden;

        > div {
            transition: margin-bottom .15s;
            margin-bottom: -100%;
        }

        &.expanded > div {
            margin-bottom: 0;
        }
    }
</style>