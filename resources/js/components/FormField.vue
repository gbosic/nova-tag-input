<template>
    <default-field :field="field" :errors="errors">
        <template slot="field">
            <vue-tags-input v-model="tag" :tags="tags" @tags-changed="tagsChanged"
                            :add-on-key="addOnKeys"
                            :allow-edit-tags="field.allowEditTags"
                            :placeholder="field.placeholder"
                            :autocompleteItems="filteredItems"
                            :validation="validation"
            />
        </template>
    </default-field>
</template>

<script>
import { FormField, HandlesValidationErrors } from 'laravel-nova'
import VueTagsInput from '@johmun/vue-tags-input';

export default {
    data() {
        return {
            tag: '',
            tags: [],
            addOnKeys: [13, ',', ';'],
            autocompleteItems: [],
            validation: [
                {
                    classes: 'no-email',
                    rule: /^(([^<>()\[\]\\.,;:\s@"]+(\.[^<>()\[\]\\.,;:\s@"]+)*)|(".+"))@((\[[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\])|(([a-zA-Z\-0-9]+\.)+[a-zA-Z]{2,}))$/,
                    disableAdd: true,
                }
            ]
        }
    },
    mixins: [FormField, HandlesValidationErrors],

    props: ['resourceName', 'resourceId', 'field'],

    mounted() {
        // Set up default parameters
        this.autocompleteItems = (this.field.autocompleteItems) ? this.field.autocompleteItems : this.autocompleteItems;
        this.addOnKeys = (this.field.addOnKeys) ? this.field.addOnKeys : this.addOnKeys;
        this.validation = this.field.fieldToValidate === "phone"
            ? [
                {
                    classes: 'no-number',
                    rule: /^\+?[1-9]\d{1,14}$/,
                    disableAdd: true,
                }
            ]
            : this.validation;
    },

    methods: {
        /*
         * Set the initial, internal value for the field.
         */
        setInitialValue() {
            this.value = this.field.value || '';
            if (this.value !== '') {
                let tags = [];
                this.value.forEach(function (item, index) {
                    // Check if the data needs conversion
                    tags.push((typeof item === "object" && item.hasOwnProperty('text')) ? item : {'text': item});
                });
                // Store the tags array
                this.tags = tags;
            }
        },

        /**
         * Fill the given FormData object with the field's internal value.
         */
        fill(formData) {
            formData.append(this.field.attribute, JSON.stringify(this.tags) || '')
        },

        /**
         * Update the field's internal value.
         */
        handleChange(value) {
            this.value = value
        },

        /**
         * Handles the tagChanged event
         * @param newTags
         */
        tagsChanged(newTags) {
            this.tags = newTags;
        }
    },

    computed: {
        filteredItems() {
            return this.autocompleteItems.filter(i => {
                return i.text.toLowerCase().indexOf(this.tag.toLowerCase()) !== -1;
            });
        },
    },

    components: { VueTagsInput }
}
</script>
