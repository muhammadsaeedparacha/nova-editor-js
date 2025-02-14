<template>
    <DefaultField
        :field="field"
        :errors="errors"
        :show-help-text="showHelpText"
        :full-width-content="true"
        @keydown.stop
    >
        <template #field>
            <div
                :id="`editor-js-${field.attribute}`"
                ref="input"
                class="editor-js"
            />
        </template>
    </DefaultField>
</template>

<script>
import { DependentFormField, FormField, HandlesValidationErrors } from 'laravel-nova';

export default {
    mixins: [DependentFormField, FormField, HandlesValidationErrors],

    props: ['resourceName', 'resourceId', 'field'],

    methods: {
        /*
             * Set the initial, internal value for the field.
             */
        setInitialValue() {
            this.value = this.field.value;

            const self = this;

            const currentContent = (typeof self.field.value === 'object')
                ? self.field.value
                : JSON.parse(self.field.value);

            const editor = NovaEditorJS.getInstance({
                /**
                 * Wrapper of Editor
                 */
                holder: `editor-js-${self.field.attribute}`,

                /**
                 * This Tool will be used as default
                 */
                defaultBlock: self.field.editorSettings.initialBlock,

                /**
                 * Default placeholder
                 */
                placeholder: self.field.editorSettings.placeholder,

                /**
                 * Enable autofocus
                 */
                autofocus: self.field.editorSettings.autofocus,

                /**
                 * Internalization config
                 */
                i18n: {
                    /**
                     * Text direction. If not set, uses ltr
                     */
                    direction: (self.field.editorSettings.rtl ?? false) ? 'rtl' : 'ltr',
                },

                /**
                 * Initial Editor data
                 */
                data: currentContent,

                readOnly: self.currentlyIsReadonly,


                /**
                 * Min height of editor
                 */
                minHeight: 35,

                onReady() {

                },
                onChange() {
                    editor.save().then((savedData) => {
                        self.handleChange(savedData);
                    });
                },
            }, self.field);
        },

        /**
         * Fill the given FormData object with the field's internal value.
         */
        fill(formData) {
            const value = typeof this.value === 'string' ? this.value : JSON.stringify(this.value);
            formData.append(this.field.attribute, value || '');
        },

        /**
         * Update the field's internal value.
         */
        handleChange(value) {
            this.value = JSON.stringify(value);
        },
    },
};
</script>
