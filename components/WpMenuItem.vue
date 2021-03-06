<template>
    <li :class="classes">
        <a
            v-show="!isRelative || isMailTo"
            target="_blank"
            :href="item.url"
            class="menu-link link-external"
            @click="menuInteracted"
            v-html="item.label"
        />

        <nuxt-link
            v-show="isRelative && !isHash"
            :to="relativeUrl"
            class="menu-link link-internal"
            v-scroll-to="{
                el: `${relativeUrl}`,
                duration: 700,
                lazy: false,
                easing: 'ease-in-out',
                offset: -100,
           }"
            :class="item.cssClasses?item.cssClasses:''"
            @click.native="menuInteracted"
        >
            <span class="menu-link-text">{{ item.label }}</span>
        </nuxt-link>

        <span
            v-show="isHash"
            class="menu-label"
            v-html="item.label"
        />

        <!-- TODO Make prop for sub-menus to be drop downs -->
        <ul
            v-show="hasSubMenu"
            class="sub-menu"
        >
            <wp-menu-item
                v-for="(subItem, i) in getChildren"
                :key="`sub-${i}`"
                class="menu-item sub-menu-item"
                :item="subItem"
                @menu-interacted="menuInteracted"
            />
        </ul>
    </li>
</template>

<script>
import _get from "lodash/get"

export default {
    props: {
        item: {
            type: Object,
            default: () => {},
        },
    },
    computed: {
        classes() {
            return [
                "wp-menu-item menu-item",
                {
                    "is-realtive": this.isRelative,
                },
                { "has-sub-menu": this.hasSubMenu },
                { "is-disabled": this.isHash },
            ]
        },
        getChildren() {
            return _get(this, "item.childItems.nodes", [])
        },
        hasSubMenu() {
            return this.getChildren.length
        },
        isRelative() {
            return this.item.target !== "_blank" && !this.isMailTo
        },
        isHash() {
            return this.item.label == "#"
        },
        isMailTo() {
            return this.item.url.includes("mailto:")
        },
        relativeUrl() {
            let url = this.item.url
            // Replace all these things
            const replaceThese = [
                _get(this, "$store.state.siteMeta.frontendUrl", ""),
                _get(this, "$store.state.siteMeta.backendUrl", ""),
            ]
            replaceThese.forEach((element) => {
                url = url.replace(element, "")
            })
            return url
        },
    },
    methods: {
        menuInteracted(event) {
            this.$emit("menu-interacted", event)
            this.$store.commit("SET_MENU", false)
        },
    },
}
</script>
