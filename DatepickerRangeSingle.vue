<template>
    <v-menu v-model="menu" :close-on-content-click="false" :nudge-right="40"
            transition="scale-transition" offset-y min-width="auto">
        <template v-slot:activator="{ on, attrs }">
            <v-text-field
                outlined
                ref="input"
                hide-details="auto"
                v-model="formattedDate"
                :label="dateRange.length && dateRange[0] && dateRange[1] ? $t('Selected Range') : $t('ALL Time')"
                :prepend-inner-icon="icon"
                :placeholder="inputPlaceholder"
                :autofocus="autofocus"
                :autocomplete="autocomplete"
                :append-icon="postIcon || null"
                :reverse="$vuetify.rtl"
                :loading="loading"
                :rules="validationRules"
                :required="required"
                validate-on-blur
                v-bind="attrs"
                v-on="on"
                readonly
                clearable
                @click:clear="clearDate"
            ></v-text-field>
        </template>
        <v-card>
            <v-row>
                <v-col>
                    <v-date-picker v-model="dateRange" range @update:modelValue="updateDate"></v-date-picker>
                </v-col>
                <v-col>
                    <v-list>
                        <v-list-item v-for="(preset, key) in presets" :key="key" @click="setPreset(preset)">
                            <v-list-item-title>{{ key }}</v-list-item-title>
                        </v-list-item>
                    </v-list>
                </v-col>
            </v-row>
        </v-card>
    </v-menu>
</template>

<script>
export default {
    name: "JetDateRangePicker",
    props: {
        value: {
            type: Array,
            default: () => ["", ""]
        },
        autofocus: Boolean,
        autocomplete: {
            type: String,
            default: "off"
        },
        icon: {
            type: String,
            default: "mdi-calendar"
        },
        postIcon: {
            type: String,
            default: ""
        },
        loading: Boolean,
        required: Boolean,
        placeholder: {
            type: String,
            default: ""
        }
    },
    data: () => ({
        menu: false,
        dateRange: ["", ""],
        presets: {
            "Today": [new Date().toISOString().substr(0, 10), new Date().toISOString().substr(0, 10)],
            "Yesterday": [
                new Date(new Date().setDate(new Date().getDate() - 1)).toISOString().substr(0, 10),
                new Date(new Date().setDate(new Date().getDate() - 1)).toISOString().substr(0, 10)
            ],
            "This Month": [
                new Date(new Date().getFullYear(), new Date().getMonth(), 1).toISOString().substr(0, 10),
                new Date().toISOString().substr(0, 10)
            ],
            "Last Month": [
                new Date(new Date().getFullYear(), new Date().getMonth() - 1, 1).toISOString().substr(0, 10),
                new Date(new Date().getFullYear(), new Date().getMonth(), 0).toISOString().substr(0, 10)
            ],
            "This Year": [
                new Date(new Date().getFullYear(), 0, 1).toISOString().substr(0, 10),
                new Date().toISOString().substr(0, 10)
            ],
            "Last Year": [
                new Date(new Date().getFullYear() - 1, 0, 1).toISOString().substr(0, 10),
                new Date(new Date().getFullYear() - 1, 11, 31).toISOString().substr(0, 10)
            ]
        }
    }),
    computed: {
        formattedDate() {
            return this.dateRange[0] && this.dateRange[1] ? `${this.dateRange[0]} - ${this.dateRange[1]}` : "";
        },
        inputPlaceholder() {
            return this.placeholder || this.$t('ALL Time');
        },
        validationRules() {
            return [];
        }
    },
    watch: {
        value: {
            immediate: true,
            handler(newVal) {
                this.dateRange = newVal;
            }
        }
    },
    methods: {
        updateDate() {
            this.$emit('input', this.dateRange);
            this.menu = false;
        },
        setPreset(range) {
            this.dateRange = range;
            this.updateDate();
        },
        clearDate() {
            this.dateRange = ["", ""];
            this.$emit('input', this.dateRange);
        }
    }
};
</script>

<style scoped>
</style>
