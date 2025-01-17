<template>
    <Collapsible :clickable-text="sectionName" :href="href">
        <template #content>
            <div class="border border-1 rounded">
                <Collapsible
                    class="property p-3 border border-top-1"
                    v-for="(property, propertyKey) in sortSchemaByRequired(properties)"
                    :key="propertyKey"
                    :arrow="false"
                    :clickable-text="propertyKey"
                    :href="propertyKey"
                >
                    <template #additionalButtonText>
                        <span v-if="property['$required']" class="text-danger"> *</span>
                    </template>
                    <template #buttonRight="{collapsed}">
                        <div class="d-flex flex-grow-1 align-items-center justify-content-between">
                            <div class="d-flex gap-1">
                                <Tooltip v-if="showDynamic && !isDynamic(property)" class="d-flex" title="Non-dynamic">
                                    <Cancel class="text-danger" />
                                </Tooltip>
                                <Tooltip v-if="property['$beta']" class="d-flex" title="Beta">
                                    <AlphaBBox class="text-warning" />
                                </Tooltip>
                                <Tooltip v-if="property['$deprecated']" class="d-flex" title="Deprecated">
                                    <Alert class="text-warning" />
                                </Tooltip>
                            </div>
                            <div class="d-flex gap-2">
                                <template v-for="type in extractTypeInfo(property).types" :key="type">
                                    <a v-if="type.startsWith('#')" :href="type" @click.stop>
                                        <button class="d-flex fw-bold type-box rounded fs-7 px-2 py-1 bg-transparent">
                                            <span class="ref-type">{{ className(type) }}</span><eye-outline />
                                        </button>
                                    </a>
                                    <div v-else class="type-box rounded fs-7 px-2 py-1">
                                        {{ type }}
                                    </div>
                                </template>
                                <ChevronDown v-if="collapsed" />
                                <ChevronUp v-else />
                            </div>
                        </div>
                    </template>
                    <template #content>
                        <PropertyDetail :show-dynamic="showDynamic" :is-dynamic="showDynamic && isDynamic(property)" :property="property" :text-sanitizer="replaceText">
                            <template #markdown="{content}">
                                <slot :content="content" name="markdown" />
                            </template>
                        </PropertyDetail>
                    </template>
                </Collapsible>
            </div>
        </template>
    </Collapsible>
</template>

<script setup lang="ts">
    import {extractTypeInfo, className} from "../../utils/schemaUtils";
    import ChevronDown from "vue-material-design-icons/ChevronDown.vue";
    import Collapsible from "../misc/Collapsible.vue";
    import Tooltip from "../misc/Tooltip.vue";
    import PropertyDetail from "./PropertyDetail.vue";
    import ChevronUp from "vue-material-design-icons/ChevronUp.vue";
    import Alert from "vue-material-design-icons/Alert.vue";
    import Cancel from "vue-material-design-icons/Cancel.vue";
    import AlphaBBox from "vue-material-design-icons/AlphaBBox.vue";
    import type {JSONProperty} from "./PropertyType.vue";
    import type {JSONSchema} from "./SchemaToHtml.vue";
    import EyeOutline from "vue-material-design-icons/EyeOutline.vue";

    withDefaults(defineProps<{ href?: string, sectionName: string, properties: [JSONProperty], showDynamic: boolean }>(), {
        href: Math.random().toString(36).substring(2, 5),
        showDynamic: true
    });

    const isDynamic = (property: JSONProperty): boolean => {
        if (property["$dynamic"] === true) {
            return true;
        }

        if (property["$dynamic"] === false) {
            return false;
        }

        if (property.oneOf?.some(prop => prop["$dynamic"] === true)) {
            return true;
        }

        return false;
    };

    const replaceText = (str: string): string => {
        str = str?.split("```")[0]?.replace(/`([^`]*)`/g, "<code>$1</code>");
        str = str?.replace(
            /\[(.*?)\]\((.*?)\)/g,
            (match, text, url) => {
                if (text && url) {
                    return `<a href="${url}" rel="nofollow" target="_blank">${text}</a>`;
                } else {
                    return match;
                }
            }
        );
        return str?.split("```")[0];
    };

    function sortSchemaByRequired<T extends NonNullable<NonNullable<JSONSchema["properties"]>["properties"]>>(schema: T): T {
        const requiredKeys = [];
        const nonRequiredKeys = [];

        for (const key in schema) {
            if (typeof schema[key] === "object") {
                if (schema[key].$required) {
                    requiredKeys.push(key);
                } else {
                    nonRequiredKeys.push(key);
                }
            }
        }

        const sortedKeys = [...requiredKeys.sort(), ...nonRequiredKeys.sort()];

        const sortedSchema: T = {} as T;
        sortedKeys.forEach(key => {
            sortedSchema[key] = schema[key];
        });

        return sortedSchema;
    }
</script>

<style lang="scss" scoped>
    @use "../../scss/variables" as variables;

    .type-box, :deep(.type-box) {
        border: 1px solid variables.$blue !important;

        .ref-type {
            padding-right: 0.625rem;
            border-right: 1px solid var(--ks-border-primary);

            + * {
                margin-left: 0.625rem;
            }
        }
    }

    .properties {
        border: 1px solid;
        border-radius: var(--bs-border-radius);
    }
</style>